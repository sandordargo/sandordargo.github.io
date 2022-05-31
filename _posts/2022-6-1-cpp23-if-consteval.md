---
layout: post
title: "C++23: Consteval if to make compile time programming easier"
date: 2022-6-1
category: dev
tags: [cpp, cpp23, const, consteval]
excerpt_separator: <!--more-->
---
Let's continue our exploration of C++23 features! This week we discuss [how to call `consteval` functions from not explicitly constant evaluated ones](https://en.cppreference.com/w/cpp/language/if#Consteval_if).

[This paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1938r0.html), this new feature is also a good example to see how C++ evolves. C++20 introduced 2 new keywords, [`consteval` and `constinit`](https://www.modernescpp.com/index.php/c-20-consteval-and-constinit). Although they've been good additions, in the meanwhile the community found some bugs and also came up with some ideas for improvement. And here they are shipped with the next version of C++!

## What is `if consteval`?

The amount of `const*` syntax is clearly growing in C++. `const` was part of the original language and then we got `constexpr` with C++11. C++17 introduced `if constexpr`, C++20 brought us `consteval` and `constinit`, and with C++23 we are going to get `if consteval` (often referred to as *consteval if*).

Let's see what the latest addition is about.

A *consteval if statement* has no condition. Better to say, it's the condition itself. If it's evaluated in a *manifestly constant-evaluated context*, then the following compound statement is executed. Otherwise, it's not. In case there is an `else` branch present, it will be executed as you'd expect it.

If that helps readability, you can also use `if !consteval`. The following two pieces of code are equivalent.

```cpp
if !consteval {
  foo(); 
} else {
  bar();
}

// same as

if consteval {
  bar();
} else {
  foo();
}
```

## How to call `consteval` functions?

To answer that question, let's remind us of the difference between a `constexpr` and a `consteval` function. A `constexpr` function's return value can be computed at compile-time or during run-time. A `consteval` function is guaranteed to be executed during compile time, it's also called an *immediate function*.

In C++, we have a tendency of moving more and more computations to compile time. As such we slightly increase the compile-time (although it still goes down due to better compilers and more powerful computers), but we speed up the runtime. Following these trends and benefitting from compile-time computations, you might want to call `consteval` functions from `constexpr` functions. But it's not going to work with C++20.

```cpp
consteval int bar(int i) {
    return 2*i;
}

constexpr int foo(int i) {
    return bar(i);
}

int main() {
  [[maybe_unused]] auto a = foo(5);
}
/* 
In function 'constexpr int foo(int)':
error: 'i' is not a constant expression
      |        return bar(i);
      |                ~~~^~~
*/
```

It makes sense. After all, as `foo(int)` is a `constexpr` function, it can be executed at runtime too. But what if you really want to call a `consteval` function from a `constexpr` function when it's executed at compile time?

In C++20, `consteval` functions could invoke `constepxr` ones, but not the other way around. Even if you try to surround the call of the `consteval` function with `std::is_constant_evaluated()`, it won't change. The following example is not going to work, because `i` is still not a constant expression:

```cpp
consteval int bar(int i) {
    return 2*i;
}

constexpr int foo(int i) {
    if (std::is_constant_evaluated()) {
        return bar(i);
    }
    return 2*i;
}

int main() {
  [[maybe_unused]] auto a = foo(5);
}
/*
main.cpp: In function 'constexpr int foo(int)':
main.cpp:6:14: error: 'is_constant_evaluated' is not a member of 'std'
    6 |     if (std::is_constant_evaluated()) {
      |              ^~~~~~~~~~~~~~~~~~~~~
main.cpp:7:19: error: 'i' is not a constant expression
    7 |         return bar(i);
      |                ~~~^~~

*/
```

[This proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1938r0.html) fixes it, by adding the new language feature of `if consteval`. Use that to call consteval functions from constexpr ones. In fact, not only from constexpr ones but from any function. Just make sure that you set the `-std=c++2b` compiler flag.


```cpp
consteval int bar(int i) {
    return 2*i;
}

int foo(int i) {
    if consteval {
        return bar(i);
    }
    return 2*i;
}

int main() {
  [[maybe_unused]] auto a = foo(5);
}
```

While `if consteval` behaves exactly as `if (std::is_constant_evaluated)`, it's superior to it because it doesn't need any header include, its syntax is crystal clear, plus you can invoke consteval functions if it evaluates to true.

## Conclusion

In this article, we learnt about a new C++ feature, `if consteval` that will help us invoke `consteval` functions when the context is constant-evaluated, yet it's not explicitly declared so.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
