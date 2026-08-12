---
layout: post
title: "C++26: no more UB in lexing and preprocessing"
date: 2025-2-26
category: dev
tags: [cpp, cpp26, undefinedbehaviour, lexing]
excerpt_separator: <!--more-->
---
If you ever used C++, for sure you had to face undefined behaviour. Even though it gives extra freedom for implementers, it's dreaded by developers as it may cause havoc in your systems and it's better to avoid it if possible.

Surprisingly, even the lexing process in C++ can result in undefined behaviour. Thanks to Corentin Jabot's work and his [P2621R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2621r3.pdf) that won't be the case anymore. As it was accepted as a defect report starting from C++98, in fact, you benefit from this already if you use a new enough compiler.

Truth be told, compilers didn't do anything dangerous. They handled the below cases safely and deterministically. So this change is really about updating the standard and matching implementers' work.

Let's quickly see the three cases.

## Unterminated strings

```cpp
// unterminated string used to be UB
const char * foo = "
```

Who would have thought that an unterminated string or a character was UB?! Despite the permissive standard, all major compilers identified it as ill-formed. From now on, even the standard says so.

## Universal character names produced by macros

> *Universal character names (UCNs) in C++ are a way to represent Unicode characters in source code using a standardized syntax. They allow you to include characters that may not be directly representable in your source file encoding, such as non-ASCII characters or characters from other scripts. The syntax is either `\uXXXX` or `\UXXXXXXXX` for 4- or 8-digit hexadecimal values.*

If you wrote macros that expanded to universal character names, you risked undefined behaviour! In reality, all major compilers supported this use case properly, and from now on the standard defines this. 

```cpp
#define CONCAT(x, y) x ## y
int CONCAT(\, u0393) = 0; // UB: universal character name formed by macro expansion
```

By the way, have you ever used the `##` preprocessing token? I haven't, at least I don't remember. I haven't written a macro for the last few years.

So `##` is called the token-pasting operator and lets us join separate tokens into one single, just like in the above example. 

## Spliced universal character name

According to the standard, if universal character names are spliced across lines, that's also undefined behaviour.

```cpp
int \\ // UB : universal character name accross spliced lines
u\
0\
3\
9\
1 = 0;
```

While the above was UB, all major compilers supported it except for MSVC which considered it an error. With the acceptance of [P2621R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2621r3.pdf), it's officially considered well-formed.

I'm still wondering who and why would write such code.

## Update: preprocessing is never undefined either

*This section was added later, after [P2843R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2843r3.pdf) (Alisdair Meredith) was adopted for C++26 at the Sofia meeting in June 2025.*

P2621R3 above cleaned up undefined behaviour in lexing (translation phases 1–3). P2843R3 finishes the job by eliminating *all* remaining undefined behaviour in the preprocessor (phase 4). The paper's key insight is that the preprocessor simply transforms source code before compilation — there is no runtime, so "undefined behaviour" was never the right term. Most cases become ill-formed (with a required diagnostic), and a few remain ill-formed with no diagnostic required. Unlike P2621R3, this one is *not* a DR — it applies to C++26 only.

There are quite a few cases. Let me highlight the most interesting ones.

### Token pasting producing invalid tokens

When the `##` operator pastes two tokens and the result isn't a valid preprocessing token, that used to be undefined behaviour. Now it's ill-formed with a required diagnostic:

```cpp
#define DO_CONCAT(a,b) a##b
#define CONCAT(a,b) DO_CONCAT(a,b)
#define MINUS -

auto x = CONCAT(MINUS, word);  // pastes "-" and "word" → "-word" is not a valid token
// GCC and Clang already diagnose this; C++26 now requires a diagnostic
```

### Stringizing producing invalid strings

When the `#` operator produces something that isn't a valid string literal:

```cpp
#define TO_TEXT(a) #a
#define TEXT(a) TO_TEXT(a)
#define BACKSLASH \\

const char *x = TEXT(BACKSLASH);  // produces "\" — unterminated string
```

This is now ill-formed with a required diagnostic. The paper also fixes stringizing of raw string literals containing newlines — those newlines are now escaped as `\n` in the resulting string, resolving CWG1709.

### Redefining predefined macros

Using `#define` or `#undef` on predefined macros like `__cplusplus`, `__FILE__`, or `__LINE__` used to be undefined behaviour. Now it's ill-formed:

```cpp
#define __cplusplus 12345678L   // ill-formed
#undef __LINE__                 // ill-formed
```

Compilers already diagnose many of these cases, although the paper identifies several gaps in current implementations.

### Macros expanding to `defined`

When a macro expands to the `defined` operator in an `#if` directive, that used to be undefined behaviour. The paper initially proposed making it well-defined (all compilers already handled it), but following feedback at the Sofia meeting, it was kept as ill-formed, no diagnostic required. In practice, all major compilers accept it and evaluate `defined` correctly, so nothing changes for users — but the standard still doesn't bless it:

```cpp
#define MACRO defined
#if MACRO(MACRO)   // was UB, now IFNDR (compilers still accept it)
#endif
```

### The rest

The paper also covers: preprocessing directives appearing inside macro arguments (now ill-formed, with a required diagnostic), invalid `#include` forms after macro expansion (still ill-formed, no diagnostic required), invalid and out-of-range `#line` directives (now ill-formed), and defining keywords as macros (ill-formed with a required diagnostic, with the existing restriction moved from the library requirements into the core preprocessor specification). It resolves CWG1709 and seven open preprocessor issues, CWG2575–CWG2581. CWG2575 is only partially resolved: macro-generated `defined` becomes IFNDR rather than undefined behaviour.

Together with P2621R3, translation phases 1 through 4 — lexing and preprocessing — are now entirely free of undefined behaviour.

## Conclusion

In this article, we've reviewed the three kinds of undefined behaviour that [P2621R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2621r3.pdf) removes from lexing and the cases that [P2843R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2843r3.pdf) removes from preprocessing. Together, the entire front end of the compiler — from reading source characters to producing preprocessing tokens — is now free of undefined behaviour in C++26.

{% include connect-deeper.html %}
