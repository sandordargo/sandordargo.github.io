---
layout: post
title: "C++26: Reducing undefined behaviour"
date: 2026-7-29
category: dev
tags: [cpp, cpp26, undefinedbehaviour, safety]
excerpt_separator: <!--more-->
---
Nowadays, safety is in the focus of C++. Whether it's at committee meetings, conference talks, or hallway discussions, the topic keeps coming up. C++26 made some big steps forward in this area, and one recurring theme is the reduction of cases leading to undefined behaviour.

With the words of one of the presented proposals, *"undefined behaviour has all but unbounded potential for bad program behaviour, including security leaks, data corruption, deadlocks, and so on"*. The simple act of violating a precondition can silently turn a perfectly correct-looking program into one that exhibits any of these.

In today's article, we look at how and where C++26 addresses this. A couple of these changes we have already covered on this blog, but they are worth reiterating in this context. And there is one important change we haven't talked about yet. Let's start with that.

<!--more-->

## P3144R2: Deleting a Pointer to an Incomplete Type Should be Ill-formed

Before C++26, deleting a pointer to an incomplete type was undefined behaviour. The only exception was when the complete class type happened to have a trivial destructor and no class-specific deallocation function. In all other cases, the compiler simply could not know what to do.

Consider this example:

```cpp
// widget.h
struct Widget;  // forward declaration only

void deleteWidget(Widget* p) {
    delete p;  // What destructor should be called?
               // Is there a custom operator delete?
}
```

```cpp
// widget.cpp
struct Widget {
    ~Widget() { /* non-trivial cleanup */ }
    static void operator delete(void* p) { /* custom deallocation */ }
};
```

At the point of the `delete` expression, the compiler doesn't know whether `Widget` has a non-trivial destructor or a custom `operator delete` - the former is quite often the case, while the latter is relatively rare. It cannot determine at translation time whether the program has well-defined behaviour or undefined behaviour - that determination depends upon information typically available only to the linker.

[P3144R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3144r2.pdf) makes deleting a pointer to an incomplete class type ill-formed. Full stop. It's now a compilation error.

### The road to ill-formed

Interestingly, this wasn't the initial direction. Several alternatives were considered which are presented in the proposal. Let's enumerate a few of them:

- **Deprecation first**: The original plan was to deprecate such deletes and only make them ill-formed in a later standard. The fear was that going straight to ill-formed would face adoption resistance.
- **Require the compiler to do the right thing**: Force the implementation to generate correct destructor calls and deallocation even for incomplete types. A nice idea, but nobody knew how to implement it efficiently.
- **Don't call the destructor**: Define the behaviour as ending the object's lifetime without running its destructor. This was combined with the concept of *erroneous behaviour* (borrowed from [P2795R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2795r5.html)) and deprecation.
- **Ill-formed unconditionally**: Including for types with trivial destructors. This would break some existing code that currently works fine, thanks to Hyrum's Law.

After thorough analysis, the originally preferred solution was the deprecation path combined with erroneous behaviour. The idea was to preserve well-defined behaviour in all cases but mark the pattern as deprecated. The erroneous behaviour classification would give compilers better diagnostics, both at compile time and at runtime.

But it turned out that the committee was more courageous. The courage is based on the fact that all major compilers already warn on such deletes without even trying to determine whether the complete type would have a trivial destructor. In other words, in practice, developers were already being told not to do this. So the committee went for the simplest and most impactful choice: make it ill-formed.

If you have existing code that deletes through an incomplete type, you'll need to make the type complete at the point of deletion. This is the right thing to do anyway - it ensures the destructor actually runs.

## P2795R5: Erroneous Behaviour for Uninitialized Reads

We [covered this one in detail previously](https://www.sandordargo.com/blog/2025/02/05/cpp26-erroneous-behaviour), but it fits perfectly into the theme of reducing undefined behaviour, so let's recap.

Reading an uninitialized local variable used to be undefined behaviour. Starting from C++26, it becomes *erroneous behaviour* - a new category that means the behaviour is well-defined but incorrect. The compiler is recommended to diagnose it.

```cpp
void foo() {
    int d;  // d has an erroneous value
    bar(d); // erroneous behaviour, not UB anymore
}
```

Instead of the program silently doing anything (as UB permits), an uninitialized object is now initialized to an implementation-specific value. Reading it is a conceptual error that compilers are encouraged to diagnose through warnings or runtime errors.

If you intentionally want to leave a variable uninitialized for performance reasons, C++26 gives you the `[[indeterminate]]` attribute. But in that case, reading the variable before initialization is still undefined behaviour - you're explicitly opting into it.

The beauty of this change is that it works just by recompiling existing code. No manual code changes needed - the compiler will tell you where you forgot to initialize variables.

## P2748R5: Disallow Binding a Returned Reference to a Temporary

This one we also [discussed in a dedicated article](https://www.sandordargo.com/blog/2025/06/11/cpp26-no-binding-to-returned-reference-to-temporary). It turns a common dangling-reference pattern from undefined behaviour into a compilation error.

Before C++26, the standard said that the lifetime of a temporary bound to the returned value in a function return statement is not extended. The temporary is simply destroyed at the end of the full-expression in the return statement. This was a recipe for dangling references.

```cpp
auto&& f1() {
    return 42;  // was UB, now ill-formed
}

const double& f2() {
    static int x = 42;
    return x;   // implicit conversion to double creates a temporary
                // was UB, now ill-formed
}
```

C++26 replaces the vague "lifetime is not extended" wording with a clear rule: a return statement that binds the returned reference to a temporary expression is ill-formed. The code won't compile. This eliminates a whole class of subtle lifetime bugs at compile time rather than letting them manifest as mysterious crashes at runtime.

## P3471R4: Standard Library Hardening

The changes above are all on the language side. But C++26 also addresses safety on the library side through standard library hardening, which we [covered in a dedicated article](https://www.sandordargo.com/blog/2026/05/13/cpp26-library-hardening).

The idea behind hardening is simple: many standard library operations have preconditions that, when violated, lead to undefined behaviour. Accessing `vector[10]` when the vector has only 5 elements is undefined behaviour. So is calling `front()` on an empty `optional`, or dereferencing an invalid `span` index.

[P3471R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3471r4.html) standardizes a mechanism for implementations to check these preconditions at runtime. When a check fails, the program can trap or abort instead of silently corrupting memory. It covers containers like `std::vector`, `std::array` and `std::deque`, views like `std::span` and `std::string_view`, and wrappers like `std::optional` and `std::expected`.

What makes this approach practical is that it's opt-in and requires no code changes - you just recompile with hardening enabled (e.g. `-fhardened` in Clang and GCC). Real-world deployments report roughly 0.3% performance overhead while catching thousands of bugs. That's a remarkable trade-off.

## Conclusion

C++26 tackles undefined behaviour from multiple angles. On the language side, deleting a pointer to an incomplete type is now ill-formed, reading uninitialized variables becomes erroneous rather than undefined, and returning a reference to a temporary is a compilation error. On the library side, standard hardening gives implementations a standardized way to catch precondition violations at runtime with minimal overhead.

None of these changes require new programming paradigms or major rewrites. Most of them work simply by recompiling existing code with a C++26-conforming compiler. That's the kind of progress that matters: making existing code safer without asking developers to change how they think.

{% include connect-deeper.html %}
