---
layout: post
title: "Rspec internals (part 1)"
date: 2015-02-17 13:53:30 -0500
comments: true
categories: 
---



If you wonder how Rspec is built and is curious about the internals of such a tool then this article is for you. After watching an episode from the excellent [Gary Bernhard's Destroy all software series](https://www.destroyallsoftware.com/talks) ,i decided to blog about it to make sure i understand how it works.  

We will use Minitest TestCase to test the most basic Rspec syntax. As usual let's write two tests, one that pass and another one that fails. And we will write the code that defines the Rspec features in spec.rb. but lets start with just the tests:
```ruby
require 'minitest/autorun'
require_relative 'spec'

class TesDecribe < MiniTest::Unit::TestCase
  def test_that_it_can_pass
    describe 'some description' do
      it 'has some property' do

      end
    end
  end

  def test_that_it_can_fail
    assert_raises(IndexError) do
      describe 'some failing test' do
        it 'fails' do
          raise IndexError
        end
      end
    end
  end
end
```

Running this will throw a bunch of errors :

```
Finished in 0.001565s, 1277.9553 runs/s, 638.9776 assertions/s.

  1) Failure:
TestDescribe#test_that_it_can_fail [test.rb:13]:
IndexError expected but nothing was raised.

2 runs, 1 assertions, 1 failures, 0 errors, 0 skips
```

Interesting, It seems like the  describe method is defined somehow but the block inside is not executed. It is time to start working on that spec.rb file

```ruby
#lets define describe method that will pick a description and a block and let's call that block
def describe description,&block
  block.call
end
```

let's run our test again

```
Finished in 0.001240s, 1612.9032 runs/s, 806.4516 assertions/s.

  1) Error:
TestDescribe#test_describe_method:
NoMethodError: undefined method `it' for #<TestDescribe:0x007fb8158a2558>
    test.rb:10:in `block in test_describe_method'
    test.rb:4:in `call'
    test.rb:4:in `describe'
    test.rb:9:in `test_describe_method'


  2) Failure:
TestDescribe#test_that_it_can_fail [test.rb:17]:
[IndexError] exception expected, not
Class: <NoMethodError>
Message: <"undefined method `it' for #<TestDescribe:0x007fb8158a1e50>">
---Backtrace---
test.rb:19:in `block (2 levels) in test_that_it_can_fail'
test.rb:4:in `call'
test.rb:4:in `describe'
test.rb:18:in `block in test_that_it_can_fail'
---------------
```

This is the same Exception, the block inside `describe` gets executed but there is an undefined method `it` inside. The `it` method is pretty much the same as `describe`, it will take a description and a block and will execute that block. Let's update our spec.rb:

```ruby
#lets define describe method that will pick a description and a block and let's call that block
def describe description,&block
 block.call
end

def it description,&block
 block.call
end
```
 
 let's run our tests again:
```
Finished in 0.000968s, 2066.1157 runs/s, 1033.0579 assertions/s.

2 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

Good this is working, i can use that `describe do .... end` syntax, but how about assertions, i want to be able to do something like `2.should == 2`. As Usual let's do the same and build 2 tests, a passing one and a failing one.

```ruby
class TestAssertion  < MiniTest::Test
  def test_that_it_can_pass
    2.should == 2
  end

  def test_that_it_can_fail
    assert_raises AssertionError do
      1.should ==  2
    end
  end
end
```

Obviously running test will not pass:
```
Finished in 0.001312s, 3048.7805 runs/s, 762.1951 assertions/s.

  1) Error:
TestAssertion#test_that_it_can_fail:
NameError: uninitialized constant TestAssertion::AssertionError
    test.rb:17:in `test_that_it_can_fail'


  2) Error:
TestAssertion#test_that_it_can_pass:
NoMethodError: undefined method `should' for 2:Fixnum
    test.rb:13:in `test_that_it_can_pass'
```

I see two things , an undefined method `should` and an undefined Exception AssertionError. Let's update spec.rb:
```ruby
...
#this is easy fix, just define an exception
class AssertionErro < Exception
end

```

In order to implement the `should` we need to create an assertion for the object passed and it needs to work for all ruby objects so we will implement it for the `Object` class. You also need to remember that '==' is just syntactic sugar offered by ruby and it is the equivalent of 'equal?'.  Basically, `2 == 2` is the same as `2.equal?(2)` . this is very important because we will define a `==` method in our `DelayedAssertion` class. Enough talking, let's take a look at the code in spec.rb:

```ruby
class Object
  def should
    DelayedAssertion.new(self)
  end 
end

#lets define DelayedAssertion class now

class DelayedAssertion
  def initialize(subject)
    @subject = subject
  end
  #here where the magic happens
  def ==(other)
    raise AssertionError unless @subject == other
  end
end
```

let's run the test now:
```
# Running:

....

Finished in 0.001384s, 2890.1734 runs/s, 1445.0867 assertions/s.

4 runs, 2 assertions, 0 failures, 0 errors, 0 skips
```

Wow,  we have been able to implement some major features of Rspec. In future post i will try to add support to should method such as  `something.should be_true` or `array.should have(4).things`. You can also add your implementation as a comments.
