---
layout: post
title: "C++26: Standard library hardening"
date: 2026-5-13
category: dev
tags: [cpp, cpp26, safety]
excerpt_separator: <!--more-->
---
Undefined behavior (UB) in C++ is one of the hardest categories of bugs to deal with. It can silently corrupt memory, cause crashes far from the actual mistake, or — worst of all — just happens to work on your machine. A significant share of UB in real codebases comes not from exotic language features, but from basic misuse of the standard library: accessing a `vector` out of bounds, calling `front()` on an empty container, or dereferencing an empty `optional`.

C++26 addresses this directly with standard library hardening, introduced via [P3471R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3471r4.html).

<!--more-->

## What is library hardening?

Library hardening converts certain undefined behavior in the standard library into detectable contract violations at runtime. When a hardened precondition is violated, the runtime reacts before any other observable side effect — think of it as adding bounds checking to operations that are UB today.

This is not a new idea. All three major standard library implementations already ship vendor-specific hardening modes. The problem is that these mechanisms are all different, non-portable, and inconsistently specified. [P3471R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3471r4.html) standardizes what implementations already do.

The standardized version of hardening becomes the first use of contracts (specified in [P2900](https://isocpp.org/files/papers/P2900R6.pdf)). This aligns perfectly with the direction of the language.

## The motivation: real-world evidence

The strongest argument for this feature is [Google's production experience, which the proposal references](https://security.googleblog.com/2024/11/retrofitting-spatial-safety-to-hundreds.html): applying hardened libc++ across *"hundreds of millions of lines of C++"* code found over 1,000 bugs, including security-critical ones. The average performance overhead was a surprisingly low **0.30%** — one third of a percent. That overhead came down thanks to the compiler's ability to eliminate redundant checks during optimization.

The impact went beyond security: teams observed a **30% reduction in baseline segmentation fault rates** in production, pointing to improved code correctness across the board.

What a remarkable result!

## What gets hardened?

The proposal focuses deliberately on *memory safety preconditions* only. Checking all preconditions *"is an explicit non-goal"* — the goal is to catch the checks that are cheap to add and high-value to have.

### `std::span`

- `operator[]`: requires `idx < size()`
- `front()`, `back()`: require non-empty
- `first()`, `last()`, `subspan()`: validate  that `count` and `offset` are in bounds
- Constructors from a range: verifies the extent matches

### `std::string_view`

- `operator[]`: requires `pos < size()`
- `front()`, `back()`: require non-empty
- `remove_prefix()`, `remove_suffix()`: require `n <= size()`

### Sequence containers (`vector`, `deque`, `list`, `forward_list`, `array`, `string`)

- `operator[]`: requires `n < size()` (for `basic_string`, `n <= size()` since accessing the null terminator is valid)
- `front()`, `back()`: require `!empty()`
- `pop_front()`, `pop_back()`: require `!empty()`

### `std::optional` and `std::expected`

- `operator->()`, `operator*()` on `optional`: require `has_value()`
- Value access on `expected`: requires `has_value()`
- `error()` on `expected`: requires `!has_value()`

### `std::mdspan`

- `operator[]`: validates all multidimensional indices within their extents
- Constructor: verifies static extents match during conversions

### `std::bitset` and `std::valarray`

- `operator[]`: requires `pos < size()` / `n < size()`

Some things are *intentionally* left out. Iterator-based operations like `erase()`, associative containers, and algorithms are deferred — they require more complex validity checks that don't fit the scope of this proposal.

## Concrete examples

Here are a couple of examples of what library hardening catches that today silently causes UB:

```cpp
std::vector<int> v = {1, 2, 3};
int x = v[5];       // contract violation: 5 >= 3
v.pop_back();
v.pop_back();
v.pop_back();
v.pop_back();       // contract violation: !empty() is false
```

```cpp
std::string_view sv("hello");
char c = sv[10];          // contract violation: 10 >= 5
sv.remove_prefix(10);     // contract violation: 10 > 5
```

```cpp
std::optional<int> opt;
int x = *opt;     // contract violation: has_value() is false
```

```cpp
std::span<int, 5> sp(data, 3);  // contract violation: extent mismatch
sp.first<10>();                  // contract violation: 10 > size()
```

None of these are difficult-to-imagine-to-happen situations. These are the kinds of bugs that slip through in code reviews and only surface in production.

## How con you activate it?

The proposal does *not* standardize the activation mechanism — this is intentionally left to implementations. In practice, you will see something like:

- A compiler flag: `-fhardened` or `-D_LIBCPP_HARDENING_MODE=...`
- Or a build-system option specific to the implementation

One deliberate design choice: in a hardened implementation, you cannot override a hardened precondition check with `ignore` semantics. The whole point of hardening is to provide a firm safety baseline. If you could turn off individual checks arbitrarily, the guarantee would be meaningless. Besides, as it's specified that a hardened precondition violation is a contract violation, *"we are already past the point where ignore could bypass the check"*.

To check whether your compiler already supports hardening, you can use [the various feature-test macros listed here](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3471r4.html#feature-test-macros).

At the moment of writing, GCC 15 and MSVC 19.44 already partially implemented [P3471R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3471r4.html).

## Conclusion

Library hardening is one of those features that makes you wonder why it wasn't standardized sooner. It catches real bugs, at negligible cost, across code you don't have to change. The three major implementations have been doing their own versions of this for years — C++26 finally makes it portable and consistent.

If you care about memory safety in your C++ code (and in 2026, you should), hardened standard library mode should be part of your build configuration.

## Connect deeper

If you liked this article, please
- hit on the like button,
- [subscribe to my newsletter](https://sandor-dargo.kit.com/e19f29b0a1)
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
