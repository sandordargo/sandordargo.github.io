---
layout: post
title: "C++26: std::inplace_vector"
date: 2026-X-X
category: dev
tags: [cpp, cpp26, containers]
excerpt_separator: <!--more-->
---
Until C++26, the standard library had no vector-like container with compile-time fixed capacity whose element storage is guaranteed to live inside the container object itself. `std::vector` obtains storage through its allocator — typically the heap. `std::array` has a fixed size. If you need a dynamically-resizable array that never allocates — because you're on bare metal, because allocation latency is unacceptable, or because you need the data to live inside the object for serialization — there was no standard answer.

C++26 adds `std::inplace_vector<T, N>` in the new `<inplace_vector>` header, originally introduced by [P0843R14](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p0843r14.html) and subsequently refined by later C++26 papers (P3981R2, P4022R0, P3074R7). It's essentially `boost::static_vector` standardized: a vector-like container with a compile-time fixed capacity `N`, where all elements are stored inline within the object itself.

<!--more-->

## The basics

The API should feel familiar if you've used `std::vector`. You get `push_back`, `emplace_back`, `insert`, `erase`, `pop_back`, `clear`, random-access iterators, `operator[]`, `at()`, `front()`, `back()`, `data()` — the usual suspects. The key difference is that `capacity()` is `static constexpr` and always returns `N`:

```cpp
// https://godbolt.org/z/Te4bnrjjj
#include <inplace_vector>

std::inplace_vector<int, 8> v;
v.push_back(1);
v.push_back(2);
v.push_back(3);

assert(v.size() == 3);
static_assert(v.capacity() == 8);  // always 8, no matter what
assert(v[0] == 1);
```

Unlike `std::array`, a default-constructed `inplace_vector` doesn't require `T` to be default-constructible. An `array<T, N>` always stores `N` live objects; an `inplace_vector<T, N>` only has `size()` constructed elements — there's no pre-initialization of unused capacity slots.

## Three ways to handle overflow

What happens when you `push_back` into a full `inplace_vector`? Unlike `std::vector`, it can't just allocate more memory. P0843 gives you three tiers, depending on how much control you want:

### Throwing: `push_back` / `emplace_back`

The standard modifiers throw `std::bad_alloc` when the vector is full. The choice of `bad_alloc` (rather than `length_error`) was deliberate: existing code that catches allocation failures via `bad_alloc` handles `inplace_vector` overflow automatically.

```cpp
std::inplace_vector<int, 2> v = {1, 2};
v.push_back(3);  // throws std::bad_alloc — no room
```

One nice difference from `std::vector`: `push_back` returns a reference to the inserted element. `std::vector::push_back` returns `void` — an ABI compatibility issue that prevented the existing API from being changed (it was attempted for C++20 but backed out). Since `inplace_vector` is a new type, no such baggage exists.

### Fallible: `try_push_back` / `try_emplace_back`

If you don't want capacity exhaustion reported by an exception, the `try_*` variants return `std::optional<T&>`. On success, you get a reference to the new element. On failure, you get `nullopt` — and the argument is not consumed. (Note that construction of the element itself can still throw; what the `try_*` family avoids is the `bad_alloc` on capacity overflow.)

```cpp
// https://godbolt.org/z/66q794K14
std::inplace_vector<std::string, 2> v;
v.push_back("first");
v.push_back("second");

std::string s = "third";
auto result = v.try_push_back(std::move(s));
if (!result) {
    // s is NOT moved-from — still holds "third"
    assert(s == "third");
}
```

This "no effects on failure" guarantee is important. You don't lose your data just because the container was full.

### Unchecked: `unchecked_push_back` / `unchecked_emplace_back`

When you can prove there's room, the `unchecked_*` variants have a precondition that `size() < capacity()`. Violating it is undefined behaviour:

```cpp
std::inplace_vector<int, 100> v;
for (int i = 0; i < 100; ++i)
    v.unchecked_push_back(i);  // we know there's room
```

The precondition lets implementations avoid handling capacity exhaustion on this path, which can make these operations cheaper in performance-sensitive code — true to C++'s principle of not paying for what you don't use. On freestanding implementations — where the throwing `push_back` and `emplace_back` are not available — the `try_*` and `unchecked_*` families are particularly relevant.

## Trivially copyable

If `T` is trivially copyable (or `N` is 0), then `inplace_vector<T, N>` is itself trivially copyable. This enables bytewise transfer in environments where the relevant representation, ABI, and address-space assumptions are satisfied. `std::vector` can never be trivially copyable — it always contains a pointer to its storage.

This property propagates naturally from `T`'s special member functions: trivial copy constructor if `T` has one, trivial destructor if `T` has one.

## How it compares

| | `std::array<T, N>` | `std::inplace_vector<T, N>` | `std::vector<T>` |
|--|-----|-----|-----|
| Capacity | Fixed at `N` | Fixed at `N` | Dynamic |
| Size | Always `N` | 0 to `N` | 0 to `max_size()` |
| Storage | Inline | Inline | Via allocator (typically heap) |
| T default-constructible? | For default construction | No | No |
| Trivially copyable? | If `T` is | If `T` is | Never |
| Typical move/swap | O(N) | O(size) | O(1) |
| Allocator-aware? | No | No | Yes |

The move/swap cost is worth calling out: moving or swapping an `inplace_vector` moves or swaps every element, because there's no heap pointer to transfer. This is O(size), not O(1) like `std::vector`. For small `N` this doesn't matter; for large `N` it's something to keep in mind.

## Constexpr

`std::inplace_vector` is `constexpr`-capable in C++26, including for non-trivial element types — thanks to P3074R7 and P3726R2 which resolved the underlying union-lifetime issues. Whether a particular operation can be evaluated at compile time depends on whether the corresponding operations on `T` are valid during constant evaluation.

## Conclusion

`std::inplace_vector` fills the gap between `std::array` and `std::vector` that the standard library has had since the beginning. It gives you a dynamically-resizable container with no heap allocation, trivial copyability when possible, and three tiers of overflow handling to match your error-handling style. If you've been using `boost::static_vector` or a hand-rolled equivalent, this is the standardized replacement you've been waiting for.

{% include connect-deeper.html %}
