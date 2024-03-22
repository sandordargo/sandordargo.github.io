---
layout: post
title: "C++23: Encoding related changes"
date: 2024-3-20
category: dev
tags: [cpp, cpp23, utf8, encoding]
excerpt_separator: <!--more-->
---
Today, we are going to go through a couple of changes introduced by C++23 where the common theme is encoding. These changes are all related to how the code we write will be encoded by the compiler. Often these papers don't introduce big changes, simply the common and desirable behaviour got standardized.

Yet, these are all important to have code that works as expected and that is portable between different platforms and compilers.

## Trimming whitespaces before line splicing

As of C++20, what's the output of the below piece of code ([Godbolt](https://godbolt.org/z/qq378vvKM))?

```cpp
int main() {
int i = 1
// \ 
+ 42
;
return i;
}
```

Without having seen how the code is formatted I would have said that it's `1 + 42`, so `43`. And actually, that's the real output. On MSVC. Not on other compilers, such as GCC and Clang.

These latter ones return `1`. The reason is that they trim the trailing whitespaces before handling `\` splicing. The new-line character following `\` is a whitespace, it's deleted, therefore `+ 42` ends up on the same line as `// \` and it becomes part of the comment.

Just for the record, both implementation strategies are valid.

[P2223R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2223r2.pdf) change this situation and set the GCC/Clang implementation strategy into ~~stone~~ standard. This might be a breaking one for code that is only compiled with MSVC, but it shouldn't happen on a big scale.

