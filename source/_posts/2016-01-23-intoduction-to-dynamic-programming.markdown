---
layout: post
title: "Intoduction to Dynamic Programming"
date: 2016-01-23 22:16:49 -0500
comments: true
categories: 
---

As a big fan of programming puzzles, I realized that the harder the problems get, the less equipped you are to solve them if you don't use dynamic programming techniques. I know it sounds very taunting and fancy but Dynamic programming is not. It is not as elegant as Recursion and requires additional data structure to solve intermediate solutions most of the time.

Wikipedia has a better as follows: `dynamic programming is a method for solving a complex problem by breaking it down into a collection of simpler subproblems, solving each of those subproblems just once, and storing their solutions - ideally, using a memory-based data structure`.

This definition is very interesting. As a software developer, don't we always try to break down all problems into a collection of simpler problems? The answer is yes. Do we always try to store their solutions in a data structure? Not so sure.

In this post we will try to solve a very common problem (ie Fibonacci Sequence) in two ways and demonstrate on how the methods of dynamic programming can be more efficient.

## The Recusive way
This is the recursive way to solve it:

```ruby
def fib n
    return 0 if n == 0
    return 1 if (n == 1 || n == 2)
    return fib(n-1)+fib(n-2)
end
```

let's see how ```fib(4)``` is calculated:

```
fib(5) =          fib(4)            +     fib(3)
                    |                        |
            fib(3)  +  fib(2)          fib(2) + fib(1)
              |          |               |        |
              |          1               1        1
            fib(2) + fib(1)      
              |       |
              1       1
```
It seems a bit confusing at first (that's recursion in general), but what I want you to notice is that we are solving the same problems over and over again. For example, fib(2) is solved twice. You can imagine how trying to calculate the Fibonacci sequence of a big number could result in a large amount of time. It acutally takes 2-3 minutes to find the 50th term!

## The Dynamic programming way

Since the topic is Dynamic Programming and it is all breaking down the complex problems to simpler ones and storing their solution, this is called [Memoization](https://en.wikipedia.org/wiki/Memoization) by the way. Let's try this. In our example the intermediate solution could be fib(3), so let's store it in an array so we don't have to recalculate it, And let's do this for all the other numbers.
Here is what the solution should look like:

```ruby
def dynamic_fib n
  fibonacci = Array.new
  fibonacci[0] = 0
  fibonacci[1] = fibonacci[2] = 1
  (3).upto(n) do |index|
    fibonacci[index] = fibonacci[index-1]+fibonacci[index-2]
  end
  fibonacci[n]
end
```
This solution looks more efficient, it will find the Fibonacci Sequence with `n` complexity. Let's benchmark it though:

```ruby
require 'benchmark/ips'

Benchmark.ips do |x|
  x.report 'recursive fib 30' do
    fib 30
  end

  x.report 'dynamic fib 30' do
    dynamic_fib 30
  end
  x.compare! # Print the comparison
end
```

```
Calculating -------------------------------------
    recursive fib 30     1.234k i/100ms
      dynamic fib 30    40.252k i/100ms
-------------------------------------------------
    recursive fib 30     12.358k (± 3.8%) i/s -     62.934k
      dynamic fib 30    489.616k (±10.1%) i/s -      2.455M

Comparison:
      dynamic fib 30:   489616.3 i/s
    recursive fib 30:    12357.8 i/s - 39.62x slower
```

# Conclusion
Once you see how it works, it would seem very natural to implement these methods to solve a certain class of problems. Whenever you see recursion you might have a chance to optimize your code using Dynamic Programming.




