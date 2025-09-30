---
layout: post
title: "C++26: std::optional<T&>"
date: 2025-10-1
category: dev
tags: [cpp, cpp26, optional, reference]
excerpt_separator: <!--more-->
---
If you're regular on my blog, you know that I share what I learn about new C++ language and library features ever since C++20. You probably also read my [CppCon2025 Trip Report](). And here is where the two meets each other.

I attended a great presentation by Steve Downey about [`std::optional<T&>`](). It's worth noting that Steve is the father of optional references, he authored [P2988R12](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2988r12.pdf) along with Peter Sommerlad.

Let's start with a bit of history. Yes, it's about the end of 2025, so C++2017 is history in a certain way. `std::optional<T>` was introduced by C++17. It provides us a way to express whether a value is there while using value semantics. You don't need to rely on pointers. But C++17 `std::optional` cannot hold references - well, unless you use `std::reference_wrapper`... C++26 fixes this problem.

## What is `std::optional<T&>`

`std::optional<T&>` has three key characteristics
- unlike `std::optional<T>`, this is not an owning type. It just holds a reference to a value
- it provides both reference and value semantics
- it can hold a pointer to the underlying object of type T or`nullptr`

Let's talk a little bit more about this last point. While `optional<T>` is like a variant of `T` and `std::monostate`, in this case it could somehow replace a variant of `T&` and `nullptr`. Essentially a pointer. We can use `std::optional<T&>` instead of a non-owning raw pointer.

Does this mean one less reason to use raw pointers?

I mean one should not use owning raw-pointers since C++11 at all. Though passing around raw pointers is still acceptable to express that those are not-owning. C++26 might change that. At least with optionals we can spare the `nullptr` checks, even if one way or anotehr we still have to check if the optional is engaged.

## Key considerations

Making the least surprising and least dangerous choices. They are usually the same.

### Assign or Rebind?

Take this piece of code. What should it do?

```cpp
// copied directly from Steve Downey CppCon presentation
Cat fynn;
Cat loki;
std::optional<Cat&> maybeCat1;
std::optional<Cat&> maybeCat2{fynn};
maybeCat1 = fynn;
maybeCat2 = loki;
``` 

One of the most important design decisions where what these assignments should do? Should they be allowed in the first place?

They could have been assignments. Meaning that at the end, `maybeCat1` referring to `fynn` would have contained a copy of `loki` as `maybeCat2` was referring `fynn`.

But eventually state independence won and `operator=` for `optional<T&>` always means rebinding. So in the above example, at the end, `maybeCat1` binds to `fynn` and `maybeCat2` binds to `loki` instead of the original `fynn`.

Two notes:
- so in the above code `maybeCat1 = fynn` changes nothing, it's superfluous
- while one cannot rebind a `T&`, it's possible for `optional<T&>`


### What about `make_optional()`?

`make_optional()` returns an optional of `T`. But let's not forget that it's a function template and it's largely supplanted by CTAD. `std::make_optional<T&>()` used to create an `optional<T>`. And in fact, it's not going to change, it will still create an `optional<T>` and **NOT** an `optional<T&>`. 

Passing in a reference to create an optional reference would always dangle.

It's also worth noting that the main usage of `make_optional<T&>` are tests cases.

### The question of constness

Should `optional<T&>` model shallow or deep `const`? In other words, should `operator->()` or `operator*()` return `T&` or `const T&` for a `const optional<T&>`?

The designers went with shallow constness, so the dereference operators return `T&`. If we need deep constness and we expect to have `const T&` returned in the above cases, we can still use `optional<const T&>`.


### The least dangerous `value_or`

What should `value_or` return for `optional<T&>`. According to the presentation and the paper. There was no strong consensus over this question, but the least - strong - objections was about the option that it would return a value, a.k.a. `T`.

This is the solution that will least likely causing issues and also allows the common case of using a literal as the alternative value.

Note that the author intends to propose free functions, such as `reference_or`, `value_or`, `or_ivoke` and `yield_if` over all types modeling optional-like.

You might read about it on this blog in the near future.

## Conclusion


## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!  