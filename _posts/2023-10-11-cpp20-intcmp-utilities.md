---
layout: post
title: "How to compare signed and unsigned integers in C++20?"
date: 2023-10-11
category: dev
tags: [cpp, cpp20, cleancode, integercomparison]
excerpt_separator: <!--more-->
---
Comparing two numbers should be easy right? Maybe it should, yet it's not the case in C++ even if we constrain the comparison to the domain of integral numbers.

If you try to compare a signed with an unsigned integer there are several possible outcomes. It might actually work and you will never know what you risked. Maybe it will not work as you expected and you'll spend quite some time scratching your head about what just happened. It's also possible that it will not work according to your expectation but it will go unnoticed.

Another option is that you get a compiler warning either by specifically turning on `-Wsign-compare` or `-Wextra`. If it's combined with `-Werror`, the compilation will even break and you must fix it. In this and the coming two articles, we are going to talk about integer comparisons.

Let's start by checking a bit deeper what can go wrong and then we walk through in detail the C++20 solution.

## The problem of comparing a signed with an unsigned

What are integral types in C++?

There are quite a few of them: `bool`, `char`, `char8_t`, `char16_t`, `char32_t`, `short`, `int`, `long`, `long long`, and the unsigned versions of them. Let's put aside that we can also *cv* qualify them and that there are a big bunch of implementation-defined types such as `uint32_t` et al.

Even though these are all integrals, `bool` and `char` types are not meant to store numbers. We are limiting our focus to `short`, `int`, `long`, `long long` and their unsigned versions.

If you take the signed and unsigned versions of the same type, they are going to occupy the same size in memory. For example, both a `short` and an `unsigned short` will need 2 bytes. 2 bytes give us 2\*\*16 possibilities, 2\*\*16 different values to store. Therefore the range of a short is from -2\*\*8 to 2\*\*8-1, while the range of an `unsigned short` is from 0 to 2\*\*16-1.

The bitwise representations of *-1* and *4294967295* are the same, but depending on how those two bytes are flagged, their interpretation will be different. Unsigned integers can only carry non-negative numbers, therefore there is no need to reserve a bit for the sign. The least significant bit represents 2^0, the next one 2^1, then 2^2 and so on.

On the other hand, a signed integer can hold both negative and positive numbers. To be able to represent all of them, it uses the two's complement form.
- The most significant bit represents the sign (0 for positive, 1 for negative numbers)
- To convert a positive value to its negative counterpart, you invert all the bits and then add 1

But what does this mean in practice?

If you try to interpret *-1* as an `unsigned int` - assuming a 4-byte size - the result will be something big, in this case *4294967295*. *-1* is represented as *111111111'111111111'111111111'111111111* in memory. The first byte shows that we deal with a negative value, then we have to negate everything bitwise and subtract -1 to get the value it stores. By negating bitwise we get 0 and if we subtract -1 we are at -1.

But if `111111111'111111111'111111111'111111111` is treated as an unsigned number we simply get the biggest possible positive number that can be represented on 32 bits.

This also means that big enough unsigned numbers cannot be represented in 2's complement form given that the size of the variable stays the same. It's straightforward given that there is one bit reserved to store the sign of the stored number.

This difference in representation makes it potentially unsafe to compare signed and unsigned numbers to each other.

To put in code the above, unless you use `-Werror` the below will compile. If you don't use `-Wextra` or `-Wsign-compare` you won't even get a warning.

```cpp
int main() {
    static_assert(static_cast<unsigned int>(-1) > 42);

    constexpr int n = -1;
    constexpr size_t m = 42;
    static_assert(n > m);
    
    return 0;
}
```

## The modern solution

C++20 introduced the so-called *"intcmp"* functions in the `<utility>` header to provide a safe way to compare signed and unsigned integers and at the same time get mathematically reasonable results. In other words, they will treat `-1` smaller than any non-negative number.

First, let's see what are the available functions and what are their meaning:

|   Function    | Meaning  |
|---------------|--------------|
|  std::cmp_equal(n, m) | n == m |
|  std::cmp_not_equal(n, m) | n != m |
|  std::cmp_less(n, m) | n < m |
|  std::cmp_greater(n, m) | n > m |
|  std::cmp_less_equal(n, m) | n <= m |
|  std::cmp_greater_equal(n, m) | n >= m |

This means that when reading out the function name the relation such as "less" is always compared to the first first parameter. The first parameter is less than or greater than the second.

While these functions are only available since C++20, they are easy to backport as they require no new language features (C++17 suffices) and the reference implementation on C++ Reference is good enough.

Let me first put here the code and then let's go through it.

