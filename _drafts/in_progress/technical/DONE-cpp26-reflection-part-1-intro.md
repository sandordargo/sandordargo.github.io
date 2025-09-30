---
layout: post
title: "C++ Reflections What is it and why does it matter?"
date: 2025-X-X
category: dev
tags: [cpp, cpp26, concepts, templates]
excerpt_separator: <!--more-->
---
You've defined an `enum class` in C++—strongly typed, scoped, and safe. But the moment you want to print it as a string, you're stuck.

Do you write a `switch`? A `map`? A macro? There's no built-in `to_string()` for enums, and every solution feels like unnecessary boilerplate. Every single time.

This is exactly the kind of problem reflection promises to solve. It gives the compiler access to the names of your `enum` values — and lets you generate that `to_string()` automatically.

Let's dive into what C++ reflection is and why it could finally bring elegance to problems like this — and much more.

## What is reflection, anyway?

Reflection is the ability of a program to inspect and manipulate its own structure and metadata — such as types, functions, members, or attributes — from within the language itself.

C++ will also allow generating code based on this information. That's also known as **reflective metaprogramming**.

In practice, it means asking:
- "What fields does this struct have?"
- "What's the name of this member?"
- "What is the return type of this function?"
- "Does this type have a specific attribute?"

And then generating or injecting code based on the answers.

Before we go further, let's clarify the differences between two types of reflection: **static** and **runtime**.


| Feature                 | **Static Reflection** (Compile-Time)                 | **Runtime Reflection**                      |
|------------------------|-------------------------------------------------------|---------------------------------------------|
| **When it happens**    | During compilation                                    | While the program is running                |
| **Performance impact** | None at runtime                                       | Potential runtime overhead                  |
| **Usage**              | Code generation, validation, metaprogramming          | Plugins, scripting, runtime decisions       |
| **Examples**           | C++ proposals, Circle compiler, Rust macros   | Java `getClass()`, C# `Type.GetProperties()`|
| **Flexibility**        | Safer and faster, but less dynamic                    | Very dynamic, but harder to optimize        |


C++ reflection will be **compile-time reflection**, which means you'll be able to write type-safe logic that adapts to your data structures — without runtime cost.

In the rest of this series, we'll explore how this eliminates boilerplate and opens the door to more powerful generic programming.


## Why does C++ need reflection?

We started this post with a common pain point: printing enum values as strings. But there's much more.

We already started this article by a common pain point. Representing enum values as a string. But there is obviously much more. Let's look into some other problems.

The real issue is **repetition**.

Without reflection, we constantly retype the same boilerplate code:
- **Manual JSON serialization**: Writing `to_json()` and `from_json()` for every struct.
- **Visitor patterns**: Tedious overloads or `std::visit` wrappers.
- **UI/config binding**: Mapping fields to UI or config keys, duplicating names and types.

Take this simple struct:

```cpp
struct User {
    std::string name;
    int age;
};
```

To serialize it to JSON, you might write something like this:

```cpp
nlohmann::json to_json(const User& user) {
    return nlohmann::json{
        {"name", user.name},
        {"age", user.age}
    };
}
``` 

Now do that again. And again. And again.

But with reflection, it could be as simple as:

```cpp
nlohmann::json to_json(const User& user) {
    return reflect_to_json(user); // automatic, written only once
}
```

No more repeating field names. No more missed updates when a field changes. The compiler already knows the structure — reflection just lets you use that knowledge.

And this is only the beginning. Reflection can reduce boilerplate across debugging tools, schema generation, validation, and more.

## What's Happening in the Standard?

Reflection isn't new — languages like Java, C#, and Rust have had some form of it for years. But for C++, it's long been a missing piece.

That's changing with C++26.

[As Herb Sutter reported](https://herbsutter.com/2025/06/21/trip-report-june-2025-iso-c-standards-meeting-sofia-bulgaria/), at the June 2025 ISO C++ meeting in Sofia, the committee accepted seven proposals for compile-time reflection:
- [P2996R13](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r12.html) "Reflection for C++26": the basic foundation 
- [P3394R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3394r3.html) "Annotations for reflection" : the ability reflect additional attribute information
- [P3293R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3293r2.html) "Splicing a base class subobject": adding better support for treating base class subobjects uniformly with member subobjects
- [P3491R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3491r2.html) "define_static_{string,object,array}" adding functions that were split off from the main reflection paper P2996
- [P1306R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p1306r4.html) "Expansion statements" adding "template for" to make it easy to loop over reflection data at compile time.
- [P3096R12](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3096r9.pdf) "Function parameter reflection in reflection for C++26": adding support for reflecting function parameters.
- [P3560R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3560r1.html) "Error handling in reflection": enabling compile-time exception handling as the error handling model for reflection code.

This is a huge **first step**. Of course, more work is ahead—C++29 will likely expand reflection further — but C++26 lays a solid and usable foundation.

As with any language feature, the Committee moves carefully. Once something becomes part of C++, it's very hard to change. So the goal was to ship something useful and correct — even if not yet complete.


## What you'll learn in this series

This series will explore both the syntax and semantics of C++ reflection, as well as practical real-world use cases.

I'll keep it as realistic and hand-on as possible while
- reflecting on types and members
- building tools that we can further extend and use in real-life
- removing boilerplate from our code.

## Conclusion

Reflection is finally coming to C++ — and it's about time.

In this series, we'll learn how to use it to write cleaner, smarter code with less repetition. Next up, we'll set up a playground and reflect on our first struct.

Let's start reflecting. 🙂

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!