---
layout: post
title: "C++26: more constexpr in the standard library"
date: 2025-4-30
category: dev
tags: [cpp, cpp26, constexpr, standardlibrary]
excerpt_separator: <!--more-->
---
Last week, we discussed [language features that are becoming `constexpr` in C++26](https://www.sandordargo.com/blog/2025/04/23/cpp26-constexpr-language-changes). Today, let’s turn our attention to the standard library features that will soon be usable at compile time. One topic is missing: exceptions. As they need both core language and library changes, I thought they deserved their own post.

## [P2562R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2562r1.pdf): `constexpr` stable sorting

This paper proposes making `std::stable_sort`, `std::stable_partition`, `std::inplace_merge`, and their `ranges` counterparts usable in constant expressions. While many algorithms have become `constexpr` over the years, this family related to stable sorting had remained exceptions — until now.

The recent introduction of `constexpr` containers gives extra motivation for this proposal. If you can construct a container at compile time, it’s only natural to want to sort it there, too. More importantly, a `constexpr std::vector` can now support efficient, stable sorting algorithms.

A key question is whether the algorithm can meet its computational complexity requirements under the constraints of constant evaluation. Fortunately, `std::is_constant_evaluated()` provides an escape hatch for implementations. For deeper details, check out the [proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2562r1.pdf) itself.

## [P1383R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p1383r2.pdf): More `constexpr` for `<cmath>` and `<complex>`

While [P0533](https://www.sandordargo.com/blog/2023/05/24/cpp23-constexpr#constexpr-for-cmath-and-cstdlib) made many `<cmath>` and `<cstdlib>` functions `constexpr`-friendly in C++23, it only addressed functions with trivial behavior — those no more complex than the basic arithmetic operators.

Floating-point computations can yield different results depending on compiler settings, optimization levels, and hardware platforms. For instance, calculating `std::sin(1e100)` may produce varying outcomes due to the intricacies of floating-point arithmetic at such scales. The paper discusses these challenges and suggests that some variability in results is acceptable, given the nature of floating-point computations.

[The proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p1383r2.pdf) accepts the need for a balance between strict determinism and practical flexibility. It suggests that while some functions should produce consistent results across platforms, others may inherently allow for some variability.	

## [P3074R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3074r7.html): trivial `union`s (was `std::uninitialized<T>`)

To implement static, in-place, `constexpr`-friendly containers like non-allocating vectors, you often need uninitialized storage — typically via `union`s. However, default behavior for special members of unions has been limiting: if not all alternatives are trivial, the special member is deleted. This presents a problem for `constexpr` code where a no-op destructor isn't quite the same as a trivial one.

The road to solving this wasn’t short: [P3074R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3074r7.html) went through seven revisions and considered five possible solutions—including library-based approaches, new annotations, and even a new union type. Ultimately, the committee decided to just *make it work* with minimal changes to the user experience.

But how?

For unions, the default constructor - if there is no default member initializer - is always going to be trivial. If the first alternative is an implicit-lifetime time, it begins its life-time and becomes the active member.

The defaulted destructor is deleted if either the union has a user-provided default constructor or there exists a variant alternative that has a default member initializer and that member’s destructor is either deleted or inaccessible. Otherwise, the destructor is trivial.

This excerpt from the proposal shows the changes well.

```cpp
// trivial default constructor (does not start lifetime of s)
// trivial destructor
// (status quo: deleted default constructor and destructor)
union U1 { string s; };

// non-trivial default constructor
// deleted destructor
// (status quo: deleted destructor)
union U2 { string s = "hello"; }

// trivial default constructor
// starts lifetime of s
// trivial destructor
// (status quo: deleted default constructor and destructor)
union U3 { string s[10]; }

// non-trivial default constructor (initializes next)
// trivial destructor
// (status quo: deleted destructor)
union U4 { string s; U4* next = nullptr; };
```

## [P3372R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3372r2.html): `constexpr` containers and adaptors

Hana Dusíková authored [a massive proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3372r2.html) that boils down to a simple goal: make (almost) all containers and adaptors `constexpr`.

Up until now, only a handful of them were `constexpr`-friendly (`std::vector`, `std::span`, `std::mdspan`, `std::basic_string` and `std::basic_string_view`). From now on, the situation will be flipped. Almost everything will be `constexpr`-friendly. There is one exception and one constraint:
- `std::hive` is not included, because it doesn't have a stable wording yet
- if you want to use unordered containers at compile-time, you must provide your own hashing facility, because `std::hash` cannot be made `constexpr`-friendly due to its requirements. Its result is guaranteed to be consistent only with the duration of the program.

Happy days!


## [P3508R0](www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3508r0.html): Wording for "`constexpr` for specialized memory algorithms"

Such a strange title, isn't? Wording for *something*...

As it turns out, there was already a paper accepted ([P2283R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2283r2.pdf)) making specialized memory algorithms `constexpr`-friendly. Algorithms that are essential for implementing `constexpr` container support, yet they were forgotten from C++20.

These algorithms are (both in `std` and in `std::ranges` namespaces):
- `uninitialized_value_construct`
- `uninitialized_value_construct_n`
- `uninitialized_copy`
- `uninitialized_copy_result`
- `uninitialized_copy_n`
- `uninitialized_copy_n_result`
- `uninitialized_move`
- `uninitialized_move_result`
- `uninitialized_move_n`
- `uninitialized_move_n_result`
- `uninitialized_fill`
- `uninitialized_fill_n`


When the paper was made, the necessary implementation change was to use `std::construct_at` instead of *placement new*, as `std::consturct_at` was already `constexpr`. But in the meantime, [P2747R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2747r2.html) was accepted and *placement new* in the core language also became `constexpr`. Therefore, the implementation of the above functions doesn't have to be changed, only their signatures have to be updated to support `constexpr`. Hence, the wording change.

## [P3369R0](www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3369r0.html): `constexpr` for `uninitialized_default_construct`

We saw that the `constexpr` *placement new* affected [P2283R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2283r2.pdf) and raised the need for a wording change performed in [P3508R0](www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3508r0.html). But that's not the only side-effect it had. From the above-listed algorithm families, one is missing: `uninitialized_default_construct`. The reason is that `uninitialized_default_construct` cannot be implemented with `std::construct_at` as it always performs value initialization, default initialization was impossible.

But with `constexpr` *placement new* this is not an issue anymore, therefore `uninitialized_default_construct` can also be turned into `constexpr`.

## Conclusion

C++26 marks a huge step forward for `constexpr` support in the standard library. From stable sorting algorithms to containers, from tricky union rules to specialised memory functions, compile-time programming is becoming more and more supported. 

In the next article, we’ll cover compile-time exceptions! 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!  