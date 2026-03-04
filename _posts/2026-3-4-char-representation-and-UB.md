---
layout: post
title: "Accessing inactive union members through char: the aliasing rule you didn't know about"
date: 2026-3-4
category: dev
tags: [cpp, unions, ub, aliasing]
excerpt_separator: <!--more-->
---
[I recently published an article on a new C++26 standard library facility, `std::is_within_lifetime`](https://www.sandordargo.com/blog/2026/02/18/cpp26-std_is_within_lifetime). As one of my readers, Andrey, pointed out, one of the examples contains code that seems like undefined behavior. But it's also taken — almost directly — from [the original proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2641r4.html), so it's probably not UB. And that's correct, it's not undefined behavior.

Let's first examine the example and the UB suspect, then dive into the fine print of C++ to explain why it's not what it seems to be.

## The suspect

Let's carefully examine the code below and focus on the `else` branch of the compile-time conditional.

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

Did it raise your eyebrows?

What happens if the active member of the union is `b`? We still make our comparison through `c`. But isn't accessing an inactive member of a union always undefined behavior?

## The fine print almost nobody knows about

From my already referenced article, [let's jump back to the original proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2641r4.html). It explicitly states that this is not UB, though it doesn't explain it in depth. However, it does reference some details about type similarity and certain exceptions:

> *If a program attempts to access the stored value of an object through a glvalue whose type is not similar to one of the following types the behavior is undefined:*
>
> *(11.1) the dynamic type of the object,*
>
> *(11.2) a type that is the signed or unsigned type corresponding to the dynamic type of the object, or*
>
> *(11.3) a char, unsigned char, or std::byte type.*

The rule states that accessing an object through a glvalue of an unrelated type is undefined behavior *unless* the glvalue type is the dynamic type, the signed/unsigned corresponding type, or a `char`, `unsigned char`, or `std::byte`.

In our case, the only exception that applies is the third one.

Let's consider the following case:

```cpp
OptBool ob(true);   // active member is bool b
ob.has_value();     // reads c
```

In this example, we're accessing the union storage through `c`, which is a `char` glvalue, while the active object is `bool`.

While accessing an inactive union member would normally be undefined behavior, the aliasing rule quoted above provides an exception. It explicitly allows reading any object's representation through a `char` (or `unsigned char` or `std::byte`). In other words, any object may be inspected via a `char` pointer or reference.

A `bool` is represented as either `0` or `1`, so comparing it to `2` — or any other `char` value — is perfectly valid and not UB.

On the other hand, reading `b` directly while `c` is the active member would be UB, because `bool` is not a character type listed in the exception above, and it's not similar to `char` either.

## Why character types matter

But why is talking about character types or bytes so important?

The morning after I wrote the first draft of this article, I stumbled upon an interesting section in [Patrice Roy's book *Memory Management in C++*](https://amzn.to/4l8fGzq). He explains that even though C++ tries to repurpose `char` as a representation of a character and uses `std::byte` to represent a byte, due to the language's C roots, a *"`char*` can alias any address in memory"* — it's really just a *"pointer to a byte"*.

That explains everything.

## Conclusion

When you see code that accesses an inactive union member through a `char`, your immediate reaction might be "that's undefined behavior!" And you'd usually be right — except when you're not.

The C++ standard contains a special exception to its strict aliasing rules. Character types (`char`, `unsigned char`, and `std::byte`) can alias any object in memory. This isn't just a quirk of the language; it reflects the fundamental nature of these types as representations of raw bytes rather than semantic values.

This exception is what makes the `OptBool` example from the `std::is_within_lifetime` proposal work. At runtime, we can safely read the union storage through a `char` member even when `bool` is active, because we're not interpreting the value — we're just inspecting its raw representation.

It's a small detail buried in the fine print of the standard. But it's these kinds of details that separate code that merely looks correct from code that actually *is* correct according to the language rules.

Have you encountered other surprising exceptions in the C++ aliasing rules?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
