---
layout: post
title: "C++23: static operator() and static operator[]"
date: 2023-7-26
category: dev
tags: [cpp, cpp23, static, calloperator]
excerpt_separator: <!--more-->
---
In this article, we are going to review two new features of C++23. Now the language allows the call operator (`operator()`) and the subscription operator (`operator[]`) to be static. Let's jump into the details.

## `static operator()`

As we saw earlier in [our big C++ algorithms tutorial](https://www.sandordargo.com/blog/2019/01/30/stl-algos-intro), function objects are extensively used in the standard library to customise the behaviour of several functions. With the introduction of ranges in C++20 (and of earlier non-standard libraries), the usage of function objects became even more widespread.

Many function objects are extremely simple. Let's take an implementation of `isEven.`

```cpp
auto isEven = [](int i) {return i % 2 == 0;};
```

When the compiler transforms this lambda into an actual function object, [it'll look somehow like this](https://cppinsights.io/lnk?code=YXV0byBpc0V2ZW4gPSBbXShpbnQgaSkge3JldHVybiBpICUgMiA9PSAwO307Cg==&insightsOptions=cpp17&std=cpp17&rev=1.0):

```cpp
class __lambda_5_19 {
public: 
    inline /*constexpr */ bool operator()(int i) const {
      return (i % 2) == 0;
    }

    using retType_5_19 = bool (*)(int);

    inline constexpr operator retType_5_19 () const noexcept {
      return __invoke;
    };

private: 
    static inline /*constexpr */ bool __invoke(int i) {
      return __lambda_5_19{}.operator()(i);
    }
};
```

It's worth noting that even though this lambda has no capture, it has no state at all, `operator()` is not static. What does it mean in practice? As it's explained in the [Motivation section of P1169R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1169r4.html#motivation), the consequence is that if `operator()` cannot be inlined, the implicit object parameter (`this`) must be passed around which manifests itself as an extra register call in the generated assembly code.

This is a violation of the fundamental principle behind C++: *you don't pay for what you don't need*. Here you do.

Thanks to [this proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1169r4.html), the above problem is going to be solved by allowing `operator()` to be a `static` member function. This new feature will come in handy both for hand-written function objects and for lambdas.

In the case of a lambda, you have to explicitly declare it `static` if you want that the generated object has a `static` call operator:

```cpp
auto isEven = [](int i) static {return i % 2 == 0;};
```

At the same time, you only declare a lambda `static` if it has no capture, if it has a capture then the compiler reminds you that you violated the rules:

```cpp
// ERROR: 'static' lambda specifier with lambda capture
auto isDivisableBy = [operand](int i) static {return i % operand == 0;};
```

## `static operator[]`

[P2589R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2589r0.pdf) makes `operator[]` consistent with `operator()` with regards of `static`ness.

The need for this consistency was expressed in some places already:
- In [the already discussed proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1169r4.html) to allow `operator()` be `static`
- In [P2128R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2128r6.pdf) allowing multidimensional subscription operators
- And in [EWG88](https://cplusplus.github.io/EWG/ewg-active.html#88) where Gabriel Dos Reis already brought up in 2014 that `operator()` and `operator[]` should be handled the same way and both should be allowed to be static.

Now this becomes valid code:

```cpp
class MyClass {
 public:
  static int operator[](size_t index);
  // ...
};
```

## Conclusion

In this post, we reviewed two papers that got accepted for C++23. The language now allows both `operator()` and `operator[]` to be `static` in the principle of not having to pay for something you don't use. In this case, that something is the `this` pointer. Probably `static` call operators will be more frequently used than the `static` subscription operator, but keeping the language consistent is a good idea!

Both these changes are already available on gcc (13) and clang(16) at the moment of publication in July 2023.

In the next post, we'll stay with the subscription operator and discuss how and why it's going to support multidimensional arrays.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!