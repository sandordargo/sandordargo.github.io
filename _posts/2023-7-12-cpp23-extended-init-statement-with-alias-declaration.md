---
layout: post
title: "C++23: Alias declarations in for loop init-statements"
date: 2023-7-12
category: dev
tags: [cpp, cpp23, for, initstatement]
excerpt_separator: <!--more-->
---

Let's continue exploring C++23 features! Sometimes, accepted proposals introduce entirely new features, sometimes they bring bug fixes and sometimes they are somewhere in between.

The small feature we discuss today is from the latter category. Loops and conditionals in C++ have been evolving in a similar direction with the introduction of init statements. As we discussed in [this article](https://www.sandordargo.com/blog/2022/11/02/statements-with-initializers-part-2-loops), since C++20 you can use a range-based for loop with an init statement:

```cpp
for (auto strings = createStrings(); auto c: strings[0]) {
  std::cout << c << " ";
}
```

What I missed in that earlier article is that we cannot only declare and initialize new variables in an init statement, but we can also use have `typedef` if that's what we need.

```cpp
std::vector pointsScored {27, 41, 32, 23, 28};

for (typedef int Points; Points points : pointsScored) {
    std::cout << "Jokic scored " << points << '\n';
}
```

You might run into a situation where scoping a `typedef` for the `for`-loop's body is useful and this feature comes in handy. At the same time, if we need an alias, it's more readable and also more widely usable to [use `using`](https://www.sandordargo.com/blog/2022/04/27/the-4-use-of-using-in-cpp#aliasing).

Sadly, with C++20's `for`-loop init statements that's not something you can do. You can use `typedef`, but not `using`.

Thanks to Jens Mauer's [paper P2360R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2360r0.html) this has changed with C++23. Alias declarations are also permitted now: 

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector pointsScored {27, 41, 32, 23, 28};

    for (int points : pointsScored) {
        std::cout << "Jokic scored " << points << '\n';
    }

    for (typedef int Points; Points points : pointsScored) {
        std::cout << "Jokic scored " << points << '\n';
    }

    for (using Points = int; Points points : pointsScored) {
        std::cout << "Jokic scored " << points << '\n';
    }
}
```

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!