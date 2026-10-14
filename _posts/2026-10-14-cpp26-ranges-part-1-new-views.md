---
layout: post
title: "C++26 Ranges: Four New Views"
date: 2026-10-14
category: dev
tags: [cpp, cpp26, ranges, views]
excerpt_separator: <!--more-->
---
C++20 shipped a powerful ranges library, but the set of view adaptors had some obvious gaps. Want to get the output as a container? No luck. At least [C++23 helped with that](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1206r7.pdf). Want to iterate over two vectors as one sequence? Write your own. Want to avoid redundant evaluations in a `transform | filter` pipeline? Write a workaround. Want a quick index sequence matching a range's size? Spell out `iota` with an ugly cast.

C++26 fills four of these gaps with new views: `views::concat`, `views::to_input`, `views::cache_latest`, and `views::indices`.

<!--more-->

## `views::concat` — iterate over multiple ranges as one

This was one of the most requested additions. [P2542R8](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2542r8.html) gives us `views::concat`, which lazily concatenates any number of ranges into a single view:

```cpp
// https://godbolt.org/z/dvMjdvv6b

std::vector<int> v1{1, 2, 3};
std::vector<int> v2{4, 5};
std::array a{6, 7, 8};

for (int x : std::views::concat(v1, v2, a))
    std::cout << x << ' ';  // 1 2 3 4 5 6 7 8
```

Before C++26, you'd either copy everything into a temporary container or write nested loops. `views::concat` gives you a single lazy sequence with no copies.

### Mixed range types

The ranges don't need to be the same type - they just need compatible element types. The view computes a common reference, value, and rvalue-reference type across all ranges using `common_reference_t` and `common_type_t`. If those don't exist, the concatenation is ill-formed.

```cpp
// https://godbolt.org/z/Tnf6v4K88

std::vector<int> vi{1, 2};
std::vector<double> vd{3.0, 4.0};
// works: common_reference_t<int&, double&> is double
for (double x : std::views::concat(vi, vd))
    std::cout << x << ' ';  // 1 2 3 4
```

There are limitations, though. The conversions from each range's reference type to the common types must be equality-preserving. You can't mix a range of move-only rvalue references with a range of prvalues if dereferencing the first would leave it in a moved-from state on repeated access. In practice this is rarely an issue - the most common case is concatenating ranges of the same or implicitly convertible value types, and that just works.

### Iterator category

The resulting iterator category depends on the weakest underlying range, with one extra rule that might surprise you: for bidirectional or above, every range *except the last* must be a `common_range` (i.e., `begin()` and `end()` return the same type). This is because decrementing across a range boundary needs `end()` to return an iterator you can decrement from - a sentinel won't do.

| All ranges are... | Extra requirement | Result |
|---|---|---|
| random_access | all-but-last are `common_range` | random_access |
| bidirectional | all-but-last are `common_range` | bidirectional |
| forward | - | forward |
| otherwise | - | input |

This means that concatenating two `std::vector`s gives you random-access iteration — the view can compute the total size, and jumping to an arbitrary index just requires figuring out which underlying range it falls into. But if you mix in a `std::list`, the weakest category wins:

```cpp
// https://godbolt.org/z/j7W67sYaa
std::vector v1{1, 2, 3};
std::vector v2{4, 5, 6};
std::list l{7, 8, 9};

// Both vectors are random_access + common_range -> random_access
auto c1 = std::views::concat(v1, v2);
static_assert(std::ranges::random_access_range<decltype(c1)>);

// vector is random_access + common, but list is only bidirectional
// -> bidirectional overall (weakest wins)
auto c2 = std::views::concat(v1, l);
static_assert(std::ranges::bidirectional_range<decltype(c2)>);
static_assert(!std::ranges::random_access_range<decltype(c2)>);
```

### Things to know

`views::concat` is a range *factory*, not a pipeable adaptor. You call it with all ranges as arguments: `views::concat(r1, r2, r3)`. You can then pipe its result onward: `views::concat(r1, r2) | views::filter(pred)`.

If you pass a single range, `views::concat(r)` is equivalent to `views::all(r)` - no `concat_view` is created, just a lightweight `all` wrapper. This is a nice optimisation: there's nothing to concatenate, so the machinery is bypassed entirely. Zero arguments won't compile - you need at least one range.

The view is never `borrowed_range`. The reason is that its iterator stores a `std::variant` of the underlying iterators internally, and that variant's lifetime is tied to the `concat_view` object. If the view were destroyed while an iterator was still alive, the variant would be dangling. It's `sized_range` only when all constituent ranges are sized - otherwise there's no way to compute a total size without iterating.

## `views::to_input` — deliberate performance downgrade

