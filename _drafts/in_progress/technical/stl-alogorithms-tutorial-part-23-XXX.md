---
layout: post
title: "The big STL Algorithms tutorial: heap operations"
date: 2021-X-X
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we are going to talk about heap operations:
<!--more-->

* `is_heap`
* `is_heap_until`
* `make_heap`
* `push_heap`
* `pop_heap`
* `sort_heap`

The first question we have to answer - before we'd start discussing the above functions one by one - is what we mean by a heap.

It's worth mentioning this because the most often a C++ developer meets the word heap is about static and dynamic memory allocations. It's about heap vs stack.

Not this time. In this case, we talk about data structures, in particular max-heaps:
- binary trees where all levels of the tree (except the last one) are fully filled. On hte last level, they are filled from left to right.
- the key stored in each node is either greater than or equal to the keys in the node's children,

We got used to the fact that standard C++ algorithms work on all the different kinds of containers. It's not the case for heap operations. They work on containers that are supporting random access iterators, such as `std::vector` or `std::deque`.

If you pass a list, your code will not compile and you'll get some horribly long error messages. [Go and try yourself](https://godbolt.org/#z:OYLghAFBqd5TKALEBjA9gEwKYFFMCWALugE4A0BIEAZugHZEDKqAhgDbYgCMALOXUYBVAM7YACgA8QAcgAMM8gCse5dq3qhUAUgBMAIT37yYzqiIEG1bPUwBhdOwCuAW3ohd5G5gAyBetgAcq4ARtikPHLkAA7oIsSW9A7Obh4xcQkMfgHBLmER3FGm2OaJTESspETJru6exaUM5ZVE2UGh4ZEmFVU1qfU9rf7teZ2FAJQm6E6kqFwyegDM/qjOOADU2ot2liJEpNisLlu42nIAgksra9ib2%2BwEeydnl7rL9KtOG1t2AG4lJFIzwuVw%2BNzudg4wDIxCQx0WpxBF38RHWLlY/gg402AHZDBd1oT1ntMCAQA89ut6K4RLj9NxyOtPOtFozeNocQARLb485E4lEUloaaon4/AVCkLoRwcaJIVg8l78klkjBOUXbcUqkCPAD6SEO0Qg1JcIgAdGFgJjJlSaWbvFjxoqQVyZJN2LIAKyKdzyRToWR2IxGYnTWa3JbcRREWQKSYQFAYFzRAicChUCBJlNpkDAET0VjRERIdBEASpojhETUEKxxQhfyVACesij5CTLhsRAA8vR2C2/eQcOjNJw60OCAdSv9q4PsJISur5m2UdgPYOHiFSM2HDhx/sCC5W26BAxmGwx3xT8IxFJxyoGepNGhgwY1AQQtXYAXOyAANa/Kw8QUP8ET7E49B/uMkzoNEFgMLOAC0JJbJyOgGEYuisOsiHdosigNPB7gQN4fTuAy3htLk%2BSqLE8REWRtEZERVEdAUJhrgCZSDIxDKEdxLSsaM7F7C0vHdIJwzUWMkwiGGcw8O6Xo%2BuOAYyJIAAcABsiFabw6zAKgqDrBA4GQdiED4MQZCbG8DLrA4yapuEtmLNw2JBhhBjRnW8YoCK0TqpQ1BZs57HYIQgKqII54cFwV4xaIEjSIOADu27RMeSkyN65C%2Bgo5Bqd26qBai6A0Osmk6XpBlGSZZlQSZjnZi5kbjD5frQeQf6RFE668IoR6LIsZrDWN40TVpeWqbIBEgFEMadeQcCwIm6BOWmwWZutLURHmBZFiWZY0BWVY1uODYFqQA5th2Xa9v247DhowBjoOhBThYM7jvOi6VselCMGu46btu127vMBUHkeMhRpMMUsHFPD8Ilt4pQVKieE%2BWivsYm5flihVwYkSEAOocOwOGk/O%2BzYchgqoehhgGLoBX8VYJG2OJlFSWxTH0Yk4l0Zk9BCTRfGcY09DNL0ji1Ko7PS4MYtjBJsspORatDDkfPuVMMwKXr665fl/qyFVun6YZxmmaQEGNZZkU2ZGjLNWFrm6B5uMdXG2UDeQQ0jRNwdjVNpuFbNJjzeQi1xstiAJiAAVBRmoU5p4EXWexCMXvFKNnkld5pRlWVqMp02DkVJXqus5WVdplu1TbDUWW7aYe%2B1Me%2BX7g0gMNo0h8HFcFWpc0Ld33W9WXMj4QHU/h6PXeddls8L5HsdupMoHxFYvBAA%3D).

Now it's time to get the details.

## `is_heap`

