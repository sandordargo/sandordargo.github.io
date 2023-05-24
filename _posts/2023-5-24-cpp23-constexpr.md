---
layout: post
title: "C++23: Even more constexpr"
date: 2023-5-24
category: dev
tags: [cpp, cpp23, constexpr, compiletime]
excerpt_separator: <!--more-->
---
Ever since C++ introduced the `constexpr` keyword in C++11, each new standard brought us more and more opportunities to make our code increasingly `constexpr`, in other words, compile-time execution friendly.

In this article, we are going to review briefly what changes with C++23 on this front.

## Non-literal variables in `constexpr` functions

[P2242R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2242r3.html) proposes to remove the restriction that a `constexpr` function cannot contain 
- a definition of a variable of a [non-literal type](https://en.cppreference.com/w/cpp/named_req/LiteralType)
- a definition of a variable of static or thread storage duration,
- or a goto statement,
- or an identifier label.

The rationale behind this change is that the presence of the listed things in a function is not a problem as long as they are not evaluated at compile-time. We should remind ourselves that a `constexpr` function may or may not be evaluated at compile-time.

Let's suppose that in a `constexpr` function we want to call a piece of code that is guaranteed to be evaluated at compile-time. Then we need to have that piece of code in a block under the condition of either [`if consteval`](https://www.sandordargo.com/blog/2022/06/01/cpp23-if-consteval) or `if (std::is_constant_evaluated())`. 

With this paper, the following code becomes valid. 

```cpp
template<typename T> constexpr bool f() {
  if (std::is_constant_evaluated()) {
    // ...
    return true;
  } else {
    T t; // This could have been problematic before
    // ...
    return true;
  }
}
struct nonliteral { nonliteral(); };
static_assert(f<nonliteral>());
```

As `nonliteral` is a non-literal type, the compilation should fail without this proposal, even though the line that causes the failure is not in a constant-evaluated context.

This change is available starting from GCC 12 and Clang 15.

## Relaxing some more `constexpr` restrictions

As I mentioned in the intro, every new standard relaxes a bit `constexpr` conditions and in fact, even the previous section discussed relaxations. But there is more. Thanks to [Barry Revzin](https://brevzin.github.io/) and [P2448R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2448r2.html).

Let's see the two main issues the paper wants to solve.

### Functions that are not *yet* `constexpr`

As I wrote earlier, a `constexpr` function might be evaluated at compile-time but might not. Whether or not a call can be constant-evaluated might depend on the parameters passed in. Often it seems straightforward that a function cannot be constant-evaluated because of the non-`constexpr` functions it calls. But never say never! More and more functions in the standard library become `constexpr` and conditionally marking functions `constexpr` - as in the next happens - is painful:

```cpp
#include <optional>

#if __cpp_lib_optional >= 202106
constexpr
#endif
void h(std::optional<int>& o) {
    o.reset();
}
```

### Explicitly defaulted functions follow different rules

Another problem Barry wanted to address is related to explicitly defaulted functions. Let's use the following template to demonstrate the issue:

```cpp
template <typename T>
struct Wrapper {
    constexpr Wrapper() = default;
    constexpr Wrapper(Wrapper const&) = default;
    constexpr Wrapper(T const& t) : t(t) { }

    constexpr T get() const { return t; }
    constexpr bool operator==(Wrapper const&) const = default;
private:
    T t;
};
```

If you mark a member function `constexpr`, it means that it *might* be evaluated in a constant-evaluation context.

On the other hand, if you `default` a member function, it means that it must be `constexpr`-compatible in all instantiations. Yet, not all the compilers complain if you violate this rule and those that raise an error, they don't raise the error in each case.

In the case of a template, this means that all instantiations should be `constexpr`-compatible. If that's not possible, the current solution is to remove the `constexpr` from the explicitly defaulted constructors and `operator==()`. We might end up with an instantiation where they would be usable in a constant-evaluation context, yet they are not marked as `constexpr`

### What is changing after all?

With the acceptance of [this proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2448r2.html), there are a couple of restrictions removed.

Constructors and destructors will follow the same rules as functions in terms of `constexpr`. In addition, an explicitly defaulted function on its first declaration will not only be implicitly inline but also implicitly `constexpr` if it satisfies the requirements of a `constexpr` function.

And even `constexpr` functions will get some relaxed rules - beyond the relaxation presented in the previous section. They can return and/or take as parameters [non-literal types](https://en.cppreference.com/w/cpp/named_req/LiteralType). 

This change is available starting from GCC 13.

## Permitting static `constexpr` variables in `constexpr` functions

[P2647R1](https://wg21.link/P2647R1) corrects a hole in the standard. As the proposal says, there is no good reason why a `constexpr` function cannot have today a `static constexpr` local variable.

While this is fine:

```cpp
char xdigit(int n) {
  static constexpr char digits[] = "0123456789abcdef";
  return digits[n];
}
```

Its `constexpr` version wouldn't compile without some workarounds presented in the [paper](https://wg21.link/P2647R1:

```cpp
constexpr char xdigit(int n) {
  static constexpr char digits[] = "0123456789abcdef";
  return digits[n];
}
```

Not anymore! The above code is becoming legit. GCC 13 and Clang 16 already support it!

## `constexpr type_info::operator==()`

At the moment, `typeid` is allowed in constant expressions, but the returned [`std::type_info`](https://www.sandordargo.com/blog/2023/03/01/binary-sizes-and-rtti#typeid-and-stdtype_info) object has no `constexpr` methods, therefore it's not practically usable in a `constexpr` context.

[P1328R1](https://wg21.link/P1328R1) makes the equality operator (`operator==()`) `constexpr` to `std::type_info` becomes practically usable in compile-time functions!

It's fascinating how far the `constexpr` came. I mean, `typeid()` and `std::type_info` are used for [Run-Time Type Information (RTTI)](https://www.sandordargo.com/blog/2023/03/01/binary-sizes-and-rtti) and yet we talk about `constexpr`...

GCC 12, Clang 17 and MSVC 19.33 already provide this feature!

## `constexpr` for `<cmath>` and `<cstdlib>`

Until now, `<cmath>` and `<cstdlib>` has barely contained any `constexpr` functions. [P0533R9](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0533r9.pdf) aims to improve on this situation in order to facilitate compile-time programming. These headers have been simply neglected so far, otherwise, there is no reason why `std::chrono::abs` is `constexpr` but `std::abs` is not.

In (8.E-G sections of [P0533R9](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0533r9.pdf) you can find the full list of functions that are going to become `constexpr`. Luckily it's quite a long list!

This is yet to be implemented by the compilers!

## `constexpr std::unique_ptr`

[P2273R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2273r3.pdf) is bringing us `constexpr` unique pointers. The requirements of `constexpr` were loosened by [P0784R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0784r7.html), adopted by C++20. With that `new` and `delete` might be used in certain `constexpr`-contexts, so it was worth trying making `std::unique_ptr` also `constexpr`. It worked out fine and it's already available in the new versions of major compilers.

The authors also tried to make `shared_ptr` `constexpr`, but due to missing compile-time atomic support it's not possible for the moment to implement `shared_ptr` according to the standard. The authors are going to explore their possibilities further.

GCC 12, Clang 16 and MSVC 19.33 already provide this feature!

## `constexpr` for integral overloads of `std::to_chars()` and `std::from_chars()`

At compile-time, there is currently no standard way to make conversions between numbers and strings. Now that a `std::string` can be instantiated at compile-time, we are much closer to get a `constexpr std::format` and this is a gap.

`std::to_chars` and `std::from_chars` are fundamental blocks for parsing and formatting as they are locale-independent, they don't throw and don't allocate memory. Except for the floating point overloads now they will become `constexpr` thanks to [P2291R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2291r3.pdf).

They will be another step forward to have `constexpr std::format`. 

GCC 13, Clang 16 and MSVC 19.34 already provide this feature!

## `constexpr std::bitset`

[P2417R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2417r2.pdf) extends the `constexpr` interface of `std::bitset`. So far, only one of the constructors and `operator[]` was marked as `constexpr`. However, [since `std::string` can be `constexpr`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0980r1.pdf), all the internals - and therefore the full API - of `std::bitset` can be `constexpr`.

GCC 13, Clang 16 and MSVC 19.34 already provide this feature!

## DR: constexpr for std::optional and std::variant

[P2231R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2231r1.html) adds some missing `constexpr` both to `std::optional` and `std::variant`. As the already referenced [P0784R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0784r7.html) made it possible to use `new` and `delete` at compile-time (by using `std::construct_at`), there was no reason to not making `optional` and `variant` fully constexpr.

GCC 11, Clang 13 and MSVC 19.31 already provide this fix!

## Conclusion

In this article, we reviewed the `constexpr` related changes in C++23. There are quite many of them, and our options for compile-time programming are growing! You can check on [C++ Reference](https://en.cppreference.com/w/cpp/23) whether they are implemented by your compiler of choice.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!