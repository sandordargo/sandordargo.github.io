---
layout: post
title: "C++26: no more UB in lexing"
date: 2025-2-26
category: dev
tags: [cpp, cpp26, structuredbindings, attributes]
excerpt_separator: <!--more-->
---
If you ever used C++, for sure you had to face undefined behaviour. Even though it gives extra freedom for implementers, it's dreaded by developers as it may cause havoc in your systems and it's better to avoid it if possible.

Surprisingly, even the lexing process in C++ can result in undefined behaviour. Thanks to Corentin Jabot's work and his [P2621R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2621r3.pdf) that won't be the case anymore. As it was accepted as a defect report starting from C++98, in fact, you benefit from this already if you use a new enough compiler.

Truth be told, compilers didn't do any dangerous. They handled the below cases safely and deterministically. So this change is really about updating the standard and matching implementers' work.

Let's quickly see the three cases.

## Unterminated strings

```cpp
// unterminated string used to be UB
const char * foo = "
```

Who would have thought that an unterminated string or a character was UB?! Despite the permissive standard, all major compilers identified it as ill-formed. From now on, even the standard says so.

## Universal character names produced by macros

> *Universal character names (UCNs) in C++ are a way to represent Unicode characters in source code using a standardized syntax. They allow you to include characters that may not be directly representable in your source file encoding, such as non-ASCII characters or characters from other scripts. The syntax is either `\uXXXX` or `\uXXXXXXX` for 4- or 8-digit hexadecimal values.*

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

## Conclusion

In this article, we've reviewed the three kinds of undefined behaviour that [P2621R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2621r3.pdf) removes from lexing. Unterminated strings and characters are officially considered errors, and both sliced and concatenated universal character names are accepted now.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)