This one sounds counterintuitive: why would you voluntarily *weaken* a range's iterator category?

[P3137R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3137r3.html) adds `views::to_input`, which wraps a range and exposes it as an input-only range. The name was deliberately chosen to differ from the existing `as_const` / `as_rvalue` adaptors - those operate on elements, while `to_input` operates on the *view itself*.

### The motivation: compile-time costs

Range adaptors maximize their iterator category for generality. But that generality has a price. Consider:

```cpp
r | views::chunk(n) | views::join
```

`chunk` sees a forward range and selects its forward-iterator codepath. What does that cost? Forward iterators support multi-pass - you can copy an iterator and traverse from both copies independently. To support this, `chunk_view`'s iterator must track both the current position *and* the end of the current chunk, comparing against both on every increment. It also caches `begin()` so that calling it repeatedly doesn't re-scan. All of this is necessary to satisfy the forward iterator contract.

But then `join` flattens the chunks back into a single sequence - and the result is input-only regardless. The multi-pass guarantee, the double tracking, the cached `begin()` - none of it was needed. The forward work was wasted.

With `views::to_input`:

```cpp
r | views::to_input | views::chunk(n) | views::join
```

Now `chunk` sees an input range and selects its simpler, cheaper codepath. Same result, less work. The benefits are both runtime (fewer comparisons, no caching) and compile-time (simpler template instantiations).

### When the adaptor is a no-op

If the source range is already input-only, non-common, and non-forward, `views::to_input` passes through via `views::all` - no wrapping overhead.

### Ties to filter safety

As we'll see in part 3 of this series, `views::to_input` also plays a role in making `filter_view` safer to use. Prepending it before a filter prevents the dangerous compositions that could previously produce undefined behaviour.

## `views::cache_latest` — stop computing things twice

If you've ever profiled a `transform | filter` pipeline and noticed the transform running twice per surviving element, [P3138R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3138r5.html) is for you.

The reason behind this is that when `filter` evaluates its predicate, it dereferences the iterator - calling the transform. When you then dereference the filter's iterator to use the result, it dereferences the underlying iterator *again*, calling the transform a second time.

```cpp
// Full example at: https://godbolt.org/z/he3Y97cWj

auto pipeline = v
    | views::transform(expensive_computation)
    | views::cache_latest  // caches the most recently dereferenced value
    | views::filter(predicate);
```

`views::cache_latest` stores the result of the most recent `operator*` call. Subsequent dereferences return the cached value until the iterator advances. For reference types, it stores a pointer; for prvalue types, the value itself.

### Design constraints

The view is deliberately restrictive:

- **Input-only.** It never models `forward_range` — multi-pass iteration would require caching more than one element.
- **Never `const`-iterable.** The cache is mutable state.
- **Not thread-safe.** Concurrent `operator*` calls on the same iterator may race. This is an explicit relaxation in the specification — input iterators aren't meant for cross-thread sharing anyway.

The cache is invalidated on `operator++`. Advancing and then dereferencing produces a fresh computation.

## `views::indices` — the iota you actually wanted

Have you ever tried to write a view of indices `[0, 1, ..., n-1]` where `n` is the size of a range?

You might have run into trouble:

```cpp
// Full example at: https://godbolt.org/z/jPsasEhYc

// C++23: doesn't compile 
// iota(int, size_t) is ill-formed due to mismatched types
auto bad = std::views::iota(0, std::ranges::size(rng));

// C++23 workaround: verbose
auto ugly = std::views::iota(std::ranges::range_size_t<decltype(rng)>{},
                              std::ranges::size(rng));
```

[P3060R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3060r3.html) fixes this with a new range adaptor, `views::indices(n)`:

```cpp
auto good = std::views::indices(std::ranges::size(rng));
// [0, 1, 2, ..., size-1] with the correct type
```

Under the hood it's just `views::iota(T(0), n)` where `T` is deduced from the argument. No new view type - it reuses `iota_view`. The argument must be integer-like; `bool` is excluded.

It's a single-argument adaptor only. If you need `indices(start, end)`, `views::iota` still has you covered.

## Conclusion

These four views close real gaps in the C++20/23 ranges toolkit. `views::concat` gives you lazy multi-range iteration without copies. `views::to_input` and `views::cache_latest` are performance tools — one avoids unnecessary work at the adaptor level, the other avoids redundant computations in the pipeline. And `views::indices` finally lets you write `iota(0, size(r))` without a type mismatch.

In the next part, we'll look at the infrastructure improvements that make ranges more practical: `approximately_sized_range`, `ranges::generate_random`, and list-initialization for algorithms.

{% include connect-deeper.html %}
