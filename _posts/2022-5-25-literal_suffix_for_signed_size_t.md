---
layout: post
title: "C++23: Literal suffix for (signed) size_t"
date: 2022-5-25
category: dev
tags: [cpp, cpp23, literals]
excerpt_separator: <!--more-->
---
Let's continue our exploration of C++23 features! This week we discuss [the extended language support for literal suffixes](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p0330r8.html).

## What is a literal suffix?

Literals can have an optional suffix which indicates the type of the literal. As such, one doesn't have to store the value in a variable of the desired type but can use the literal directly.

For example, if you need a `long` value and you don't want to rely on implicit conversions, you can pass `42L` instead of passing `42`.

While [we can define our own user-defined literals](https://www.sandordargo.com/blog/2020/10/21/user-defined-literals), for integers, C++ provides quite a few literal suffixes:
- none means that the literal is an `int`
- `U` makes an integer `unsigned`
- `L` makes integers `long`
- `LL` males them `long long`
- `ULL` (or `LLU`) turns `int`s into `unsigned long long int`s

And C++23 is going to add one, or if combined with `U` then 2 elements to this list:
- `Z` turns an `int` into the sized version of `std::size_t`
- `UZ` turns an `int` into `std::size_t`

## But why do we need this new `Z` literal suffix?

If you're an [Almost Always Auto](https://herbsutter.com/2013/08/12/gotw-94-solution-aaa-style-almost-always-auto/) person, you probably shook your head quite often when you wanted to write a good old `for` loop. But even if you just had a look into legacy code's `for` loops, you probably saw too many messed up situations with loop indexes.

Let's have a look at a simple situation:

```cpp
#include <vector>

int main() {
  std::vector<int> v{0, 1, 2, 3};
    for (auto i = 0; i < v.size(); ++i) {
      /* use both i and v[i] */
    }
}
```

We try to use `auto` for the loop index, but we got a compiler warning! `std::vector<T>::size()` returns a `std::vector<T>::size_type`, usually `std::size_t` that is an unsigned type. At the same time, `0` is deduced as a signed integer. Comparing a signed with an unsigned type leads to a compiler warning. Hopefully, you don't tolerate compiler warnings in your project, so we consider that the above example does not compile.

In case, you want to store the size of the vector for optimization reasons, you even get a hard error, reminding you that the `auto` education for `i` and `s` was not consistent!

```cpp
#include <vector>

int main() {
  std::vector<int> v{0, 1, 2, 3};
    for (auto i = 0, s = v.size(); i < s; ++i) {
      /* use both i and v[i] */
    }
}
```
What if `0u` is used for initializing `i`? It depends on whether you have a helper variable to store the size and on your system.

The worst case is that `i` will be truncated on a 64-bit system as `0u` is deduced as an `unsinged int`, while `s` is a `long unsigned int`. In a better situation, you get a compilation error because of this:

```cpp
#include <vector>

int main() {
  std::vector<int> v{0, 1, 2, 3};
    for (auto i = 0u, s = v.size(); i < s; ++i) {
      /* use both i and v[i] */
    }
}
/*
main.cpp: In function 'int main()':
main.cpp:5:10: error: inconsistent deduction for 'auto': 'unsigned int' and then 'long unsigned int'
    5 |     for (auto i = 0u, s = v.size(); i < s; ++i) {
      |   
*/
```

These were the simple examples borrowed from the accepted proposal, but you can find there many more. In general, with an existing set of literal suffixes, you can run into situations when you want the compiler to deduce the type for you for an integer literal because 
- comparing signed with unsigned elements is unsafe
- and you cannot replace `std::size_t` with `ul` (`unsigned long`) because you can run into narrowing/truncating situations when switching between 32-bit and 64-bit systems

To avoid the problems, you either have to use some verbose casts (mostly `static_cast`) or introduce a helper variable without relying on `auto` type deduction.

As mentioned in the beginning, [P0330R8] finally solves this problem by introducing `Z` and `UZ`. `Z` introduces the signed version of `std::size_t` and `UZ` the unsigned version.

With that, our previous examples should compile without any problem and unpleasant surprises as such:

```cpp
#include <vector>

int main() {
  std::vector<int> v{0, 1, 2, 3};
    for (auto i = 0UZ, s = v.size(); i < s; ++i) {
      /* use both i and v[i] */
    }
}
```

Just make sure that you compile with the option `-std=c++2b`.

## Conclusion

In this article, we saw why it's difficult to use literal suffixes and `auto` type deduction for good old loops and how [P0330R8](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p0330r8.html) solves this situation by introducing `Z`/`UZ` in C++23 to denote `std::size_t`.

Where do you think the signed version of `size_t` comes in handy?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!