---
layout: post
title: "C++23: Narrowing contextual conversions to bool"
date: 2022-6-15
category: dev
tags: [cpp, cpp23, conversions]
excerpt_separator: <!--more-->
---
In the previous article discussing new language features of C++23, [we discussed `if consteval`](https://www.sandordargo.com/blog/2022/06/01/cpp23-if-consteval). Today, we'll slightly discuss `if constexpr` and also `static_assert`. Andrzej Krzemieński proposed a paper to make life a bit easier by allowing a bit more implicit conversions. Allowing a bit more narrowing in some special contexts.

## A quick recap

For someone less experienced with C++, let's start with recapitulating what the most important concepts of this paper represent.

### `static_assert`

Something I just learned is that `static_assert` was introduced by C++11. I personally thought that it was a much older feature. It serves for performing compile-time assertions. It takes two parameters
- a boolean constant expression
- a message to be printed by the compiler in case of the boolean expression is `false`. C++17 made this message optional.

With `static_assert` we can assert the characteristics of types at compiler time (or anything else that is available knowledge at compile time.

```cpp
#include <type_traits>

class A {
public:
// uncomment the following line to break the first assertion
// virtual ~A() = default;
};

int main() {
  static_assert(std::is_trivial_v<A>, "A is not a trivial type");
  static_assert(1 + 1 == 2);
}
```

### constexpr if

`if constexpr` is a feature introduced in C++17. Based on a constant expression condition, with *constexpr if* we can select and discard which branch to compile.

Take the following example from [C++ Reference](https://en.cppreference.com/w/cpp/language/if):

```cpp
template<typename T>
auto get_value(T t)
{
    if constexpr (std::is_pointer_v<T>)
        return *t; // deduces return type to int for T = int*
    else
        return t;  // deduces return type to int for T = int
}
```

If `T` is a pointer, then the `template` will be instantiated with the first branch and the `else` part be ignored. In case, it's a value, the `if` part will be ignored and the `else` is kept. `if constexpr` has been a great addition that helps us simplify SFINAE and whatever code that is using `std::enable_if`.

### Narrowing

Narrowing is a type of conversion. When that happens the converted value is losing from its precision. Most often it's something to avoid, just like [the Core Guidelienes says in ES.46](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-narrowing).

Converting a `double` to an `int`, a `long` to an `int`, etc., are all narrowing conversions where you (potentially) lose data. In the first case, you lose the fractions and in the second, you might already store a number that is bigger than the target type can store.

Why would anyone want that implicitly?

But converting an `int` to a `bool` is also narrowing and that can be useful. When that happens `0` is converted to `false`, and anything else (including negative numbers) will result in `true`.

Let's see how [the paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1401r5.html) wants to change the status quo.

## What is the paper proposing

In fact, the proposal of Andrzej might or might not change anything for you depending on your compiler and its version. On the other hand, it definitely makes the standard compiler compliant. 

Wait, what?

Let's take the following piece of code.

```cpp
template <std::size_t N>
class Array
{
  static_assert(N, "no 0-size Arrays");
  // ...
};

Array<16> a;
```

According to - the pre-paper-acceptance version of - the standard, it should fail to compile because `N` which is 16 shouldn't be narrowed to `bool`. Still, if you compile the code with the major implementations, it will compile without any issue.

[The paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1401r5.html) updates the standard so that it matches this behaviour.

The other context where the paper changes the standard is `if constexpr`. Converting contextually a type to bool is especially useful with enums used as flags. Let's have a look at the following example:

```cpp
enum Flags { Write = 1, Read = 2, Exec = 4 };

template <Flags flags>
int f() {
  if constexpr (flags & Flags::Exec) // should fail to compile due to narrowing
    return 0;
  else
    return 1;
}

int main() {
  return f<Flags::Exec>(); // when instantiated like this
}
```

As the output of `flags & Flags::Exec` is an `int`, according to the standard it shouldn't be narrowed down to a `bool`, while the intentions of the coder are evident.

Earlier versions of Clang failed to compile this piece of code, you had to cast the condition to bool explicitly. Yet, later versions and all the other major compilers compiled the successfully.

There are 2 other cases where the standard speaks about *"contextually converted constant expression of type `bool`"*, but the paper doesn't change the situation for those. For more details on that, [check out the paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1401r5.html)!

## Conclusion

[P1401R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1401r5.html) will not change how we code, it will not or just slightly change how compilers work. But it makes the major compilers compliant with the standard. It aligns the standard with the implemented behaviour and will officially let the compilers perform a narrowing conversion to `bool` in the contexts of a `static_assert` or `if constexpr`. Now we can avoid explicitly casting expressions to bool in these contexts without guilt. Thank you, Andrzej!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!