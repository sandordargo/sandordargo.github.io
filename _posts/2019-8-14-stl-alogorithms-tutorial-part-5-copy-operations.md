---
layout: post
title: "The big STL Algorithms tutorial: modifying sequence operations - copy et al."
date: 2019-8-14
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we will discover all the modifying sequence operations whose name start with copy:
<!--more-->

* `copy`
* `copy_n`
* `copy_if`
* `copy_backward`

## `copy`

There is no big surprise about the goal of `std::copy`. It takes the elements of the input range and copies them to the output. Let here be an example:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto inputNumbers = std::vector<int>{1, 2, 3, 4, 5, 6};
  auto copiedNumbers = std::vector<int>{};
  std::copy(inputNumbers.begin(), inputNumbers.end(), copiedNumbers.begin());
  for (auto number : copiedNumbers) {
    std::cout << number << "\n";
  }
  
  return 0;
}
```

So what do you think? Will our copy operation be successful?

No, it won't be! Instead, we are facing a core dump caused by a segmentation fault. The reason is that there is simply not enough space in `copiedVectors`. Its size is zero and there is no automatic expansion of the vector unless you use the corresponding API (like [push_back()](http://www.cplusplus.com/reference/vector/vector/push_back/)).

So we have two options to choose from.

1) We can make sure that the output vector has a big enough size for example by declaring it with the size of the input like this:

```cpp
auto copiedNumbers = std::vector<int>(inputNumbers.size());
```

This approach has multiple disadvantages.
* `copiedNumbers` will be populated with the default constructed objects. Okay, in our example we use integers, but imagine if we use a big vector of custom objects that are more costly to build.

* There is another issue. What if the size of the input changes between you create copiedNumbers and you actually call the copy algorithm? Still the same segmentation fault.

2) Instead, you can use an _inserter_ which is an inserter __iterator__ and as its name suggests it will help you to add new elements to the output vector. You can use it like this:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto inputNumbers = std::vector<int>{1, 2, 3, 4, 5, 6};
  auto copiedNumbers = std::vector<int>{};
  std::copy(inputNumbers.begin(), inputNumbers.end(), std::back_inserter(copiedNumbers));
  for (auto number : copiedNumbers) {
    std::cout << number << "\n";
  }
  
  return 0;
}
```

Please note that we used `std::back_inserter` in our example that always inserts new elements at the end of its target. Just like `push_back`, but that's someone you cannot use in algorithms as it is related to a specific container, it's not an inserter iterator.

A particular problem you might think off is that our output container is empty in the beginning and it grows and grows. In how many steps? We can't really know in advance that's an implementation detail of the compiler you are using. But if your input container is big enough, you can assume that the output operator will grow in multiple steps. Resizing your vector might be expensive, it needs memory allocation, finding continuous free areas, whatever.

If you want to help with that, you might use `std::vector::reserve`, which will reserve a big enough memory area for the vector so that it can grow without new allocations. And if the reserved size is not enough, there won't be a segmentation fault or any other issue, just a new allocation.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto inputNumbers = std::vector<int>{1, 2, 3, 4, 5, 6};
  auto copiedNumbers = std::vector<int>{};
  copiedNumbers.reserve(inputNumbers.size());
  std::copy(inputNumbers.begin(), inputNumbers.end(), std::back_inserter(copiedNumbers));
  for (auto number : copiedNumbers) {
    std::cout << number << "\n";
  }
  
  return 0;
}
```

What we could observe is that `copy` doesn't insert new elements on its own, but it overwrites existing elements in the output container. It can only insert if an inserter iterator is used.

## `copy_n`

`copy` took its inputs by a pair of iterators. One marked the beginning of the input range and one the end. But what if you want to copy let's say 5 elements. Easy-peasy, you can still use copy:

```cpp
std::copy(inputNumbers.begin(), inputNumbers.begin()+5, std::back_inserter(copiedNumbers));
```

Pointer arithmetics work well on iterators, so you are free to do this. But you have a more elegant way, you can use `copy_n` and then you need only the first iterator:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto inputNumbers = std::vector<int>{1, 2, 3, 4, 5, 6};
  auto copiedNumbers = std::vector<int>();
  copiedNumbers.reserve(inputNumbers.size());
  std::copy_n(inputNumbers.begin(), 5, std::back_inserter(copiedNumbers));
  for (auto number : copiedNumbers) {
    std::cout << number << "\n";
  }
  
  return 0;
}
```

Otherwise `copy_n` has the same characteristics as `copy`.

## `copy_if`

Let's say you only want to copy certain elements of a list. For example only the even numbers? What can you do? You can simply call `copy_if` and pass your condition in the form of a unary predicator. What can it be? It can be a function object, a function pointer or simply a [lambda expression](http://sandordargo.com/blog/2018/12/19/c++-lambda-expressions). Due to its simplicity, I stick to lambdas:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto inputNumbers = std::vector<int>{1, 2, 3, 4, 5, 6};
  auto copiedNumbers = std::vector<int>();
  copiedNumbers.reserve(inputNumbers.size());
  std::copy_if(inputNumbers.begin(), inputNumbers.end(), std::back_inserter(copiedNumbers), [](int i) { return i % 2 == 0; });
  for (auto number : copiedNumbers) {
    std::cout << number << "\n";
  }
  
  return 0;
}

```

## `copy_backward`

The last algorithm for today is `copy_backward`. This algorithm copies elements from the input range but starting from the back going towards the beginning.

Does it produce a reversed order compared to the input? No, it doesn't. It keeps order. So why does this `copy_backward` exists? What is its use?

Think about the following case.

You have an input range of `{1, 2, 3, 4, 5, 6, 7}` and you want to copy the part `{1, 2, 3}` over `{2, 3, 4}`. To make it more visual:

```cpp
{1, 2, 3, 4, 5, 6, 7} => {1, 1, 2, 3, 5, 6, 7}
```
So we try to use `copy` and the output container is the same as the input.

You might try this code:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto inputNumbers = std::vector<int>{1, 2, 3, 4, 5, 6, 7};
  std::copy(std::begin(inputNumbers), std::begin(inputNumbers)+3, std::begin(inputNumbers)+1);
  for (auto number : inputNumbers) {
    std::cout << number << "\n";
  }
  
  return 0;
}

```
The output might be different compared to what you expected - it depends on your expectation and compiler:

```cpp
1
1
1
1
5
6
7
```

So what happened?

First, the first number (`inputNumbers.begin()`) is copied over the second one (inputNumbers.begin()+1). So 2 is overwritten by 1. Then the second number (`inputNumbers.begin()+1`) is getting copied to the third (`inputNumbers.begin()+2`) position. But by this time, the second number is 1, so that's what will be copied to the third. And so on.

_(It is possible that you're using a compiler that is smart enough to overcome this issue)_

`std::copy_backward` will help you to not to have this issue. First, it will copy the last element of your input range and then it will one by one towards the first element, keeping the relative order in the output. Use `copy_backward` when you copy to the right and the input range is overlapping with the output one.

## Conclusion

Today, we had a peek into the algorithms that start with the copy prefix. They are not all the copy algorithms, but the rest (like `reverse_copy`, `unique_copy`) I decided to fit in other parts.

Maybe the most important thing to remember that if you don't want to rely on your compiler smartness' and your input and output containers are the same, you have to think wise whether you should use `copy` or `copy_backward`.

Next time we’ll start learning about the move and swap and their friends. Stay tuned!
