---
layout: post
title: "C++ Concepts and the Core Guidelines"
date: 2021-10-13
category: dev
tags: [cpp, cpp20, concepts, guidelines]
excerpt_separator: <!--more-->
---
Let's get back to C++ concepts and have a look at the rules and best practices that the [Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-templates) propose.
<!--more-->

Having read them through, I found that they are incomplete (no surprise, concepts are new), yet outdated.

How is that possible?

They were written for the Concepts TS, not for the standardized version. So as you'll see, here and there it follows a syntax that is not compilable.

I'm sure it will take some years to find all the best practices and fill the guidelines. After all, they should not change frequently. 

Let's see what they offer today.

## How to use concepts

Let's start with some rules on how to use concepts.

### [T.10: Specify concepts for all template arguments](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t10-specify-concepts-for-all-template-arguments)

This rule recommends something we already discussed. You shouldn't use bare `typename T`s in the template parameter lists. 

`T` is obviously a bad name as it doesn't bring any additional information apart from that it's a template type and you should strive for better names, but the rule mainly suggests not to use these template types without constraining them.

Instead of 

```cpp
template <typename Num>
auto add(Num a, Num b) {
  return a+b;
}
```

we should use

```cpp
template <typename Num>
requires Number<Num>
auto add(Num a, Num b) {
  return a+b;
}
```

or even better:

```cpp
template <Number Num>
auto add(Num a, Num b) {
  return a+b;
}
```

### [Whenever possible use standard concepts
](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t11-whenever-possible-use-standard-concepts)

