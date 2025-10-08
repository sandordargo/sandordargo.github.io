---
layout: post
title: "C++26: range support for std::optional"
date: 2025-10-8
category: dev
tags: [cpp, cpp26, optional, range]
excerpt_separator: <!--more-->
---
I learned about the new range API of `std::optional` from Steve Downey at [CppCon 2025](https://www.sandordargo.com/blog/2025/09/24/trip-report-cppcon-2025) during his [talk about `std::optional<T&>`](https://www.sandordargo.com/blog/2025/10/01/cpp26-optional-of-reference). To be honest, I found the idea quite strange at first. I wanted to dig deeper to understand the motivation and the implications.

The first example I encountered was this:

```cpp
void doSomething(std::string const& data, 
                 std::optional<Logger&> logger = {}) {
    for (auto l : logger) {
        l.log(data);
    }
    return;
}
```

This shows that you can *iterate* over an `optional`. In fact, iterating over an optional means the loop will execute either zero or one time. That’s neat — but at first glance, you might ask: why on Earth would anyone want to do that?

After all, you could just write:

```cpp
void doSomething(std::string const& data, 
                 std::optional<Logger&> logger = {}) {
    if (logger.has_value()) {
        l->log(data);
    }
    return;
}
```

Sure, you need pointer semantics, but if you dislike the arrow operator, you can write `logger.value().log(data)`. It's slightly longer, but arguably more expressive. So what’s the real value of the range-based approach?

Well, the people writing the standard are not exactly known for adding features "just because" - they are smart and considerate. I figured there had to be more compelling examples that didn't fit into a presentation slide. And indeed, there are.

## When `optional` Meets Ranges

A great use case appears when chaining ranges involving optional values. The range API allows us to avoid explicit null checks in pipelines where missing values are simply skipped.

Here’s an example from [P3168R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3168r2.html):

```cpp
// Example from P3168R2: start from a set of values, apply multiple range operations involving optional values.
std::unordered_set<int> s{1, 3, 7, 9};
const auto flt = [&](int i) -> std::optional<int> {
    if (s.contains(i)) {
        return i;
    } else {
        return {};
    }
};

for (auto i : std::views::iota(1, 10) | std::views::transform(flt)) {
    for (auto j : i) { // no need to transform
        for (auto k : std::views::iota(0, j)) {
            // std::cout << '\a'; // do not actually log in tests
            std::ignore = k;
        }
        // std::cout << '\n'; // do not actually log in tests
    }
}
```

This is where `std::optional` as a range truly shines — it integrates seamlessly into the range pipeline and eliminates manual `if` checks. You can find [a broader set of examples in the Beman project unit tests
](https://github.com/bemanproject/optional/blob/main/tests/beman/optional/optional_range_support.test.cpp).

## Competing Proposals

Interestingly, there were competing proposals for this functionality. [P1255R12](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p1255r12.pdf) suggested two new views: `views::maybe` and `views::nullable`.
The former would have represented a view holding zero or one value, while the latter would have adapted types that may or may not contain a value.

However, the authors of [P3168R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3168r2.html) argued for a simpler, more unified approach. Since `std::optional` and the proposed `maybe_view` used identical wording: *"may or may not store a value of type T"*, introducing another type with the same semantics seemed redundant. In the end, genericity and simplicity won — we didn't need a new view when `optional` could just become one.

You can read the arguments pro and contra in the referenced papers.

## `std::optional` as a View

By adopting this range interface, `std::optional` effectively becomes a view containing at most one element. The question was how to make it so.

Two options were considered:

- Inheriting from `std::ranges::view_interface<std::optional<T>>`
- Specializing `std::ranges::enable_view<std::optional<T>>` to `true`

The first approach would have introduced unnecessary member functions, contradicting the simplicity goal. So the second option won  — `std::optional<T>` now simply specializes `ranges::enable_view` in the same spirit as `std::string_view` and `std::span`.

## Iterator Types

The authors really believe in simplicity. They kept trimming unnecessary parts between revisions. Initially, they considered defining a full suite of iterator algorithms, but realized that the existing `ranges::begin()`, `ranges::cbegin()`, et al. already do a big part of the job.

As a result, `std::optional` only defines `iterator` and `const_iterator`, both of which are implementation-defined. They were originally meant to be raw pointers, but that could have led to misuse — such as applying placement new — so implementers will need to ensure safer behavior.

## Conclusion

`std::optional`'s new range interface might look unusual at first, but it's a subtle, useful enhancement. It integrates `optional` into the range ecosystem, letting it compose cleanly with other views and transformations without explicit branching.

Rather than introducing yet another view type, the committee went with keeping things simple and making existing abstractions more composable. It might take a while to feel natural, but once you start thinking of optional as "a range of at most one", it all clicks.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!  