---
layout: post
title: "C++26: Structured bindings in conditions"
date: 2026-4-15
category: dev
tags: [cpp, cpp26, structuredbindings]
excerpt_separator: <!--more-->
---
Structured bindings were introduced in C++17 as an alternative way of declaring variables. They allow you to decompose an object into a set of named variables, where the collection of those bindings conceptually represents the original object as a whole.

```cpp
// https://godbolt.org/z/97GaMajMP

#include <cassert>
#include <string>

struct MyStruct {
    int num;
    std::string text;

    bool operator==(const MyStruct&) const noexcept = default;
};

MyStruct foo() {
    return {42, "let's go"};
}

int main() {
    const auto& [n, t] = foo();
    MyStruct ms{n, t};
    assert(ms == foo());
    return 0;
}
```
## Structured bindings before C++26

Up to C++23, structured bindings could appear in:
- simple declarations (as shown above), and
- range-based `for` loops.

```cpp
// https://godbolt.org/z/Tf8s4sdrf

#include <iostream>
#include <string>
#include <vector>

struct MyStruct {
    int num;
    std::string text;
};

int main() {
    std::vector<MyStruct> v{ {42, "hello"}, {51, "there"} };
    for(const auto&[num, text] : v) {
        std::cout << "num: " << num << ", text: " << text << '\n';
    }
    return 0;
}
/*
num: 42, text: hello
num: 51, text: there
*/
```

However, you could not use structured binding declarations in conditions (for example, in `if` or `while` statements).

## What’s new in C++26?

C++26 brings some signigicant improvements to structured bindings:
- [Individual bindings can be annotated with attributes](https://www.sandordargo.com/blog/2025/01/29/cpp26-attributes-structured-bindings)
- [Structured bindings can be declared `constexpr`](https://www.sandordargo.com/blog/2025/04/23/cpp26-constexpr-language-changes#p2686r5-constexpr-structured-bindings-and-references-to-constexpr-variables)
- [Structured bindings can introduce a parameter pack](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p1061r10.html)

And of course, there is the topic of this article:
> Structured binding declarations are now allowed directly in conditions.

This change comes from [P0963R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p0963r3.html)

## Structured bindings in conditions

As of C++26, the following becomes valid:

```cpp
// https://godbolt.org/z/bP58h9153

#include <iostream>
#include <optional>
#include <string>

struct Result {
    bool is_successful;
    std::optional<std::string> error_message;
    explicit operator bool() const noexcept { return is_successful; }
};

Result foo(int i) {
    if (i % 2 == 0) {
        return {true};
    } else {
        return {false, "that didn't work!"};
    }
}

int main() {
    for (int n : {1, 2}) {
        if (const auto& [is_successful, error_message] = foo(n)) {
            std::cout << "no error\n";
        } else {
            std::cout << "there was an error: " << error_message.value_or("")
                      << '\n';
        }
    }

    return 0;
}

/*
there was an error: that didn't work!
no error
*/
```

## What's going on here?

- The initializer (`foo(n)`) is evaluated first.
- The condition is then evaluated by invoking `operator bool()` on the initialized object.
- The structured binding is introduced, and the bindings are available in both the if and the else branches.

This mirrors the existing rules for init-statements in `if`, but now extended to structured bindings as well.

A key requirement is that the decomposed object must be contextually convertible to bool. Without an `operator bool`, the condition would be ill-formed.

## Why is this useful?

This addition significantly improves expressiveness and readability. It is especially helpful when:
- a function returns a status together with additional data,
- a result may or may not be present along with diagnostic information (for example, `std::expected`-like types),
- an operation naturally produces multiple logically related values.

[The motivation section of the paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p0963r3.html#Motivation) contains several realistic examples illustrating these use cases.

## Evaluation order and decomposition

One subtle but important design decision concerns **when** decomposition happens.

The chosen semantics are:
1) Initialize the object.
2) Evaluate the condition (via contextual conversion to `bool`).
3) Introduce and initialize the structured bindings.

This sequencing ensures that condition checking is well-defined and consistent with existing language rules, while still allowing the bindings to be used in both branches.

## Conclusion

Structured bindings in conditions may look like a small syntax sugar, but they let us write much more expressive conditional logic. By allowing decomposition and condition checking to live side by side, C++26 reduces boilerplate, improves locality, and better supports modern result types that bundle status and data together. This is a pragmatic, well-integrated evolution of a feature that has already proven its value since C++17.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
