---
layout: post
title: "C++26 Reflection: All You Need To Know About std::meta::info"
date: 2025-X-X
category: dev
tags: [cpp, cpp26, reflection, syntax]
excerpt_separator: <!--more-->
---
In the previous parts of our C++ Reflection journey, we explored [the motivations for C++ Reflection](), [the basics of the syntax](), and the essentials of [`std::meta::info`](). We also looked into [the basics of metafunctions]() to understand the role of `consteval` functions when interacting with reflection objects.

In this article, we will dig deeper into a few key metafunctions: **name, location, and type queries**.


```cpp
namespace std::meta {
  // P2996
  using info = decltype(^^::);

  template <typename R>
    concept reflection_range = /* see above */;

  // name and location
  consteval auto identifier_of(info r) -> string_view;
  consteval auto u8identifier_of(info r) -> u8string_view;

  consteval auto display_string_of(info r) -> string_view;
  consteval auto u8display_string_of(info r) -> u8string_view;

  consteval auto source_location_of(info r) -> source_location;

  // type queries
  consteval auto type_of(info r) -> info;
  consteval auto parent_of(info r) -> info;
  consteval auto dealias(info r) -> info;
```

## Name queries

The difference between the "normal" and `u8`-prefixed versions is simple:
- `identifier_of` and `display_string_of` return `std::string_view`.
- `u8identifier_of` and `u8display_string_of` return `std::u8string_view`.

>*Note: `std::string` cannot be used here because that would mean compile-time allocated strings leaking into the runtime environment — which is not allowed today.*

Given a reflection r that represents some construct X with an identifier, if that identifier can be expressed using the ordinary literal encoding, then `identifier_of(r)` will return a non-empty `string_view` containing the identifier.

`identifier_of(r)` can fail in two cases:
- the entity has no identifier (e.g., some lambdas, anonymous namespaces),
- the identifier cannot be represented in the literal encoding.

In those cases, `identifier_of` throws a `std::meta::exception`.

By contrast, `u8identifier_of` never fails for the second reason since UTF-8 can represent all identifiers.

Since the C++ grammar requires that every identifier has at least one character, `identifier_of` never returns an empty string.

So what's the difference between `identifier_of` and `display_string_of`?
- `identifier_of` gives you the raw identifier.
- for `display_string_of`, implementations are encouraged to provide a more readable string that helps identify the construct.

Let's see some examples:

```cpp
//Clang: https://godbolt.org/z/eq8GqMGvh
//EDG: https://godbolt.org/z/d9ovs9aTq
#include <experimental/meta>
#include <iostream>

struct MyStruct {
    int num;
    bool flag;
};

constexpr int square(int num) {
    std::cout << "identifier_of(^^num): " << identifier_of(^^num) << '\n';
    std::cout << "display_string_of(^^num): " << display_string_of(^^num) << '\n';
    return num * num;
}

enum class Color {
    kRed, kGreen, kBlue
};

int main() {
    constexpr auto r1 = ^^MyStruct;
    // constexpr auto r2 = ^^([](){});
    [[maybe_unused]] constexpr auto l = [](){};
    constexpr auto r3 = ^^l;
    std::cout << "identifier_of(r1): " << identifier_of(r1) << '\n';
    std::cout << "display_string_of(r1): " << display_string_of(r1) << '\n';
    // std::cout << "identifier_of(r2): " << identifier_of(r2) << '\n';
    std::cout << "identifier_of(r3): " << identifier_of(r3) << '\n';
    std::cout << "display_string_of(r3): " << display_string_of(r3) << '\n';
    
    std::cout << "identifier_of(^^Color::kBlue): " << identifier_of(^^Color::kBlue) << '\n';
    std::cout << "display_string_of(^^Color::kBlue): " << display_string_of(^^Color::kBlue) << '\n';
    
    square(42);
}

/*

identifier_of(r1): MyStruct
display_string_of(r1): MyStruct
identifier_of(r3): l
display_string_of(r3): l
identifier_of(^^Color::kBlue): kBlue
display_string_of(^^Color::kBlue): kBlue
identifier_of(^^num): num
display_string_of(^^num): num

*/
```

