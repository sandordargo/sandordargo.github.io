---
layout: post
title: "Use concepts with std::remove_cvref_t"
date: 2025-8-13
category: dev
tags: [cpp, concepts, bestpractices, templates]
excerpt_separator: <!--more-->
---
*This article was inspired by [Mateusz Pusz' The 10 Essential Features for the Future of C++ Libraries
 talk at C++ on Sea 2025.](https://cpponsea.uk/2025/session/the-10-essential-features-for-the-future-of-cpp-libraries)*

Let's talk about templates, constraints, and [concepts](https://www.sandordargo.com/tags/concepts/). We'll start with a quick reminder of why concepts are essential when working with templates. Then we'll dive into the challenge posed by reference-qualified types and finish with a practical solution.

## Avoid unconstrained templates

By now, it's well known that using unconstrained templates is discouraged. Even the C++ Core Guidelines strongly recommend against it.

[T.47](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t47-avoid-highly-visible-unconstrained-templates-with-common-names) only advises avoiding highly visible unconstrained templates with common names due to the risks of argument-dependent lookup going wrong. [T.10](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t10-specify-concepts-for-all-template-arguments) goes further, recommending that we specify concepts for every template argument to improve both simplicity and readability.

The same idea appears in [I.9](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i9-if-an-interface-is-a-template-document-its-parameters-using-concepts), which suggests documenting template parameters using concepts.

It's hard to argue with these guidelines. Concepts make code more readable — just by looking at a function, class, or variable template, the reader can immediately tell what kinds of types are accepted.

> *If you want to learn more about concepts, check out my [concepts-related articles](https://www.sandordargo.com/tags/concepts/) or [my book on concepts](https://www.sandordargo.com/books/cpp_concepts/).*

But what makes a good concept? That's a more complex topic — and one we can't fully cover in a single article.

## The problem of reference-qualified types

While concepts promote readability, Mateusz Pusz pointed out that sometimes we must sacrifice a bit of that clarity to ensure our constraints do what we intend.

Consider this function template:

```cpp
template<Quantity Q>
void foo(Q&& q);
```
Here, `Q` is constrained by the concept `Quantity`. It seems fine — and it's probably how you're used to applying concepts.

But here is the catch: `Q` is deduced from the call of `foo`, so if you pass an *lvalue* of type `MyQuantity`, `Q` will be `MyQuantity&`. But it might also be `const MyQuantity&`, `MyQuantity&&` or even `volatile MyQuantity`. 


That means the `Quantity` concept must accept all of these reference-qualified forms. But if your concept assumes a pure value type, things can go wrong.

Let's take this definition of the `Quantity` concept:

```cpp
template<typename T>
concept Quantity = requires(T t) {
    typename T::unit;  
    // some more checks...
};
``` 

In this case, if `T` is a (`const`) reference, the compilation may fail because `T::unit` is not accessible through a reference-qualified type.

Look at the full example:

```cpp
// https://godbolt.org/z/saPaKEd96
#include <concepts>
#include <iostream>

template<typename T>
concept Quantity = requires(T t) {
    typename T::unit;  
    // some more checks...
};

struct Gram {
    using unit = int;
};

template<Quantity Q>
void foo(Q&& q) {

}

int main() {
    Gram g;
    foo(g); // error: no matching function for call to 'foo(Gram&)'
}
```

## The solution: strip references

To fix this while keeping the `Quantity` concept simple — or in cases where you can't modify it — you can strip reference and cv-qualifiers from `Q` before applying the constraint:

```cpp
template<typename Q>
 requires Quantity<std::remove_cvref_t<Q>>
void foo(Q&& q);
```

Here, `Q` can be any type. Before it's checked against the `Quantity` concept, all `const`/`volatile` and reference qualifiers are removed.

Since concepts apply to the deduced type — qualifiers and all — this step ensures you're checking the actual intended type. It allows your concept to remain focused on what matters: the value type.

The downside is that this approach is more verbose, and we lose the convenient shorthand syntax for constrained templates.

C++26 will bring a solution to this readability downgrade, which we will discuss next week.

## Conclusion

Using `std::remove_cvref_t` with concepts helps enforce constraints on the intended value types and avoids subtle failures due to cv - or reference qualifications. It's a small price to pay for correctness and cleaner concepts — at least until the language evolves to make this pattern more concise. We discuss the evolution next week.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!