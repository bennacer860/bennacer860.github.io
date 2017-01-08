---
layout: post
title: "'Pair With Sum' Problem"
date: 2017-01-08 17:40:47 -0500
comments: true
categories: 
---

Recently, I Have been researching common coding interview-problems and I've discovered that most of the major tech companies like Google or facebook have overlapping problem lists. One of these problems is called `Pair With Sum` and it states the following `Find a pair of elements from an array whose sum equals a given number`. 

This request is a bit ambiguous - so let's start out by listing all the assumptions that we are making before we begin working on a solution:

   - The array contains only integers
   - The array can have duplicates
   - Return a boolean that indicates if the list has such a pair
   - We will start with an ordered (ascending order) list example

### Brute force 

Lets code the most naive solution. Basically, try every pair possible and see if their sum is equal to the given number. It would look like this in Ruby

```ruby
def has_pair_with_sum_brute_force(input, sum)
  0.upto(input.length - 1) do |n|
    0.upto(input.length - 1) do |i|
      if input[i] + input[n] == sum and n != i
        return true
      end
    end
  end
  return false
end
```

This solution would work whether the list is sorted or not. However, this is an inefficient way of solving the problem, indicated by the complexity of this algorithm at O(n^2). This will get out of control pretty quickly for large arrays. We need a solution that has a linear complexity or a O(n). Let's explore another solution where we assume that the array elemets are sorted.

### Moving pointer

```ruby
def has_pair_with_sum_in_ordered_list(input, sum)
  pointer_begining = 0
  pointer_end = input.length - 1
  while(pointer_begining < pointer_end)
    # puts pointer_begining
    # puts pointer_end
    return true if input[pointer_begining] + input[pointer_end] == sum
    pointer_end -= 1 if input[pointer_begining] + input[pointer_end] > sum
    pointer_begining += 1 if input[pointer_begining] + input[pointer_end] < sum
  end
  return false
end
```

This solution has the algorithm complexity of O(n) we are looking for, making it much more scalable and efficient. Let's go over the solution briefly. We have one pointer at the begining of the array and one at the end. we sum the value pointed with the two pointer

   - If the sum is equal to the given number then we have found a solution
   - If the sum is greater than the given number then the sum is too big, so we move the last pointer to the value before it
   - If the sum is smaller than the given number then the sum is too small so we move the last pointer to the value after it

### Saving the Complement

Imagine now that our list is not ordered, In this case, the previous method would not work. We need to come up with another algorithm with a linear complexity to retain our scalability. We note that in order for us to determine if a number has a pair, that the sum would equal to the given number, it needs to have its complement in the array. Let's assume that the first element of the array is 2 and that the given number is 8- the complement of 2 to make 8 is `8-2=6`. If we find 6 in the array, we know that it represents a valid pair.

Here's an example:

`array = [1,3,2],  given_number = 4 `

Let's go through this array and save the complements in some data structure. The ideal data structure in this case would be some kind of hash that would save only keys. The reason why we would choose this data structure is because it will ensure our collection constantly maintains unique elements (no redundancy) and it will do the element look up in a constant complexity O(1). In Ruby we can use the [Set](http://ruby-doc.org/stdlib-2.3.3/libdoc/set/rdoc/Set.html) library.

Back to our array traversal. The first element is 1, its complement is (4-1=3), let's add this complement to the complements set. The second element is 3, this number is a complement of a previous element, therefore we have a valid pair here. Let's code the method that would do that:

```ruby
def has_pair_with_sum_in_unordered_list(input, sum)
  complements = Set.new
  0.upto(input.length - 1) do |i|
    value = input[i]
    complement = sum - value
    # if we have stored the value complement then we have a winner pair
    if complements.include?(value)
      return true
    else
      complements.add(complement)
    end
  end
  return false
end
```

### benchmarking
Let's prove that the best solution is not the brute force method by using the `benchmark/ips` gem.

```
$ ruby sum_of_pair.rb
Warming up --------------------------------------
         brute force    13.000  i/100ms
pointer algorithm for ordered list
                         1.022k i/100ms
find complement algorithm for unordered list
                       647.000  i/100ms
Calculating -------------------------------------
         brute force    140.826  (± 2.8%) i/s -    715.000  in   5.082566s
pointer algorithm for ordered list
                         10.314k (± 2.8%) i/s -     52.122k in   5.057393s
find complement algorithm for unordered list
                          7.045k (±25.5%) i/s -     31.703k in   5.064816s

Comparison:
pointer algorithm for ordered list:    10314.1 i/s
find complement algorithm for unordered list:     7045.2 i/s - 1.46x  slower
         brute force:      140.8 i/s - 73.24x  slower
```

The brute force solution is 73.24 times slower than the unordered list algorithm. The unordered list algorithm is 1.46 times slower that the ordered list algorithm (very insignificant difference).


### Conclusion


As you can see, there are multiple ways to solve an these common interview questions. Each method would lead to a perfect linear complexity solution. Ensuring you have identified the right assumptions is essential in solving similar problems.
