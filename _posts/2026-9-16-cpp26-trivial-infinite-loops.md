---
layout: post
title: "C++26: Trivial infinite loops are no longer undefined behaviour"
date: 2026-9-16
category: dev
tags: [cpp, cpp26, undefinedbehaviour]
excerpt_separator: <!--more-->
---
Let's start with a question! Is this program well-defined?

```cpp
int main() {
    while (true)
        ;
}
```

If you said yes, you'd be wrong — at least before C++26. A `while (true);` loop with no side effects used to be *undefined behaviour*. Compilers were free to assume it terminates, and some — Clang in particular — would optimize it away entirely, with [spectacular consequences](https://godbolt.org/z/WYMxxeW1T):

```cpp
// https://godbolt.org/z/WYMxxeW1T
#include <iostream>

int main() {
    while (true)
        ;
}

void unreachable() {
    std::cout << "Hello world!" << std::endl;
}
```

In Clang, [this prints "Hello world!"](https://godbolt.org/z/WYMxxeW1T). The compiler removes the infinite loop, `main` falls through, and the linker-placed `unreachable()` function executes. This is not a compiler bug — it's just UB, still better than nasal demons.

Recently, I wrote about [how C++26 reduces undefined behaviour](https://www.sandordargo.com/blog/2026/07/29/cpp26-reduces-undefined-behaviour), covering changes like erroneous behaviour for uninitialized reads and making incomplete-type deletes ill-formed. I completely forgot about this one. I only realized while preparing for [an upcoming CppCon talk on C++26 features](https://cppcon2026.sched.com/event/2RT3x/the-real-story-of-c++26-beyond-the-headline-features) — so here it is now.

C++26 fixes this with [P2809R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2809r3.html). Trivial infinite loops are now well-defined. The mentioned proposal was also accepted as a defect report, so implementations may apply the fix to earlier C++ modes as well. That is why you might not be able to reproduce the old behaviour on a recent compiler even in C++20 mode.

<!--more-->

## How did we get here?

The story starts with the forward progress guarantee, introduced in C++11 alongside threading support. The standard says ([intro.progress]) that *the implementation may assume any thread will eventually do one of the following: terminate, call a library I/O function, access a volatile glvalue, or perform a synchronization or atomic operation*.

A `while (true);` loop does none of those things. Under the pre-C++26 forward-progress rules, an execution that remains in such a loop forever has undefined behaviour. The optimizer can therefore assume that execution never gets stuck there, which enables transformations that remove the loop and mark the path as unreachable.

The funny bit is that C got this right. C++11 and C11 both introduced forward-progress rules, but C included one more rule: loops whose controlling expression is a *constant expression* may **not** be assumed to terminate. So `while (1);` is well-defined in C11 and ever since.

C++ never adopted that extra rule. The result was the unnecessary divergence just described, but let's repeat it: `while (1);` was well-defined in C but undefined behaviour in C++.

But why would anyone write `while (true);` in the first place?

What I found is that this is common in embedded and kernel code as a *halt-on-error pattern*. When a fatal error occurs and there's no operating system to exit to, you simply stop:

```cpp
if (hardware_init_failed()) {
    log_error("fatal: hardware init failed");
    while (true)
        ;  // halt — there's nothing left to do
}
```

This is not simply a common pattern on bare metal — it was also undefined behaviour in C++. The consequences aren't theoretical. When the optimizer removes the loop, execution falls through into whatever code the linker placed after it — as the "Hello world!" example at the top of this article demonstrates. In an embedded system, that means a fatal error handler doesn't actually halt the device. The hardware keeps running in a corrupt state, executing whatever instructions happen to follow. In security-critical code, that's a real vulnerability.

## What C++26 changes

C++26 doesn't simply copy C's rule, though. That approach was considered and rejected. C protects a much broader set of loops — broadly, loops whose controlling expression is a constant expression — which could inhibit useful optimizations. Instead, P2809R3 defines a deliberately narrow category: the **trivial infinite loop**. It's defined by two conditions:

1. The loop must be a **trivially empty iteration statement** — meaning its body is literally empty (`;` or `{}`). Any non-empty statement in the body, even a meaningless expression statement such as `"a string";`, disqualifies it.

2. The controlling expression must be a **constant expression** that evaluates to `true`. For a `for` loop with no condition, `true` is implicit.

When both conditions are met, the loop body is replaced with a call to `std::this_thread::yield()`. This gives execution of the loop the forward-progress semantics it previously lacked.

Here's what qualifies and what doesn't:

| Code | Trivial infinite loop? |
|------|----------------------|
| `while (true);` | Yes |
| `for (;;);` | Yes |
| `do {} while (true);` | Yes |
| `constexpr bool go = true; while (go);` | Yes — `go` is a constant expression |
| `while (true) { "a string"; }` | No — body contains a statement |
| `while (true) if (done) break;` | No — body is not empty |
| `while (true) if constexpr (false) break;` | No — doesn't match the syntax of a trivially empty iteration statement |
| `bool done = false; while (!done);` | No — not a constant expression |

The change also updates the forward progress guarantee itself: a thread may now "continue execution of a trivial infinite loop" as one of the things it's assumed to eventually do. The optimizer can therefore no longer treat a trivial infinite loop as undefined behaviour and assume that execution continues past it.

## The freestanding caveat

On freestanding implementations, it is implementation-defined whether the replacement with `std::this_thread::yield()` occurs at all. That's important for bare-metal systems: turning a deliberate halt loop into a cooperative yield could introduce behaviour the programmer never intended.

## Conclusion

`while (true);` being undefined behaviour was one of those C++ facts that surprised everyone who heard it. It was an unnecessary divergence from C, it broke real embedded code, and compilers genuinely exploited it. C++26 fixes it — trivial infinite loops are now well-defined, and the compiler can no longer optimize them away.

{% include connect-deeper.html %}
