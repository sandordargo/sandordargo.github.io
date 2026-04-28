---
layout: post
title: "C++26: Structured Bindings can introduce a Pack"
date: 2026-4-22
category: dev
tags: [cpp, cpp26, structuredbindings, packs]
excerpt_separator: <!--more-->
---
Last week, we talked about how C++26 improves structured bindings by [allowing them to be used in conditionals' init statements](https://www.sandordargo.com/blog/2026/04/15/cpp26-structured-bindings-condition). We also briefly touched on other improvements coming in C++26, such as [individual binding annotations](https://www.sandordargo.com/blog/2025/01/29/cpp26-attributes-structured-bindings) and [constexpr bindings](https://www.sandordargo.com/blog/2025/04/23/cpp26-constexpr-language-changes#p2686r5-constexpr-structured-bindings-and-references-to-constexpr-variables).  

There is, however, one important enhancement to structured bindings that we haven't covered yet on this blog:

> [Structured Bindings can introduce a Pack](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p1061r10.html)

This is not the only improvement C++26 brings to parameter packs either — I already wrote about [pack indexing here](https://www.sandordargo.com/blog/2025/01/22/cpp26-pack-indexing).

## What is a pack?

Before diving into the feature itself, let's briefly clarify what a *pack* is. A reasonably accurate (and not overly simplistic) definition is the following:

> A parameter pack is a language construct that represents an arbitrary number of types or values, allowing code to accept and operate on an arbitrary number of arguments in a type-safe way.

I deliberately avoided using the word *template* in this definition. Historically, packs only existed in templated contexts, but that limitation is slowly being relaxed. Earlier versions of the proposal behind this feature even aimed to support structured binding packs in non-templated contexts. That part was eventually dropped due to implementation complexity and related objections — but the direction is still telling.

## Structured bindings with packs

If you're familiar with Python 3, what becomes possible in C++26 might already feel familiar. The proposal extends structured bindings with pack bindings, enabling far more flexible decompositions.

Here are a few examples taken directly from the paper:

```cpp
std::tuple<X, Y, Z> f();

auto [x,y,z] = f();          // OK today
auto [...xs] = f();          // proposed: xs is a pack of length three containing an X, Y, and a Z
auto [x, ...rest] = f();     // proposed: x is an X, rest is a pack of length two (Y and Z)
auto [x,y,z, ...rest] = f(); // proposed: rest is an empty pack
auto [x, ...rest, z] = f();  // proposed: x is an X, rest is a pack of length one
                             //   consisting of the Y, z is a Z
auto [...a, ...b] = f();     // ill-formed: multiple packs
```

Introducing packs into structured bindings is already powerful on its own, but it also makes it trivial to focus on individual elements at the beginning or the end of a decomposition. Combined with expressive [placeholders](https://www.sandordargo.com/blog/2025/01/08/cpp26-unnamed-placeholders) like `_`, ignoring elements becomes both easy and readable.

## A more elaborate example: dot product

Of course, packs in structured bindings really shine in more substantial use cases. To keep this article concise, I'll only share one example here, but I encourage you to read both the [Motivation](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p1061r10.html#motivation) and the [Proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p1061r10.html#proposal) sections of *P1061R10* for more.

Before C++26, computing the dot product of two tuples required either:
- nested `std::apply()` calls, or
- a less-than-readable solution involving `std::index_sequence`.

Both approaches work, but they’re cumbersome in different ways.

Personally, I find the nested `std::apply` solution more readable than the index-sequence-based one — especially when compared to what it becomes with structured binding packs. Here's the pre-C++26 version, with a double nesting of `std::apply`:



```cpp
template <class P, class Q>
auto dot_product(P p, Q q) {
    return std::apply([&](auto... p_elems){
        return std::apply([&](auto... q_elems){
            return (... + (p_elems * q_elems));
        }, q)
    }, p);
}
```

With C++26 structured binding packs, all that indirection simply disappears:

```cpp
template <class P, class Q>
auto dot_product(P p, Q q) {
    // no indirection!
    auto&& [...p_elems] = p;
    auto&& [...q_elems] = q;
    return (... + (p_elems * q_elems));
}
```


## Conclusion

Allowing structured bindings to introduce packs is a natural and powerful evolution of an already popular language feature. By potentially removing layers of indirection and boilerplate, it enables code that is not only shorter, but also clearer and easier to reason about.

This change fits a broader pattern we've been seeing in modern C++: features that were once confined to narrow use cases are becoming more accessible. Structured binding packs don't just save a few lines of code but they unlock new ways of thinking about decomposition and data access, while staying fully type-safe.

C++26 continues to refine existing abstractions rather than piling on new ones, and structured binding packs are a great example of how small, well-targeted extensions can significantly improve everyday code.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](https://sandor-dargo.kit.com/e19f29b0a1)
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!