---
layout: post
title: "C++26: Disallow Binding a Returned Reference to a Temporary"
date: 2025-6-11
category: dev
tags: [cpp, cpp26, safety, temporaries]
excerpt_separator: <!--more-->
---
In short, thanks to [P2748R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2748r5.html) by Brian Bi, it's no longer possible to return a reference that binds to a temporary expression, and that's just lovely.

## What exactly is changing?

Often, a proposal modifies a lot of wording, and it's not practical to start by reading through it all. But in this case, the changes are small and easy to digest, so let’s dive in.

This line is being **removed**:

> _"The lifetime of a temporary bound to the returned value in a function return statement (8.7.4) is not extended; the temporary is destroyed at the end of the full-expression in the return statement."_

And this line is being **added** (excluding the examples we'll discuss in a moment):

> _"In a function whose return type is a reference, other than an invented function for `std::is_convertible` ([meta.rel]), a return statement that binds the returned reference to a temporary expression ([class.temporary]) is ill-formed."_

There are two interesting shifts here:
- The language becomes more specific. It doesn't merely mention "temporaries" but refers directly to **references binding to temporary expressions**, with certain exceptions.
- The effect isn't just that lifetime extension doesn't happen, it's now a **compilation error**. The code is ill-formed.


## Some examples

Here are examples from the proposal that will also appear in the standard:

```cpp
auto&& f1() {
    return 42;  // ill-formed
}
```

The return statement in `f1` is ill-formed because it tries to return a universal reference bound to a temporary `int`. That’s no longer allowed.

```cpp
const double& f2() {
    static int x = 42;
    return x;   // ill-formed
}
```

At first glance, this might look valid - `f2` returns a reference to a static variable, right? Not quite. The static variable `x` is an `int`, but the function returns a `const double&`. So `x` gets converted to a temporary `double`, and that's what gets returned by reference. Hence, the code is ill-formed.

```cpp
auto&& id(auto&& r) {
    return static_cast<decltype(r)&&>(r);
}

auto&& f3() {
    return id(42);  // OK, but probably a bug
}
```

The `id` function is a simple perfect-forwarding helper. Technically, `f3` is valid — it's returning an rvalue reference to a temporary — but the temporary is already destroyed by the end of `id(42)`'s full expression. So while this isn't ill-formed, it likely results in a dangling reference. A silent bug waiting to happen.

## What about `std::is_convertible`?

`std::is_convertible` is a trait used to detect implicit convertibility. For example, an `int` can be converted to a `const double&`, and the author of this proposal doesn't to interfere with such compile-time checks.

Several approaches were considered. Ultimately, the most viable way was to add a special exception for `std::is_convertible`, which you can see in the updated wording.

## Conclusion

This change in C++26 tightens the rules around returning references to temporaries — and that's a good thing. It turns dangerous, bug-prone code into immediate compilation errors, making code safer and easier to reason about. It also reflects a broader trend in modern C++: giving programmers stronger guarantees and better diagnostics, even at the cost of breaking some old patterns.

You might see some legacy code break because of this change, but that's a small price to pay for eliminating a whole class of subtle lifetime bugs. And thanks to the carefully crafted exception for `std::is_convertible`, existing metaprogramming techniques remain safe and robust.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!