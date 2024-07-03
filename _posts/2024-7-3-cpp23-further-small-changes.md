---
layout: post
title: "C++23: further small changes"
date: 2024-7-3
category: dev
tags: [cpp, cpp23, unreachable, deprecation]
excerpt_separator: <!--more-->
---
In this article, let's get back to exploring C++23. We are going to have a look at some unrelated small changes in the standard, including the rarest species of changes. Deprecations!

## Printing volatile pointers

[P1147R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1147r1.html) has pointed out and fixed an interesting behaviour of `std::cout` with pointers.

Let's assume that we have `p0` and `p1` pointers pointing at the same address (`0xdeadbeef`), but their type is different. `p0` is `int*` while `p1` is `volatile int*`. If you happen to print those addresses with `cout`, the output will be different for the two pointers. While printing `p0` used to output the actual address, `p1` on the other hand used to print `1`.

The reason behind is that for the two pointers, different overloads of `std::cout` matched. `int*` was implicitly converted to `const void*`, but `volatile int*` was converted to `bool`. Thanks to [P1147R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1147r1.html), a new overload `const volatile void*` has been introduced which `const_cast`s away `volatile` and calls `const void*`. As a result, printing the values of `int*` and `volatile int*` is the same now.

## Clarifying the status of the “C headers”

[P2340R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2340r1.html) is actually both a good read and a meaningful update to the standard. You're probably aware of so-called C headers in C++.

Most ISO C headers are available in C++ in two forms. For one, you have the C-style headers following the `<name.h>` conventions and also the C++ version of them following the `<cname>` convention with the same names declared in it but within the `std` namespace.

For example, there is `<stdint.h>` with all the fixed width integer types declared in it in the global namespace, such as `int8_t`, `int16_t`, etc; and there is the C++ version called `<cstdint>` with `std::int8_t`, `std::int16_t`, etc.

Since the beginning of the C++ standard, the C headers (`<name.h>`) have been deprecated, in other words, subject to future removal, whereas everyone knew that they wouldn't go anywhere due to reasons of interoperability with C. The usage of C headers is needed when code must be valid C++ and valid C code. Otherwise, in pure C++ code, we should use the C++ versions. At the same time, the need for the C headers is not going away.

This proposal makes C headers no longer deprecated, but assigns them a state called "for foreign-language interoperability only."

## `std::unreachable()`

[P0627R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0627r6.pdf) is introducing a new standard library function in the `<utility>` header, `std::unreachable()`.

`std::unreachable()` serves to mark locations in code that are known to the compiler to be unreachable.

Let's take an example where we have a function `foo` taking an `int n` which can only have the values 0, 1, 2 or 3. The function contains a switch statement.

```cpp
void foo(int n) {
switch (n) {
	case 0:
	case 2:
		handle_0_or_2();
		break;
	case 1:
		handle_1();
		break;
	case 3:
		handle_3();
		break;
	}
}
```

The compiler will not know that `n` cannot be smaller than 0 or bigger than 3, so it might generate some extra instructions to skip the whole switch statement if the value is different.

One option would be to use an enum in these cases, but it might not always be possible and we could find other usages for `std::unreachable()`. So another way to omit such extra code is to mark other cases unreachable. It's already possible with some compiler- and/or platform-specific functions such as `__builtin_reachable()` or `__assume(false)`, but with C++23 we also get the standard way to this.

```cpp
void foo(int n) {
switch (n) {
	case 0:
	case 2:
		handle_0_or_2();
		break;
	case 1:
		handle_1();
		break;
	case 3:
		handle_3();
		break;
	default:
		std::unreachable();
	}
}
```
If `std::unreachable()` is still reached, the behaviour is undefined.

For an interesting read on design decisions and comparison with contract-based programming, check out [P0627R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0627r6.pdf).

## Deprecate `std::aligned_storage` and `std::aligned_union`

We often say that one of the superpowers of C++ is backward compatibility. Very few things get deprecated even less removed. We discussed already that [the support for garbage collection has been removed from C++](). Today we finish this article, with a couple of deprecations.

Let's start with [P1413R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1413r3.pdf) which deprecates `std::aligned_storage` and `std::aligned_union`. These functions are niche, they are mainly used for containers by utility libraries, such as *Abseil*, *Boost* or *Folly*.

The reason behind removing these functions is their poor, unsafe and sometimes even misleading APIs. They require extensive usage of `reinterpret_cast` and as such they are prone to invoke undefined behaviour. They fail to provide upper-bound guarantees on the size of the resulting type and some of the default template arguments are misleading and lead to errors.

If you want to learn about the details of these issues, read the "On the API" section of [P1413R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1413r3.pdf).

If you happen to use these `aligned_*` functions, most probably you're aware of this and the suggested replacement which is using `std::byte` in combination with the `alignas` attribute.

```cpp
- std::aligned_storage_t<sizeof(T), alignof(T)> t_buff;
+ alignas(T) std::byte t_buff[sizeof(T)];
```

## Deprecate `std::numeric_limits::has_denorm` and `std::numeric_limits::has_denorm_loss`

This change goes hand in hand with a change in `C` where `_HAS_SUBNORM` macros are getting obsoleted.

Most probably you never used these functions. Even the author of [P2614R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2614r2.pdf) claims that no code relies on these. Why this paper is interesting and we refer back once again to that superpower of backwards-compatibility because the author explicitly says that these functions are not helpful and can be even misleading, yet they cannot be removed. They should be deprecated, so marked as not to be used, but without the intent of actual removal! The reason: that would be a major compatibility break.

I'm not in the position to judge this approach, but it's definitely a reason why the standard is ever-growing and why certain people think that the language is getting more and more complex.

## Conclusion

In this article, we reviewed some small changes in the standard. We've how printing volatile pointers is getting fixed, we've seen that marking unreachable parts of code is getting standardized and we've also had a look at the deprecation of some niche or not even used functions from the standard library. 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!