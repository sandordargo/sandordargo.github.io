---
layout: post
title: "Format your own type (Part 2)"
date: 2025-7-30
category: dev
tags: [cpp, format]
excerpt_separator: <!--more-->
---
Last week, we discussed [how to write our own formatter](https://www.sandordargo.com/blog/2025/07/23/format-your-own-type-part-1) and finished with a relatively simple solution for printing a struct called `ProgrammingLanguage`. Today, we'll take it to the next level.

## Add more options for semantic versioning

Let's dream big. Instead of only handling major and minor versions (like Python 3.12), let's aim to fully support [semantic versioning](https://semver.org/).

> Semantic versioning (SemVer) is a versioning scheme that conveys meaning about the underlying changes in a release. It typically consists of three parts: *MAJOR.MINOR.PATCH*.

We should be able to print all these correctly:

```cpp
ProgrammingLanguage cpp{"C++", 20};
ProgrammingLanguage python312{"Python", 3, 12};
ProgrammingLanguage python31211{"Python", 3, 12, 11};


std::cout << std::format("{:%n%v} is fun", cpp) << '\n';  // C++20 is fun
std::cout << std::format("{:%n %v} is fun", python312) << '\n';  // Python 3.12 is fun
std::cout << std::format("{:%n %v} is fun", python31211) << '\n';  // Python 3.12.11 is fun
```

To achieve this, we first ensure that our `ProgrammingLanguage` struct can represent all parts of a semantic version:

```cpp
struct ProgrammingLanguage {
    std::string name;
    int major_version{0};
    std::optional<int> minor_version{std::nullopt};
    std::optional<int> patch_version{std::nullopt};
};
```
> While we could use `unsigned int` for version numbers, many prefer plain `int` for consistency and simplicity. I'll keep it simple here too.

Without any changes, only the `major_version` is printed by default. We want to give the user more flexibility.

The easiest solution is updating the `switch` and adding a case for minor and patch versions:

```cpp
switch (_attributes[++n]) {
    case 'n':
        out = std::format_to(out, "{}",
                             programming_language.name);
        break;
    case 'v':
        out = std::format_to(
            out, "{}", programming_language.major_version);
        break;
    case 'm':
        out = std::format_to(
            out, "{}", *programming_language.minor_version);
        break;

    case 'p':
        out = std::format_to(
            out, "{}", *programming_language.patch_version);
        break;
    case '%':
        out = std::format_to(out, "%");
        break;
}
```

This works, but the user has to specify the format precisely. For instance, to get `Python 3.12`, they must write:

```cpp
std::cout << std::format("{:%n %v.%m} is fun", python) << '\n';
```

## Add better default behaviour

It would be nice to make the default formatting take care of minor and patch versions as well if they are available.

We're going to make a couple of changes, so let's see what attributes we plan to end up with:
- `%l` to print the name of the **language**.
- `%v` to print the full **version**, dot separated. It will only print version parts that are available.
- `%m` to print the **major** version
- `%n` to print the **minor** version
- `%p` to print the **patch** version

If attributes are empty we are going to print the version as if we got the `{:%l%v}` formatting options. Instead of printing, let's use some assertions for testing purposes.

```cpp
// https://godbolt.org/z/93M9e4hrr

#include <cassert>
#include <format>
#include <iostream>
#include <optional>
#include <string>

struct ProgrammingLanguage {
    std::string name;
    int major_version{0};
    std::optional<int> minor_version{std::nullopt};
    std::optional<int> patch_version{std::nullopt};
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
                                 programming_language.major_version);
            return out;
        }
        for (auto n = 0u; n < _attributes.size(); ++n) {
            if (_attributes[n] == '%') {
                switch (_attributes[++n]) {
                    case 'l':
                        out = std::format_to(out, "{}",
                                             programming_language.name);
                        break;
                    case 'v':
                        out = std::format_to(
                            out, "{}", programming_language.major_version);
                        if (programming_language.minor_version) {
                            out = std::format_to(
                                out, ".{}",
                                *programming_language.minor_version);
                        }
                        if (programming_language.patch_version) {
                            out = std::format_to(
                                out, ".{}",
                                *programming_language.patch_version);
                        }
                        break;
                    case 'm':
                        out = std::format_to(
                            out, "{}", programming_language.major_version);
                        break;
                    case 'n':
                        out = std::format_to(
                            out, "{}", *programming_language.minor_version);
                        break;

                    case 'p':
                        out = std::format_to(
                            out, "{}", *programming_language.patch_version);
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
    using namespace std::string_literals;
    ProgrammingLanguage cpp{"C++", 20};
    ProgrammingLanguage python{"Python", 3, 12};
    ProgrammingLanguage pythonPatch{"Python", 3, 12, 10};

    assert(std::format("{}", cpp) == "C++20"s);
    std::cout << std::format("{:%l%m}", cpp) << '\n';
    assert(std::format("{:%l%m}", cpp) == "C++20"s);
    assert(std::format("{:%l%v}", cpp) == "C++20");
    assert(std::format("{:%l %v}", pythonPatch) == "Python 3.12.10");
    assert(std::format("{:%l %m.%n.%p}", pythonPatch) == "Python 3.12.10");
}
```
A good follow-up would be deciding how to handle edge cases, such as when a patch version exists without a minor version — but that goes beyond the scope of this post.

## Conclusion

[In the previous post](https://www.sandordargo.com/blog/2025/07/23/format-your-own-type-part-1), we started implementing a custom formatter to print a programming language name and version. Today, we expanded it to support semantic versioning, optional fields, and customizable format strings.

We didn't add error handling yet — like how to deal with missing minor or patch versions — but what we've done is a solid foundation if you want to continue.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
