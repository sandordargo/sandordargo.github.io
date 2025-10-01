---
layout: post
title: "C++26: std::optional<T&>"
date: 2025-10-1
category: dev
tags: [cpp, cpp26, optional, reference]
excerpt_separator: <!--more-->
---
If you're a regular reader of my blog, you know I've been sharing what I learn about new C++ language and library features ever since C++20. You probably also read my [CppCon 2025 Trip Report](https://www.sandordargo.com/blog/2025/09/24/trip-report-cppcon-2025). And this post is where the two come together.  

At CppCon I attended a great talk by Steve Downey about [`std::optional<T&>`](https://cppcon2025.sched.com/event/27bOx/stdoptionaldyatdyagg-optional-over-references). Steve is the *father of optional references*—he co-authored [P2988R12](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2988r12.pdf) with Peter Sommerlad.  

Let's start with a little history. By now — at the end of 2025 — even C++17 feels like history. That's when `std::optional<T>` was introduced, giving us a way to represent "maybe a value" with value semantics, instead of relying on pointers. But `std::optional` in C++17 (and later) couldn't hold references — unless you wrapped them in `std::reference_wrapper`.  

C++26 finally fixes this gap.  

## What is `std::optional<T&>`?

`std::optional<T&>` has three key characteristics:

- Unlike `std::optional<T>`, it is **not an owning type**. It simply refers to an existing object.  
- It provides both reference and value-like semantics.  
- Internally, it behaves like a pointer to `T`, which may also be `nullptr`.  

This last point is interesting: while `optional<T>` can be seen as `variant<T, monostate>`, `optional<T&>` is closer to `variant<T&, nullptr_t>`. In practice, it's a safer alternative to a non-owning raw pointer.  

Does this mean one less reason to use raw pointers?  

Well, owning raw pointers have been discouraged since C++11. Non-owning raw pointers are still common for expressing "I don't own this." With C++26, `optional<T&>` might replace many of those cases. You’ll still need to check engagement, but at least it’s more expressive than sprinkling `nullptr` checks everywhere.  

## Key considerations

The design of `std::optional<T&>` aimed for the **least surprising and least dangerous behavior**. Let's walk through some of the decisions.  

### Assign or Rebind?

Consider this snippet (from Steve Downey’s CppCon talk):  

```cpp
Cat fynn;
Cat loki;
std::optional<Cat&> maybeCat1;
std::optional<Cat&> maybeCat2{fynn};
maybeCat1 = fynn;
maybeCat2 = loki;
```

What should these assignments do? Should they copy objects or rebind references?

If they were true assignments, `maybeCat2 = loki;` would copy `loki` into `fynn`. That would have been surprising and error-prone.

Instead, the committee decided that `operator=` for `optional<T&>` always rebinds the reference. At the end of the snippet:
- `maybeCat1` refers to `fynn`
- `maybeCat2` now refers to `loki`, not `fynn`.

This design makes the behavior consistent and avoids accidental copies.

### What about `make_optional()`?

`make_optional()` returns an `optional<T>`, not `optional<T&>`. Even if you pass a reference, it still creates an owning optional. This is intentional: allowing `make_optional<T&>` would lead to dangling references.

In practice, `make_optional<T&>` was mostly used in test cases. With C++26, you’ll need to construct `optional<T&>` directly if you want an optional reference.

*It's also worth noting that the main usage of `make_optional<T&>` are tests cases.*

### The question of constness

Should `optional<T&>` model shallow or deep const?

For a `const optional<T&>`, should `operator*()` and `operator->()` yield `T&` or `const T&`?

The designers chose shallow constness: dereferencing a `const optional<T&>` still gives you a non-`const T&`. If you need deep constness, you can use `optional<const T&>`.

### The least dangerous `value_or`

What about `value_or`?

For `optional<T>`, it returns a `T`. For `optional<T&>`, the safest design turned out to be the same: `value_or` returns a `T` (by value).

This avoids surprising reference semantics, and it supports common use cases like providing a literal fallback:

```cpp
auto name = maybeName.value_or("Anonymous"s);
```

Steve also mentions in the paper future proposals for free functions like `reference_or`, `value_or`, `or_invoke`, and `yield_if` for all optional-like types. Stay tuned — I'll probably cover those here soon.


## Conclusion

`std::optional<T&>` fills an important gap in the language. It gives us a safer and more expressive way to model maybe-a-reference, reducing our reliance on raw pointers and reference wrappers.

It's another step toward making code both more readable and less error-prone — two things I'm always happy to see in modern C++.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!  