---
layout: post
title: "C++26: More function wrappers"
date: 2026-5-20
category: dev
tags: [cpp, cpp26, functional, typeerasure]
excerpt_separator: <!--more-->
---
C++26 continues to fill the gaps in our type-erased callable wrapper story. We already had `std::function` since C++11 and `std::move_only_function` since C++23, but there were still missing pieces. Now we're getting two new additions: `std::copyable_function` and `std::function_ref`.

<!--more-->

## What's wrong with `std::function`?

`std::function` has served us well, but it has two well-known issues. First, it can [add significantly to binary size](https://www.sandordargo.com/blog/2023/04/05/binary-size-and-templates). Second, and more fundamentally, it has a **const-correctness defect**. Its `operator()` is declared `const`, but it can invoke a non-const `operator()` on the stored callable:

```cpp
// https://godbolt.org/z/9YxcW5eqK

#include <functional>
#include <iostream>

struct Counter {
    int counter = 0;
    void operator()() {
        ++counter;
        std::cout << "modifying state: " << counter << '\n';
    }
};

int main() {
    const std::function<void()> f = Counter{};
    f();  // OK (!)
}
```

This defect is baked into the original design and cannot be fixed without breaking the ABI.

C++23 introduced [`std::move_only_function`](https://www.sandordargo.com/blog/2024/01/31/cpp23-likes-to-move-it) ([P0288R9](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0288r9.html)) which fixed the const-correctness issue and added support for cv/ref/noexcept qualifiers. But as its name suggests, you can't copy it. We still needed a wrapper that is both copyable *and* const-correct.

## `std::copyable_function`

[P2548R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2548r6.pdf) fills exactly that gap. Its design follows `std::move_only_function` closely, adding a copy constructor and copy assignment operator, and requiring that stored callables are copy-constructible.

The key difference from `std::function` is that the qualifier in the signature directly controls how `operator()` is declared:

```cpp
// https://godbolt.org/z/orxPK9Gn9

#include <functional>

struct Counter {
    int count = 0;
    int operator()() { return ++count; } // non-const
};

int main() {
    // copyable_function<int()> means operator() is non-const
    std::copyable_function<int()> f = Counter{};
    f(); // OK, non-const call

    // If you want const, you say so explicitly:
    // std::copyable_function<int() const> g = Counter{}; // ERROR!
    // Counter::operator() is not const-qualified

    std::copyable_function<int() const> h = []{ return 42; }; // OK
}
```

Just like `move_only_function`, it supports the full range of qualifier combinations -- `const`, `noexcept`, lvalue/rvalue ref-qualifiers, and any combination thereof. This is a significant improvement over `std::function`, which supports none of these.

A `copyable_function` can be implicitly converted to a `move_only_function`, but not the other way around. Like `move_only_function`, it omits the rarely useful `target()` and `target_type()` member functions.

## `std::function_ref`

[P0792R14](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p0792r14.html) adds a different kind of wrapper: a **non-owning, type-erased callable reference**. Think of it as `std::string_view` for callables.

```cpp
// With function pointer -- too restrictive, no lambdas with captures:
payload retry(size_t times, payload(*action)());

// With template -- works but bloats binary, must be in header:
template<class F>
payload retry(size_t times, F&& action);

// With function_ref -- lightweight, non-owning, no allocation:
payload retry(size_t times, std::function_ref<payload()> action);
```

It has **reference semantics**: there is no default constructor, no `operator bool`, and no `nullptr` comparison. A `function_ref` always refers to a valid callable. Every specialization is trivially copyable, meaning it can be passed in registers.

Assignment from non-function types is deleted to prevent dangling references - since `function_ref` doesn't own the callable, assigning a temporary would be a bug.

It supports `const` and `noexcept` qualifiers but not ref-qualifiers, since ref-qualifiers on a reference type wouldn't make sense.

A follow-up paper, [P3961R1](https://isocpp.org/files/papers/P3961R1.html), fixes double indirection when constructing a `function_ref` from another `function_ref`, and allows assigning a `noexcept`-qualified `function_ref` to a non-`noexcept` one - just as you'd expect from plain function pointers.

## Choosing the right wrapper

With four callable wrappers now available, here's a quick guide:

- **`function_ref`**: For callback parameters. Non-owning, zero overhead.
- **`move_only_function`**: For storing callables when you don't need to copy them. Task queues, deferred execution.
- **`copyable_function`**: For storing callables when you need copies. The modern `std::function` replacement.
- **`std::function`**: Legacy. Avoid in new code.

## Conclusion

C++26 completes the type-erased callable wrapper picture. `std::copyable_function` gives us what `std::function` should have been from the start: a copyable wrapper with correct const semantics. `std::function_ref` fills the non-owning niche, offering a lightweight, zero-allocation alternative for callback parameters. Together with `std::move_only_function`, there's no longer a reason to reach for `std::function` in new code.

{% include connect-deeper.html %}
