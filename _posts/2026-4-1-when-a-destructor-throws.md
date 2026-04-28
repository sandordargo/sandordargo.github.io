---
layout: post
title: "What happens when a destructor throws"
date: 2026-4-1
category: dev
tags: [cpp, exceptions, destructors, noexcept]
excerpt_separator: <!--more-->
---
Recently I wrote about [the importance of finding joy in our jobs on The Dev Ladder](https://devladder.substack.com/p/can-we-still-find-joy-in-programming). Mastery and deep understanding are key elements in finding that joy, especially now that generating code is cheap and increasingly done better by AI than by us.

Then a memory surfaced. I frequently ask during interviews — as part of a code review exercise — what happens when a destructor throws. Way too many candidates, even those interviewing for senior positions, cannot answer the question. Most say it's bad practice, but cannot explain why. Some say the program might terminate. Getting an elaborate answer is rare.

I'm not saying it's a dealbreaker, but it definitely doesn't help.

Let's see what actually happens.

## The role of a destructor

A destructor is the key to implementing the RAII idiom. RAII matters because after you acquire a resource, things might go south. A function might need to return early, or it might throw. Making sure resources are released is cumbersome, and the cleanest way to achieve it is to wrap both acquisition and release in an object that handles this automatically.

But what if the release itself is not successful?

Destructors have no return value, so error reporting is limited. Typical options include logging, storing error state, or (discouraged) throwing.

Why did I mark throwing an exception discouraged?

## What happens when an exception is thrown

When an exception is thrown, runtime stack unwinding starts.

First, automatic objects in the current scope are destroyed in reverse order, with their destructors executed.

If another exception is thrown during unwinding, `std::terminate` is called.

If a matching exception handler is found, execution continues there.

## What if a destructor throws with no other active exception?

Let's start with a simple example where no exception handling is ongoing:

```cpp
// https://godbolt.org/z/zn9b19jao
#include <iostream>

struct A {
    ~A() {
        std::cout << "Destructor\n";
        throw std::runtime_error("boom");
    }
};

int main() {
    try {
        A a;
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << "\n";
    }
}
```

Let's go step by step:
- we enter the `try` block
- `A a` is constructed
- `a`'s scope ends and `~A()` is called
- `A::~A()` throws

And then...

We have to stop for a second and recall an important rule:

> Since C++11, destructors are implicitly `noexcept(true)` unless declared otherwise or a base or member destructor can throw.

As an exception would leave our `noexcept` destructor, the `noexcept` guarantee is violated, so `std::terminate` is called. The `catch` block is never reached.

What if we want the destructor to be allowed to throw?

Let's update the example and mark the destructor throwable with `noexcept(false)`:

```cpp
// https://godbolt.org/z/KKhErsv6Y
#include <iostream>

struct A {
    ~A() noexcept(false) {
        std::cout << "Destructor\n";
        throw std::runtime_error("boom");
    }
};

int main() {
    try {
        A a;
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << "\n";
    }
}
```

In this case, the exception propagates normally and the `catch` block intercepts it.

So a destructor can throw as long as it's explicitly marked `noexcept(false)`.

I said *can*, not *should*. And there's a critical caveat.

## What if a destructor throws while another exception is active?

What if a destructor throws during stack unwinding? Let's update our example slightly:

```cpp
// https://godbolt.org/z/a1n75Tb4c
#include <iostream>

struct A {
    ~A() noexcept(false) {
        std::cout << "Destructor throwing\n";
        throw std::runtime_error("boom");
    }
};

int main() {
    try {
        A a;
        throw std::runtime_error("original");
    } catch (...) {
        std::cout << "caught\n";
    }
}
```

Let's go through what happens step by step:
- we enter the `try` block
- `throw std::runtime_error("original")` is thrown and stack unwinding starts
- as part of the unwinding, local objects are destroyed, so `A::~A()` is called
- `A::~A()` throws a second exception while the original is still active — `std::terminate()` is called

This termination is mandated by the C++ standard and no `catch` block is reached.

The rule exists because otherwise the runtime would need to track multiple simultaneous propagations, which would be complex and almost certainly ambiguous.

## Conclusion

When a destructor throws, the outcome depends on context.

If no other exception is active and the destructor is explicitly marked `noexcept(false)`, the exception propagates normally and can be caught. But this is the exception — in both senses of the word. Since C++11, destructors are implicitly `noexcept(true)`, so a throwing destructor will call `std::terminate` by default, bypassing any `catch` block entirely.

The real danger is the second scenario: a destructor throwing while stack unwinding is already in progress. Even with `noexcept(false)`, this always calls `std::terminate`. You cannot let an exception escape a destructor during unwinding — the standard simply does not allow it.

This is why the conventional wisdom holds: destructors should not throw. If resource release fails, the alternatives — logging the error, setting an error flag, or storing the failure state for later inspection — are far safer than propagating exceptions out of a destructor. The `noexcept(false)` opt-out exists, but it requires careful handling and should only be used when you can guarantee that the destructor is never called during stack unwinding.

Have you ever encountered a codebase that threw from destructors deliberately? How was the error handling structured?

## Connect deeper

If you liked this article, please
- hit on the like button,
- [subscribe to my newsletter](https://sandor-dargo.kit.com/e19f29b0a1)
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
