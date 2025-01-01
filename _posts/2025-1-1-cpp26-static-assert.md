---
layout: post
title: "C++26: user-generated static_assert messages"
date: 2025-1-1
category: dev
tags: [cpp, cpp26, static_assert]
excerpt_separator: <!--more-->
---
Our first quest into the world of C++26 was about [`=delete` with an optional error message](https://www.sandordargo.com/blog/2024/12/18/cpp26-delete-with-a-reason), which improves the readability of the source code and potentially the error messages. In this next part of our journey, we will continue to focus on readability improvements, particularly those for error messages.

With C++26 and the acceptance of [P2741R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2741r3.pdf), we are getting better error messages for `static_assert`. Compile time assertions were introduced in C++11 with a mandatory message. Since C++17, the message is only optional. And with C++26, `static_assert` evolves further and the message can be a constant expression instead of an unevaluated string. In other words, it can have some dynamic elements.

This was not the first attempt to make such a feature part of C++, but back in 2014, when something similar was proposed, C++ didn't have enough compile-time programming capabilities to make this easily implementable.

## Why is this change useful?

Before looking at some concrete examples of utilisation, let's dig into the motivations behind this feature.

Obviously, we want better error messages. C++ is infamous for its horrendous error logs when the compilation fails. Even though we must admit that it's more and more just a legacy, the situation improved a lot over the course of the years. In any case, it was mostly template-related error messages that were behind the biggest issues.

While compile-time assertions with their custom messages already meant an improvement, user-generated diagnostic messages will further improve the situation.

But that's not the only motivation. Sometimes you don't want different, or better messages. You just want the same message in several assertions without having to duplicate it. Without this change, the following piece of code doesn't compile:

```cpp
constexpr std::string_view sv{"abraka"};
static_assert(true, sv);
/*
error: 'static_assert' with a user-generated message is a C++26 extension [-Werror,-Wc++26-extensions]
     static_assert(true, sv);
*/
```

Starting from C++26, we'll be able to reuse error messages.

Moreover, this feature will come in handy with reflection as well as it can use the name of instantiated template parameters.

## Some more examples

We already said that we'll be able to reuse error messages. Now let's see two more concrete examples of how we'll be able to use them. 

For example, since the size of a type is available at compile-time, we can use that in our assertion messages.

```cpp
static_assert(sizeof(S) == 1, std::format("Unexpected sizeof: expected 1, got {}", sizeof(S)));
```

To be fair, in many cases `S` would be a template parameter and we can already use [concepts](https://www.sandordargo.com/tags/concepts/) with a similarly readable error message:

```cpp
template <typename S> requires (sizeof(S) == 1)1)
void foo(S s) {
   // ...
}
foo(42);
/*
<source>:25:6: note: candidate template ignored: constraints not satisfied [with S = int]
   25 | void foo(S s) {
      |      ^
<source>:24:33: note: because 'sizeof(int) == 1' (4 == 1) evaluated to false
   24 | template <typename S> requires (sizeof(S) == 1)
*/
```

There is another example with `enum`s which I like even more. Imagine that you have two enums that you have to keep in sync, they should have the same number of elements. You might even add an enumerator called `count` to both your enums.

You could already write a `static_assert` comparing the values of the `count` enumerators: 

```cpp
#include <type_traits>

enum class A {
    a,
    b,
    c,
    count,
};

enum class B {
    a,
    b,
    c,
    count,
};

static_assert(static_cast<std::underlying_type_t<A>>(A::count) == 
              static_cast<std::underlying_type_t<B>>(B::count),
              "The number of enumerators in A and B must be the same");
int main() {
    
}
```

But now, you'll be able to use directly their values as well, for better error messages. Without having to use tricks.

```cpp
static_assert(static_cast<std::underlying_type_t<A>>(A::count) == 
              static_cast<std::underlying_type_t<B>>(B::count),
              std::format("The number of enumerators in A ({}) and B ({}) must be the same", static_cast<std::underlying_type_t<A>>(A::count), static_cast<std::underlying_type_t<B>>(B::count)));
```

We must admit that the excessive use of `static_cast`s is not very readable though.

## Conclusion
`static_assert` further evolves with C++26. Our tool for compile-time assertion will support user-generated error messages. As such, we'll be able to reuse messages as well as enrich them with information available at compile-time making diagnostics easier to read.

At the moment of publication, this feature is already supported by GCC 14 and Clang 17.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)