`is_heap` in its singles form takes two parameters and returns a boolean. If the input range is a *max heap*, it returns true, otherwise false.

The two input parameters are denoting the beginning and the end of the range to check.

As we got used to it, there are two optional parameters. At the last position, you might pass in a binary predicate, a comparator that would return `true` if the first argument is smaller than the second one.

Since C++17, you can pass in an optional execution policy before all the other parameters.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
 
int main()
{
    std::vector<int> orderedNumbers { 1, 2, 3, 4, 5 };
 
    std::vector<int> numbersInHeapOrder { 5, 4, 3, 1, 2 };
 
    std::cout << std::boolalpha;
    std::cout << "orderedNumbers.is_heap()?: " 
              << std::is_heap(orderedNumbers.begin(), orderedNumbers.end())
              << '\n';
    std::cout << "numbersInHeapOrder.is_heap()?: " 
              << std::is_heap(numbersInHeapOrder.begin(), numbersInHeapOrder.end())
              << '\n';
}
/*
orderedNumbers.is_heap()?: false
numbersInHeapOrder.is_heap()?: true
*/
```

## `is_heap_until`

`is_heap_until` finds the longest range that is a *max heap* starting from the first input parameter that denotes the beginning of the range to check up until the second input that signifies the last element to check.

The return value will be a pointer that points at the end of the longest *map heap* found.

As usual you have the possibility to pass in a custom comparator and since C++17 an execution policy.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
 
int main()
{
    std::vector<int> numbers { 5, 4, 3, 1, 2, 6 };
 
    std::cout << std::boolalpha;
    std::cout << "numbers are organized as a max heap?: " 
              << std::is_heap(numbers.begin(), numbers.end())
              << '\n';
    std::cout << "numbers until the last but one position "
              << "are organized as a max heap?: " 
              << std::is_heap(numbers.begin(), numbers.end()-1)
              << '\n';
    std::cout << "the first element not part of the largest heap: " 
              << *(std::is_heap_until(numbers.begin(), numbers.end()))
              << '\n';
}
/*
numbers are organized as a max heap?: false
numbers until the last but one position are organized as a max heap?: true
the first element not part of the largest heap: 6
*/
```

## `make_heap`

While the previous two presented functions were non-intrusive, they don't change the passed in container, `make_heap` does.

You pass in a range of elements in any order and you'll get it back with the data organized into a *max heap*.

You can also pass in your custom comparator as a third parameter.

Unlike in another cases, there is no option to pass in an execution policy. If you think about it, it makes sense. It'd be pretty hard to construct a heap in parallel.

The function is void, meaning that it doesn't return anything.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
 
int main()
{
    std::vector<int> numbers { 1, 2, 3, 4, 5 };
 
    std::cout << std::boolalpha;
    std::cout << "numbers are organized as a max heap?: " 
              << std::is_heap(numbers.begin(), numbers.end())
              << '\n';
    for(const auto n : numbers) {
      std::cout << n << ' ';
    }
    std::cout << '\n';
    
    std::make_heap(numbers.begin(), numbers.end());
    
    std::cout << "what about now?: " 
              << std::is_heap(numbers.begin(), numbers.end()-1)
              << '\n';
    for(const auto n : numbers) {
      std::cout << n << ' ';
    }
    std::cout << '\n';
}
/*
numbers are organized as a max heap?: false
1 2 3 4 5 
what about now?: true
5 4 3 1 2 
*/
```

Just as a side note, there is no `make_heap_copy`, or similar function that would leave the original input unchanged and build the heap somewhere else.

But you can make your copy first and then turn it into a heap.

## `push_heap`

From time to time, there are functions in the standard library and in the `<algorithm>` header that doesn't exactly work the way you'd expect based on its name.

Or at least, not how I'd expect.

I thought that `push_heap` would insert an element into a range that is already organized into a heap.

Not exactly.

It takes a range denoted by its beginning and end and an optional comparator.

It assumes that all the elements, but the last one are organized into a *max heap* and takes that last element and inserts it into a heap.

So it doesn't take care of adding an element to the container. Before calling `push_heap`, `is_heap` on the full container would potentially return `false`, but `is_heap(v.begin(), v.end()-1)` is requried to return `true`. After calling `push_heap`, even `is_heap` must return true.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
 
int main()
{
    std::vector<int> numbers { 5, 4, 3, 1, 2, }; 
    
    std::cout << std::boolalpha;
    std::cout << "numbers are organized as a max heap?: " 
              << std::is_heap(numbers.begin(), numbers.end())
              << '\n';
              
    numbers.push_back(42);
 
    std::cout << std::boolalpha;
    std::cout << "after adding 42, numbers are organized as a max heap?: " 
              << std::is_heap(numbers.begin(), numbers.end())
              << '\n';
    std::cout << "numbers are organized as a max heap "
              << "until the last but one element?: " 
              << std::is_heap(numbers.begin(), numbers.end()-1)
              << '\n';
    for(const auto n : numbers) {
      std::cout << n << ' ';
    }
    std::cout << '\n';
    
    std::push_heap(numbers.begin(), numbers.end());
    
    std::cout << "what about now, are all numbers in a heap?: " 
              << std::is_heap(numbers.begin(), numbers.end())
              << '\n';
    for(const auto n : numbers) {
      std::cout << n << ' ';
    }
    std::cout << '\n';
}
/*
numbers are organized as a max heap?: true
after adding 42, numbers are organized as a max heap?: false
numbers are organized as a max heap until the last but one element?: true
5 4 3 1 2 42 
what about now, are all numbers in a heap?: true
42 4 5 1 2 3 
*/
```

