---
layout: post
title: "My battle against signed/unsigned comparison: the most usual violations"
date: 2023-10-18
category: dev
tags: [cpp, cpp20, cleancode, integercomparison]
excerpt_separator: <!--more-->
---
As we discussed last week, comparing numbers with different signs can be dangerous in C++. If you try to compare a signed with an unsigned integer, you might get a result that makes no sense if you approach the question from a mathematical point of view. At least, with the right compiler settings, you'd get a warning.

We also saw that C++20 offers an easy and safe way to compare numbers anytime and it will always return you the result you'd expect.

As such, the warnings also go away.

Whatever project I work on, introducing or using a stricter set of warnings and treating them as errors come up over time. An unsafe integer comparison is one of the nastiest warnings to fix because the fix is sometimes not evident but at the same time, usually, you have a lot of similar warnings.

In this article, I'll share with you according to my experience the 3 most common ways `-Wsign-compare` warnings are invoked.

## Using a wrongly typed loop control variable

The simplest and the most frequent form of this abuse is when the loop control variable is wrongly typed. Most often it means that the loop control is either declared as `int i = 0` or `auto i = 0` which are both signed integers, but they are compared against the size of a standard container which is a `size_t`.

```cpp
std::vector<int> v; 

for(int i = 0; i < v.size(); ++i) {
  std::cout << i;
}
```

This is not very dangerous because on most platforms an `int` is 4 bytes and a `size_t` is an `unsigned long long` that is 8 bytes. As such, in the comparison `i` is statically cast to an `unsigned long` or `unsigned long long` which is safe as long as `i` is positive. If `i` is negative you will have surprises, but if you modify `i` in the body of your loop, you have bigger design issues.

Another possible problem is that a signed `int` and therefore `i` cannot represent all the positive values `size_t` can, so with numbers that require a larger type than `int` to represent positive values, this comparison will fail. To be fair, that is rarely a problem.

```cpp
#include <iostream> 
#include <limits> 

int main()
{
    std::cout << "std::numeric_limits<int>::max(): "    << std::numeric_limits<int>::max() << '\n';
    std::cout << "std::numeric_limits<size_t>::max(): " << std::numeric_limits<size_t>::max() << '\n';
    return 0;
}

/*
std::numeric_limits<int>::max(): 2147483647
std::numeric_limits<size_t>::max(): 18446744073709551615
*/
```

To fix this issue you simply have to change the declaration of `i` to `size_t i = 0` or to `auto i = 0uz` ([on C++23](https://www.sandordargo.com/blog/2022/05/25/literal_suffix_for_signed_size_t)), `auto i = 0u` is not a good option, [read here why](https://www.sandordargo.com/blog/2022/05/25/literal_suffix_for_signed_size_t). 

I also saw that some user-defined containers return `int` or `ssize_t` as a length, and the loop control variable is defined as `size_t`. As long as the maximum - due to a potential error condition - is not negative, all is fine. But if the `ssize_t length()` method might return `-1` for example, you'll end up with a loop that will go until one before the maximum value of a `size_t` (`std::numeric_limits<size_t>::max()-1`) 

This is also easy to fix, you simply have to match the type of the loop index with the type of the other value in the stop condition.

## Mixed conditions in loop control

A bit more complex and slightly less frequent case is when you set a more complex stop condition in a loop.

Take this example:

```cpp
#include <iostream>

class Container {
    public:
    size_t length() const {
        return 42;
    }
};

int maxLength() {
    return 576;
}

int main() {
   Container c;
   for (size_t i = 0; i < c.length() &&  i <= maxLength(); ++i) {
     std::cout << i << '\n';
   }
   return 0;
}
```

You cannot fix such an error by modifying the type of the loop index. If `i` is signed, the first condition will emit a signed/unsigned comparison warning, if it's unsigned, then the second one will.

The fix depends on your possibilities in terms of modifying code.

If you can safely and rightly change the type of one of the variables in the expression, then do so. In our case that would either be the return type of `Container::length()` or the return type of `maxLength()`.

If you cannot do that for any reason, you have to make that comparison safe one way or another.

If you can, use [the standard integer comparison tools that I wrote about last week](https://www.sandordargo.com/blog/2023/10/11/cpp20-intcmp-utilities). In the above example, the `for`-loop control block would look like this:

```cpp
for (size_t i = 0; i < c.length() && std::cmp_less_equal(i, maxLength()); ++i)
```

That's safe and still quite readable.

If you don't have access to it, backport it. You have a reference implementation on C++ Reference and you can check the actual implementations on GitHub ([clang here](https://github.com/llvm/llvm-project/blob/main/libcxx/include/__utility/cmp.h), [gcc here](https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/std/utility), [msvc here](https://github.com/microsoft/STL/blob/main/stl/inc/utility)).

If you don't want to backport it... I think you really should... But let's imagine that you don't. In that case, use a `static_cast` to cast one of the values in the conditional to overcome the warning. To choose your cast, you have to think wisely to make sure that nothing can go wrong considering the potential values of both variables.

In our case, we should ask ourselves the question if it's safe to cast `i` to an `int`. For that we have to know if `i` can have a value that is bigger than `std::numeric_limits<int>::max()`. If it might be larger, then casting it to `int` potentially leaves us with a negative value that we clearly don't want. If we know for sure that the container in this case cannot be so large (because for example, it contains the number of countries in the world), we can go ahead. We can also cast it to a larger type. So for example instead of an `int` we can use a `long long`.

Or we have to consider if it's safe to cast the other value, in this case, that would mean casting `maxLength()` to a `size_t`. Again, for that, we have to know if realistically it can return a negative value. If so, it's an unsafe cast.

It's better to backport those new integer comparison utilities.

## Saving values into the wrong type

Another frequent root cause behind signed/unsigned comparisons is that one of the variables participating in the comparison is saved into the wrong type. The called method would return the right type, but it's saved into a wrongly typed variable.

```cpp
#include <iostream>
#include <vector>

size_t foo() {
    return 42;
}

int main() {
   std::vector<int> v;

   int bar = foo();

   if (v.size() == bar) {
     std::cout << "same size\n";
   } else {
     std::cout << "different size\n";
   }
}
```

You'd be surprised to see how frequently this is the case and how ignorant we - developers - are.

As `std::vector<T>::size()` returns `size_t` and `bar` is an `int`, we have a warning. But in such cases, it's easy to fix the problem, we simply have to update the declaration of `bar` so that it matches the return type of `foo` and the problem is gone.

## Conclusion

In this article, we reminded ourselves how comparing signed with unsigned types can go wrong. Then I shared with you the 3 most common cases I encountered while fighting against `-Wsign-compare` warnings.

They are often trivial to fix and they are often the result of ignorance and bad compiler settings. But there might be some cases when using [the C++ intcmp utilities](https://www.sandordargo.com/blog/2023/10/11/cpp20-intcmp-utilities) is the best way to go.

Next week, let's continue with the three strangest cases I encountered battling signed/unsigned comparison warnings.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!