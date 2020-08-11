---
layout: post
title: "The big STL Algorithms tutorial: modifying sequence operations - rotate functions"
date: 2020-8-12
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we will continue with 4 algorithms that either help us to rotate elements around a given element in the input range or they just shift elements around:
<!--more-->

* `rotate`
* `rotate_copy`
* `shift_left` / `shift_right`

Up until now the meaning of the STL function names was pretty straightforward. Finding something or reversing a container doesn't need much explanation. But what the hell is rotating a container?

The simplest explanation I could come up with is the following. If you think you have a more understandable one, let me know in the comments.

To rotate a container by one simply means to take (or `pop`) its first element and put (or `push_back`) it back at the end of the container.

Here is an example:
```
// First rotation
{0, 1, 2, 3, 4, 5, 6, 7, 8, 9} -> {1, 2, 3, 4, 5, 6, 7, 8, 9, 0}
// Second rotation
{1, 2, 3, 4, 5, 6, 7, 8, 9, 0} -> {2, 3, 4, 5, 6, 7, 8, 9, 0, 1}
// Third rotation
{2, 3, 4, 5, 6, 7, 8, 9, 0, 1} -> {3, 4, 5, 6, 7, 8, 9, 0, 1, 2}
```

Now that we know what rotating means, let's get started!

## `rotate`

In the STL we got used to defining the input container by two iterators, denoting the beginning and the end of the container we wish to operate on. It's not the case for `rotate`. The first and the third(!) parameter is to pass in the iterators pointing at the first and last elements, what is the one in the middle for?

It's also an iterator! It should point to the element that you want to be the new first element. So for example if you want to rotate by one, you could call it like this:
```cpp
 std::rotate(numbers.begin(), numbers.begin()+1, numbers.end());
```

This also means that by using `rotate`, you don't define how many rotations you want to perform, but rather that you want to rotate until that middle parameter shows up at the beginning of the container.

Here is a complete example:
```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
  std::vector<int> numbers {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
  
  std::rotate(numbers.begin(), numbers.begin()+5, numbers.end());
  
  std::cout << "numbers after rotation: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  // numbers after rotation:  5 6 7 8 9 0 1 2 3 4
  return 0;
}
```

If you are interested in undefined behaviours caused by non-intended usage I encourage you to define another container and replace one of the parameters with an iterator pointing at another vector of ints.

That's basically the only surprise `rotate` can give you.

It's worth noting that the signature slightly changed with C++11. Before, it was returning nothing, and since then it returns the iterator pointing to the element that now contains the value previously pointed by the first parameter.

## `rotate_copy`

As usual, the STL provides us two types of the same algorithm - where it makes sense -, by default it modifies the passed in range: goodbye immutability, hello more optimal memory consumption; but usually there is another version where the algorithm gets a `_copy` suffix and an extra parameter.

When this second flavour is used, the input range goes unchanged and the results are copied to a new container. This new container is passed in only by denoting its beginning. Thus you have to make sure that either the output container is big enough to accommodate all the results or you have to pass it in as inserter. Most often this would mean `std::back_inserter`.

This is the case for `rotate`/`rotate_copy` as well. `rotate_copy` takes its first three parameters the same way as rotate (beginning of input, new first element after rotating, then end of input range) and then as an extra fourth element, we have to pass in the beginning of the output range.

Here is an example:


```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
  const std::vector<int> numbers {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
  std::vector<int> rotatedNumbers;
  rotatedNumbers.reserve(numbers.size());
  
  std::rotate_copy(numbers.begin(), numbers.begin()+5, numbers.end(), std::back_inserter(rotatedNumbers));
  
  std::cout << "rotatedNumbers: ";
  for (const auto& number : rotatedNumbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  // rotatedNumbers:  5 6 7 8 9 0 1 2 3 4
  return 0;
}
```

## `shift_left` / `shift_right`

These two are brand new algorithms added to the STL with C++20. When I first saw them, I had misconceptions. I thought that they could be the elementary operations of `std::rotate`, so basically rotating by one in a direction or the other. I was wrong. So what it is?

As usual, you'll send in the input range by specifying its beginning and its end and as an additional parameter, you can specify the number of positions to shift.

What does shift mean then? If you want to shift left by 1, it means that every element will be moved by 1 to the left! For right shifting, it's obviously to the right.

[CppReference](https://en.cppreference.com/w/cpp/algorithm/shift) says that if you try to shift by a bigger number than the length of the full input, nothing would happen. That's something I could validate on [wandbox](https://wandbox.org/) with GCC 11 and C++2a. But it also says that nothing would happen if `n` is negative. In that case, I got a core dump. Implementation might change it the future, I guess. Anyway, this is C++2a, not C++20 yet.

What you also have to keep in mind is that if you shift left by one, you basically lose the first element (at position 0) in your input. So shifting by one to left and then one to the right, will not keep your range identitical to the original one:

```cpp
std::vector<int> numbers {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
std::shift_left(numbers.begin(), numbers.end(), 1); // {1, 2, 3, 4, 5, 6, 7, 8, 9, 9}
std::shift_right(numbers.begin(), numbers.end(), 1); // {1, 1, 2, 3, 4, 5, 6, 7, 8, 9}
```

That's the difference with rotate. What is taken from the beginning of the end is not put to the other end of the container, but it's just discarded.

But are the items copied or moved? I'd encourage you to try it. You can simply wrap the above integers into a small class and log in the special functions when they are called.

I made this experiment, and if your type supports move semantics, they will be used. Otherwise, copy assignments are used.

## Conclusion

Today, we learned about 4 algorithms that help us to rotate/shift the elements around in a container. Probably rotating is more interesting. Elements are moved to the left and what we would drop, it gets pushed_back to the end. Probably the most readable way to use it is by expressing the second parameter with the beginning of the input container + the number of shifts/rotations by one you want to perform. But the most readable way might depend on your use case. 

For some interesting real-life use cases, I'd invite you to watch [Sean Parent's talk from GoingNative 2013](http://channel9.msdn.com/Events/GoingNative/2013/Cpp-Seasoning).

Next time we’ll learn about `shuffle`, `random_shuffle`, and `sample` algorithms. Stay tuned!