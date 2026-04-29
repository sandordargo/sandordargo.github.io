---
layout: post
title: "C++26: string and string_view improvements"
date: 2026-4-29
category: dev
tags: [cpp, cpp26, string, string_view]
excerpt_separator: <!--more-->
---
Let's continue our exploration of C++26 improvements. Today we focus on `string_view`. Some types got new constructors accepting `string_view`s, and concatenation of `string`s and `string_view`s just got easier.

But let's start with a brief reminder of what a `string_view` is.

<!--more-->

## Reminder: the role of `string_view`

`std::string_view` was introduced in C++17 and its purpose is to provide read-only access to a string-like object. It can often replace `const string&` parameters and offers a significant performance gain. It's generally advisable to use it whenever you'd pass an immutable string-like input that you cannot move from source to target.

We covered the topic earlier in more depth [here](https://www.sandordargo.com/blog/2022/07/13/why_to_use_string_views).

## P2495R3: Interfacing `stringstream`s with `string_view`

A `stringstream` is a good old tool for dealing with operations on string-based streams. While [C++23 introduced `spanstream`s](https://www.sandordargo.com/blog/2023/12/06/cpp23-strtream-strstream-replacement), due to fundamental semantic differences, `stringstream`s are not dead and it's important to maintain them.

Being a good old tool also means they predate `string_view`. Given the available set of constructors, if you want to initialize a `stringstream` from a `string_view`, you first have to manually convert it into a `string`.

[P2495R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2495r3.pdf) fixes this by adding new constructors accepting `string_view`s.

It's worth noting that this is a purely additive library change — it doesn't break existing code.

At the moment of publication, this change is already available on Clang 19.

## P2697R1: Interfacing `bitset` with `string_view`

[P2697R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2697r1.pdf) is very similar to the previous one, both in nature and structure. In fact, it was written by the same author, Michael Florian Hava.

This paper extends the set of constructors `std::bitset` offers.

> `std::bitset` represents a fixed-size sequence of N bits. Bitsets can be manipulated through standard logic operators and they can be converted to and from strings and integers.

Up until C++26, constructing a `bitset` from a `string_view` required converting it to a temporary `string` first — an unnecessary copy. Alternatively, one could access `string_view`'s underlying content through its `data()` accessor, but that's unergonomic and can lead to problems related to null-termination.

Thanks to [P2697R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2697r1.pdf), these problems are gone and you can directly construct a `bitset` from a `string_view`.

At the moment of publication, this change is already available on Clang 18.

## P2591R5: Concatenation of strings and string views

[This one](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2591r5.html) is a bit more involved than the previous two, both in terms of paper size and design decisions required.

The basic problem: up until C++26, you cannot concatenate a `std::string` with a `std::string_view`:

```cpp
// https://godbolt.org/z/K4MWe87K6
#include <iostream>
#include <string>
#include <string_view>

std::string concat(std::string s, std::string_view sv) {
    return s + sv; // ERROR
}

int main() {
    std::string hello{"hello "};
    std::string_view world{"world"};
    std::cout <<  concat(hello, world);
}
/*
<source>: In function 'std::string concat(std::string, std::string_view)':
<source>:6:14: error: no match for 'operator+' (operand types are 'std::string' {aka 'std::__cxx11::basic_string<char>'} and 'std::string_view' {aka 'std::basic_string_view<char>'})
    6 |     return s + sv;
      |            ~ ^ ~~
      |            |   |
      |            |   std::string_view {aka std::basic_string_view<char>}
      |            std::string {aka std::__cxx11::basic_string<char>}
*/
```

Had we used two `string` parameters, there would be no issue at all. Without [P2591R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2591r5.html), the only workarounds are unergonomic or error-prone:
- convert the view to a string, which is inefficient (`s + std::string(sv)`)
- access the underlying data of the view, which is error-prone due to null-termination issues (`s + sv.data()`)
- use `+=`, `append()`, or `insert()`, which are tedious

API-wise, the fix is simple: add `operator+` overloads that work with `std::string` and `std::string_view` in both orders.

You might wonder why this wasn't in the standard already. If you work in a large organisation, the story that the author of [P2591R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2591r5.html) dug up might sound familiar: the overloads were held back for a potential future string builder feature that never arrived. And even if it had, a string builder wouldn't justify leaving `operator+` overloads out.

I won't cover all the design considerations here, but if you're interested, read [Section 4 of the proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2591r5.html#designdecisions). It's worth noting that the preferred design changed several times across the five revisions.

The final approach uses free non-friend function templates that accept anything convertible to a `string_view`. If you take the previous example and enable the C++26 flag, you'll see that [it works](https://godbolt.org/z/KjhacaT4v).

One thing this approach deliberately does not support is concatenating an object convertible to `string` with another `string` — and that's expected; it's actually one of the reasons the current design was chosen.

At the moment of publication, this change is already available on Clang 19 and GCC 15.

## Conclusion

C++26 brings three focused improvements to the `string`/`string_view` ecosystem. `stringstream` and `bitset` both gain constructors that accept `string_view` directly, removing the need for unnecessary temporary strings. And `operator+` finally works between `std::string` and `std::string_view`, eliminating a long-standing ergonomic rough edge. None of these are headline features, but they all remove friction that you've probably run into before.

## Connect deeper

If you liked this article, please
- hit on the like button,
- [subscribe to my newsletter](https://sandor-dargo.kit.com/e19f29b0a1)
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
