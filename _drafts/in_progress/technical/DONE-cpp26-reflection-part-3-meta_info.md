---
layout: post
title: "C++26 Reflection: All You Need To Know About std::meta::info"
date: 2025-X-X
category: dev
tags: [cpp, cpp26, reflection, syntax]
excerpt_separator: <!--more-->
---
During the previous parts of our C++ Reflection journey, we explored [the motivations for C++ Reflection]() and [the basics of the syntax](). We concluded by introducing the `std::meta::info` type — the common result of applying the reflection operator and the fundamental building block for splicing generated code.

Let's now take a closer look at what `std::meta::info` really is and how it works.

## `std::meta::info` - One Type to Reflect Them All

As a reminder, you can find `std::meta::info` in the `<meta>` or `<experimental/meta>` header, depending on your toolchain.

While there was considerable interest in designing multiple reflection types — each tailored to specific language elements like types, variables, and functions — the committee ultimately chose a single opaque type. This approach helps decouple the type system from the precise structure of the language today, making the feature more robust to future changes.

As explained in [P2996](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r12.html#why-a-single-opaque-reflection-type), choosing a unified type avoids locking in today's syntax or semantics. For instance, it avoids complications that would have arisen from changes like C++11’s decision to treat references as first-class types.

Using a single reflection type also makes it straightforward to store collections of heterogeneous reflections—types, variables, functions, and more — in a uniform way.

## A Surprisingly Simple Definition

Despite its power, `std::meta::info` is deceptively simple in definition:

```cpp
namespace std {
  namespace meta {
    using info = decltype(^^::);
  }
}
```

This represents the type returned when reflecting on the global namespace.

While basic in appearance, `std::meta::info` can represent:
- any type or type alias
- any free standing or member function
- any variable, including static and non-static member variables and structured bindings
- any enumerator
- any template
- any namespace (including the global namespace - as we can expect from the above definition) or namespace alias
- any object that is a permitted result of a constant expression 
- any value with structural type that is a permitted result of a constant expression 
- the null reflection for default constructed instances of `std::meta::info`

> Structural types (introduced in C++20) are literal types that have only public, non-mutable data members and no virtual functions. These can be used as non-type template parameters.

Currently, one limitation is that reflection over expressions (e.g., walking through a function body’s expressions) is not supported—but this is being considered for future proposals.

## A Scalar Type with Identity Semantics

`std::meta::info` is a scalar type. You can compare it for equality and inequality, but not ordering. Two `info` instances are equal if and only if they reflect the exact same entity.

Here are some examples, directly taken from [P2996](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r12.html#comparing-reflections):

```cpp
static_assert(^^int == ^^int);
static_assert(^^int != ^^const int);
static_assert(^^int != ^^int &);

using Alias = int;
static_assert(^^int != ^^Alias);
static_assert(^^int == dealias(^^Alias));

namespace AliasNS = ::std;
static_assert(^^::std != ^^AliasNS);
static_assert(^^:: == parent_of(^^::std));
```
Here, references and `const` qualifiers make reflections distinct. Even a type alias has a different reflection from the underlying type, though you can normalize them with metafunctions like `dealias`.

One interesting nuance: the reflection of `int` is distinct from the reflection of a member variable of type `int`.

```cpp
int x;
struct S { int y; };
static_assert(^^x == ^^x);
static_assert(^^x == ^^S::y);
```

We can also reflect on values. And even value reflections show this behavior. Two identical constants will yield distinct info objects:

```cpp
constexpr int i = 42, j = 42;

static_assert(^^i != ^^j);  // 'i' and 'j' are different entities.
static_assert(constant_of(^^i) == constant_of(^^j));    // Two equivalent values.
static_assert(^^i != std::meta::reflect_object(i))      // A variable is distinct from the
                                                        // object it designates.
static_assert(^^i != std::meta::reflect_constant(42));  // A reflection of an object
                                                        // is not the same as its value.
```

## No Member Functions—Only Metafunctions

You might have noticed that we never invoked any member functions on `std::meta::info` directly. That's because you don't work with reflection data via object-oriented interfaces. Instead, everything is driven by `consteval` metafunctions.

Reflection in C++26 is highly composable and functional in nature. Rather than having member functions like `get_type()` or `get_name()`, you pass info instances to metafunctions like `meta::name_of`, `meta::type_of`, or `meta::members`.

This design also means you can easily write your own metafunctions that operate on info — something we'll cover in more detail next week.

## Conclusion

`std::meta::info` is the cornerstone of C++26 reflection. It's a single, opaque, and scalar type that represents any reflectable language entity — from types to values to namespaces. Its design emphasizes simplicity, extensibility, and future-proofing.

While it might feel unusual at first that info exposes no direct interface, this is by design. The real power lies in `consteval` metafunctions, which offer clean and composable ways to extract information, generate code, and build powerful tools like serializers, validators, and command-line interfaces.

In future articles, we'll dive into how to use these info instances with the standard-provided metafunctions — and how to write your own.

Reflection isn't magic—but it can feel like it once you know how to harness it.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!