First of all, on the top of the listing, you have two Godbolt links. [One compiled with a Clang fork](//Clang: https://godbolt.org/z/eq8GqMGvh), the [other with the EDG implementation](https://godbolt.org/z/d9ovs9aTq).

The main difference is that EDG hasn't implemented yet `std::meta::display_string_of` - that might have changed by the time you're reading this. Another difference is that the Clang implementation doesn't reflect directly on a lambda expression (`r2`). EDG does it, but it cannot take the identifier of it, not being a constant-expression.

In the examples above, identifiers and display strings look the same. But with more advanced constructs, you'll start seeing differences.

## Location query

The location query is handled by `std::meta::source_location_of`. It returns an unspecified [std::source_location](https://en.cppreference.com/w/cpp/utility/source_location.html). Implementations are encouraged to return the actual source location of the reflected entity.

At the moment, only Clang has implemented this. Example:

```cpp
// https://godbolt.org/z/Mnrb1xT9E
#include <experimental/meta>
#include <iostream>

constexpr void print(const std::source_location location)
{
    std::cout << "file: "
              << location.file_name() << '('
              << location.line() << ':'
              << location.column() << ") `"
              << location.function_name() << "`: "
              << '\n';
}

struct MyStruct {
    int num;
    bool flag;
};

constexpr int square(int num) {
    print(source_location_of(^^num));
    return num * num;
}

enum class Color {
    kRed, kGreen, kBlue
};

int main() {
    constexpr auto r1 = ^^MyStruct;
    [[maybe_unused]] constexpr auto l = [](){};
    constexpr auto r2 = ^^l;

    print(source_location_of(r1));
    print(source_location_of(r2));
    print(source_location_of(^^Color::kBlue));
    print(source_location_of(^^square));
    
    square(42);
}
/*
file: /app/example.cpp(30:37) `int main()`: 
file: /app/example.cpp(25:19) ``: 
file: /app/example.cpp(19:15) ``: 
file: /app/example.cpp(19:26) `int square(int)`: 
*/
```

As we can observe, `std::meta::source_location_of` correctly returns a source_location of the type/value reflected.

Here is a simplified example to see the difference between when we reflect on a type and an instance of it:

```cpp
// https://godbolt.org/z/7vjWThcKf
#include <experimental/meta>
#include <iostream>

constexpr void print(const std::source_location location)
{
    std::cout << "file: "
              << location.file_name() << '('
              << location.line() << ':'
              << location.column() << ") `"
              << location.function_name() << "`: "
              << '\n';
}

struct MyStruct {
    int num;
    bool flag;
};


int main() {
    constexpr auto r1 = ^^MyStruct;

    MyStruct ms;

    print(source_location_of(r1));
    print(source_location_of(^^ms));
}
/*
file: /app/example.cpp(14:8) ``: 
file: /app/example.cpp(23:14) `int main()`: 
*/
```
## Type queries

Let's finish this post with type queries. We have three of them:

- `auto type_of(info r) -> info`
- `auto dealias(info r) -> info`
- `auto parent_of(info r) -> info`

### `type_of`

If `r` reflects a typed entity, `type_of(r)` gives you the reflection of its type. If `r` itself is a type or something without a type (e.g., a namespace), `type_of` throws.

An important detail: `type_of` never returns an alias; it always returns the dealiased type.

```cpp
// https://godbolt.org/z/xa7hejTj7
#include <experimental/meta>
#include <iostream>
#include <string>

namespace foo {

}

struct MyStruct {
    int num;
    bool flag;
};


int main() {
    constexpr auto r1 = ^^MyStruct;

    [[maybe_unused]] MyStruct ms;
    constexpr std::string s{"hello"};
    
    using AliasedMyStruct = MyStruct;
    [[maybe_unused]] AliasedMyStruct ams;
    
    // Error: a type has no type
    // constexpr auto t1 = type_of(r1);

    // Error: a namespace has no type
    // constexpr auto t2 = type_of(^^foo);
    constexpr auto t3 = type_of(^^ms);
    std::cout << display_string_of(t3) << '\n';

    constexpr auto t4 = type_of(^^s);
    std::cout << display_string_of(t4) << '\n';

    constexpr auto t5 = type_of(^^ams);
    std::cout << display_string_of(t5) << '\n';
}
/*
MyStruct
const basic_string<char, char_traits<char>, allocator<char>>
MyStruct
*/
```

We can also observe that if we pass a reflection of a type or of a namespace (that is not a type) then `type_of` fails just as expected.


### `dealias`

If we have the reflection of an aliased type and we want to get the dealiased version, we can use `dealias` to do that.

```cpp
// https://godbolt.org/z/v6zME6njW
#include <experimental/meta>
#include <iostream>
#include <string>

struct MyStruct {
    int num;
    bool flag;
};


int main() {
    constexpr auto r1 = ^^MyStruct;
    
    using AliasedMyStruct = MyStruct;
    constexpr auto r2 = ^^AliasedMyStruct;

    static_assert(r1 != r2);
    static_assert(r1 == dealias(r2));
    static_assert(r1 == dealias(r1));

    [[maybe_unused]] MyStruct ms;
    static_assert(^^ms == dealias(^^ms));


    std::cout << display_string_of(r1) << '\n';
    std::cout << display_string_of(r2) << '\n';
    std::cout << display_string_of(dealias(r2)) << '\n';
}
/*
MyStruct
AliasedMyStruct
MyStruct
*/
```
We can also observe that if we call `dealias` on a reflection that is not representing an aliased type, or not even a type, `dealias` returns its input.

### `parent_of`

Let's move on and delve into `std::meta::parent_of`. You might think at first that it's about discovering relationships an inheritance tree. It's not. It's about finding the enclosing scope.

So for example, the parent of `Base`, `Derived1` and `Derived2` in the below listing are all the same. It's the same anonymous namespace. *(Notice as well, that anonymous namespaces have no identifiers.)*

You can also see that not only namespaces can be parents, but a function, a class as well. Any scope that is enclosing the reflected entity.

```cpp
// https://godbolt.org/z/ErKz1v9h5
#include <experimental/meta>
#include <iostream>

namespace {
class Base {};

class Derived1 : public Base {};
class Derived2 : public Base {};
}

namespace foo {
struct MyStruct {
    int num;
    bool flag;
};
}

int main() {
    int j{42};

    static_assert(parent_of(^^Derived1) == parent_of(^^Derived2));
    std::cout << "parent of Base: " << display_string_of(parent_of(^^Base)) << '\n';
    std::cout << "parent of Derived1: " << display_string_of(parent_of(^^Derived1)) << '\n';
    std::cout << "parent of Derived2: " << display_string_of(parent_of(^^Derived2)) << '\n';
    std::cout << "parent of foo::MyStruct: " << display_string_of(parent_of(^^foo::MyStruct)) << '\n';
    std::cout << "parent of foo::MyStruct::flag: " << display_string_of(parent_of(^^foo::MyStruct::flag)) << '\n';
    std::cout << "parent of j: " << display_string_of(parent_of(^^j)) << '\n';

    // Error: There is no identifier of an anonymous namespace
    // std::cout << "parent of Base: " << identifier_of(parent_of(^^Base)) << '\n';
    std::cout << "parent of foo::MyStruct: " << identifier_of(parent_of(^^foo::MyStruct)) << '\n';
}
/*
parent of Base: (anonymous-namespace)
parent of Derived1: (anonymous-namespace)
parent of Derived2: (anonymous-namespace)
parent of foo::MyStruct: foo
parent of foo::MyStruct::flag: MyStruct
parent of j: main
parent of foo::MyStruct: foo
*/
```

`parent_of` only throws if something has no parent.

But what has no parent? For example the global namespace, language linkages (other than C++) or types that neither classes/structs or enums. (What that can be?)

## Conclusion

In this article, we explored some of the most fundamental metafunctions that make `std::meta::info` truly useful:

- **Name queries** help us retrieve identifiers and human-readable display strings.
- **Location queries** give us precise `std::source_location` data.
- **Type queries** allow us to navigate from entities to their types, `dealias` them, and discover their enclosing scope.

Taken together, these tools form the backbone of C++26 reflection: they allow us to bridge source-level constructs with compile-time introspection in a structured way.

Stay tuned for the next part of our reflection discovery.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!