---
layout: post
title: "C++26: std::format improvement (Part 1)"
date: 2025-7-9
category: dev
tags: [cpp, cpp26, format]
excerpt_separator: <!--more-->
---
C++26 brings a series of improvements to `std::format`, continuing the work started in C++20 and refined in C++23. These changes improve formatting consistency, runtime safety, and user ergonomics. There are so many of these updates, that I decided to divide them into two articles.

## Arithmetic overloads of `std::to_string` and `std::to_wstring` use `std::format`

[P2587R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2587r3.html) by Victor Zverovich proposes replacing `sprintf` with `std::format` in the arithmetic overloads of `std::to_string` and `std::to_wstring`. 

The motivation?

`std::to_string` has long been known to produce not-so-great and misleading results (differing from `iostreams`), especially with floating-point values. `std::to_string` uses the global C locale. In practice, it's unlocalized. Also, it often places the decimal points to suboptimal places:

```cpp
std::cout << std::to_string(-1e-7);  // prints: -0.000000
std::cout << std::to_string(0.42); // prints: 0.420000
```
These outputs are imprecise and often unnecessary. By leveraging `std::format` (and ultimately `std::to_chars`), we now get clearer, shorter representations. The representations of floating-point overloads become also unlocalized and use the shortest decimal representations.

As a result, the above outputs would change as follow:

```cpp
std::cout << std::to_string(-1e-7);  // prints: -1e-7
std::cout << std::to_string(0.42); // prints: 0.42
```

This change has some effects on existing code. When `std::to_string` is used with floating-point arguments, the output becomes more precise and/or shorter. Also, when the C locale is explicitly set, the decimal point will no longer be localized. But such usage seems very low.

This change is already available in GCC 14.

## Type checking format args 

While `std::format` already performs compile-time checks for format strings, certain errors — particularly those involving dynamic formatting specifications like width or precision — can still lead to runtime errors.

For example:

```cpp
std::format("{:>{}}", "hello", "10"); // Runtime error
```
In this case, the dynamic width specifier `{}` expects an integral type, but receives a `const char*`, leading to a runtime error.

[P2757R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2757r3.html) suggests enhancing the `basic_format_parse_context` to access the types of format arguments during compile-time parsing. This change allows the compiler to detect mismatches between format specifiers and argument types, turning potential runtime errors into compile-time errors.

The `{fmt}` library already implements a similar feature through its compile_parse_context, which stores type-erased information about argument types to facilitate compile-time checks.

This change is already available in GCC 15.

## Formatting pointers	

[P2510R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2510r3.pdf) makes pointer types compatible with `std::format`. There is nothing new under the sun in a sense that the options are already available for integer types.

While for integer types we already had a rich selection of formatting options, for pointer types it's been not the case. We either had to compromise or use some tricks. Like using `reinterpret_cast<uintptr_t>` or writing a custom formatter.

```cpp
#include <iostream> 
#include <format>

int main() { 
    auto ptr = new int(42);
    std::cout << &ptr << '\n';
    // std::cout << std::format("{:#018x}", ptr) << '\n'; // error in C++23
    std::cout << std::format("{:#018x}", reinterpret_cast<uintptr_t>(ptr)) << '\n';
    std::cout << std::format("{:#018X}", reinterpret_cast<uintptr_t>(ptr)) << '\n';

    delete ptr;
} 
```

With this proposal two kinds of formatting will be available for pointer types:
- zero padding, so `std::format("{:018}", ptr);` would result in an output like `0x00007ffe0325c4e4`
- lower/uppercase output with `p` or `P`, so `std::format("{:P}", ptr);` would result in output like `0X7FFE0325C4E4`

This change is already available in all the three major compilers, GCC 15, Clang 17 and MSVC 19.40.

## Member `std::basic_format_arg::visit()`

[P2637R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2637r3.html) brings member `visit` functions. Up until now, `std::visit` and `std::visit_format_arg` were only available as free standing functions.

For `std::visit_format_arg`, the main reason was to mimic `std::visit`. For `std::visit`, the reason was the need to forward constness and value categories. But with [C++23's deducing `this`](https://www.sandordargo.com/blog/2022/02/16/deducing-this-cpp23), these requirements can be satisfied by member functions.

> Have you already used `std::basic_format_arg`? It was introduced by C++20 and it provides access to a formatting arguments. It behaves like a variant of most of the builtin types, plus some more.

Let's have a look at the examples from the standard how this proposal changes the code we have to write:

```cpp
// Before:
auto format(S s, format_context& ctx) {
  int width = visit_format_arg([](auto value) -> int {
    if constexpr (!is_integral_v<decltype(value)>)
      throw format_error("width is not integral");
    else if (value < 0 || value > numeric_limits<int>::max())
      throw format_error("invalid width");
    else
      return value;
    }, ctx.arg(width_arg_id));
  return format_to(ctx.out(), "{0:x<{1}}", s.value, width);
}

// After:
auto format(S s, format_context& ctx) {
  int width = ctx.arg(width_arg_id).visit([](auto value) -> int {
    if constexpr (!is_integral_v<decltype(value)>)
      throw format_error("width is not integral");
    else if (value < 0 || value > numeric_limits<int>::max())
      throw format_error("invalid width");
    else
      return value;
    });
  return format_to(ctx.out(), "{0:x<{1}}", s.value, width);
}
```

This change is already available in GCC 15 and Clang 18/19 (`std::variant`/`stdbasic_format_arg`).

## Conclusion

C++26 is shaping up to be a great release for anyone using `std::format`. Whether it's more accurate outputs, safer templates, or simpler code, these updates make it even easier to use.

Stay tuned for Part 2, where we'll explore runtime formatting improvements, fixes for undefined behavior, and new formatters for standard types like `std::filesystem::path`.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
