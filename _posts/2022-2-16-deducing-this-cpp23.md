---
layout: post
title: "C++23: Deducing this"
date: 2022-2-16
category: dev
tags: [cpp, cpp23, typededuction, overloads]
excerpt_separator: <!--more-->
---
A few weeks ago, I participated in the first [AFNOR](https://www.afnor.org/en/) meeting of my life. AFNOR is the French standardization organization, part of the ISO group and I've recently joined the group responsible for the standardization of C++. 
<!--more-->

Before going there, I asked around at my company, what would be my peers interested in. What features would they really like to see shipped with C++23? Maybe I can find a way to offer my help and work on those features.

One of the inputs I received was about [*deducing `this`*](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0847r7.html). I didn't know it so I had a look at the proposal.

In this article, I'd like to share in a nutshell what I learnt about [this proposal](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0847r7.html).

And the good news is that it's been already accepted, it's going to be part of C++23.

## What is this `this` about?

So what is the proposal of [Gašper Ažman](https://twitter.com/atomgalaxy), [Sy Brand](https://twitter.com/TartanLlama), [Ben Deane](https://twitter.com/ben_deane) and [Barry Revzin](https://twitter.com/BarryRevzin) about?

They propose *"a new way for specifying or deducing the value category of the expression that a member-function is invoked on*". In other words, they want to have *"a way to tell from within a member function whether the expression it’s invoked on is an lvalue or an rvalue; whether it is `const` or `volatile`; and the expression’s type"*.

## Why would that be useful?

I completely understand if the above abstract leaves you a bit puzzled, though after rereading it a few times I found it very precise. Let's see a couple of examples that motivated this proposal. 

[As explained in the proposal](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0847r7.html#motivation), since C++03, member functions can have *cv-qualifiers* and they can also be overloaded based on these qualifications. It's worth noting that it's far more common to overload a member function based on the `const` qualifier than based on the `volatile`.

Most commonly the `const` and non-`const` overloads do the very same thing, *"the only difference is in the types being accessed and used"*.

Since C++11, the number of possible overloads doubled as [we can overload member functions based on reference qualifiers](https://www.sandordargo.com/blog/2018/11/25/override-r-and-l0-values#use--or--for-function-overloading).

This means that for a member function `Foo::bar`, we can have all these overloads:

```cpp
void Foo::bar() & { /* ... */ }
void Foo::bar() && { /* ... */ }
void Foo::bar() const & { /* ... */ }
void Foo::bar() const && { /* ... */ }
```

Still, all the implementations would be the same.

How to deal with that?

We either write the same logic four times or three functions delegate to the fourth or maybe all of them would delegate to a `private` (`static`) helper.

None of them is very effective.

The proposal would simplify this situation.

## How would the new syntax look like?

The authors of the proposal considered [four different syntaxes](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0847r7.html#was-this-syntax-considered), and in the end, they come up with this one:

```cpp
struct X {
    void foo(this X const& self, int i);

    template <typename Self>
    void bar(this Self&& self);
};
```

*"A non-`static` member function can be declared to take as its first parameter an explicit object parameter, denoted with the prefixed keyword `this`."* It can be deduced following normal function template deduction rules.

A function with an explicit object parameter cannot be `static`, `virtual` and they cannot have `cv`- or `ref`-qualifiers.

Any calls to such members will deduce and interpret the object arguments as the `this` annotated parameter and handle the subsequent arguments as the coming parameters. In other words, you don't have to pass explicitly anything as `this`.

For the detailed rules, name lookups and overload resolutions, I'd recommend you to [read the proposal ](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0847r7.html#name-lookup-candidate-functions). Still, I would like to mention how different `cv`/`ref` overloads with implicit object types can be made explicit.

```cpp
struct X_implicit {
  void foo() &;

  void foo() const&;

  void bar() &&;
};

struct X_explicit {
  void foo(this X&);

  void foo(this X const&);

  void bar(this X&&);
};
```

Of course, for the inexperienced reader, `X_explicit` offers a much more understandable semantics on what function should be invoked based on the type of `X` at the moment of the call.

## How (deducing) `this` will be useful for us?

The design of a programming language is never supposed to be *l'art pour l'art*. A new feature, a new syntax should always bring clear benefits to the community. Let's see a couple of real-world examples of how deducing `this` will be useful for us.

I'll show you a couple of examples, for the full list please [refer to the proposal](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0847r7.html#real-world-examples).


## Deduplicating overloads

At the beginning of this article, when I wrote about the motivations of this proposal, I mentioned that sometimes we have to implement different overloads based on *cv*- or *ref*-qualifiers and very often we have to provide the very same implementations multiple times.

By using the explicit object parameter, we can get rid of the code duplication as the type of the object will be deduced.

```cpp

template <typename T>
class OptionalNotDeducingThis {
  // ...
  constexpr T* operator->() {
    return addressof(this->m_value);
  }

  constexpr T const*
  operator->() const {
    return addressof(this->m_value);
  }
  // ...
};

template <typename T>
class OptionalDeducingThis {
  // ...
  template <typename Self>
  constexpr auto operator->(this Self&& self) {
    return addressof(self.m_value);
  }
  // ...
};

```

## CRTP simplified

[The Curiously Recurring Template Pattern (CRTP)](https://www.sandordargo.com/blog/2019/03/13/the-curiously-recurring-templatep-pattern-CRTP) is among the most popular design patterns of modern C++. It's often talked about on different blogs, conferences and used in many libraries nowadays.

It implements polymorphism without the cost of virtual tables by adding functionality to a derived class through the base. The derived class is passed to the base class as a template argument.

With the proposal of deducing `this`, we can use standard inheritance as the explicit objects already deduce the type derived objects.

```cpp
template <typename Derived>
struct AddPostfixIncrementWithCRTP {
    Derived operator++(int) {
        auto& self = static_cast<Derived&>(*this);

        Derived tmp(self);
        ++self;
        return tmp;
    }
};

struct AType : AddPostfixIncrementWithCRTP<AType> {
    AType& operator++() { /* ... */ }
};


struct AddPostfixIncrementWithDeducingThis {
    template <typename Self>
    auto operator++(this Self&& self, int) {
        auto tmp = self;
        ++self;
        return tmp;
    }
};


struct AnotherType : AddPostfixIncrementWithDeducingThis {
    AnotherType& operator++() { /* ... */ }
};

```

## Recursive Lambdas

I wrote about recursive lambda functions and the Y-combinator in my [Trip Report of CPPP 2021](https://www.sandordargo.com/blog/2021/12/15/trip-report-cppp2021#three-catchy-ideas). The class templates used as helpers are far from being simple, but they let you write lambdas that can refer to themselves:

```cpp
#include <functional>

template<class Fun>
class y_combinator_result {
  Fun fun_;
public:
  template<class T>
  explicit y_combinator_result(T&& fun):
    fun_(std::forward<T>(fun)) {}

  template<class ...Args>
  decltype(auto) operator()(Args &&...args) {
    return fun_(std::ref(*this),
                std::forward<Args>(args)...);
  }
};

template<class Fun>
decltype(auto) y_combinator(Fun &&fun) {
  return y_combinator_result<std::decay_t<Fun>>(std::forward<Fun>(fun));
}

auto gcd = y_combinator([](auto gcd, int a, int b) -> int {
  return b == 0 ? a : gcd(b, a % b);
});
std::cout << gcd(20, 30) << std::endl;
```

By using the explicit object parameter, referring to the self is not a problem anymore. If the proposal of deducing this will be accepted, writing recursive lambdas will be greatly simplified:

```cpp
auto gcd = [](this auto self, int a, int b) -> int {
    return b == 0 ? a : self(b, a % b);
}
std::cout << gcd(20, 30) << std::endl;
```

## Conclusion

In this example, we saw one of the most popular and most-waited proposed features of C++23, [deducing `this`](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0847r7.html). In the next version of C++, we will be able to take an explicit object type parameter in member functions. With the help of it, we will be able *"tell from within a member function whether the expression it’s invoked on is an lvalue or an rvalue; whether it is `const` or `volatile`; and the expression’s type"*

As we saw, this addition will give us tools to greatly simplify our code when we have multiple overloads for the same member functions, not to mention the CRTP patterns or recursive lambda functions.

What is the C++23 feature you are waiting for the most?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!