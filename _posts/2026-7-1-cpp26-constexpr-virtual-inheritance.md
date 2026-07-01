---
layout: post
title: "C++26: constexpr virtual inheritance"
date: 2026-7-1
category: dev
tags: [cpp, cpp26, constexpr, inheritance]
excerpt_separator: <!--more-->
---
`constexpr` has come a long way since its introduction in C++11. Back then, `constexpr` functions were extremely restricted and could essentially contain only a single `return` statement. C++14 relaxed those restrictions, allowing local variables, loops, and multiple statements. C++20 was another major milestone, bringing support for `constexpr` dynamic allocation, enabling types such as `std::string` and `std::vector` to become usable in constant evaluation, and allowing `constexpr virtual` functions. [C++23 further relaxed the rules](https://www.sandordargo.com/blog/2023/05/24/cpp23-constexpr) by permitting static local `constexpr` variables and `try` blocks inside `constexpr` functions. C++26 continues the trend, adding support for [exception handling](https://www.sandordargo.com/blog/2025/05/07/cpp26-constexpr-exceptions) during constant evaluation and several other improvements [in the language](https://www.sandordargo.com/blog/2025/04/23/cpp26-constexpr-language-changes) and [in the library](https://www.sandordargo.com/blog/2025/04/30/cpp26-constexpr-library-changes) we've already covered on this blog.

But somehow [P3533R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3533r2.html) by Hana Dusíková slipped under my radar - something I realized during her excellent talk on the state of `constexpr` at [ACCU On Sea](https://www.sandordargo.com/blog/2026/06/24/trip-report-accu-on-sea-2026). This paper removes one of the last remaining syntactical restrictions on `constexpr`: the prohibition of virtual inheritance.

<!--more-->

## A quick recap on virtual inheritance

I've written about [virtual inheritance before](https://www.sandordargo.com/blog/2020/12/23/virtual-inheritance), so I'll keep this brief.

Virtual inheritance solves the diamond problem. Consider a `Person` base class with `Student` and `Worker` both inheriting from it. If `TeachingAssistant` inherits from both `Student` and `Worker`, it ends up with two copies of `Person` — and any call to a `Person` member becomes ambiguous:

```cpp
struct Person {
    virtual ~Person() = default;
    virtual void speak() {}
};

struct Student: Person {};
struct Worker: Person {};

struct TeachingAssistant: Student, Worker {};

TeachingAssistant ta;
ta.speak(); // error: ambiguous
```

Virtual inheritance tells the compiler to keep only one shared instance of the base class, `Person`:

```cpp
struct Student: virtual Person {};
struct Worker: virtual Person {};

struct TeachingAssistant: Student, Worker {};

TeachingAssistant ta;
ta.speak(); // OK — only one Person subobject
```

The cost is a slightly larger object (extra vtable pointers) and more complex construction semantics — the most derived class is responsible for initializing the virtual base. That's why you should only use it when you actually need it. For the full picture, check out [the earlier article](https://www.sandordargo.com/blog/2020/12/23/virtual-inheritance).

## What the proposal changes

Before P3533R2, the standard explicitly forbade constructors and destructors of types with virtual bases from being `constexpr`. In fact, Clang went even further and rejected *all* member functions of such types as `constexpr`, not just constructors and destructors.

This meant code like the following was ill-formed:

```cpp
struct Common {
    unsigned counter{0};
};

struct Left : virtual Common {
    unsigned value{0};
    constexpr const unsigned& get_counter() const {
        return Common::counter;
    }
};

struct Right : virtual Common {
    unsigned value{0};
    constexpr const unsigned& get_counter() const {
        return Common::counter;
    }
};

struct Child : Left, Right {
    unsigned x{0};
    unsigned y{0};
};

constexpr auto ch = Child{}; // error before C++26
static_assert(&ch.Left::get_counter() == &ch.Right::get_counter());
```

With P3533R2 accepted into C++26, this code just works. You can construct objects with virtual bases at compile time, and `constexpr` member functions in such hierarchies are perfectly fine.

## Why this matters

You might think: virtual inheritance is rare enough at runtime, who needs it at compile time? The answer is the standard library.

`std::ios_base` — the foundation of the entire iostream hierarchy — uses virtual inheritance. That single fact has blocked the `constexpr`-ification of streams. And streams being non-`constexpr` has cascading effects. For example, `<chrono>`'s exception types `chrono::nonexistent_local_time` and `chrono::ambiguous_local_time` need stream formatting. Making `basic_istream_view` usable in `constexpr` range-based parsing also depends on this.

By lifting this restriction, P3533R2 unblocks a whole chain of library improvements.

There's a deeper point, too. Before this paper, the standard defined a concept called *constexpr-suitable*: a function was constexpr-suitable if it wasn't a coroutine and its class had no virtual bases. With virtual inheritance now allowed, the only remaining restriction is coroutines (which [P3367](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3367r0.html) is addressing for C++29). Once coroutines are handled too, every function will be constexpr-suitable and the term becomes meaningless.

In other words, `constexpr` is moving toward being a simple opt-in rather than a keyword guarded by a list of syntactical restrictions. The language is converging on a model where only the *evaluation* properties of code — defined in `[expr.const]` — determine what can be constant-evaluated.

## Conclusion

P3533R2 is a small proposal with outsized impact. It removes the last significant syntactical restriction on `constexpr` (aside from coroutines), and more importantly, it unblocks the constexpr-ification of streams and related library facilities.

It's also a sign of where C++ is heading. Each standard chips away at the things you *can't* do at compile time, and we're approaching a point where `constexpr` simply means "the compiler may evaluate this" — no asterisks, no exceptions lists. That's a cleaner language.

{% include connect-deeper.html %}
