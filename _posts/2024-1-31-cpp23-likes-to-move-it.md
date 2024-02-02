---
layout: post
title: "C++23 likes to move it!"
date: 2024-1-31
category: dev
tags: [cpp, cpp23, movesemantics, moveoperations]
excerpt_separator: <!--more-->
---
C++23 is going to bring us a few changes regarding move operations. It mostly means extended support in the standard library, but there is also one change directly in the language. Let's start with that.

## Simpler implicit move

[P2266R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2266r3.html) is a quite great proposal both in terms of its high-quality explanation and the amount of proposed changes in wording. I think if you're interested in exploring a readable proposal, [this](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2266r3.html) might be the one.

Let me try to summarize it briefly.

Since the introduction of move semantics and rvalue references in C++11, we can return move-only types by value:

```cpp
struct Widget {
    Widget(Widget&&);
};

Widget one(Widget w) {
    return w;
}
```

C++14 extended this support so that even converting constructors accepting an rvalue type can be called to invoke an implicit move.

```cpp
struct RRefTaker {
    RRefTaker(Widget&&); // here is the converting constructor
};

RRefTaker two(Widget w) {
    return w;
}
```

As you can see, the two examples follow the same logic, in a sense they are quite symmetric.

C++20 introduced some changes in copy elision and overload resolution. As a consequence `two()` would work even if `w` was taken as an rvalue (`Widget&&`). But because of a wording issue, if it had to take a `Widget&&` and return the same `Widget&&` we would get an error. The original wording is rather complex as it's also claimed in the abstract of [P2266R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2266r3.html). The proposal fixes the mentioned defect and also simplifies the specification by saying that a returned move-eligible id-expression is always an xvalue.

> In C++, an eXpiring value (xvalue in short) represents an expiring or about-to-expire value. It typically occurs in the context of a move operation or when the value is no longer needed in its current location. Examples include the result of `std::move()` and `operator[]` applied to an rvalue.

GCC 13 and Clang 13 already support this!

## std::move_only_function

[P0288R9](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0288r9.html) addresses a request that has been opened in 2014. Nobody can say that this is a reckless change!

`std::move_only_function` is like `std::function`, but for move-only types. It's a wrapper for any constructible callable targets, such as functions, lambdas, function objects or bind expressions. The wrapper can only be moved as its copy assignment operator and copy constructor are deleted. It also supports cv/ref/noexcept qualifiers in function types.

You (will) find this new utility in the `<functional>` wrapper.

GCC 12 and MSVC 19.32 already support this!

*Given that this has been awaited for a long time and that [`std::function` has a bad reputation](https://www.sandordargo.com/blog/2023/04/05/binary-size-and-templates), I plan to experiment with `std::move_only_function` in a few weeks and report back the results.*. 

## Adding move-only types support for comparison concepts

C++20 introduced [concepts](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations) and even though we have move semantics in C++ since C++11, many important concepts do not support move-only types, such as `std::equality_comparable_with`, `totally_ordered_with` or `three_way_comparable_with`.

The reason is often that these concepts are implemented in a way that two `const&` types have to be convertible to the non-reference `std::common_reference_t` which means that the two types have to be copyable.

[P2404R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2404r3.pdf) aims to relaxing this requriement by replacing `common_reference_with<const remove_reference_t<T>&, const remove_reference_t<U>&>` with `comparison-common-type-with <T, U>` in `std::three_way_comparable_with` and in `std::equality_comparable_with`. It also relaxes the preconditions in `std::totally_ordered_with` so that it supports move-only types.

`comparison-common-type-with <T, U>` is an exposition-only concept intended to determine that there exists a common supertype of `T` and `U` even if they are move-only types.

> [Exposition-only means that they are not required by the standard, they just illustrate the intentions of the authors.](https://stackoverflow.com/a/34493177) An exposition-only definition is also a hint to the implementers about what the committee had in mind. 

MSVC 19.36 already supports this!

## Relaxing range adaptors to allow for move-only types

Even though ranges were introduced to C++ many years later than move-semantics, still several range adaptors require that the types they store are copy-constructible.

[P2494R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2494r2.html) slightly modifies the views that currently use the exposition-only type `copyable-box` for user-provided predicates. That slight modification is that `copyable-box` is renamed to `movable-box`, but no constraints are modified for the changed views.

It goes further though for `transform`-like views, where requirements on invocables are relaxed. Now their invocables don't have to be copy-constructible anymore, move-constructability is enough. These views are:

- `single_view,`
- `transform_view,`
- `zip_transform_view`, and
- `adjacent_transform_view`. 

MSVC 19.34 already supports this!

## `std::move_iterator` should not always be `input_iterator`

In order to understand the goals of this paper, let's start with a bit of C++ history. In C++17, you could only subtract two iterators, if they were random access iterators. At the same time, there was no other way to get the size of a range.

> If you want a short reminder on what iterators are, [read this article](https://www.sandordargo.com/blog/2022/03/16/iterators-vs-pointers#what-is-an-iterator).

C++20 improved this situation and even input iterators can be subtracted from their ranges' "end" marked by a so-called `sized_sentinel_for` object and the standardized ranges can provide their size in a cheaper way than subtraction. Given that the range is a so-called sized range.

On the other hand, if the range is unsized, there are some issues. If a range is unsized, you cannot construct a `vector` out of it by one single allocation, since we don't know the size. Instead, the compiler has to loop over the range and keep pushing back to the vector.

```cpp
some_unsized_forward_range | views::move | ranges::to<vector>()
```
In C++20, `move_iterator<T*>` is an `input_iterator` which can be problematic. You can go through `input_iterator`s normally multiple times without a problem, but you cannot do the same with `move_iterator` because if you do so, the second time you already pass through moved-from objects. Therefore library authors have to be very cautious and recognize if they deal with move_iterator.

[P2520R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2520r0.html) fixes this problem by not having `move_iterator<Iterator>::interator_concept` always `input_iterator_tag`. `iterator_concept` will be based on the actual `Iterator`. `iterator_concept` will only be `input_iterator_tag` if `Iterator` doesn't model `random_access_iterator`, `bidirectional_iterator` or `forward_iterator`. If `Iterator` models one of those then the concept will be the corresponding tag (`<iterator concept>_tag`).

As such, the above example can work fine with a single allocation, because if `Iterator` is a random access iterator (such as for `T*`), then the compiler can get the size without accessing any item, without passing through the range.

Clang 17 and MSVC 19.34 already support this!

## Conclusion

In this article, we reviewed how C++23 relaxes many requirements. More and more operations that required copy-constructible objects now will only require move-constructible ones. These are quite logical changes that are also good for performance.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!