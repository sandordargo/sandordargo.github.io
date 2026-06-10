---
layout: post
title: "C++26: Cleaning up string literals"
date: 2026-6-10
category: dev
tags: [cpp, cpp26, string literals]
excerpt_separator: <!--more-->
---
The two papers we are covering today are complementary in a philosophical sense. They both improve how string literals are handled in C++26. [P2361R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2361r6.html) tackles strings that are never evaluated — the ones that only exist at compile time. [P1854R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p1854r4.html) tackles evaluated string literals, making non-encodable characters ill-formed instead of implementation-defined. Let's start with the unevaluated side.

> To be precise, we are not only talking about C++26 here. The changes for evaluated contexts (P1854R4) are taken as a defect report with immediate effect starting from the first standardized version, C++98.

<!--more-->

## P2361R6: Unevaluated strings

Not all string literals in C++ end up in your compiled program. Some are used exclusively at compile time — the compiler reads them, acts on them, and then they're gone. They never make it into the binary.

These strings appear in the following contexts:

- `_Pragma` directives
- `#line` directives
- `[[nodiscard]]` and `[[deprecated]]` attributes
- `extern` linkage specifications
- `asm` statements
- [`static_assert` messages](https://www.sandordargo.com/blog/2025/01/01/cpp26-static-assert)
- Literal operator names

Since these strings are never evaluated at runtime, they don't need to be converted to the execution encoding or any encoding specified by a prefix like `L`, `u8`, `u`, or `U`. Despite this, before C++26, the standard didn't formally distinguish them from regular string literals. Compilers handled them inconsistently.

### What changes

P2361R6 introduces the concept of an *unevaluated string* and establishes clear rules:

- **No encoding prefix is allowed.** Writing `static_assert(false, L"bad")` or `static_assert(false, u8"bad")` becomes ill-formed.
- **The string is not converted to the execution encoding.** The compiler keeps the original sequence of characters for diagnostic purposes.
- **Among escape sequences, only `universal-character-name`s and `simple-escape-sequence`s (except `\0`) are permitted.** These are replaced by the corresponding Unicode codepoints. Numeric escape sequences (like `\x1B` or `\077`) and conditional escape sequences are ill-formed. Regular characters — including non-printable ones embedded directly in the source — are still allowed.

You might consider the last point too restrictive, but it makes sense. Since the compiler doesn't know which encoding to convert these strings to, numeric escape sequences have no meaningful interpretation. There's no way for them to denote a valid code unit in an unknown encoding. Simple escape sequences like `\n` or `\t` are fine because they map to well-known Unicode codepoints.

Despite the changes, how diagnostic messages are presented does not change. That remains a quality-of-implementation concern. The proposal explicitly does not restrict which Unicode characters can appear in unevaluated strings — only which escape mechanisms can be used. So non-printable characters, control characters, and invisible characters can still appear directly in the source text or via simple escape sequences. The compiler just has to figure out how to display them.

### Are these breaking changes?

Technically, yes. But a large survey of over 90 million lines of open source code found essentially no real-world code affected by these restrictions. The only uses of encoding prefixes on `_Pragma` or `static_assert` strings turned up in Clang's own test suite.

### Future directions

The paper explicitly notes that these changes don't prevent supporting constant expressions in `static_assert` or attributes in the future. One could imagine a grammar where:

```cpp
static_assert(some_condition, u8"explanation");
```

becomes valid again — not as an unevaluated string, but as a constant expression that happens to be a `u8` string literal. That would be the subject of a separate proposal.

## P1854R4: Making non-encodable string literals ill-formed

Now let's turn to strings that *are* evaluated — the ones that end up in your binary. Before C++26, if you put a character into a string literal that couldn't be represented in the literal's associated encoding, the behavior was implementation-defined. MSVC would silently use `?` as a substitute for unrepresentable characters. GCC would emit a diagnostic. Clang, which always uses UTF-8 for narrow literals, avoided the problem entirely.

This kind of implementation-defined behavior is dangerous. It can lead to silently incorrect programs — your string looks right in the source code, but the compiled binary contains something different.

[P1854R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p1854r4.html) makes this straightforward: if a character cannot be represented in the literal's associated encoding, the program is ill-formed. No more silent substitution.

This paper is part of a broader effort — alongside [P2362R3](https://wg21.link/p2362r3) and [P2621R3](https://wg21.link/p2621r3) — to make lexing less surprising when non-Latin1 characters appear in source code.

### What changes for string literals

Consider this scenario: your execution character set is ASCII, and you write:

```cpp
const char* greeting = "こんにちは";
```

Before C++26, MSVC would compile this but replace each character with `?`, producing `"?????"` in the binary. It does emit a warning for each character, but the build succeeds — leaving you with a corrupted string. With P1854R4, this becomes a compile error. The intent is clear: a program should either preserve the same sequence of abstract characters as in the source, or refuse to compile.

If you actually need a UTF-8 encoded string regardless of the execution encoding, you can use `u8`:

```cpp
const char8_t* greeting = u8"こんにちは";  // always valid, always UTF-8
```

### What changes for multicharacter literals

The paper also tightens the rules for multicharacter literals like `'abcd'`. These are already conditionally-supported and implementation-defined in their value, so the paper doesn't try to remove them. But it does address a visual ambiguity problem.

Some Unicode characters are composed of multiple code points that form a single visible grapheme. For example, `'é'` written as `e` followed by `COMBINING ACUTE ACCENT` looks like a single character but is actually two code points. This means what *appears* to be a character literal is actually a multicharacter literal — with all the implementation-defined weirdness that implies.

P1854R4 requires that each element in a multicharacter literal must be representable as a single code unit in the ordinary literal encoding. This excludes combining characters and multi-byte code points, which eliminates the visual ambiguity. You can still write `'abcd'` (each character fits in one code unit), but accidentally creating a multicharacter literal from what looks like a single character is now caught at compile time.

### Impact on C

This makes a behavior that is implementation-defined in C ill-formed in C++. GCC already exhibited the behavior proposed by this paper across all language modes, so the practical impact on cross-language code is minimal.

## Conclusion

Both P2361R6 and P1854R4 are about the same fundamental idea: string literals in C++ should behave predictably. Unevaluated strings shouldn't pretend to have an encoding they'll never use. Evaluated strings shouldn't silently corrupt characters they can't represent.

These aren't flashy features. You won't rewrite any code to use them. But they close off subtle sources of bugs and portability issues that have existed since the earliest days of C++. Combined with related work like P2362R3 and P2621R3, they represent a steady effort to make the C++ lexer less surprising — and that's exactly the kind of improvement the language needs.

## Connect deeper

If you liked this article, please
- hit on the like button,
- [subscribe to my newsletter](https://sandor-dargo.kit.com/e19f29b0a1)
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
