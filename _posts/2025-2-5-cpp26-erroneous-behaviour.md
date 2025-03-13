---
layout: post
title: "C++26: erroneous behaviour"
date: 2025-2-5
category: dev
tags: [cpp, cpp26, templates, packindexing]
excerpt_separator: <!--more-->
---
If you pick a random talk at a C++ conference these days, there is a fair chance that the speaker will mention safety at least a couple of times. It's probably fine like that. The committee and the community must think about improving both the safety situation and the reputation of C++.

If you follow what's going on in this space, you are probably aware that people have different perspectives on safety. I think almost everybody finds it important, but they would solve the problem in their own way.

A big source of issues is certain manifestations of undefined behaviour. It affects both the safety and the stability of software. I remember that a few years ago when I was working on some services which had to support a 10x growth, one of the important points was to eliminate undefined behaviour as much as possible. One main point for us was to remove uninitialized variables which often lead to crashing services. 

Thanks to [P2795R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2795r5.html) by Thomas Köppe, uninitialized reads won't be undefined behaviour anymore - starting from C++26. Instead, they will get a new behaviour called "erroneous behaviour".

The great advantage of erroneous behaviour is that it will work just by recompiling existing code. It will diagnose where you forgot to initialize variables. You don't have to systematically go through your code and let's say declare everything as `auto` to make sure that every variable has an initialized value. Which you probably wouldn't do anyway. 

But what is this new behaviour that on C++ Reference is even listed [on the page of undefined behaviour](https://en.cppreference.com/w/cpp/language/ub)? It's well-defined, yet incorrect behaviour that compilers are **recommended** to diagnose. *Is recommended enough?!* Well, with the growing focus on safety, you can rest assured that an implementation that wouldn't diagnose erroneous behaviour would be soon out of the game.

Some compilers can already identify uninitialized reads - what nowadays falls under undefined behaviour. For example, clang and gcc with `-ftrivial-auto-var-init=zero` have already offered default initialization of variables with automatic storage duration. This means that the technique to identify these variables is already there. The only thing that makes this approach not practical is that you will not know which variables you failed to initialize.

Instead of default initialization, with erroneous behaviour, an uninitialized object will be initialized to an implementation-specific value. Reading such a value is a conceptual error that is recommended and encouraged to be diagnosed by the compiler. That might happen through warnings, run-time errors, etc.

```cpp
void foo() {
  int d;  // d has an erroneous value
  bar(d); // that's erroneous behaviour!
}
```

So looking at the above example, ideally `int d;` should be already diagnosed at compile-time as a warning. If it's ignored, at some point, `bar(d);` will have an effect during program execution, but it should be well-defined, unlike undefined behaviour where anything can happen.

> It's worth noting that undefined behaviour and having erroneous values is not possible in constant expressions. In other words, `constexpr` protects from it.

Initializing an object to anything has a cost. What if you really want to avoid it and initialize the object later? Will you be able to still do it without getting the diagnostics? Sure! You just have to be deliberate about that. You cannot just leave values uninitialized by accident, you must mark them with C++26's new attribute, `[[indeterminiate]]`.

```cpp
void foo() {
  int d [[indeterminate]];  // d has an indeterminate value
  bar(d); // that's undefined behaviour!
}
```

We must notice in the example, that `d` doesn't have an erroneous value anymore. Now its value is simply [indeterminate](https://en.cppreference.com/w/cpp/language/attributes/indeterminate). On the other hand, if we later use that variable still without initialization, it's undefined behaviour!

Above, we've only talked about variables with automatic storage duration. That's not the only way to have uninitialized variables. Moreover, probably it's not even the main way, think about dynamic storage duration, think about pointers! Also, if any member is left uninitialized, the parent object's value will be considered either indeterminate or erroneous.


```cpp
struct S {
  S() {}
  int num;
  std::string text;
};

int main() {
  [[indeterminate]] S s1; // indeterminate value
  std::cout << s1.num << '\n' // this is UB as s1.num is indeterminate
  S s2;
  std::cout << s2.num << '\n' // this is still UB, s2.num is an erroneous value
}
```
Not only variables variables but function parameters can also be marked `[[indeterminate]]`.

```cpp
struct S {
  S() {}
  int num;
  std::string text;
};

void foo(S s1 [[indeterminate]], S s2)
{
    bar(s1.num); // undefined behavior
    bar(s2.num); // erroneous behavior
}
```

At the point of writing (January 2025), no compiler provides support for erroneous behaviour.

## Conclusion

C++26 introduces erroneous behaviour in order to give well-defined, but incorrect behaviour for reading uninitialized values. Soon, compilers will be recommended to diagnose every occurrence of reads of uninitialized variables and function parameters.

Also, if something is not initialized at a given moment on purpose, you can mark it with the `[[indeterminate]]` attribute following the don't pay for what you don't need principle.

This new behaviour is a nice step forward in terms of C++'s safety.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)