---
layout: post
title: "C++23: Preprocessing directives"
date: 2022-9-7
category: dev
tags: [cpp, cpp23, preprocessing]
excerpt_separator: <!--more-->
---
The ISO Committee accepted two proposals for C++23 related to preprocessing directives. [P2334R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2334r1.pdf) is about introducing `#elifdef` and `#elifndef` and [P2437R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2437r1.pdf) is introducing `#warning.`

## What is a preprocessing directive?

First, let's discuss what are preprocessing or prepocessor directives. They can be included both in header and implementation files and they start with a `#` sign. Such lines are examined and resolved before the compilation starts and as the name suggests that is done by the preprocessor. Such directives are only one line long and end with a newline, so no semicolon is required. In case you want longer directives, you have to end the line with `\` in order to signal to the preprocessor that you'd continue the definiton on the next line. 

If you think about simple one-liner preprocssing directives, you can think about the `#include` statements. What happens for #include statements is that they are basically textually replaced by files being included.

Another (in)famous usage of preprocessor directives is macros! With the directive `#define` we can introduce shortcuts for literals, functions, classes, whatever. The problem is that again, it's just a textual replacement before the compilation starts. Then if you face with a syntax error or if you have to debug, you have quite a difficult job because what was compiled was textually different from what you have in your source code. But enough on why macros are bad. [Read more here.]()

If you are looking for an example for a multiline directive, it's usually a macro defined with `#define` such as 

```cpp
#define square(x) \
      ((x)        \
        *         \
      (x))        \
```


## `#elifdef` and `#elifndef`

C++23 is introducing `elifdef` and `elifndef`. Interestingly this is coming from WG14, so the ISO work group working on the C language. These new directives are going to be part of C23, and the C++ working group (WG21) decided to adopt these changes in order to avoid preprocessor incompatibilities with C.

In any case, it's going to help save some keystrokes, and it reads relatively well.

Probably you've seen `#ifndef` at least in some header guards. `#ifndef identifier` serving a shortcut for `#if !defined(identifier)` and there is also `#ifdef identifier` is a shorthand for `#if defined(identifier)`. In the C++ language we don't only have `if`, but there is also `else if` for supporting multibranch conditionals. It's similar in the preprocessor directives world, where we don't only have `#if`, but also `#elif`. Until now, they had no similar shorthands, but C23 and C++23 makes life easier for those who use these multibranch conditional preprocessor directives. And in fact, not just easier but also more symmetric and readable.

There is very little reason to have shorthands for one and not the other.

## `#warning`

As I explained in [C++23: Narrowing contextual conversions to bool](https://www.sandordargo.com/blog/2022/06/15/cpp23-narrowing-contextual-conversions-to-bool), a new standard doesn't always change the compiler implementations. Sometimes, a new standard just gets closer to existing implementations. 

It's the case with the introduction of `#warning` to the standard. It had been already implemented by all the major compilers, and now both the C and the C++ standard is going to adopt it.

So what does `#warning`? It invokes diagnostic message by the preprocessor, without stopping the translation. That's the difference between `#error` and `#warning.` The former stops the translation, while the latter does not.

Its usage could look like this:

```
#ifndef FOO 
#warning "FOO defined, performance might be limited"
#endif
```

## Conclusion

It might be a bit suprising, but C++23 (and C23) introduces changes to the preprocessor directives. With the introduction of `#elifdef` and `#elifndef` the language becomes more round as we already had `#ifdef` and `#ifndef` and with the standardization of `#warning`, the standard comes closer to the compiler implementations.

How often do you use preprocessor directives apart from `#include` and header gurads?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
