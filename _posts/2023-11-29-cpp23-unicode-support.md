---
layout: post
title: "C++23: Growing unicode support"
date: 2023-11-29
category: dev
tags: [cpp, cpp23, unicode, i18n]
excerpt_separator: <!--more-->
---
The standardization committee has accepted (at least) four papers which clearly show a growing Unicode support in C++23. Let's review what those papers cover.


## C++ Identifier Syntax using Unicode Standard Annex 31

In short, you won't be able to use emojis in identifiers. Personally, I didn't even know it was possible. Who on Earth would use such an identifier in production code?

```cpp
int 🕐 = 0;
```

In any case, I read out 3 issues that [P1949R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1949r7.html) fixes.

First, up until now, you could use certain emojis, but certain others you could not.

```cpp
int ⏰ = 0; //not valid
int 🕐 = 0;

int ☠ = 0; //not valid
int 💀 = 0;
```

Second, you could not use gender and skin tone modification. Including those is apparently complex, so the proposal is to remove all emojis.

Third, *"some words in some scripts, such as Persian, Malayalam, and Sinhala, require the use of zero-width joiners and non-joiners to render properly. These words will no longer be well-formed identifiers."*

This change is bad for funny conference slides, but good for everyone else. One less thing to pay attention to in a [code review](https://www.sandordargo.com/blog/2021/10/06/airy-code-reviews).

## Remove non-encodable wide character literals and multicharacter wide character literals

C++ currently allows writing wide character literals that cannot fit `wchar_t`. Interestingly these might compile today:

```cpp
wchar_t a = L'🤦‍♀️'; // \U0001f926
wchar_t b = L'ab';
wchar_t c = L'é'; // \u0065\u0301
```

On Clang, the first two failed to compile saying that "wide character literals may not contain multiple characters", Gcc simply emitted a warning indicating that "multi-character literal cannot have an encoding prefix".

As you can already see, until C++23 different interpretations could interpret these differently, now these become ill-formed thanks to [P2362R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2362r3.pdf).

## Delimited escape sequences

[P2290R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2290r3.pdf) brings us a new way to write octal, hexadecimal or universal character name escape sequences.

Before C++23, an octal escape sequence accepts 1 to 3 digits as arguments and it's introduced by a backslash `\`, from now on you use `\o{n...}` where inside the braces you use an arbitrary number of octal digits.

For hexadecimal characters, it used to be `\xn...` where `n...` is replaced by an arbitrary number of hexadecimal digits and also becomes `\x{n...}`. 

It's worth noting that both `\x{nnnn}` and `\o{nnnn}` must represent a value that can be represented in a single code unit of the encoding of the string or character literal they are a part of.

Similarly, for Unicode characters, you can use `\u{n...}`. Until now, it was mandatory to specify 4 hexadecimal digits after `\u` or 8 after `\U`. That meant that you must specify leading zeros all the time which is a bit cumbersome. It's not the case anymore with `\u{n....}`, you can omit the leading zeros.

```cpp
constexpr char o1 = '\123';
constexpr char o2 = '\o{123}';
static_assert(o1 == o2);

constexpr char h1 = '\x0f';
constexpr char h2 = '\x{0f}';
static_assert(h1 == h2);

constexpr char u1 = '\u0023';
constexpr char u2 = '\u{23}';
static_assert(u1 == u2);
```

## Named universal character escapes

Since C++11 we can use universal character names such as `U'\u0100'` which stands for the UTF-32 character literal with `U+0100` which has the name `{LATIN CAPITAL LETTER A WITH MACRON}`. We can also use characters in combination such as `u8"\u0100\u0300"` which combines `{LATIN CAPITAL LETTER A WITH MACRON}` with `{COMBINING GRAVE ACCENT}`.

Thanks to [P2071R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2071r2.html), we will be able to use directly the Unicode-assigned names instead of the Unicode point values. We'll have to introduce these names after the `\N` escape sequence:

```cpp
U'\N{LATIN CAPITAL LETTER A WITH MACRON}' // Equivalent to U'\u0100'
u8"\N{LATIN CAPITAL LETTER A WITH MACRON}\N{COMBINING GRAVE ACCENT}" // Equivalent to u8"\u0100\u0300"
``` 

[This proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2071r2.html) is a relatively long, but at the same time also an interesting read about different considerations and objections regarding this new feature. One of the concerns was the sheer size of the Unicode name database that contains the codes (e.g. `U+0100`) and the names (e.g. `{LATIN CAPITAL LETTER A WITH MACRON}`). It's around 1.5 MiB which can significantly impact the size of compiler distributions. The authors proved that a non-naive implementation can be around 300 KiB or even less.

Another open question was how to accept the Unicode-assigned names. Is `{latin capital letter a with macron}` just as good as `{LATIN CAPITAL LETTER A WITH MACRON}`? Or what about `{LATIN_CAPITAL_LETTER_A_WITH_MACRON}`? While the Unicode consortium standardized an algorithm called [UAX44-LM2](https://www.unicode.org/reports/tr44/tr44-24.html#UAX44-LM2) for that purpose and it's quite permissive, language implementors barely follow it. C++ is going to require an exact match with the database therefore the answer to the previous question is no, `{latin capital letter a with macron}` is not the same as `{LATIN CAPITAL LETTER A WITH MACRON}`. On the other hand, if there will be a strong need, the requirements can be relaxed in a later version.

If you want to know more, [read P2071R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2071r2.html).

## Conclusion

As we saw today, C++23 made some important steps towards supporting Unicode in a consistent way. We are not going to be able to use emojis in identifiers. From now on, non-encodable wide character literals and multicharacter wide character literals are ill-formed. Delimiter escape sequences will become more readable for octal, hexadecimal and Unicode sequences whose leading zeros might be omitted from now on. And still speaking about universal character escapes, instead of hexadecimal codes, from now on, we can also use the named versions. When I say from now on, of course, I mean C++23 and I assume compiler support that you can always check here on [C++ Reference](https://en.cppreference.com/w/cpp/compiler_support).

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!