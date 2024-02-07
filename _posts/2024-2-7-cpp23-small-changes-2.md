---
layout: post
title: "C++23: More small changes"
date: 2024-2-7
category: dev
tags: [cpp, cpp23, string_view, pairs]
excerpt_separator: <!--more-->
---
In this post, we continue discovering the changes introduced by C++23. We are going to look into three (and a half) small changes, each affecting constructors of some standard library types. We're going to see how new constructors for container types, a new range constructor for `string_view` and some default template arguments for `pair`.

Read on for the details.

## Iterators pair constructors for stack and queue

As of C++20, almost all container-like objects (containers and container-adaptors) can be initialized with a pair of iterators.

```cpp
std::vector<int> v{42, 51, 66};
std::list<int> l(v.begin(), v.end());
```

All of them, except for `std::stack` and `std::queue`. They don't provide such overloads. If you want to initialize them with a pair of iterators, you need an intermediary `std::initiailizer_list`.

```cpp
std::vector<int> v{42, 51, 66};
// std::queue<int> q1(v.begin(), v.end()); // DOESN'T COMPILE!
std::queue<int> q2({v.begin(), v.end()});
```

This inconsistency, at first, looks like a small inconvenience. But its effects are much deeper. While using the `stack` or `queue` on its own is not a big burden, if you want to offer functionality that works with all container-like objects, you have a higher price to pay.

Due to the lack of an iterator-pair-based constructor, you either have to write special implementations or not support them. For the `ranges` library that is definitely a problem. Deducing types using CTAD is not possible either.

[P1425R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1425r4.pdf) fixes this situation by adding the iterator-pair-based constructors for `stack` and `queue` to the standard as well as the necessary deduction guidelines.

## (Explicit) Range constructor for std::string_view

[P1989R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1989r2.pdf) proposes a range constructor to `std::string_view`. [P24990R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2499r0.html) also makes it explicit.

```cpp
template<class charT, class traits = char_traits<charT>>
class basic_string_view {
public:
 // [...]

 template <class R>
 constexpr basic_string_view(R&& r);
};
```

This was not the first time that this constructor was proposed, but previously it was rejected due to some concerns about when the constructor would be chosen over a `string_view` conversion function. This concern has been addressed in [P1989R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1989r2.pdf) with the set of constraints applied to `R`. Let's enumerate those constraints:
- `R` must model `std::ranges::contiguous_range` and `std::ranges::size_range`
- `std::ranges::range_value_t<R>` and `charT` must be the same
- `R` must not be convertible to `const charT*`
- `d.operator ::std::basic_string_view<charT, traits>()` must not be a valid expression, where `d` is an lvalue of type `remove_cvref_t<R>`
- if the qualified-id `R::traits_type` is valid and denotes a type, `is_same_v<remove_reference_t<R>::traits_type, traits>` must be true

The last two constraints aim exactly to avoid this use case. Thanks to them, the new `string_view` constructor won't be selected, when a type has the conversion a conversion function.

On the other hand, if a type otherwise satisfying the constraints has a conversion operator to a different `basic_string_view`, notably `basic_string_view<charT, some-other-traits-type>`, while not itself defining `using type_traits = some-other-traits-type`, a program that was previously ill-formed will call the new range overload.

## Default Arguments for `pair`'s Forwarding Constructor

[P1951R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1951r1.html) proposes to adopt a similar strategy that has been used for other types (such as `std::optional` or `std::exchange`) to accomodate braced initializers.

This strategy is rather simple, the forwarding constructor of `pair` should use `T1` and `T2` as default arguments of `U1` and `U2`, so that braced initializers may be used as constructor arguments.

Here is the changed constructor signature. Only the defaulting is new in it.

```cpp
template<class U1=T1, class U2=T2> 
  constexpr explicit(see below) pair(U1&& x, U2&& y);

``` 

The motivation behind this change is that due to the lack of this defaulting when a simple pair of braces are used to create one of the values (`std::pair<std::string, std::vector<std::string>> p("hello", {});
`), an inefficient constructor would be chosen and extra copies would be made.

With this simple proposed change, `pair`'s forwarding constructor would be chosen which in this case would use move instead of copy operations. In fact, with this change, lots of existing code's behaviour will change, but for the better. In many cases, where there were copies, there will be moves. That's welcome.

## Conclusion

In this post, we learned about some constructor-related changes introduced by C++23. We saw that now `std::stack` and `std::queue` can be initialized from iterator pairs; `std::string_view`s can be created from ranges and we can construct `std::pair` via a forwarding constructor more often than before.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!