We might remember [Matt Godbolt saying earlier in one of his talks that one of the superpowers of C++ is backward compatibility](https://www.youtube.com/watch?v=5P7fnH9cCR0) and here the change potentially breaks code. At the same time, it's worth noting that implementation-defined behaviour would break here and not standardized behaviour.

The reason behind going in this direction is that many IDEs, code formatters and other tools already discard trailing whitespaces as the Google style guide and most of its derivations already forbid them.

## Removing mixed wide string literal concatenation

Adjacent string literals are concatenated.

```cpp
#include <iostream>

int main() {
     auto a = "Hello" "World";
     std::cout << a << '\n';
}
/*
HelloWorld
*/
```

In the above example, the value of `a` is *HelloWorld*. Fair enough. But what if the string literals have encoding prefixes, such as `L""`, `u8""`, `u""` or `U""`? As long as only one of the literals has such a prefix, the resulting concatenated string will have the same prefix. But if string literals involved in the concatenation have different prefixes, the behaviour is implementation-defined.

```cpp
#include <iostream>

int main() {
  { auto a = L"" + u""; }
  { auto a = L"" + u8""; }
  { auto a = L"" + U""; }

  { auto a = u8"" L""; }
  { auto a = u8"" u""; }
  { auto a = u8"" U""; }

  { auto a = u"" L""; }
  { auto a = u"" u8""; }
  { auto a = u"" U""; }

  { auto a = U"" L""; }
  { auto a = U"" u""; }
  { auto a = U"" u8""; }
}
```

All the major compilers issue an error.

And thanks to [P2201R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2201r1.html), that behaviour is getting standardized. All mixed-encoding prefixes are ill-formed.

## Consistent character literal encoding

As of C++20, *"the value of character literals in preprocessor conditional is not required to be identical to that of character literals in expressions"*.

`#if 'A' == '\x41'` and `if ('A' == 0x41)` might have different results. According to a survey involving more than 1300 open source projects available on *vcpkg*, the expectation is clear. The primary use case for these macros is to detect narrow literal encoding at compile-time. In fact, all the major compilers treat these literals as if they were encoded in narrow literals.

Therefore, [P2316R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2316r2.pdf), doesn't change what people expect from C++, it won't even change the implementation of all the major compilers, it simply adjusts the standard to real-life behaviour and expectations. With its adoption, the standard guarantees that the value of character literals in preprocessor conditional is identical to that of character literals in
expressions.

## Missing feature test macros for C++20 core papers

As it's pointed out in [P2493R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2493r0.html), some feature-test macros have been missing for the language or they have not been bumped as necessary. This paper proposed to bump `__cpp_concepts` from *201907L* to *202002L*, and for `__cpp_constexpr` a bump to *202110L* is coming from another paper.

Without these bumped values, compilers cannot know whether some features are available or not and as such, they cannot use them.

## Character sets and encodings

[P2314R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2314r4.html) introduces a few terminology changes and also a couple of behaviour changes. Changes that won't directly influence most people's life who code in C++. Let me try to share with you the gist of this paper.

The main change is that "*universal-character-names* are no longer formed in the translation phase". Instead, all Unicode input characters are retained throughout the translation.

For example, up until C++20, if you had a "stringizing" preprocessor operator such as `#define S(x) # X`, then both `S(Köppe)` and `S(K\u00f6ppe)` could have been translated into "K\\u00f6ppe", but with this paper, both translates into simply `Köppe`. Interestingly, all the major compilers have been already in line with [P2314R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2314r4.html), so no change to them.

That was the main behaviour "change" - which is not really a change, but rather an adjustment of the standard to reflect implementation behaviour.

Here are also the terminologies introduced by this paper:
- *translation character set*: the abstract character set used during translation; can represent the character equivalent of all valid universal-character-names
- *basic character set*: minimum character set needed to express C++ program source
- *basic literal character set*: minimum set of characters expressible by literals
- *ordinary / wide literal encoding*: compile-time encoding used for initializing string literal objects

## Support for UTF-8 as a portable source file encoding

From [P2295R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2295r6.pdf), I learned something I would have never thought of. In C++, it's impossible to write code that can be strictly considered portable.

It was a surprising fact to me, as portability always seemed an important goal when writing code for enterprises in C++. And now I learned from [P2295R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2295r6.pdf) that even though in practice you could write portable code, in theory, it was not the case.

The reason behind this is that "the set of source file character sets is implementation-defined, which makes writing portable C++ code impossible." Thanks to this paper, it will be mandatory to accept UTF-8 as an input format which will both increase general portability and ensure that Unicode-related features can be used widely.

It's worth noting that all the major compilers support UTF-8 as source file encoding and Clang only supports this file format.

The paper doesn't go further than mandating to accept UTF-8 encoded source files. It doesn't try to provide a standard way to detect the encoding of the files to be compiled. It also doesn't restrict in any way what encodings should be accepted. It also doesn't speak about the conservation of code point sequences during the translation, or in other words the formation of "*universal-character-names*", that's what the above-mentioned [P2314R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2314r1.html) is for.

Interestingly, today compilers validate (or not) non-UTF-8 inputs differently. MSVC emits a warning if you use some invalid UTF-8 characters even in your comments. Clang doesn't check comments but it emits errors if it finds invalid Unicode characters in string literals.

GCC supports different input character sets. For non-UTF-8 inputs it makes sure that they decode cleanly, otherwise, it raises an error. On the other hand, UTF-8 input is not decoded at all, its handling is inconsistent with others.

## Clarify the handling of encodings in localized formatting of chrono types

One last proposal that is related to encodings is [P2419R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2419r2.html). C++20 added formatting of chrono types with `std::format` but left unspecified what happens during localized formatting when the locale and literal encodings do not match.

Let's take the example directly from the published paper:

```cpp
std::locale::global(std::locale("Russian.1251"));
auto s = std::format("День недели: {}", std::chrono::Monday);
```

In this case, we can have a mismatch between the UTF-8 local encoding and the "Russian.1251". The standard hasn't specified what should happen in such scenarios until the acceptance of [P2419R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2419r2.html).

One option is to have *"День недели: Пн"*. In that case, everything is in UTF-8. The other option would be to have *"День недели: \xcf\xed"*, where "\xcf\xed" is in *Russian.1251* and it's not valid UTF-8. This is also called a "Mojibake" and is undesirable.


> Mojibake (文字化け) is a term in Japanese that translates to "character transformation" or "character corruption" in English. It refers to the phenomenon where text that is encoded or decoded incorrectly results in a display of garbled or unreadable characters. Mojibake is often seen when there is a mismatch between the encoding used to store or transmit text and the encoding expected by the software or system trying to interpret that text. 

With the acceptance of [P2419R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2419r2.html), implementations are allowed to do transcoding or substituting the locale so that the result is in a consistent encoding and there should be no mojibakes.

## Conclusion

In this article, we reviewed changes that are related to how the compiler encodes our code. We saw that there is less and less implementation-defined behaviour in that area. While the behaviour often doesn't change, it becomes standardized. We also saw that C++ has a growing Unicode support. It's nice to see how portability is getting better and better.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
