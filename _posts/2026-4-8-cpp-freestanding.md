---
layout: post
title: "Freestanding standard library"
date: 2026-4-8
category: dev
tags: [cpp, freestanding]
excerpt_separator: <!--more-->
---
[In some earlier articles](https://www.sandordargo.com/blog/2026/03/18/cpp26-span-improvements#p2833r2-freestanding-library-inout-expected-span), we used the term *freestanding*. We said that a freestanding implementation is one that operates without the support of a hosted operating system. Think embedded systems, OS kernels, or bare-metal environments where heap allocation, system calls, and exception support are typically unavailable. The C++ standard defines a minimal subset of the language and library that must work in such constrained environments.

But I wanted to go slightly deeper, because the term doesn't come up much in everyday C++ — at least not until you start reading the standard more carefully or until you need to use such an implementation.

<!--more-->

## Two kinds of implementations

This is quite logical. If something is called *freestanding*, there must be an alternative — the one we're used to. It does have a name: *hosted*.

To check which one you're on, inspect the `__STDC_HOSTED__` macro. A value of `1` means you're on a hosted implementation; `0` means freestanding.

A few key differences:
- On a hosted implementation, a program can have more than one thread running concurrently. On a freestanding implementation, this is implementation-defined.
- On a hosted implementation, a program must contain a global `main` function. On a freestanding implementation, that requirement is implementation-defined.
- A hosted implementation must provide all standard headers. For freestanding, even that is implementation-defined — though the standard can mandate that specific entities must be provided or even deleted.

> What does a program do without `main()`? Start-up and termination can still be executed, including the constructors and destructors for objects with [static storage duration](https://www.sandordargo.com/blog/2022/05/18/scope-linkage-name#storage-duration).

Note that a vendor is not required to provide a freestanding implementation at all.

## What is actually guaranteed in freestanding

Let's go beyond *"it's implementation-defined"*. The standard does require a small but useful subset of the library to be available in freestanding environments.

You can generally rely on low-level, self-contained utilities. Headers such as `<cstdint>`, `<cstddef>`, `<limits>`, and `<type_traits>` are part of that minimal set. These provide fixed-width integer types, basic size definitions, compile-time type inspection, and numeric limits — all things that don't depend on an operating system. Parts of `<new>` are available too, although that doesn't necessarily mean you have a working heap or global allocation facilities.

What's notably absent are components that inherently depend on OS services: no `<thread>`, no `<filesystem>`, and typically no `<iostream>`. Even dynamic memory and exception support are not guaranteed. Freestanding gives you the building blocks of the language and library, but not the higher-level conveniences.

## The evolution of freestanding in modern C++

Freestanding used to be very limited — almost to the point where the standard library was barely usable. That's no longer true.

Starting from C++20 and continuing through C++23 and C++26, there has been a clear effort to [expand what is available in freestanding environments](https://www.sandordargo.com/blog/2026/03/18/cpp26-span-improvements#p2833r2-freestanding-library-inout-expected-span). More and more headers are being made freestanding-friendly, particularly those that don't rely on OS-level services. This includes parts of the algorithms library, various utility components, and with C++26, facilities like `std::span`, `std::expected`, `std::mdspan`, and smart pointer adapters like `out_ptr` and `inout_ptr`.

As a result, algorithms and utilities that used to be restricted to hosted implementations are increasingly usable without an operating system. You can write more expressive, higher-level code even in constrained environments, without having to reinvent everything from scratch.

This evolution didn't happen by accident. There has been a strong push from domains like embedded systems and game development, where developers want both performance and better abstractions. The goal is clear: make modern C++ usable everywhere, not just on full-fledged operating systems.

## Conclusion

Freestanding is not a niche technicality — it's a foundational distinction in the C++ standard. It clarifies what you can actually count on when the runtime environment offers minimal guarantees: no OS, no heap, no exceptions. The fact that the standard has been steadily expanding the freestanding subset from C++20 through C++26 reflects a genuine commitment to making modern C++ viable in resource-constrained environments.

If you work on embedded software, OS kernels, or bare-metal targets, understanding what freestanding guarantees — and what it doesn't — is directly relevant. And even if you don't, knowing the distinction sharpens your understanding of why certain standard library components are designed the way they are, particularly around exceptions, allocation, and OS dependencies.

Do you work with freestanding implementations? What environment are you targeting?

## Connect deeper

If you liked this article, please
- hit on the like button,
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
