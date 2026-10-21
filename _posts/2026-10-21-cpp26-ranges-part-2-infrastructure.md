---
layout: post
title: "C++26 Ranges: Better Infrastructure"
date: 2026-10-21
category: dev
tags: [cpp, cpp26, ranges, algorithms]
excerpt_separator: <!--more-->
---
C++26 ranges isn't only about [new view adaptors](https://www.sandordargo.com/blog/2026/10/14/cpp26-ranges-part-1-new-views). Three changes landed that don't give you new ways to compose pipelines but fix real problems: containers can now reserve space for lazy ranges that don't know their exact size, `<random>` finally works with ranges, and `ranges::to` no longer forces you into a `|` pipeline when a simple function call would do.

<!--more-->

## `approximately_sized_range` and `reserve_hint`

Here's a scenario that might have bothered you: you have a `views::transform` pipeline that maps Unicode codepoints to their uppercase equivalents. A single "ss" might become "SS" — you can't know the output length in advance. When you `ranges::to<std::vector>()` the result, the implementation has two bad choices: traverse the range twice (once to count, once to fill), or call `push_back` repeatedly and eat the reallocations.

[P2846R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2846r6.html) introduces a middle ground with a new concept and a new customization point object.

### `sized_range` vs `approximately_sized_range`

A `sized_range` gives you the *exact* element count in O(1) via `ranges::size`. An `approximately_sized_range` gives you a *close enough* estimate in O(1) via `ranges::reserve_hint`. Every `sized_range` automatically satisfies `approximately_sized_range` — `reserve_hint` falls back to `ranges::size` when it's available.

### The `ranges::reserve_hint` customization point

`ranges::reserve_hint(r)` resolves in this order:

1. `ranges::size(r)` if the range is `sized_range`
2. `r.reserve_hint()` member function
3. ADL-found `reserve_hint(r)` free function

The result must be non-negative and representable in `range_difference_t`. It must evaluate in O(1).

### Usage

A custom view provides `reserve_hint` by forwarding from its base:

```cpp
template <input_range V>
class uppercase_view : public ranges::view_interface<uppercase_view<V>> {
    V base_;
public:
    constexpr auto reserve_hint()
        requires ranges::approximately_sized_range<V>
    {
        return ranges::reserve_hint(base_);
    }
    // ...
};
```

The consumer side is where the payoff comes. `std::vector`'s construction from an `approximately_sized_range` of `MoveConstructible` elements now calls `reserve()` with the hint. When the hint is accurate, zero reallocations.

### Which views propagate the hint?

This is where the design gets thoughtful. Views that can reasonably estimate output size propagate `reserve_hint`: `take`, `drop`, `adjacent`, `chunk`, `slide`, `stride`, `zip` (minimum of hints), `cartesian_product` (product of hints).

Views where the output size is fundamentally unpredictable do *not* propagate it: `filter`, `take_while`, `drop_while`, `join`, `split`. A `filter` view might drop everything or keep everything — any hint would be misleading.

### Why not a (lower, upper) bound?

The design was inspired by Rust's `Iterator::size_hint`, which returns a `(lower, Option<upper>)` pair. P2846 deliberately chose a single value. The Rust community's experience was that the upper bound was rarely populated and often confusing. A single best-effort estimate is simpler and in practice just as useful for calling `reserve()`.

## `ranges::generate_random`

C++11's `<random>` API gives you one value at a time: you call `engine()` or `dist(engine)` in a loop. If you want to fill a container, you write a loop. This makes auto-vectorization nearly impossible — the compiler sees one call at a time and can't prove that batching them would produce the same sequence.

[P1068R11](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p1068r11.html) adds `std::ranges::generate_random`, a ranges-friendly way to fill a range with random numbers:

```cpp
std::mt19937 engine{42};
std::vector<int> v(1000);

// Fill with raw engine output:
std::ranges::generate_random(v, engine);

// Fill with a distribution:
std::uniform_int_distribution<int> dist{1, 100};
std::ranges::generate_random(v, engine, dist);
```

### The batch optimization hook

The real design insight is that `generate_random` is a customization point. The algorithm first checks whether the engine (or distribution) provides a `generate_random` member function for batch generation. If so, it calls that. If not, it falls back to element-wise generation via `ranges::generate`.

No standard-library engine provides this member yet — today the fallback always fires. But the interface is ready for custom engines that wrap vectorized backends. When those engines implement the batch member, existing code using `ranges::generate_random` gets the speedup for free.

### Argument order

The API follows `ranges::fill`-style conventions: range first, then engine, then optional distribution. Both range-based and iterator-pair overloads exist.

```cpp
// Range overload
std::ranges::generate_random(v, engine);

// Iterator-pair overload
std::ranges::generate_random(v.begin(), v.end(), engine, dist);
```

The distribution overload requires `std::is_arithmetic_v` on the result type — no user-defined numeric types.

## List-initialization for algorithms

This one is a pure ergonomic fix. Consider:

```cpp
std::vector<std::complex<double>> v = { {1.0, 2.0}, {3.0, 4.0}, {5.0, 6.0} };

// C++23: you must spell out the type
auto it = std::find(v.begin(), v.end(), std::complex<double>{3.0, 4.0});

// C++26: braced-init-list works
auto it2 = std::find(v.begin(), v.end(), {3.0, 4.0});
```

[P2248R8](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2248r8.html) makes this work by giving the value-type template parameter a default. The same pattern applies to a long list of algorithms.

### How it works

Before C++26, `std::find` looks like:

```cpp
template <class InputIterator, class T>
InputIterator find(InputIterator first, InputIterator last, const T& value);
```

Template argument deduction can't deduce `T` from a braced-init-list `{3.0, 4.0}`. C++26 adds a default:

```cpp
template <class InputIterator,
          class T = typename iterator_traits<InputIterator>::value_type>
InputIterator find(InputIterator first, InputIterator last, const T& value);
```

Now `T` defaults to the iterator's value type, and the braced-init-list initializes a `T` — which is `std::complex<double>` in our example. Member functions like `v.push_back({3.0, 4.0})` already worked this way; algorithms just couldn't because of template deduction rules.

### Which algorithms benefit

`find`, `count`, `fill`, `remove`, `replace`, `lower_bound`, `upper_bound`, `binary_search`, `search_n`, and their ranges counterparts. Container erasure functions (`std::erase`) benefit too.

Notable exclusions: `replace_copy` doesn't get a default because its single `T` serves both the old and new values — the committee decided the default would be ambiguous. Numeric algorithms from `<numeric>` are excluded due to lack of consensus.

### Ranges algorithms: reordering for projections

The ranges versions needed a subtler fix. The value type must account for projections — `ranges::find(v, {3.0}, &point::x)` should deduce `T` as `double`, not `point`. P2248 reorders the template parameters so `Proj` precedes `T`, enabling `T` to default to `projected_value_t<iterator_t<R>, Proj>` — a new alias that correctly handles projections and proxy iterators (like `vector<bool>`).

## Conclusion

These three changes don't introduce new ways to compose views, but they make the existing ranges ecosystem meaningfully more practical. `approximately_sized_range` eliminates unnecessary reallocations when materializing lazy pipelines. `generate_random` is both a convenience and a future-proofing hook for vectorized random generation. And list-initialization for algorithms removes a paper cut that has annoyed every C++ programmer who's ever written `std::find` with a compound type.

In [the next part](/blog/2026/XX/XX/cpp26-ranges-part-3-parallel-and-safety), we'll cover the headliner: parallel range algorithms, plus the safety improvements for `filter_view`.

{% include connect-deeper.html %}