## `pop_heap`

Just like `push_heap`, `pop_heap` will make sure that the range between the first and the last but one element is organized as a heap. But before it makes the according changes, it swaps the first and the last element of the passed in range.

The input parameters are the same as for `push_heap`, so it takes two iterators denoting the first and last elements of the range you work with and it also accepts an optional comparator.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
 
int main()
{
    std::vector<int> numbers { 9, 8, 3, 1, 2, 6}; 
    
    std::cout << std::boolalpha;
    std::cout << "numbers are organized as a max heap?: " 
              << std::is_heap(numbers.begin(), numbers.end())
              << '\n';
              
    std::pop_heap(numbers.begin(), numbers.end());
 
    std::cout << std::boolalpha;
    std::cout << "after calling pop_heap, numbers are organized as a max heap?: " 
              << std::is_heap(numbers.begin(), numbers.end())
              << '\n';
    std::cout << "numbers are organized as a max heap "
              << "until the last but one element?: " 
              << std::is_heap(numbers.begin(), numbers.end()-1)
              << '\n';
    for(const auto n : numbers) {
      std::cout << n << ' ';
    }
    std::cout << '\n';
}
/*
numbers are organized as a max heap?: false
after calling pop_heap, numbers are organized as a max heap?: false
numbers are organized as a max heap until the last but one element?: true
8 6 3 1 2 9 
*/

```
## `sort_heap`

This is our last algorithm for today, with `sort_heap` we leave the realms of heaps. Just like the passed in container.

Call `sort_heap` on a range, and you'll get back your container where the elements are sorted in an ascending order, so the input range loses its *max heap* property.

If you wonder about why `std::sort_heap` exists when we `std::sort`, I don't have a clear answer for you. Since C++11, `std::sort` will always work within the complexity of *O(n\*logn)*, while for `std::sort_heap` we also have *2\*n\*logn* comparisions, which is the same order of magnitude.

[My test](https://quick-bench.com/q/b7CC4i7SEJk2t9TuwVtffXXusHI) were showing `std::sort` consistently faster by a factor 3-4.

At the same time, [I found someone saying](https://stackoverflow.com/a/63086885/3238101) in terms of memory requirements `std::sort` has a requirement for *O(logn)* memory on the stack while `std::sort_heap` only for *O(1)* meaning that in the microcontroller world `std::sort_heap` is preferabble to avoid stack overflow.

Otherwise it seems not a lot of usecases for `std::sort_heap`. Nevertheless here is an example on how to use it:

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers{1, 2, 3, 4, 5};
  std::make_heap(numbers.begin(), numbers.end());
  for(const auto n : numbers) {
    std::cout << n << ' ';
  }
  std::cout << '\n';
  
  std::sort_heap(numbers.begin(), numbers.end());
  for(const auto n : numbers) {
    std::cout << n << ' ';
  }
  std::cout << '\n';
}
/*
5 4 3 1 2 
1 2 3 4 5 
*/
```

# Conclusion

This time, we learned about *heap* algorithms that work not on the heap memory but on "heaply organized" data structures. I ope you found it interesting.

Next time we'll discuss *minimum/maximum operations*.

Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!











static void Sort(benchmark::State& state) {
  std::vector<int> numbers;
  for (size_t i=0; i < 100000; ++i) {
    numbers.push_back(i);
  }
  std::make_heap(numbers.begin(), numbers.end());
  for (auto _ : state) {
    std::sort(numbers.begin(), numbers.end());
  }
}
// Register the function as a benchmark
BENCHMARK(Sort);

static void SortHeap(benchmark::State& state) {
  std::vector<int> numbers;
  for (size_t i=0; i < 100000; ++i) {
    numbers.push_back(i);
  }
  std::make_heap(numbers.begin(), numbers.end());
  for (auto _ : state) {
    std::sort_heap(numbers.begin(), numbers.end());
  }
}
BENCHMARK(SortHeap);