This rule reminds me of something we discussed in [Loops are bad, algorithms are good! Aren't they?](https://www.sandordargo.com/blog/2020/05/13/loops-vs-algorithms).

Whenever the standard library offers you what you need, take it and use it. Reinventing the wheel is dangerous and useless.

Whatever you find in the standard library is better tested, often more expressive and in the vast majority of cases, it provides better performance compared to what you'd write.

It's the same idea for concepts as for algorithms. Why would it be any different?

### [T.12: Prefer concept names over auto for local variables](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t12-prefer-concept-names-over-auto-for-local-variables)

This is very similar to T10 which advocates for no bare template parameters, no template parameters without a constraint on them.

In this case, it's not about the `typename` keyword, but about `auto`. If we consider `typename` an unconstrained template parameter, we can also consider `auto` as an unconstrained type. In another word, `auto` is the weakest concept.

Instead of using `auto n = calculate();` we use write `Number auto n = calculate();`. In this case, it's worth noting that the rule is outdated as it's still using Concepts TS in which one could use a concept not with but instead of `auto` which is a bit misleading as it's difficult to know whether what you see is a type or a concept.

### [T.13: Prefer the shorthand notation for simple, single-type argument concepts](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t13-prefer-the-shorthand-notation-for-simple-single-type-argument-concepts)

As we saw earlier both [for functions](https://www.sandordargo.com/blog/2021/02/17/cpp-concepts-4-ways-to-use-them) and [classes](https://www.sandordargo.com/blog/2021/02/24/cpp-concepts-with-classes), there are several different ways to use concepts, to introduce constraints on your template parameters.

One way was to use the requires clause:

```cpp
template <typename T>
requires Number<T>
auto add(T a, T b) {
  return a+b;
}
```

It's quite readable, but it's more verbose than necessary.

This rule advocates for using the shorthand notation instead, to use what we call today the constrained template parameters:

```cpp
template <Number T>
auto add(T a, T b) {
  return a+b;
}
```

Or, when you have the possibility go even further and use the abbreviated function template form of

```cpp
auto add(Number auto a, Number auto b) {
  return a+b;
}
```

## How to define concepts

Let's continue with some rules on how to define concepts. With time, this can be the most important section of the core guidelines on concepts. Writing concepts is easy, writing good concepts that are meaningful and carry some semantic meaning is difficult.

### [T.20: Avoid “concepts” without meaningful semantics](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t20-avoid-concepts-without-meaningful-semantics)

A good concept should do more than enforcing the existence of certain functions, it should do more than requiring a certain API.

A good concept will also communicate semantics.

For example, it's more than enforcing having the `operator+` defined, it's communicating that the type modelling a concept is a *number*.

### [T.21: Require a complete set of operations for a concept
](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t21-require-a-complete-set-of-operations-for-a-concept)

This next rule is closely related to the previous one. If you want to have meaningful semantics it's hardly useful to model a number only supporting addition.

You need to put in a bit more work and model all the necessary operations, all the necessary comparisons. In general, all the functions that make a type modelling a useable concept.

### [T.22: Specify axioms for concepts](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t22-specify-axioms-for-concepts)

First, we have to understand what an axiom is.

An axiom or assumption is a statement that is taken to be true, it serves as a premise or starting point for further reasoning and arguments. We take an axiom valid without any evidence.

If you want to express [axioms](https://akrzemi1.wordpress.com/2012/01/11/concept-axioms-what-for/) in code, they would be Boolean expressions. C++20 doesn't support axioms, but it might change in the future.

For the time being, you can express axioms as comments:

```cpp
template<typename T>
    // The operators +, -, *, and / for a number are assumed to follow the usual mathematical rules
    // axiom(T a, T b) { a + b == b + a; a - a == 0; a * (b + c) == a * b + a * c; /*...*/ }
    concept Number = requires(T a, T b) {
        {a + b} -> std::convertible_to<T>;   // the result of a + b is convertible to T
        {a - b} -> std::convertible_to<T>;
        {a * b} -> std::convertible_to<T>;
        {a / b} -> std::convertible_to<T>;
    } 
```

### [T.23: Differentiate a refined concept from its more general case by adding new use patterns](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t23-differentiate-a-refined-concept-from-its-more-general-case-by-adding-new-use-patterns)

If you have two concepts where one is the refined version of the other, use the general one in the refined pattern and add some additional requirements.

Let's say we have this concept:

```cpp
template<typename I>
concept bool Input_iter = requires(I iter) { ++iter; };
```

In order to define `Fwd_iter` correctly, do not write it from scratch: 

```cpp
template<typename I>
concept bool Fwd_iter = requires(I iter) { 
  ++iter;
  iter++; 
}
```

Instead use, the more generic version and add the extra rules:

```cpp
template<typename I>
concept bool Fwd_iter = Input_iter<I> && requires(I iter) { iter++; }
```
This helps both the reader to understand that they have to deal with a more refined version and the compiler can also find the good concept at overload resolution time.

### [T.24: Use tag classes or traits to differentiate concepts that differ only in semantics](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t24-use-tag-classes-or-traits-to-differentiate-concepts-that-differ-only-in-semantics)

As we discussed earlier a good concept does not only express syntactic requirements, but it's also about semantics.

What if the syntactical requirements are the same for two concepts, but they have different semantics?

In order to disambiguate them, we have to add some syntactical differences.

A way of doing this is to write a tag class or a trait (either a standard or a user-defined one) and make a requirement on it:

```cpp
template<typename I>    // iterator providing random access
bool RA_iter = ...;

template<typename I>    // iterator providing random access to contiguous data
bool Contiguous_iter =
    RA_iter<I> && is_contiguous<I>::value;  // using is_contiguous trait

```

### [T.25: Avoid complementary constraints](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t25-avoid-complementary-constraints)

It's not a good idea to use complementary constraints meaning that in one function overload you make some requirements and in the other, you require its negation:

```cpp
template<typename T>
    requires !C<T>    // bad
void f();

template<typename T>
    requires C<T>
void f();
```

Instead of the negated one, just use a general template without negated constraints.

```cpp
template<typename T>   // general template
    void f();

template<typename T>   // specialization by concept
    requires C<T>
void f();
```

Why is it a bad idea to use the negated form? As we saw earlier in [C++ Concepts and logical operators](https://www.sandordargo.com/blog/2021/05/05/cpp-concepts-and-logical-operators), negations can be more difficult to handle due to subsumption rules. Besides, it's much less readable to achieve the same effect, not to mention maintainability.

Just keep it stupid simple.

### [T.26: Prefer to define concepts in terms of use-patterns rather than simple syntax](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t26-prefer-to-define-concepts-in-terms-of-use-patterns-rather-than-simple-syntax)

When I read this title first, I didn't really understand. But the Core guidelines provide a great example.

You might have some helper concepts or type traits such as `has_equal<T>` and `has_not_equal<T>`. They would let you (re)create `EqualityComparable` like this

```cpp
template<typename T> concept EqualityComparable = has_equal<T> && has_not_equal<T>;
``` 

It's not unreadable, but it's better if you use the requires body to express your constraints by writing how you want to use the types modeling the concept:

```cpp
template<typename T> concept EqualityComparable = requires(T a, T b) {
    { a == b } -> std::same_as<bool>;
    { a != b } -> std::same_as<bool>;
};
```

Remember, humans are great at following patterns. Use that as a feature!

## Additional rules

As we mentioned, there is plenty of space left in the guidelines for additional rules on concepts. 

At the moment of writing, I found one among ["Template Interfaces"](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#template-interfaces). If you found more, let me know so I can include them.

### [T.41: Require only essential properties in a template’s concepts](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t41-require-only-essential-properties-in-a-templates-concepts)

You might remember from unit testing, that you shouldn't assert every detail, every internal of a class as it makes your tests brittle. Unit tests should assert just to the right level of detail.

The idea is similar to concepts. A concept should not require too many details and definitely not things that are unrelated.

For example, a concept modelling sortable types, should not require I/O operations at all. The ability of a project to print itself has nothing to do with sortability. If that is required, it should be modelled in a different concept, such as `Printable` or `Streamable`.

A good API is strict enough, but loose at the same time and it's definitely stable. This rule helps to achieve the desired level of looseness and stability.

## Conclusion

Today, we discussed the already existing best practices and recommendations on concepts in the [Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-templates
).

There are already a decent number of rules, even though they are not up to date with C++20, they are still based on the Concepts TS. Nevertheless, they serve as a good basis for further discussion as our experience of writing concepts grows.

Let me know about your best practices.

**If you want to learn more details about C++ concepts, [check out my book on Leanpub](https://leanpub.com/cppconcepts)!**

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!