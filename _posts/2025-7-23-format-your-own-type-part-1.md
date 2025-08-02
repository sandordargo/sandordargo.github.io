---
layout: post
title: "Format your own type (Part 1)"
date: 2025-7-23
category: dev
tags: [cpp, format]
excerpt_separator: <!--more-->
---
I recently published two posts about how C++26 improves `std::format` and the related facilities. (If you missed them, here are [Part 1](https://www.sandordargo.com/blog/2025/07/09/cpp26-format-part-1) and [Part 2](https://www.sandordargo.com/blog/2025/07/16/cpp26-format-part-2)). Now it's time to explore how you can **format your own types** using `std::format`.

But let's start from the beginning.

## Print to the console with `std::format`

`std::format` was introduced in C++20 and is based on Victor Zverovich’s `<fmt>` library, which in turn was inspired by Python's string formatting capabilities.

Let's skip the fancy formatting options and simply see how to interpolate values using `std::format`.

```cpp
#include <format>
#include <iostream>
#include <string>

int main() {
    std::string language{"C++"};
    int version{20};
    std::cout << std::format("{}{} is fun", language, version) << '\n';
}

/*
C++20 is fun
*/
```

That was easy.

Now imagine you want to print your own type. That won't work by default.

```cpp
#include <format>
#include <iostream>
#include <string>

struct ProgrammingLanguage {
    std::string name;
    int version{0};
};

int main() {
    ProgrammingLanguage cpp{"C++", 20};
    std::cout << std::format("{} is fun", cpp) << '\n'; // ERROR
}
```

You'll see a long error message, but the essential part is this:

```
/opt/compiler-explorer/gcc-trunk-20250609/include/c++/16.0.0/format:4995:10: error: static assertion failed: std::formatter must be specialized for each type being formatted
 4995 |         (is_default_constructible_v<formatter<_Args, _CharT>> && ...),
```

We need to specialize `std::formatter`.

## Specialize `std::formatter` the easy way

To create a valid specialization, we need two methods with specific signatures:
- `parse`
- `format`

The `parse` method processes format specifiers, and `format` generates the formatted output.

Here is a minimal but working example, something we can survive with:

```cpp
#include <format>
#include <iostream>
#include <string>

struct ProgrammingLanguage {
    std::string name;
    int version{0};
};

template <>
struct std::formatter<ProgrammingLanguage> {
    std::formatter<std::string> _formatter;
    constexpr auto parse(std::format_parse_context& parse_context) {
        return _formatter.parse(parse_context);
    }

    auto format(const ProgrammingLanguage& programming_language,
                std::format_context& format_context) const {
        std::string output = std::format("{}{}", programming_language.name,
                                         programming_language.version);
        return _formatter.format(output, format_context);
    }
};

int main() {
    ProgrammingLanguage cpp{"C++", 20};
    std::cout << std::format("{} is fun", cpp) << '\n';
}
```
`parse` is as simple as it gets, it simply delegates parsing `format_parse_context` to the member formatter of a `string`.

`format` is a bit more interesting. First of all, we mark it `auto` to avoid typing the return type, which is `std::format_context::iterator`. In this example, we opted for a simple implementation where we take members of an instance of `ProgrammingLanguage` and use them directly to create a string. 

## A more sophisticated formatter

Sometimes simple output isn't enough. Consider this:

- For C++, "C++20" looks fine.
- But for Python, "Python312" is not ideal — we'd prefer "Python 3.12".

This means we want:
- A space between name and version for some languages
- Support for complex version numbers

Let's first handle spaces.

We want this:

```cpp
ProgrammingLanguage cpp{"C++", 20};
ProgrammingLanguage python{"Python", 312};

std::cout << std::format("{:%n%v} is fun", cpp) << '\n';  // C++20 is fun
std::cout << std::format("{:%n %v} is fun", python) << '\n';  // Python 312 is fun
```

But how to handle the formatters?

In `parse()` we can assume that our `parse_context` begins at a `{` and we need to return an iterator to the closing brace (`}`). All what's inside we save into a member string (`":%n%v"` and `":%n %v"` in the above example). That's what we'll have to interpret while formatting.

```cpp
constexpr auto parse(std::format_parse_context& parse_context) {
    auto it = std::ranges::find(parse_context, '}');
    _attributes = std::string(parse_context.begin(), it);
    return it;
}
```

The role of `format()` is to actually go through the `_attributes` string and format the output accordingly.

Let's have a look at the below piece of code.

```cpp
auto format(const ProgrammingLanguage& programming_language,
            std::format_context& format_context) const {
    auto out = format_context.out();
    for (auto n = 0u; n < _attributes.size() - 1; ++n) {
        if (_attributes[n] == '%') {
            switch (_attributes[++n]) {
                case 'n':
                    out = std::format_to(out, "{}",
                                         programming_language.name);
                    break;
                case 'v':
                    out = std::format_to(out, "{}",
                                         programming_language.version);
                    break;
                case '%':
                    out = std::format_to(out, "%");
                    break;
            }
        } else {
            out = std::format_to(out, "{}", _attributes[n]);
        }
    }
    return out;
}
```

The idea is to iterate over the attributes string and if a character is `%` then we interpret the next character as a formatter and increment the iteration further. `%n` will be interpreted as a name, `%v` as a version and `%%` is escaped to `%`. If a character is not `%` we just add it to the output.

Are we good?

The problem is that if we try to print something without any formatters, we won't get anything! Worse than that `std::cout << std::format("{} is fun", cpp) << '\n';` results in a segmentation fault.

The reason of the segmentation fault is this `for` loop control:

```cpp
for (auto n = 0u; n < _attributes.size() - 1; ++n)
```

If `_attributes.size()` is `0`, then subtracting `1` from it won't result in `-1`, but in `std::numeric_limits<unsigned int>::max() - 1`, which is a fairly big number. And there we have a problem.

We can fix it by modifying the comparison to `n <= _attributes.size();`

Now `std::format("{} is fun", cpp)` results in an empty string. Not ideal.

Let's handle empty attributes separately, as a default case.

```cpp
if (_attributes.empty()) {
    out = std::format_to(out, "{}{}", programming_language.name,
                         programming_language.version);
    return out;
}
```

That's pretty similar to what we had in the previous example. Our code looks like this:

```cpp
// https://godbolt.org/z/sTnKoz7c6
#include <format>
#include <iostream>
#include <string>

struct ProgrammingLanguage {
    std::string name;
    int version{0};
};

template <>
struct std::formatter<ProgrammingLanguage> {
    std::string _attributes;
    constexpr auto parse(std::format_parse_context& parse_context) {
        auto it = std::ranges::find(parse_context, '}');
        _attributes = std::string(parse_context.begin(), it);
        return it;
    }

    auto format(const ProgrammingLanguage& programming_language,
                std::format_context& format_context) const {
        auto out = format_context.out();
        if (_attributes.empty()) {
            out = std::format_to(out, "{}{}", programming_language.name,
                                 programming_language.version);
            return out;
        }
        for (auto n = 0u; n <= _attributes.size(); ++n) {
            if (_attributes[n] == '%') {
                switch (_attributes[++n]) {
                    case 'n':
                        out = std::format_to(out, "{}",
                                             programming_language.name);
                        break;
                    case 'v':
                        out = std::format_to(out, "{}",
                                             programming_language.version);
                        break;
                    case '%':
                        out = std::format_to(out, "%");
                        break;
                }
            } else {
                out = std::format_to(out, "{}", _attributes[n]);
            }
        }
        return out;
    }
};

int main() {
    ProgrammingLanguage cpp{"C++", 20};
    ProgrammingLanguage python{"Python", 312};

    std::cout << std::format("{} is fun", cpp) << '\n';
    std::cout << std::format("{:%n%v} is fun", cpp) << '\n';
    std::cout << std::format("{:%n %v} is fun", python) << '\n';
}

/*
C++20 is fun
C++20 is fun
Python 312 is fun
*/
```

We will continue next week to handle multi-number versions.

## Conclusion

We've seen how to make `std::format` work with your custom types — from basic support to handling user-defined format strings. This not only makes your code cleaner but also enables richer formatting capabilities tailored to your domain.

In the next article, we'll take this one step further by implementing support for more complex version formats, such as *3.12* instead of just *312*, and explore how to integrate conditional formatting logic based on the language type.

Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
