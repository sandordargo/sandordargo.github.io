---
layout: post
title: "C++26: Span improvements"
date: 2026-3-18
category: dev
tags: [cpp, cpp26, span]
excerpt_separator: <!--more-->
---
A while back, we talked about how [using `std::span` instead of C-style arrays](https://www.sandordargo.com/blog/2024/11/06/std-span) makes your code safer and easier to reason about. `std::span`, added in C++20, is a non-owning view over a contiguous sequence of objects - think of it as a `string_view`, but for arrays. C++23 continued building on this foundation, bringing related utilities such as [`<spanstream>`](https://www.sandordargo.com/blog/2023/12/06/cpp23-strtream-strstream-replacement) and [`mdspan`](https://www.sandordargo.com/blog/2023/08/15/cpp23-mdspan-mdsarray), the multidimensional cousin of `span`.

Now let's see what additional improvements we are getting with C++26.

<!--more-->

## [P2447R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2447r6.html): `std::span` over an initializer list

Just like a `string_view` can replace `const string&` in most cases — as long as null-termination is not required — `span<const int>` can similarly replace `const vector<int>&` in most cases. However, there was one ergonomic gap: an initializer list - e.g. `{1, 2, 3}` - failed to bind to a `span` function parameter. You needed double braces to make it work:

```cpp
void take(std::span<const int> v);

take({1, 2, 3});    // error before C++26
take({{1, 2, 3}});  // works before C++26
```

What this paper really proposes is fairly simple: *"`std::span<const T>` should be convertible from an appropriate braced-initializer-list. In practice this means adding a constructor from `std::initializer_list`"*.

After C++26, the first call will just work.

Note that this is a breaking change. The paper includes an example in the [*Breaking changes*](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2447r6.html) section - mainly around overload resolution ambiguity when both a `span<const int>` overload and another overload are candidates for `{1, 2, 3}`. In real life, this will barely cause any issues.

## [P2821R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2821r5.html): `span.at()`

This is a very simple addition - I'd even say a complement that should have been part of `std::span` from the very beginning. It adds bounds-checked lookup to `span`.

All other contiguous containers and views - `std::string`, `std::string_view`, `std::vector`, `std::array`, `std::deque` - offer both unsafe indexing via `operator[]` and safe, throwing access via `at()`. `span` was the odd one out.

It's not entirely clear from the paper why `at()` was originally omitted. My best guess is that the original authors wanted to avoid a throwing member function. Maybe it was just an oversight.

One way or the other, just like other contiguous containers and `string_view`, `span` is also going to have bounds-checked lookup:

```cpp
std::array<int, 4> arr = {1, 2, 3, 4};
std::span<int> s{arr};

s[10];    // undefined behaviour
s.at(10); // throws std::out_of_range
```

## [P2833R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2833r2.html): Freestanding Library: inout expected span

[P2833R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2833r2.html) extends the scope of _freestanding_ with several facilities that are largely unrelated to each other. But before we dive in, let's stop for a second and define what freestanding is.

> *A __freestanding__ implementation is one that operates without the support of a hosted operating system. Think embedded systems, OS kernels, or bare-metal environments where heap allocation, system calls, and exception support are typically unavailable. The C++ standard defines a minimal subset of the language and library that must work in such constrained environments.*

The above proposal adds the following facilities to freestanding:
- `out_ptr` and `inout_ptr`
- most of `expected`
- most of `span`
- `mdspan`

As freestanding environments generally cannot support exceptions, parts of these facilities cannot be included. Specifically, `span::at()` (since it throws `std::out_of_range`) and all overloads of `std::expected::value()` are excluded from the freestanding subset.

## [P3029R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3029r1.html): Better `mdspan` CTAD

While the title suggests we are only talking about class template argument deduction for [`mdspan`](https://www.sandordargo.com/blog/2023/08/15/cpp23-mdspan-mdsarray), the proposal actually covers regular `std::span` as well, to keep their deduction rules consistent.

Their CTAD becomes `integral_constant`-aware for more efficient type deduction. In practice, this means the compiler now checks whether the second argument passed to the `span` constructor - the one specifying the size or the end of the span - satisfies an `_integral-constant-like_` concept. If it does, the extent can be deduced as a static (compile-time) value rather than a dynamic one.

```cpp
int arr[5] = {1, 2, 3, 4, 5};
int* p = arr;

// Before C++26: deduced as span<int, dynamic_extent>
std::span s1(p, std::integral_constant<std::size_t, 5>{});

// After C++26: deduced as span<int, 5>
std::span s2(p, std::integral_constant<std::size_t, 5>{});
```

So `span(p, std::integral_constant<size_t, 5>{})` will now be deduced as `span<int, 5>` instead of `span<int, dynamic_extent>`. This preserves compile-time size information that was previously silently lost during deduction.

## Conclusion

C++26 brings four targeted improvements to `span` and its ecosystem. We get more ergonomic initialization via the new `initializer_list` constructor, a long-overdue `at()` for bounds-checked access, `span` (along with `expected`, `out_ptr`, `inout_ptr`, and `mdspan`) in freestanding environments, and smarter CTAD that preserves compile-time size information. None of these are groundbreaking changes, but they round off rough edges and make `span` a more complete and consistent tool in the standard library.

## Connect deeper

If you liked this article, please
- hit on the like button,
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
