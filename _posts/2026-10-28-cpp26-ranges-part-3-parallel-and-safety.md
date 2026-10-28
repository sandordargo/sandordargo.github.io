---
layout: post
title: "C++26 Ranges: Parallel Algorithms and Safer Filters"
date: 2026-10-28
category: dev
tags: [cpp, cpp26, ranges, parallel, algorithms]
excerpt_separator: <!--more-->
---
In [part 1](https://www.sandordargo.com/blog/2026/10/14/cpp26-ranges-part-1-new-views) we looked at the new views C++26 adds, and in [part 2](https://www.sandordargo.com/blog/2026/10/21/cpp26-ranges-part-2-infrastructure) we covered the infrastructure improvements. This final part is about the biggest gap in C++20 ranges: you couldn't run them in parallel. C++17 added execution policies to the classic `std::` algorithms, but C++20 ranges never got them. If you wanted parallelism, you had to decompose your ranges pipeline back into iterator pairs. C++26 finally closes this gap - and while it was at it, the committee also fixed a long-standing foot-gun in `filter_view` that could silently produce undefined behaviour.

<!--more-->

## Parallel range algorithms

[P3179R9](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3179r9.html) brings execution policies to over 50 range algorithms. The syntax is straightforward - the execution policy goes first:

```cpp
std::vector<int> v = /* ... */;

std::ranges::sort(std::execution::par, v);

std::ranges::for_each(std::execution::par_unseq,
    std::views::iota(0, n),
    [&](int i) { data[i] *= 2; });
```

All the execution policies you know from C++17 work: `seq`, `par`, `par_unseq`, `unseq`.

### Which algorithms?

Essentially all of the non-numeric ones. The list includes `for_each`, `for_each_n`, `transform`, `copy`, `copy_if`, `move`, `fill`, `generate`, `replace`, `replace_if`, `remove`, `remove_if`, `unique`, `reverse`, `rotate`, `sort`, `stable_sort`, `partial_sort`, `nth_element`, `merge`, `partition`, `stable_partition`, `find`, `find_if`, `find_if_not`, `find_last`, `find_end`, `find_first_of`, `adjacent_find`, `count`, `count_if`, `all_of`, `any_of`, `none_of`, `equal`, `mismatch`, `search`, `swap_ranges`, `min_element`, `max_element`, `minmax_element`, `includes`, `set_union`, `set_intersection`, `set_difference`, `set_symmetric_difference`, `is_sorted`, `is_partitioned`, `is_heap`, `contains`, `starts_with`, `ends_with`, among others.

There are some notable absences though, such as numeric algorithms (`reduce`, `transform_reduce`) and algorithms without C++17 parallel counterparts (`push_heap`, `binary_search`).

### The key constraint: random access and sized

Here's the design decision that matters most. Parallel range algorithms require `random_access_range` **and** `sized_range` — not just `forward_range`. Parallel backends need random access for divide-and-conquer decomposition, and they need a known size to partition the work.

This means you can't directly parallelize a `filter_view` (which is bidirectional at best) or a `views::iota` over an unbounded range. You'd need to materialize or constrain the range first.

### Projections and output ranges

Projections are fully supported. The output side is also ranges-aware - output is a range (not a bare output iterator), ensuring bounds are known:

```cpp
std::vector<int> src = {1, 2, 3, 4, 5};
std::vector<int> dst(5);
std::ranges::transform(std::execution::par, src, dst,
    [](int x) { return x * x; });
```

### A few gotchas

`for_each` returns only the iterator, *not* the function object. With parallel execution, the callable is copied across threads - accumulating state in it would be a data race. If you need to collect results, use `transform` into an output range.

Callables must be `copy_constructible` (not just `move_constructible`), since the parallel backend may copy them across worker threads.

`constexpr` support is not included yet.

## Filter view safety improvements

`filter_view` has been one of the more dangerous standard views since C++20. [P3725R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3725r3.pdf) addresses three related foot-guns.

### The foot-guns

**1. Composing with `reverse` can crash.** Consider:

```cpp
auto result = v
    | views::filter(pred)
    | views::reverse
    | views::as_rvalue
    | ranges::to<std::vector>();
```

Depending on the values and predicate, this can overwrite memory of other objects leading to a core dump caused by undefined behaviour. The problem is subtle: `reverse` on a `filter_view` interacts badly with the cached `begin()`.

**2. Modifying elements through a filter iterator is UB.** If you write through a `filter_view` iterator and the modification changes whether the predicate holds, the behaviour is undefined - even when it appears to work:

```cpp
for (auto& entity : entities | views::filter(&Entity::isDead)) {
    entity.bringBackToLife();  // UB: invalidates the predicate
}
```

**3. Filter views don't work with `const`.** Passing a filtered range to a function taking `const auto&` fails to compile because `filter_view` caches `begin()` internally, requiring mutability.

### The fix

P3725 makes two changes:

First, `filter_view` now provides `const begin()` and `const end()` when the underlying range is an *input range* (not forward). Input ranges are single-pass, so no `begin()` caching is needed.

Second, the UB wording for modifying elements through filter iterators is relaxed. It's now well-defined as long as the predicate is not re-evaluated on the modified element.

### The practical pattern: `views::to_input`

The recommended fix for the dangerous compositions is to prepend `views::to_input` (which we covered in [part 1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3725r3.pdf)) before the filter:

```cpp
auto safe = v
    | views::to_input
    | views::filter(pred);  // well-defined, const-iterable
```

This downgrades the range to input-only, which prevents `reverse` from compiling (it requires bidirectional), turning the runtime UB into a compile-time error. It also enables const iteration since input ranges don't need `begin()` caching.

## Smaller ranges cleanups

C++26 also ships several defect resolutions and smaller fixes in the ranges space. None of these made it to their own post, but they're worth knowing about:

- **`std::basic_const_iterator` convertibility** ([P2836R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2836r1.html)) — `const_iterator` now follows its underlying type's convertibility, fixing cases where conversions that should work were rejected.

- **Removing the common reference requirement from indirectly invocable concepts** ([P2997R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2997r1.html)) - the `indirectly_readable` and related concepts had an overly strict `common_reference` requirement that rejected valid use cases with proxy iterators. Relaxed.

- **Algorithm function objects** ([P3136R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3136r1.html)) - standardizes `ranges::fold_left` and related algorithm function objects.

- **Converting contiguous iterators to pointers** ([P3349R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3349r0.html)) - allows implementations to convert contiguous iterators to pointers for better codegen in algorithms.

- **View iterator/sentinel constructors made private** ([P3059R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3059r2.html)) - user-defined constructors that were accidentally part of the public API are now private, preventing misuse.

## Conclusion

With parallel range algorithms, C++26 finally unifies the two halves of the standard algorithms story that have been split since C++17 and C++20. You can now compose views and run the result in parallel - as long as the range is random-access and sized. The `filter_view` safety improvements and `views::to_input` adaptor (covered in [part 1](https://www.sandordargo.com/blog/2026/10/14/cpp26-ranges-part-1-new-views)) work together to turn a class of silent UB into compile-time errors. And the handful of DR fixes tighten up edge cases that have bitten early adopters of C++20 ranges.

This wraps up the C++26 ranges mini-series. The range library isn't done evolving — there are plenty of features deferred to C++29 — but the C++26 additions fill the most painful gaps.

{% include connect-deeper.html %}
