---
layout: post
title: "Should you use final?"
date: 2025-4-9
category: dev
tags: [testing, tdd, unittesting, final, mocking]
excerpt_separator: <!--more-->
---
Today, let's discuss the not-too-frequently-used `final` specifier. I think there aren’t many good reasons to use it — something the C++ Core Guidelines also suggests. And even when you *could* use it, you probably *shouldn’t*.

> `final` is not technically a keyword, it's an *"identifier with special meaning"*. It can be used as a name for objects or functions, but if used in the right place, it gains special behavior. For readable code, though, don’t use it as an identifier!


Let’s start with a quick recap of the `final` specifier, then move on to a nasty example I recently ran into.

## What is `final`

In C++, `final` is a specifier used to prevent further derivation or overriding. It can be applied to both classes and virtual functions.

### `final` classes

When final is applied to a class, it prevents other classes from inheriting from it.

```cpp
class Base {};

class Derived final : public Base {};  // OK

class AnotherDerived : public Derived {};  // Error! Derived is final

```

### `final` virtual functions

When final is applied to a virtual function, it prevents derived classes from overriding it.


```cpp
class Base {
public:
    virtual void foo() {} 

    // void bar() final {} // error: only virtual member functions can be marked 'final'
};

class Derived : public Base {
public:
    void foo() final {}
};

class AnotherDerived : public Derived {
	// void foo() override {}  // error: foo() is marked final in Derived
};  

```

It's worth noting that you cannot mark a non-virtual function as final.

## When to use it?

Don't use `final` a lot. More than that, use it *less* than you probably would! The Core Guidelines talks about `final` usage in two rules.

[C.139](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c139-use-final-on-classes-sparingly) says that you should use `final` on classes sparingly. The reason? It’s rarely needed for logical reasons and can hurt the extensibility of your class hierarchies.

Some claim that marking classes `final` improves performance, but that's more of an urban legend — or at least only true in specific cases. As always: **measure**.

In the next section, I'll show how a `final` class can be too limiting.

Before that, let's briefly mention the other rule, [C.128](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c128-virtual-functions-should-specify-exactly-one-of-virtual-override-or-final). It says that (virtual) functions should specify exactly one of `virtual`, `override` or `final`. Using only one is explicit and precise:

- `virtual` -> "this is a new `virtual` function".
- `override` -> "this is a non-final overrider"
- `final` -> "this is a final overrider."

Though you should use `final` on functions just as sparignly as on classes.

## Don't use `final` on mocks!

Recently, I was cleaning up the outputs of a test harness. To reduce noise, I decided to wrap a mock class with `NiceMock<T>`.

> If you set no expectations on a mocked call, but it's invoked, the test harness will report "uninteresting calls". Using `NiceMock<T>` suppresses those. If you use `StrictMock<T>`, uninteresting calls will cause test failures. *[Look for more info here](https://google.github.io/googletest/gmock_cook_book.html#NiceStrictNaggy)*.

When I tried to compile, it failed! After some wondering and digging, I discovered that `NiceMock<T>` and `StrictMock<T>` both inherit from `T`. In fact, they use the [CRTP](https://www.sandordargo.com/blog/2019/03/13/the-curiously-recurring-templatep-pattern-CRTP) design pattern. So, if you try to use `NiceMock<T>` on a class `T` that has been marked `final` then the compilation will fail, because `NiceMock` cannot inherit from `T`.

Too bad...

If you have the rights to change the source code of `T`, good for you. Go ahead and consider remove `final` from class declaration. If you cannot do that, you might ask the owners to do so or you simply have to give up on using `NiceMock`.

Nevertheless, if you are using `gmock` and you are writing some mock classes, don't mark then `final`. Don't make life harder for your users or for yourself.

## Conclusion

In this article, we talked about the final specifier. You can use it on classes and member functions to prevent inheritance and overrides, respectively.

While it may seem useful at first glance, it doesn’t offer performance benefits in most cases—and it often just adds unnecessary constraints. That’s why the Core Guidelines recommend using it sparingly.

We saw a concrete example where marking a mock class as `final` prevented its use with` NiceMock<T>` or `StrictMock<T>`, since they inherit from the mock class behind the scenes.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
