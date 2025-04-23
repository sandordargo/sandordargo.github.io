---
layout: post
title: "C++26: more constexpr in the core language"
date: 2025-4-23
category: dev
tags: [cpp, cpp26, structuredbindings, constexpr]
excerpt_separator: <!--more-->
---
Since `constexpr` was added to the language in C++11, its scope has been gradually expanded. In the beginning, we couldn't even use `if`, `else` or loops, which were changed in C++14. C++17 added support for `constexpr` lambdas. C++20 added the ability to use allocation and use `std::vector` and `std::string` in constant expressions. In this article, let's see how constexpr evolves with C++26. To be more punctual, let's see what language features become more `constexpr`-friendly. We'll discuss library changes in a separate article, as well as `constexpr` exceptions, which need both language and library changes.

## [P2738R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2738r1.pdf): `constexpr` cast from `void*`

Thanks to the acceptance of [P2738R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2738r1.pdf), starting from C++26, one can cast from `void*` to a pointer of type `T` in constant expressions, if the type of the object at that adress is exactly the type of `T`.

Note that conversions to interconvertible - including pointers to base classes - or not related types are not permitted.

The motivation behind this change is to make several standard library functions or types work at compile time. To name a few examples: *std::format*, *std::function*, *std::function_ref*, *std::any*. The reason why this change will allow many more for more `constexpr` in the standard library is that storing `void*` is a commonly used compilation firewall technique to reduce template instantiations and the number of symbols in compiled binaries.

## [P2747R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2747r2.html): `constexpr` placement new

As `std::construct_at` is a limited tool that only allows to perform value initialization but not others such as default or list initialization, there has been a need to make placement new usable in constant expressions.

At the same time, placement new is a very, maybe even too flexible tool and to use it in a safe way requires casting to `void*` and then back to `T*`. This faced some issues, but the acceptance of [P2738R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2738r1.pdf) and the ability of casting from `void*` in constant expressions made the impossible possible.

If you are looking for more details, check [P2747R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2747r2.html).


## [P2686R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2686r5.pdf): constexpr structured bindings and references to constexpr variables

This is a rather long (20 pages) proposal and I found it not particularly easy to read. That's not the fault of the authors, the problem is hard to address. The paper which is based on another, went through 5 revisions, discusses various solutions, and lists the wording changes on more than 10 pages.

Long story short, you'll be able to declare structured bindings `constexpr`.

As structured bindings behave like references, the same restrictions apply as to `constexpr` references. Those restrictions become more relaxed. Before, a `constexpr` reference had to bind to a variable with static storage duration, so that the address doesn't change from one evaluation to another. With C++26, in addition, variables with automatic storage duration are also accepted if and only if the address is constant relative to the stack frame in which the reference or the structured binding lives.

In practice, this means that you cannot have a `constexpr` reference in a lambda to bind to an enclosing function. The reason is that in order to access that variable, the expression is something like `this->__x` where `__x` represents the captured address of `x`. As we don't know at compile time what object `this` points to, it's not a constant expression.

## Conclusion

In this article, we reviewed how `constexpr` evolves in the C++26 core language. We are getting `constexpr` cast from `void*`, placement `new`, structured bindings and even exceptions (not discussed today). In the next article, we'll see how the standard library's constexpr support evolves.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!  