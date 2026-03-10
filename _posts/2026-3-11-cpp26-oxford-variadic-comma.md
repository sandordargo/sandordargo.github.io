---
layout: post
title: "C++26: The Oxford variadic comma"
date: 2026-3-11
category: dev
tags: [cpp, cpp26, variadics, deprecation]
excerpt_separator: <!--more-->
---
C++26 brings us a small but meaningful cleanup to the language: deprecating ellipsis parameters without a preceding comma. This change, proposed in [P3176R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3176r1.html), aims to improve C compatibility, reduce confusion, and pave the way for future language features.

The proposal's name is a playful reference to the [Oxford comma](https://en.wikipedia.org/wiki/Serial_comma) - that final comma before "and" in a list. Just as the Oxford comma clarifies lists in English, this proposal mandates a comma before the ellipsis in function parameters.

## The problem

First, let's clarify terminology. This proposal is about *ellipsis parameters* - [the C-style variadic parameters](https://www.sandordargo.com/blog/2023/05/03/variadic-functions-vs-variadic-templates) you might know from `printf`. These are different from template parameter packs, even though both use the `...` syntax.

Currently, C++ allows two ways to declare an ellipsis parameter:

```cpp
void foo(int, ...);  // with comma (C-compatible)
void foo(int...);    // without comma (C++-only)
```

The second form without the comma originates from early (pre-standard) C++ function prototypes, but has remained part of standardized C++ ever since. Interestingly, C has *never* allowed the comma to be omitted. The C standard, unchanged since C89, requires:

```cpp
// In C, only this form is valid:
int printf(char*, ...);
```

C++ later added support for the comma-separated form for C compatibility, but kept the old syntax for [backwards compatibility](https://www.youtube.com/watch?v=0_UttFDnV3k). This creates an awkward situation where `(int, ...)` is compatible with both languages, but `(int...)` only works in C++.

## Why is this confusing?

The real confusion comes from template parameter packs, introduced in C++11. Consider this example:

```cpp
template<class Ts>
void f(Ts...); // well-formed: a parameter of type Ts followed by an ellipsis parameter
```

Many users associate `(T...)` with a parameter pack, not with an ellipsis parameter. Instead, it's a single parameter of type `Ts` followed by an ellipsis parameter! To actually declare a parameter pack, you need:

```cpp
template<class... Ts>
void f(Ts... args);  // args is a parameter pack
```

The situation gets even more confusing with abbreviated function templates:

```cpp
// abbreviated variadic function template
void g(auto... args);

// abbreviated non-variadic function template with ellipsis parameter
void g(auto args...);
```

These two look similar but have completely different meanings. The latter should be deprecated.

## The curious case of six dots

Perhaps the most bizarre syntax enabled by the current rules is this:

```cpp
void h(auto......);  // equivalent to (auto..., ...)
```

Yes, that's six consecutive dots - if I counted it right. This declares a function template parameter pack followed by an ellipsis parameter. While technically possible to use (if the pack belongs to a surrounding class template), the syntax strongly suggests all dots apply to `auto`, which is misleading.

## What's being deprecated?

C++26 will deprecate ellipsis parameters without a preceding comma. Specifically:

```cpp
// Deprecated in C++26:
void f(int...);
void g(auto args...);
template<class T> void h(T...);  // T is not a parameter pack

// Preferred (and C-compatible):
void f(int, ...);
void g(auto args, ...);
template<class T> void h(T, ...);
```

The standalone ellipsis parameter remains valid:

```cpp
void f(...);  // still valid, C-compatible, unambiguous
```

## Impact

This is a pure deprecation - removal had been already refused before. No existing code becomes ill-formed. Any deprecated uses can be mechanically transformed by adding a comma before the ellipsis. This transformation is simple enough to be automated by tooling.

The proposal doesn't estimate how much code will be affected, though the author found several dozen occurrences of the `T......` pattern in a GitHub code search. The real number of affected declarations is likely non-trivial, finding them requires semantic analysis since `(T...)` could be either an ellipsis parameter or a parameter pack depending on context.

This deprecation clears the path for future language features. The syntax `(int...)` has already blocked proposals like [P1219R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1219r2.html) *"Homogeneous variadic function parameters"*, which would have given this syntax a new meaning.

By deprecating the comma-less form, the committee preserves design space for future evolution while improving consistency with C and reducing a common source of confusion.

## Conclusion

The Oxford variadic comma is a small change with multiple benefits: better C compatibility, reduced confusion with parameter packs, and preserved design space for future features. While the title is playful, the motivation is serious - cleaning up a historical artifact that serves little purpose in modern C++.

If you're using ellipsis parameters, start adding that comma before the `...`. Your code will be more C-compatible, less confusing, and ready for whatever comes next.

## Connect deeper

If you liked this article, please
- hit on the like button,
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
