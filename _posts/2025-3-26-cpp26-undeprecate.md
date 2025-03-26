---
layout: post
title: "C++26: an undeprecated feature"
date: 2025-3-26
category: dev
tags: [cpp, cpp26, undeprecate, allocators]
excerpt_separator: <!--more-->
---
During the last two weeks, first we saw what are the language features deprecated or removed from C++26 then we did the same for library features. Life is not so straight and easy though. Sometimes, features cannot be removed after deprecation. There is an example for that in C++26 too. Which we are going to review today. 

## Undeprecate `polymorphic_allocator::destroy` for C++26

C++23 deprecated the `std::polymorphic_allocator::destroy` member function and instead of having it removed, it's being added back to C++26 by [P2875R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2875r4.pdf).

The short reason for the deprecation and a hoped removal was that the purposes of `std::polymorphic_allocator::destroy` are satisfied by `std::allocator_traits` too. But it turned out in practice, that some use-cases of `polymorphic_allocator::destroy` don't involve generic code that would use `std::allocator_traits`.

But reading through [the proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2875r4.pdf) unveils a more complex and interesting story. To understand what happened, we must travel back in time almost ten years.


There was already an issue filed for C++17 claiming that the implementations of the above-mentioned `polymorphic_allocator::destroy` function and `allocator_traits::destroy` function are equivalent. And that was true! That led to the deprecation of `polymorphic_allocator::destroy`.

But in C++20, the contract of `allocator_traits::destroy` changed! The implementation doesn't produce the same code anymore as `polymorphic_allocator::destroy`. It might call `destroy_at` - in case the allocator doesn't have a `destroy` member function - which adds another level of indirection, it's not `noexcept` itself, the optimizer might be as efficient anymore in removing unwinding code.

And while, C++23 - despite the above contract change - finally deprecated `polymorphic_allocator::destroy`, `allocator_traits` must still dispatch calls to it. That's because it will dispatch calls whenever the allocator's have a destroy member function.

An additional and important problem with `allocator_traits::destroy` is that it takes non-`const&` to an allocator and it might not work correctly when you deal with hierarchies of allocators as the right type must be known at compile time. `polymorphic_allocator` was designed to be type-agnostic through type deduction of the pointer.

If these problems wouldn't be enough on their own, `polymorphic_allocator::destroy` is a natural counterpart of `polymorphic_allocator::construct`. It just feels right and easy to use the two together.

As a result, `polymorphic_allocator::destroy` is undeprecated and kept as part of the standard library.

## Conclusion

In this article, we saw that the deprecation of functionality in C++ doesn't necessarily mean a guaranteed removal. Sometimes, intentions and contracts change or people understand that deprecation is simply not the right direction. In C++26, as far as I found, there is one (library) feature undeprecated, and that's `polymorphic_allocator::destroy`. For further analysis, feel free to read [P2875R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2875r4.pdf).

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
