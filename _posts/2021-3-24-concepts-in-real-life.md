---
layout: post
title: "C++ concepts in real life"
date: 2021-3-24
category: dev
tags: [cpp, concepts, cpp20]
excerpt_separator: <!--more-->
---
During the last month or so, we examined the ins and outs of C++ concepts. We checked their [main motivations](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations), we saw [how we can use them with functions](https://www.sandordargo.com/blog/2021/02/17/cpp-concepts-4-ways-to-use-them), [with classes](https://www.sandordargo.com/blog/2021/02/24/cpp-concepts-with-classes) and [what kind of concepts are shipped with the standard library](https://www.sandordargo.com/blog/2021/03/03/cpp-concepts-in-standard-library). Then during the last two weeks, we discovered how to write our own ones ([part I](https://www.sandordargo.com/blog/2021/03/10/write-your-own-cpp-concepts-part-i), [part II](https://www.sandordargo.com/blog/2021/03/17/write-your-own-cpp-concepts-part-ii)).
<!--more-->
To conclude this series, let's see two real-world examples of useful concepts.

## Numbers finally

We've been playing with a concept called `Number` for weeks. I've always said it's incomplete. Let's have a quick reminder of why:


```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;

auto add(Number auto a, Number auto b) {
  return a+b;
}

int main() {
    std::cout << "add(1, 2): " << add(1, 2) << '\n';
    std::cout << "add(1, 2.14): " << add(1, 2.14) << '\n';
    // std::cout << "add(\"one\", \"two\"): " << add("one", "two") << '\n'; // error: invalid operands of types 'const char*' and 'const char*' to binary 'operator+'
    std::cout << "add(true, false): " << add(true, false) << '\n';
}

/*
add(1, 2): 3
add(1, 2.14): 3.14
add(true, false): 1
*/
```
Our problem is that even though we only want to accept integrals and floating-point numbers, `bool`s are also accepted. `bool`s are accepted because `bool` is an integral type.

There is even worse! `add(0, 'a')` returns 97 as `a` is a character and as such it's considered an integral type. The ASCII code of `a` is 97 and if you add that to 0, you get the result of this call.

But let's say, we really want to accept numbers and let's say in the constrained world of _real numbers_.

We have to limit the types we accept. As `std::is_floating_point` returns `true` only for `float`, `double` and `long double`, there is no problem there. But floating-point numbers are not enough and as we already saw, `std::is_integral` returns `true` for some types that we might not want to accept as numbers.

The following types and their `const` and/or `unsgined` versions are considered integral:

- `bool`,
- `char`, `char8_t`, `char16_t`, `char32_t`, `wchar_t`, 
- `short`, `int`, `long`, `long long`

But we only want to accept the types from the third line, booleans and characters are not our cups of tea.

Before C++20, we'd have to either disallow certain overloads or use static assertions with templates to make sure that only certain types would be accepted.

```cpp
template<typename T>
T addPreCpp20(T a, T b) {
    static_assert(std::is_integral_v<T>, "addPreCpp20 requires integral types");
    return a+b;
}

// ...
std::cout << addPreCpp20(1,2) << '\n'; // valid
std::cout << addPreCpp20(1,2.14) << '\n'; // woulnd't compile, static assertion fails
```

The main problem with these that we'd have to do the same steps for each function, for each parameter.

With overloads, we might end up with a too-long list of combinations (when you have 3 numeric parameters that you want to constrain), or your templates are either too repetitive or just too complex for most working on the codebase.

C++20 brought us concepts and we have to define our `Number` concept only once, and then [it's easy to use it](https://www.sandordargo.com/blog/2021/02/17/cpp-concepts-4-ways-to-use-them).

Just repeat our requirements:
- we want to accept floating-point numbers
- we want to accept integral numbers
- we don't want to accept integral types that can be converted to `int`s such as `bool`s and `char`s.

As the first trial, you might try something like this

```cpp
#include <concepts>

template <typename T>
concept Number = (std::integral<T> || std::floating_point<T>) 
                 && !std::same_as<T, bool>
                 && !std::same_as<T, char>
                 && !std::same_as<T, char8_t>
                 && !std::same_as<T, char16_t>
                 && !std::same_as<T, char32_t>
                 && !std::same_as<T, wchar_t>;

auto add(Number auto a, Number auto b) {
  return a+b;
}              
```

But we are not done yet. The following compiles and prints 139!

```cpp
unsigned char a = 'a';
std::cout << add(a, 42);
```

We have to include all the unsigned versions! Luckily only `char` has an unsigned eversion. `const`s we don't have to permit as those as a `const char` would be automatically considered a `char` and therefore it wouldn't compile.

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Number = (std::integral<T> || std::floating_point<T>) 
                 && !std::same_as<T, bool>
                 && !std::same_as<T, char>
                 && !std::same_as<T, unsigned char>                 
                 && !std::same_as<T, char8_t>
                 && !std::same_as<T, char16_t>
                 && !std::same_as<T, char32_t>
                 && !std::same_as<T, wchar_t>;

auto add(Number auto a, Number auto b) {
  return a+b;
}

int main() {
    std::cout << "add(1, 2): " << add(1, 2) << '\n';
    std::cout << "add(1, 2.14): " << add(1, 2.14) << '\n';
    // std::cout << "add(\"one\", \"two\"): " << add("one", "two") << '\n'; // error: invalid operands of types 'const char*' and 'const char*' to binary 'operator+'
    // std::cout << "add(true, false): " << add(true, false) << '\n'; // unsatisfied constraints
    // const char c = 'a';
    // std::cout << add(c, 42); // unsatisfied constraints
    // unsigned char uc = 'a';
    // std::cout << add(uc, 42); // unsatisfied constraints
}
/*
add(1, 2): 3
add(1, 2.14): 3.14
*/
```

## Utility functions constrained

Utility functions are most often not used in the enclosing class - if there is any - but with other types. 

Usually using them doesn't make sense but only with certain types. If the number of types is limited enough, or maybe they are even tied to a class hierarchy, it's straightforward how or at least with what you can use the utilities.

But if the available types are broad enough, often they are templatized. In such cases, documentation and (template) parameter names can come to the rescue. It's better than nothing, but not optimal.

As we all learned, the best documentation is code. The best way to document behaviour is through unit tests and through code that expresses its own intentions. If it can make unintended usage impossible, even better! Preferably by compilation errors, or at worst with runtime failures. (Watch [this video](https://www.youtube.com/watch?v=nLSm3Haxz0I) by Matt Godbolt on the topic!)


Concepts provide a concise and readable way to tell the reader about the types that are supposed to be used.

By checking a codebase I often work with, I found some helper functions encoding messages by taking the values from some data objects. The data objects these helper functions can deal with are nowhere listed and the parameter names offer very little help. As the business object taken are also templatized, you'll end up either with a try-and-fail approach our you have to dig deep in the code to understand what it does with the passed-in objects, how they are accessed, etc.

```cpp
template <typename BusinessObject>
void encodeSomeStuff(BusinessObject iBusinessObject) {
  doStuff();
  // ...
}
```

With concepts, we could make this simpler by creating a concept that lists all the characteristics of the business objects this encoder is designed to deal with and that's it!

```cpp
template <typename BusinessObjectWithEncodeableStuff_t>
concept BusinessObjectWithEncodeableStuff = requires (BusinessObjectWithEncodeableStuff_t bo) {
  bo.interfaceA();
  bo.interfaceB();
  { bo.interfaceC() } -> std::same_as<int>;
};


void encodeSomeStuff(BusinessObjectWithEncodeableStuff auto iBusinessObject) {
  doStuff();
  // ...
}
```

Or if the concept would not be used in other places, you might not want to name it, just use it like you would use an [immediately invoked lambda function](https://www.sandordargo.com/blog/2020/02/19/immediately-invoked-lambda-functions) without attaching any name to it.

```cpp
template <typename BusinessObjectWithEncodeableStuff>
requires requires (BusinessObjectWithEncodeableStuff bo) {
  bo.interfaceA();
  bo.interfaceB();
  { bo.interfaceC() } -> std::same_as<int>;
}
void encodeSomeStuff(BusinessObjectWithEncodeableStuff iBusinessObject) {
  doStuff();
  // ...
}
```

Do you see that `requires` is written twice written twice? It's not a typo! This is finally a good place to use nested constraints. We cannot directly use a parameter in a template function with a `requires` clause, but it's possible to use an unnamed constraint, or if you prefer to say so a nested constraint.

With the demonstrated ways, we won't simplify our utilities, but we'll make them self-documenting. By using concepts they reveal with kind of types they were meant to be used. Should you try to compile them with any different parameter, you'll receive quite decent error messages from the compiler.

## Conclusion

Today, in the last part of the C++20 concepts series we saw two real-life examples of how concepts can make our code more expressive, how they can increase the understandability and maintainability of our code.

I hope you enjoyed this series as much as I did, let me know in the comments if you feel that I should have covered some topics more deeply.

If you're looking forward to getting even more examples and more verbose explanations that wouldn't fit the size limits of blog posts, [enter your e-mail address here](https://leanpub.com/cppconcepts) to get notified once my book on concepts is released!

