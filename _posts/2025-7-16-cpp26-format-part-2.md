---
layout: post
title: "C++26: std::format improvements (Part 2)"
date: 2025-7-16
category: dev
tags: [cpp, cpp26, format]
excerpt_separator: <!--more-->
---
In [Part 1](https://www.sandordargo.com/blog/2025/07/09/cpp26-format-part-1), we explored the improvements C++26 brings to `std::format` — from better `to_string` behavior to compile-time safety checks. In this part, we look at runtime formatting, defect fixes, and support for new types like `std::filesystem::path`.

## Runtime format strings

[P2216R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2216r3.html) brought quite some improvements to `std::format`, including compile-time checking for format strings. Sadly, in use cases where format strings were only available at runtime, users had to go with the type-erased formatting version, [std::vformat](https://en.cppreference.com/w/cpp/utility/format/vformat.html):

```cpp
std::vformat(str, std::make_format_args(42));
```

Using two different APIs is not a great user experience, moreover,  `std::vformat` was designed to be used by formatting function writers and not by end users. In addition, you might run into undefined behaviour, detailed in the next section.

To overcome this situation, [P2918R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2918r2.html) adds [`std::runtime_format`](https://en.cppreference.com/w/cpp/utility/format/runtime_format.html) so you can mark format strings that are only available at run-time. As such you can opt out of compile-time format strings checks. This makes the API cleaner and the user code will read better as it shows better the intentions. 

```cpp
// Before:
std::vformat(str, std::make_format_args(42));

// After:
std::format(std::runtime_format(str), 42);
```

This change is already available in GCC 14 and Clang 18.

## DR20: `std::make_format_args` now accepts only lvalue references instead of forwarding references

[P2905R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2905r2.html) fixes unintended consequences of *[P2216R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2216r3.html) std::format improvements*. It offered checking format strings at compile-time (as we've just seen), so in use cases where format strings were only available at runtime, users had to go with the type-erased formatting version: [std::vformat](https://en.cppreference.com/w/cpp/utility/format/vformat.html).

The problem is that in innocent-looking code, like below, users face undefined behaviour as format arguments store references to temporaries which are destroyed before use:

```cpp
std::string str = "{}";
std::filesystem::path path = "path/etic/experience";
auto args = std::make_format_args(path.string());
std::string msg = std::vformat(str, args);
```

The fix is that `std::make_format_args` should take lvalue references instead of forwarding references, rejecting code like the above. It's been already implemented in `fmt` and this is knowingly and deliberately a breaking change. Luckily, this defectous feature hasn't been widely used.

This fix is already available in all the three major compilers, GCC 14, Clang 18 and MSVC 19.40.

## DR20: Fix formatting of code units as integers

[P2909R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2909r4.html) fixes a defect report. An earlier proposal introduced a bug into how `char`s are formatted. Whether `char` is signed or unsigned is implementation defined and `std::format` (through `std::to_chars`) always promotes a `char` to an `int` -  which is always signed.

As `char` is always used as a code unit type in `std::format` (and in other text processing facilities) - and a sometimes signed integer output has been surprising to the users. Here is an example:

```cpp
for (char c : std::string("🤷")) {
  std::print("\\x{:02x}", c);
}

/*
output is either this
\xf0\x9f\xa4\xb7

or this
\x-10\x-61\x-5c\x-49
*/
```

The fix proposed by the author, [Victor Zverovich](https://vitaut.net/), is to always convert a character type to the unsigned version of it when it's getting formatted. Though it's not the goal, it results in the same behaviour as `printf` has.

This fix is already available in all the three major compilers, GCC 13.3, Clang 18 and MSVC 19.40.

## `std::formatter<std::filesystem::path>`

Thanks to [P2845R8](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2845r8.html)m we will be able to print our standard paths nicely formatted.

Wasn't that available already?

Not yet. Although [P1636](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1636r2.pdf) already proposed formatters for library types, `std::filesystem::path` was removed because of some issues which are solved by now by the current proposal.

The previously proposed formatter always printed paths quoted. That is not always suitable for paths, notably when the path is a multiline one:

```cpp
std::cout << std::format("{}", std::filesystem::path("multi\nline"));
/*
The below output is not a valid C++ string
"multi
line"
*/
```

In the end end, a formatter is being added as per [P2286](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2286r8.html#filesystempath), and by default it doesn't print paths quoted. But it also adds a "debug format specifier" where control characters are escaped and the output is quoted.

```cpp
auto p = std::filesystem::path("/usr/bin");
std::cout << std::format("{}", p);
// output:
// /usr/bin

auto p = std::filesystem::path("multi\nline");
std::cout << std::format("{}", p);
// output:
// multi
// line
auto p = std::filesystem::path("multi\nline");
std::cout << std::format("{:?}", p);
// output: 
// "multi\nline"
```

The other problem was UTF-8 encoding on certain platforms when intermediary conversion are performed. The problem has been earlier solved by [P2093R14](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2093r14.html) which is now applied here as well. As a result paths with UTF-8 as a literal encoding will be printed as one would expect.

This change is not yet available in any of three major compilers.

## Conclusion

C++26 makes `std::format` more robust with safer, and cleaner API. The changes might look small in isolation, but together, they significantly improve the day-to-day developer experience.

If you missed Part 1, check it out [here](https://www.sandordargo.com/blog/2025/07/09/cpp26-format-part-1).

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!