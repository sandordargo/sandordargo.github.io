---
layout: post
title: "C++26: std::is_within_lifetime"
date: 2026-2-18
category: dev
tags: [cpp, cpp26, unions, constexpr]
excerpt_separator: <!--more-->
---
When I was looking for the next topic for my posts, my eyes stopped on `std::is_within_lifetime`. Dealing with lifetime issues is a quite common source of bugs, after all. Then [I clicked on the link](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2641r4.html) and I read *Checking if a union alternative is active*. I scratched my head. Is the link correct?

It is — and it totally makes sense.

Let's get into the details and first check what [P2641R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2641r4.html) is about.

## What does `std::is_within_lifetime` do?

C++26 adds `bool std::is_within_lifetime(const T* p)` to the `<type_traits>` header. This function checks whether `p` points to an object that is currently within its lifetime during constant evaluation.

The most common use case is checking which member of a union is currently active. Here's a simple example:

```cpp
union Storage {
  int i;
  double d;
};

constexpr bool check_active_member() {
  Storage s;
  s.i = 42;

  // At this point, 'i' is the active member
  return std::is_within_lifetime(&s.i);  // returns true
}
```

In this example, after assigning to `s.i`, that member becomes active. The function `std::is_within_lifetime(&s.i)` returns `true`, confirming that `i` is within its lifetime. If we checked `std::is_within_lifetime(&s.d)` at this point, it would return `false` since `d` is not the active member.

## Properties and the name

The function has some interesting design choices that are worth discussing.

### It's `consteval` only

`std::is_within_lifetime` is `consteval`, meaning it can only be used during compile-time. You cannot call it at runtime.

This might seem limiting, but it's actually by design. The purpose of this function is to solve problems that exist specifically in the constant evaluation world. At runtime, you have other mechanisms available like tracking state with additional variables. The compiler doesn't maintain the same level of lifetime tracking information at runtime that it does during constant evaluation.

### Why a pointer instead of a reference?

The function takes a pointer rather than a reference, which might seem unusual for a query operation. The reasoning is straightforward: passing by reference can introduce complications with temporary objects and lifetime extension rules.

A pointer makes the intent explicit — you're asking about a specific memory location, not about a value or a reference that might be bound to various things. It's a cleaner semantic fit for what the function actually does.

### Why not "*is_union_member_active*"?

You might wonder why the feature has such a general name when the primary use case is specifically about unions. The answer is that the committee chose to solve the problem at a more fundamental level.

Instead of adding a union-specific check, they provided a general mechanism to query object lifetime. This means `std::is_within_lifetime` can potentially be useful in other constant evaluation scenarios where you need to know if an object exists.

The generalization makes the feature more powerful and future-proof, even if the primary use case today is checking union member activity.

## The original motivation

The proposal was driven by a very specific problem: implementing an `Optional<bool>` with minimal storage overhead. Imagine you want to create a type that can either hold a boolean value or be empty, using as little memory as possible.

Here's the challenge:

```cpp
struct OptBool {
  union { bool b; char c; };

  constexpr auto has_value() const -> bool {
    // How do we check if 'b' is the active member?
    // We can't just read it - that's undefined behavior if 'c' is active!
  }
};
```

At runtime, you can track the active member with a sentinel value in `c` — for example, using `2` to indicate "no value" since `bool` only uses `0` or `1`. But during constant evaluation, this becomes problematic. The compiler needs to know which union member is active without relying on runtime tricks.

Before C++26, there was simply no standard way to check this at compile time. With `std::is_within_lifetime`, the solution becomes straightforward:

```cpp
struct OptBool {
  union { bool b; char c; };

  constexpr auto has_value() const -> bool {
    if consteval {
      return std::is_within_lifetime(&b);
    } else {
      return c != 2;  // sentinel value
    }
  }

  constexpr bool value() const {
    return b;
  }
};
```

During compile-time evaluation, we use `std::is_within_lifetime` to check if `b` is the active member. At runtime, we fall back to checking the sentinel value. This gives us the best of both worlds: compile-time correctness and runtime efficiency.

## Compiler support

At the moment of writing (February 2026), none of the major compilers support this feature yet. As with many C++26 additions, we'll need to wait for implementations to catch up with the standard.

## Conclusion

C++26's `std::is_within_lifetime` is a focused addition that solves a real problem in constant evaluation: checking which union member is active without invoking undefined behavior. While the motivating use case came from implementing space-efficient optional types, the committee wisely chose to address the underlying problem more generally.

The function's design — taking a pointer, being consteval-only, and having a broad name — reflects careful consideration of both current needs and potential future applications. It's a small but well-designed piece that makes constexpr evaluation more practical and expressive.

## Connect deeper

If you liked this article, please
- hit on the like button,
- [subscribe to my newsletter](https://sandor-dargo.kit.com/e19f29b0a1)
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
