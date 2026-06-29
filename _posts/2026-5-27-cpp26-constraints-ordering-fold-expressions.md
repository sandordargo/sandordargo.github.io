---
layout: post
title: "C++26: Ordering of constraints involving fold expressions"
date: 2026-5-27
category: dev
tags: [cpp, cpp26, concepts, templates]
excerpt_separator: <!--more-->
---
You have two overloads of `g()`. One requires `A<T>` for each element in a pack, the other requires `C<T>` — where `C` is a stricter concept that subsumes `A`. Both apply to the types you're passing. The compiler should pick the more constrained version. But instead it complains about an ambiguous call.

This is a limitation of how C++20 and 23 handle constraints that use fold expressions — fixed in C++26.

<!--more-->

## Constraint subsumption, briefly

Subsumption is the compiler's mechanism for ordering overloads by how constrained they are. If concept `C` is defined as `A && B`, then any type satisfying `C` also satisfies `A`. The compiler recognizes this and, when resolving an overload, prefers the more constrained one — no ambiguity.

[As we saw a few years back](https://www.sandordargo.com/blog/2021/05/05/cpp-concepts-and-logical-operators), subsumption has quirks. In particular, negated expressions wrapped in parentheses break it: `(!A<T>)` used in two separate places counts as two distinct atomic constraints, because subsumption is based on the syntactic source location of expressions, not their logical content.

The problem discussed here is similar in nature but involves fold expressions.

## The problem: fold expressions are opaque to subsumption

Consider this setup:

```cpp
#include <type_traits>

template <class T> concept A = std::is_move_constructible_v<T>;
template <class T> concept B = std::is_copy_constructible_v<T>;
template <class T> concept C = A<T> && B<T>;

template <class... T> requires (A<T> && ...) void g(T...);
template <class... T> requires (C<T> && ...) void g(T...);
```

`C<T>` subsumes `A<T>`. So the second overload is strictly more constrained: any pack of types satisfying `(C<T> && ...)` also satisfies `(A<T> && ...)`, but not necessarily the other way around.

The compiler should prefer the second overload. But in C++23 it cannot:

```cpp
// C++23: error: call to 'g' is ambiguous
g(std::string{}, std::vector<int>{});
```

The reason is that `(A<T> && ...)` and `(C<T> && ...)` are both treated as single atomic constraints by the constraint normalization rules. Subsumption checks whether one atomic constraint *is the same as* another — by source location — but it does not look *inside* a fold expression to check whether the pattern in one subsumes the pattern in another.

The fold expression forms a wall that the subsumption machinery can't see through.

## The solution: fold expanded constraints

[P2963R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2963r3.pdf) introduces a new category of constraints — **fold expanded constraints** — alongside the existing conjunctions, disjunctions, and atomic constraints.

A fold expanded constraint captures both the pattern inside the fold and the fold operator (`&&` or `||`). The subsumption rules are then extended: a fold expanded constraint `(P<Ts> && ...)` subsumes `(Q<Ts> && ...)` when:
- both expand the same parameter pack,
- both use the same fold operator, and
- `P<T>` subsumes `Q<T>` for a single type `T`.

With that change, the example above finally works in C++26:

```cpp
// C++26: selects the second overload, since C subsumes A
g(std::string{}, std::vector<int>{});
```

### Operators must match

One important constraint: the fold operators must be the same. A `(P<Ts> && ...)` constraint does not subsume `(P<Ts> || ...)`, and vice versa. The two forms make logically distinct claims — the `&&` variant requires `P` for *all* pack elements; the `||` variant requires it for *at least one*. They are incomparable for subsumption purposes.

## Connection to P2841

P2963R3 was originally derived from [P2841](https://www.sandordargo.com/blog/2025/08/20/cpp26-P2841), the proposal that introduced concept and variable-template template parameters. The connection is that P2841 opens the door to concept parameter packs — that is, packs of concepts rather than types.

When a fold expression's pattern contains an unexpanded concept template parameter pack (enabled by P2841), P2963R3 requires a special case: the constraint is fully decomposed into atomic conjunctions or disjunctions based on pack size, rather than treated as a single fold expanded constraint. This ensures that subsumption still works correctly when concepts themselves are passed as template arguments.

*At the time of writing, Clang 19 supports P2963R3.*

## Conclusion

Constraint subsumption has always had sharp edges — negations wrapped in parentheses, expressions with different source locations that mean the same thing but can't be compared. Fold expressions added another such edge: conceptually ordered constraints that the compiler treated as incomparable.

P2963R3 plugs that gap by making fold expanded constraints proper participants in subsumption. You can now write variadic overloads constrained by fold expressions, and the compiler will correctly order them by how constrained they are — the way you'd expect it to work all along.

{% include connect-deeper.html %}
