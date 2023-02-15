---
layout: post
title: "The evolution of enums"
date: 2023-2-15
category: dev
tags: [cpp, cpp20, cpp23, enums]
excerpt_separator: <!--more-->
---
Constants are great. Types are great. Constants of a specific type are really great. This is why `enum class`es are just fantastic.

Last year, we talked about [why we should avoid using boolean function parameters](https://www.sandordargo.com/blog/2022/04/06/use-strong-types-instead-booleans). One of the solutions proposed uses strong types, in particular using `enum`s instead of raw booleans. This time, let's see how `enum`s and the related support evolved during the course of the years.

## Unscoped enumerations

Enumerations are part of the original C++ language, in fact, they were taken over from C. Enumerations are distinct types with a restricted range of values. The range of values is restricted to some explicitly named constants. Let's quickly have a look at an `enum`.

```cpp
enum Color { red, green, blue };
```

After having read this very small example, it's worth noticing two things:
- The `enum` itself is a singular noun, even though it enumerates multiple values. We use such conventions because we keep in mind that it will be always used with one value. If you take a `Color` function parameter, one colour will be taken. When you compare against a value, you'll compare against one value. E.g. it reads better to compare against `Color::red` than against `Colors::red`
- The enumerator values are not written in *ALL_CAPS*! Even though there is a fair chance that you are used to that. I also used to do that. So why didn't I follow that practice? Because for writing this article, I checked the core guidelines and [Enum.5](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#enum5-dont-use-all_caps-for-enumerators) clearly says that we should not use *ALL_CAPS* in order to avoid clashes with macros. By the way, [Enum.1](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#enum5-dont-use-all_caps-for-enumerators) clearly said that we should use enumerations over macros.

Since C++11, the number of possibilities to declare an `enum` grew. C++11 introduced the possibility of specifying the underlying type of an enum. If it's left undefined, the underlying type is implementation-defined but in any case, it's an integral type.

How to specify it? Syntax-wise it might seem a bit like inheritance! Though there are no access levels to define.

```cpp
enum Color : int { red, green, blue };
```

With that you can be sure what the underlying type is. Still, it must be an integral type! For example, it cannot be a string. Should you try that and you'll get a very explicit error message:

```
main.cpp:4:19: error: underlying type 'std::string' {aka 'std::__cxx11::basic_string<char>'} of 'Color' must be an integral type
    4 | enum Color : std::string { red, green, blue };
      |                   ^~~~~~
```

Note that [the core guidelines advocate against this practice](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#enum7-specify-the-underlying-type-of-an-enumeration-only-when-necessary)! You should only specify the underlying value if it is necessary.

Why can it be necessary? It gives us two reasons.

- If you know that the number of choices will be very limited and you want to save a bit of memory:
```cpp
enum Direction : char { north, south, east, west,
     northeast, northwest, southeast, southwest }; 
```

- Or if you happen to forward declare an `enum` then you also must declare the type:

```cpp
enum Direction : char;
void navigate(Direction d);

enum Direction : char { north, south, east, west,
     northeast, northwest, southeast, southwest }; 
```

You can also specify the exact value of one or all the enumerated values as long as they are `constexpr`.

```cpp
enum Color : int { red = 0, green = 1, blue = 2 };
```

Once again, [the guidelines recommends us not to do this](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#enum8-specify-enumerator-values-only-when-necessary), unless it's necessary! Once you start doing it, it's easy to make mistakes and mess it up. We can rely on the compiler assigning subsequent values to the subsequent enumerator values.

- A good reason to specify the enumerator value is to define only the starting value. If you define the months and you don't want to start with zero.
```cpp
enum Month { jan = 1, feb, mar, apr, may, jun,
                   jul, august, sep, oct, nov, dec }; 
```
- Another reason can be if you want to define the values as some meaningful character
```cpp
enum altitude: char {
    high = 'h',
    low = 'l'
}; 
```
- One other reason can be emulating some bitfields. So you don't want subsequent values, but you always want the next power of two
```cpp
enum access_type { read = 1, write = 2, exec = 4 };
```

## Scoped enumerations

In the previous section, we saw declarations such as `enum EnumName{};`. C++11 brought a new type of enumeration called scoped `enum`s. They declared either with the `class` or with the `struct` keywords and there is no difference between those two.

The syntax is the following:

```cpp
enum class Color { red, green, blue };
```

For scoped enumerations the default underlying type is defined in the standard, it is `int`. This also means that if you want to forward declare a scoped `enum`, you don't have to specify the underlying type. If it is meant to be `int`, this is enough:

```cpp
enum class Color;
```

Apart from how the syntactical differences between how they are declared, what other differences exist?

Unscoped `enum`s can be implicitly converted to their underlying type. Implicit conversions are often not what you want, and scoped `enum`s don't have this *"feature"*. Exactly because of the unwelcome implicit conversions, [the Core Guidelines strongly recommends using scoped over unscoped `enum`s](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#enum3-prefer-class-enums-over-plain-enums).

```cpp
void Print_color(int color);

enum Web_color { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };
enum Product_info { red = 0, purple = 1, blue = 2 };

Web_color webby = Web_color::blue;

// Clearly at least one of these calls is buggy.
Print_color(webby);
Print_color(Product_info::blue);
```

Unscoped `enum`s export their enumerators to the enclosing scope which might lead to name clashes. On the other hand, with scoped `enum`s, you must always specify the name of the `enum` alongside with the enumerators.

```cpp
enum UnscopedColor { red, green, blue };
enum class ScopedColor { red, green, blue };

int main() {
    [[maybe_unused]] UnscopedColor uc = red;
    // [[maybe_unused]] ScopedColor sc = red; // Doesn't compile
    [[maybe_unused]] ScopedColor sc = ScopedColor::red;
}
```

## What else

Now that we saw how un/scoped `enum`s work and what are the differences between them, let's see what other `enum` related functionalities the language or standard library offers.

### `std::is_enum`

C++11 introduced the `<type_traits>` header. It includes utilities to check the properties of types. Not surprisingly `is_enum` is there to check whether a type is an `enum` of not. It returns `true` both for scoped and unscoped versions.

Since C++17, `is_enum_v` is also available for easier usage.

```cpp
#include <iostream>
#include <type_traits>

enum UnscopedColor { red, green, blue };
enum class ScopedColor { red, green, blue };
struct S{};

int main() {
    std::cout << std::boolalpha
              << std::is_enum<UnscopedColor>::value << '\n'
              << std::is_enum<ScopedColor>::value << '\n'
              << std::is_enum_v<S> << '\n';
}

```

### `std::underlying_type`

`std::underlying_type` was also an addition to C++11. It helps us retrieve the underlying type of an `enum`. Until C++20 if the checked `enum` is not completely defined or not an `enum`, the behaviour is undefined. Starting with C++, the program becomes ill-formed for incomplete `enum` types.

C++14 introduced a related helper, `std::underlying_type_t`.

```cpp
#include <iostream>
#include <type_traits>

enum UnscopedColor { red, green, blue };
enum class ScopedColor { red, green, blue };
enum class CharBasedColor : char { red = 'r', green = 'g', blue = 'b' };

int main() {
 
  constexpr bool isUnscopedColorInt = std::is_same_v< std::underlying_type<UnscopedColor>::type, int >;
  constexpr bool isScopedColorInt = std::is_same_v< std::underlying_type_t<ScopedColor>, int >;
  constexpr bool isCharBasedColorInt = std::is_same_v< std::underlying_type_t<CharBasedColor>, int >;
  constexpr bool isCharBasedColorChar = std::is_same_v< std::underlying_type_t<CharBasedColor>, char >;

  std::cout
    << "underlying type for 'UnscopedColor' is " << (isUnscopedColorInt ? "int" : "non-int") << '\n'
    << "underlying type for 'ScopedColor' is " << (isScopedColorInt ? "int" : "non-int") << '\n'
    << "underlying type for 'CharBasedColor' is " << (isCharBasedColorInt ? "int" : "non-int") << '\n'
    << "underlying type for 'CharBasedColor' is " << (isCharBasedColorChar ? "char" : "non-char") << '\n'
    ;
}
```

### Using-enum-declaration since C++20

Since C++20, use can use `using` with `enum`s. It introduces the enumerator names in the given scope.

The feature is smart enough to raise a compilation error in case a second `using` would introduce an enumerator name that was already introduced from another `enum`.

```cpp
#include <type_traits>

enum class ScopedColor { red, green, blue };
enum class CharBasedColor : char { red = 'r', green = 'g', blue = 'b' };

int main() {
  using enum ScopedColor; // OK!
  using enum CharBasedColor; // error: 'CharBasedColor CharBasedColor::red' conflicts with a previous declaration
}
```

It's worth noting that it doesn't recognize if an unscoped enum already introduced an enumerator name in the given namespace. In the following example, there is already `red`, `green`, and `blue` available from `UnscopedColor`, still, the `using` of `ScopedColor` with the same enumerator names is accepted.

```cpp
#include <type_traits>

enum UnscopedColor { red, green, blue };
enum class ScopedColor { red, green, blue };

int main() {
  using enum ScopedColor;
}
```

### C++23 brings `std::is_scoped_enum`

C++23 will introduce one more `enum` related function in the `<type_traits>` header, one of it is `std::is_scoped_enum` and it's helper function `std::is_scoped_enum_v`. As the name suggests and the below snippet proves, it checks whether it is argument is a **scoped** `enum` or not.

```cpp
#include <iostream>
#include <type_traits>
 
enum UnscopedColor { red, green, blue };
enum class ScopedColor { red, green, blue };
struct S{};

int main() 
{
    std::cout << std::boolalpha;
    std::cout << std::is_scoped_enum<UnscopedColor>::value << '\n';
    std::cout << std::is_scoped_enum_v<ScopedColor> << '\n';
    std::cout << std::is_scoped_enum_v<S> << '\n';
    std::cout << std::is_scoped_enum_v<int> << '\n';
}
/*
false
true
false
false
*/
```

If you want to try out C++23 features, use the `-std=c++2b` compiler flag.

### C++23 introduces `std::to_underlying`

C++23 will introduce another library feature for `enum`s. The `<utility>` header will be enriched with `std::to_underlying`. It converts an `enum` to its underlying type. As mentioned, this is a library feature, meaning that it can be implemented in earlier versions.

This one is can be replaced with a `static_cast` if you have access only to earlier versions: `static_cast<std::underlying_type_t<MyEnum>>(e);`.

```cpp
#include <iostream>
#include <type_traits>
#include <utility>
 
enum class ScopedColor { red, green, blue };

int main() 
{
    ScopedColor sc = ScopedColor::red;
    [[maybe_unused]] int underlying = std::to_underlying(sc);
    [[maybe_unused]] int underlyingEmulated = static_cast<std::underlying_type_t<ScopedColor>>(sc);
    [[maybe_unused]] std::underlying_type_t<ScopedColor> underlyingDeduced = std::to_underlying(sc);
}
```

As a reminder, let me repate that if you want to try out C++23 features, use the `-std=c++2b` compiler flag.

## Conclusion

In this article, we discussed all the language and library features that are about enumerations. We saw how scoped and unscoped `enum`s differ and why it's better to use scoped `enum`s. That's not the only [Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-enum) recommendation we discussed. 

Then we checked how the standard library has been enriched during the years supporting an easier work with `enum`s. We also had a sneak peek into the future and checked what C++23 will bring for us.


## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
