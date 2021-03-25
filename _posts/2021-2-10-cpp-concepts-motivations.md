---
layout: post
title: "The concept behind C++ concepts"
date: 2021-2-10
category: dev
tags: [cpp, cpp20, concepts]
excerpt_separator: <!--more-->
---
The idea of concepts is one of the major new features added to C++20. Concepts are an extension for templates. They can be used to perform compile-time validation of template arguments through boolean predicates. They can also be used to perform function dispatch based on properties of types.
<!--more-->

With concepts, you can *require* both syntactic and semantic conditions. In terms of syntactic requirements, imagine that you can impose the existence of certain functions in the API of any class. For example, you can create a concept `Car` that requires the existence of an `accelerate` function:

```cpp
#include <concepts>

template <typename C>
concept Car = requires (C car) {
  car.accelerate()
};
```

Don't worry about the syntax, we'll get there next week.

Semantic requirements are more related to mathematical axioms, for example, you can think about associativy or commutativity:

```
a + b == b + a // commutativity
(a + b) + c == a + (b + c) // associativity
```
There are concepts in the standard library expressing semantic requirements. Take for example [`std::equality_comparable`](https://en.cppreference.com/w/cpp/concepts/equality_comparable).

It requires that 
- the two equality comparison between the passed in types are commutative,
- `==` is symmetric, transitive and reflexive, 
- and `equality_comparable_with<T, U>` is modeled only if, given any lvalue t of type `const std::remove_reference_t<T>` and any lvalue u of type `const std::remove_reference_t<U>,` and let C be `std::common_reference_t<const std::remove_reference_t<T>&, const std::remove_reference_t<U>&>`, `bool(t == u) == bool(C(t) == C(u))`.

Though this latter one is probably a bit more difficult to decipher. Anyway, if you are looking for a thorough article dedicated to semantic requirements, read [this one by Andrzej Krzemieński](https://akrzemi1.wordpress.com/2020/10/26/semantic-requirements-in-concepts/).

## The motivation behind concepts

We have briefly seen from a very high-level what we can express with concepts. But why do we need them in the first place?

For the sake of example, let's say you want to write a function that adds up two numbers. You want to accept both integral and floating-point numbers. What are you going to do?

You could accept `double`s, maybe even `long double`s and return a value of the same type. 

```cpp
#include <iostream>

long double add(long double a, long double b) {
    return a+b;
}

int main() {
  int a{42};
  int b{66};
  std::cout << add(a, b) << '\n';
}
```

The problem is that when you call `add()` with two `int`s, they will be cast to `long double`. You might want a smaller memory footprint, or maybe you'd like to take into account the maximum or minimum limits of a type. And anyway, it's not the best idea to rely on implicit conversions. 

Implicit conversions might allow code to compile that was not at all in your intentions. It's not bad by definition, but implicit conversions should be intentional and not accidental.

In this case, I don't think that an intentional cast is justified.

Defining overloads for the different types is another way to take, but it is definitely tedious.

```cpp
#include <iostream>

long double add(long double a, long double b) {
  return a+b;
}

int add(int a, int b) {
  return a+b;
}

int main() {
  int a{42};
  int b{66};
  std::cout << add(a, b) << '\n';
}
```
Imagine that you want to do this for [all the different numeric types](https://en.cppreference.com/w/cpp/language/types). Should we also do it for combinations of `long double`s and `short`s? Eh... Thanks, but no thanks.

Another option is to define a template!

```cpp
#include <iostream>

template <typename T>
T add(T a, T b) {
    return a+b;
}

int main() {
  int a{42};
  int b{66};
  std::cout << add(a, b) << '\n';
  long double x{42.42L};
  long double y{66.6L};
  std::cout << add(x, y) << '\n';
  
}
```
If you have a look at [CPP Insights](https://cppinsights.io/lnk?code=I2luY2x1ZGUgPGlvc3RyZWFtPgoKdGVtcGxhdGUgPHR5cGVuYW1lIFQ+ClQgYWRkKFQgYSwgVCBiKSB7CiAgICByZXR1cm4gYStiOwp9CgppbnQgbWFpbigpIHsKICBpbnQgYXs0Mn07CiAgaW50IGJ7NjZ9OwogIHN0ZDo6Y291dCA8PCBhZGQoYSwgYikgPDwgJ1xuJzsKICBsb25nIGRvdWJsZSB4ezQyLjQyTH07CiAgbG9uZyBkb3VibGUgeXs2Ni42TH07CiAgc3RkOjpjb3V0IDw8IGFkZCh4LCB5KSA8PCAnXG4nOwogIAp9Cg==&insightsOptions=cpp17&std=cpp17&rev=1.0) you will see that code was generated both for an `int` and for a `long double` overload. There is no static cast taking place at any point.

Are we good yet?

Unfortunately, no.

What happens if you try to call `add(true, false)`? You'll get a `1` as `true` is promoted to an integer, summed up with `false` promoted to an integer and then they will be turned back (by `static_cast`) into a boolean.

What if you add up two string? They will be concatenated. But is that really what you want? Maybe you don't want that to be a valid operation and you prefer a compilation failure.

So you might have to forbid that [template specialization](https://dev.to/pgradot/forbid-a-particular-specialization-of-a-template-4348). And for how many types do you want to do the same?

What if you could simply say that you only want to add up integral or floating-point types. In brief, rational numbers. And here come `concepts` into the picture.

With concepts, you can easily express such requirements on template parameters.

You can precise requirements on
- the validity of expressions (that certain functions should exist in the class' API)
- the return types of certain functions
- the existence of inner types, of template specializations
- the type-traits of the accepted types

How? That's what we are going to explore in this series on C++ concepts.

## What's next?

During the next couple of weeks we are going to discuss:

- [the 4 ways to use a concept](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations)
- [how to use concepts with functions](https://www.sandordargo.com/blog/2021/02/17/cpp-concepts-4-ways-to-use-them)
- [how to use concepts with classes](https://www.sandordargo.com/blog/2021/02/24/cpp-concepts-with-classes)
- [what kind of predefined concepts the standard library introduced](https://www.sandordargo.com/blog/2021/03/03/cpp-concepts-in-standard-library)
- how to write our own concepts ([part I](https://www.sandordargo.com/blog/2021/03/10/write-your-own-cpp-concepts-part-i) and [part II](https://www.sandordargo.com/blog/2021/03/17/write-your-own-cpp-concepts-part-ii))

Stay tuned!