```cpp
/*
Template parameters T and U should be numbers that we can ensure both with concepts or with static assertions
*/

template<class T, class U>
constexpr bool cmp_equal(T t, U u) noexcept
{
    if constexpr (std::is_signed_v<T> == std::is_signed_v<U>)
        return t == u;
    else if constexpr (std::is_signed_v<T>)
        return t >= 0 && std::make_unsigned_t<T>(t) == u;
    else
        return u >= 0 && std::make_unsigned_t<U>(u) == t;
}
 
template<class T, class U>
constexpr bool cmp_not_equal(T t, U u) noexcept
{
    return !cmp_equal(t, u);
}
 
template<class T, class U>
constexpr bool cmp_less(T t, U u) noexcept
{
    if constexpr (std::is_signed_v<T> == std::is_signed_v<U>)
        return t < u;
    else if constexpr (std::is_signed_v<T>)
        return t < 0 || std::make_unsigned_t<T>(t) < u;
    else
        return u >= 0 && t < std::make_unsigned_t<U>(u);
}
 
template<class T, class U>
constexpr bool cmp_greater(T t, U u) noexcept
{
    return cmp_less(u, t);
}
 
template<class T, class U>
constexpr bool cmp_less_equal(T t, U u) noexcept
{
    return !cmp_less(u, t);
}
 
template<class T, class U>
constexpr bool cmp_greater_equal(T t, U u) noexcept
{
    return !cmp_less(t, u);
}
```

We only have to understand `cmp_equal` and `cmp_less` as the rest is implemented with the help of these two.

### `cmp_equal`

Let's repeat the reference implementation of `cmp_equal`:

```cpp
template<class T, class U>
constexpr bool cmp_equal(T t, U u) noexcept
{
    if constexpr (std::is_signed_v<T> == std::is_signed_v<U>)
        return t == u;
    else if constexpr (std::is_signed_v<T>)
        return t >= 0 && std::make_unsigned_t<T>(t) == u;
    else
        return u >= 0 && std::make_unsigned_t<U>(u) == t;
}
```

What we first see when we look at the function is that we use type traits to decide how to perform a comparison:
- if both numbers are signed or both are unsigned, then we simply check if they are equal or not
- if only the first parameter is a signed number then we check if it's non-negative and by casting it to an unsigned type does that equal to the second value
- if only the second parameter is a signed number, we do the same thing as in the previous case, just by replacing the role of the two parameters.

Let's play with these a bit, what do they mean in practice?

Let's ignore the case when we pass two numbers with the same signedness.

If we compare `int{-5}` and `unsigned int{5}`, we go to the second branch and we fail on the first condition as `t` is a negative number.

What happens if we pass `int{5}` and `unsigned int {5}`. We go to the second branch and the first condition evaluates to `true`. If we cast `int{5}` to `unsigned int`, it will keep its value and we can safely check if they are equal.

If we swap the two parameters, we could observe the same set of events.

Let's go to `cmp_less`, it's probably more interesting.

### `cmp_less`

Let's put here the reference implementation as a reminder:

```cpp
template<class T, class U>
constexpr bool cmp_less(T t, U u) noexcept
{
    if constexpr (std::is_signed_v<T> == std::is_signed_v<U>)
        return t < u;
    else if constexpr (std::is_signed_v<T>)
        return t < 0 || std::make_unsigned_t<T>(t) < u;
    else
        return u >= 0 && t < std::make_unsigned_t<U>(u);
}
```

Again, at first glance, we can see that we have three compile-time `if` branches based on the signedness of the two parameters.
- If both are signed or if both are unsigned, we do a simple comparison
- If only the first one is signed, we check whether it's a negative number or(!) if we cast it to the unsigned version, is it smaller than `u`
- If only the second parameter is signed, then we check whether it's positive and if it is, then we cast it to unsigned and check if it's bigger than `t`

This implementation is interesting because the second and third branches follow a different logic.

Let's go through them step by step.

If we compare `int{-5}` and `unsigned int{5}`, we go to the second branch. `t`'s type is signed and its value is negative, so we know that it must be smaller than any `u` as it's an unsigned value, it cannot be negative. We can stop there, we know that `t < u`.

If we compare `int{5}` and `unsigned int {5}`, we still go to the second branch, but as `t` is non-negative, we can safely cast to its unsigned type and compare it to `u`.

If we compare `unsigned int{5}` to `int{-5}`, we go to the third branch. We check if `u` is non-negative, but it's not, so we can stop there because we know that `t` which is unsigned in that case, must be greater.

If we compare `unsigned int{5}` and `int {5}`, we go again to the second branch. As `u` is positive, we can safely cast it to its unsigned type and compare it to `t`.

This implementation is safe, smart and correct. And if you go it through and take the time to understand it, it's also straightforward. But it also shows that probably if you try to do this every time on your own, there is a fair chance that you'll make some mistakes so [it's better to use the standard version](https://www.sandordargo.com/blog/2020/05/13/loops-vs-algorithms) now that we have one.

## Conclusion

In this article, we saw why it's error-prone to compare two integral numbers with differently signed types and how the same bitwise representations can be interpreted as two completely different numbers. Then we saw that C++20's *intcmp* utilities solve this issue and we also had a deep dive into the implementation of these new utility functions.

Comparing integers with different signs is not simply error-prone, but depending on your compilation settings it might also emit warnings or even errors. In the next two articles, I'll share with you what are the most common and most horrendous violations I've seen so far while I was trying to remove such warnings over the last few years.

Stay tuned.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!