---
layout: post
title: "Constructing Containers from Ranges in C++23"
date: 2025-5-21
category: dev
tags: [cpp, cpp23, ranges, containers]
excerpt_separator: <!--more-->
---
I’ve written plenty on this blog about [standard algorithms](https://www.sandordargo.com/tags/algorithms/), but far less about [ranges](https://www.sandordargo.com/tags/ranges/). That’s mostly because, although I’ve had production-ready compilers with C++20 ranges since late 2021, the original ranges library lacked a few key capabilities.

The biggest gap was at the *end* of a pipeline: you could transform data lazily, but you *couldn't* drop the result straight into a brand-new container. What you got back was a **view**; turning that view into, say, a `std::vector` still required the old iterator-pair constructor.

C++23 fixes that in two complementary ways:

* **`std::to`** (an adaptor that finishes a pipeline by **converting to a container**), and  
* **`from_range` constructors** on every standard container.

Today we’ll focus on the second improvement, because it’s the one you can implement in your own types, too.

## The `from_range` constructor

Every standard container now supports a new set of constructors that make integration with ranges smoother — the so-called `from_range` constructors.

These constructors allow you to build a container directly from a range, rather than from a pair of iterators. 

### Before C++23

```cpp
std::vector<int> v(some_range.begin(), some_range.end());
```

### Starting from C++23:

```cpp
std::vector<int> v(std::from_range, some_range);
```

The first parameter is the tag `std::from_range`, an instance of the trivial type `std::from_range_t`.
Using a tag object serves two purposes:

- It disambiguates these overloads from the legacy iterator-pair ones.
- It makes code read like a sentence: "construct this container from a range."


That’s already clearer, but it gets even better if you use it with a series of pipelines:

```cpp
#include <vector>
#include <ranges>
#include <iostream>

int main() {
    const std::vector<int> data{1,2,3,4,5,6,7,8,9};

    auto result = std::vector<int>(
        std::from_range,
        data
        | std::views::filter([](int n){ return n % 2 == 0; })
        | std::views::transform([](int n){ return n * n; })
    );

    for (int x : result) std::cout << x << ' ';   // 4 16 36 64
}
```

No temporary container, no explicit pairs of iterators - just a pipeline ending in a container.


## Concluions

Starting from C++23, standard containers support a new set of constructor overloads. These constructors take a `std::from_range` tag, a range and an optional allocator. These `from_range` constructors make it easier to construct containers from ranges, helping make C++ code more concise, more expressive, and less error-prone.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
