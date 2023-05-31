---
layout: post
title: "C++23: two additional noexcept functions"
date: 2023-5-31
category: dev
tags: [cpp, cpp23, noexcept, exceptions]
excerpt_separator: <!--more-->
---
If my math is correct, there are 125 changes / fixes / new features in C++23 and we are progressively covering them on this blog. I try to go from topic to topic. There are some topics with many smaller changes, such as [`constexpr`](https://www.sandordargo.com/blog/2023/05/24/cpp23-constexpr), there are some significant topics where even one topic must be in its own post such as [the stacktrace library](https://www.sandordargo.com/blog/2022/09/21/cpp23-stacktrace-library), and there are also some shorter posts with few and quite small changes on a given topic. Today we are going to discuss `noexcept` related changes.

> If you haven't read it, I'd recommend reading my article on `noexcept` and its effects [on binary size](https://www.sandordargo.com/blog/2023/03/29/binary-size-and-exceptions)

We can definitely see two trends in the proposals accepted for the latest standards. They try to make more and more functions:
- [`constexpr`](https://www.sandordargo.com/blog/2023/05/24/cpp23-constexpr), and
- `noexcept`

The two main changes presented today fit into this trend. But let's start with discussing a bit of `noexcept` policies in the standard.

## `noexcept`, but with conditions

The current policies on whether something in the standard can be `noexcept` or not is defined by [P1656R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p1656r2.html). Let me summarize the gist of it here.

As we got used to it, a destructor should never throw and therefore even if you don't mark it `noexcept` they implicitly are!

There might be standard library functions marked unconditionally `noexcept` if the committee fully agrees that the given function cannot throw.

The standard specifies a couple of special member functions and library functions that can be marked conditionally `noexcept`, based on the underlying types and the types these functions operate on. Nothing else can be marked `noexcept`, except for library functions that are designed for compatibility with C. Those can be unconditionally `noexcept`.

> Most often, we mark functions either `noexcept` or not. So we often mark our functions unconditionally `noexcept`. But `noexcept` can take a compile-time computable condition, such as `std::is_nothrow_move_constructible_v<T> && std::is_nothrow_assignable_v<T&, U>`

Here is the list that can be marked conditionally `noexcept` according to C++20.
- `std::swap`
- the copy-constructor and -assignment operator
- the move-constructor and -assignment operator

This list is getting modified in the C++23 standard.

It's also worth noting that an implementation can mark conditionally `noexcept` a function even if it's not listed by the standard so.

## Add a conditional noexcept specification to `std::exchange`

One of the primary use cases for `std::exchange` is implementing the move constructor and move assignment operator. In a certain way, it's quite similar to `std::swap`. Yet, while `std::swap` and move operations can be conditionally `noexcept`, it was not the case for `std::exchange`.

Up until C++23.

[P2401R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2401r0.html) makes `std::exchange` conditionally `noexcept`. The conditions are the same as for move operations: `is_nothrow_move_constructible_v<T> && is_nothrow_assignable_v<T&, U>`.

## Add a conditional noexcept specification to `std::apply`

With [the introduction of `zip` algorithms in C++23](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2321r2.html), people in and around the committee started to talk once again more and more about `std::apply`.

The reason is that `std::apply` could be effectively used to implement these new algorithms. In fact, in the previously referenced proposal `apply` appears quite a few times. Sadly, `std::apply` is not `noexcept`.

But, if we have a look into the exposition-only implementation of `apply`, we can see that it uses `invoke` and `get`. The latter is `noexcept` and the former is conditionally `noexcept`, so there is no reason why `std::apply` should not be conditionally `noexcept`.

And that becomes the new reality with C++23:

```cpp
template<class F, class Tuple>
  constexpr decltype(auto) apply(F&& f, Tuple&& t) noexcept(see below);
  
// Let I be the pack 0, 1, ..., (tuple_size_v<remove_reference_t<Tuple>>-1). The exception specification is equivalent to: noexcept(invoke(std::forward<F>(f), get<I>(std::forward<Tuple>(t))...)).
```

## Conclusion

In this article, we reviewed how the standard defines its policies towards the `noexcept` specification and we also see that two standard library functions (`std::apply` and `std::exchange`) are becoming `noexcept`. 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!