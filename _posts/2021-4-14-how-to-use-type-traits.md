---
layout: post
title: "How to use type traits?"
date: 2021-4-14
category: dev
tags: [cpp, typetraits, templates]
excerpt_separator: <!--more-->
---
As a spin-off of the [concepts series](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations), I delved into the world of type traits and last week we started to discuss what type traits are and how they are implemented.
<!--more-->

As I prefer to keep my articles somewhere between 5 and 10 minutes of reading time, I decided to stop right there. With the basic understanding of type traits, now it's time to see how to use them. We are going to see how they can set conditions for compiling different template specializations and then how they can alter types.

## Conditional compilation

As we've already mentioned we can use type traits to disallow the usage of templates with certain types based on their characteristics. Just to emphasize, this has no runtime costs, all the checks (and errors) happen at compile-time.

Let's see a basic example.

Let's say that we want to write a function called `addSigned(T a, T b)` where we only add unsigned number thus we are sure that the result is bigger than any of the inputs (we ignore overflow errors).

If we write a simple template, the problem is we can still call it with unsigned numbers.

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
T addUnsigned(T a, T b) {
    return a + b;
}


int main() {
    int a = 5;
    int b = -6;
    auto s = addUnsigned(a, b);
    if (s < a || s < b) {
        std::cout << "Oh, oh! The sum is smaller than one of the inputs!\n";
    } else {
        std::cout << "OK! The sum is larger than any of the inputs!s\n";
    }
}
/*
Oh, oh! The sum is smaller than one of the inputs!
*/
```

Type traits can help us solve this issue in different ways.

### static_assert

We can simply statically assert that `T` is an unsigned type.

```cpp
template <typename T>
T addUnsigned(T a, T b) {
    static_assert(std::is_unsigned<T>::value, "T must be unsigned!" );
    return a + b;
}
```

It's worth reminding ourselves, that when used in a boolean context, we can't simply use `std::is_unsigned<T>` as it's already a type that is not boolean - it inherits from `std::integral_constant` - but we need its `value` static member constant that is a `bool`. Since C++17 we can use `std::is_unsigned_v<T>` directly.

So `static_assert` takes the compile-time boolean as a first parameter and an error message as the second parameter.

Then if we use it with some other types, we'll get the - hopefully - nice error message from the compiler.

```
main.cpp: In instantiation of 'T addUnsigned(T, T) [with T = int]':
main.cpp:14:30:   required from here
main.cpp:6:40: error: static assertion failed: T must be unsigned, but it's
    6 |     static_assert(std::is_unsigned<T>::value, "T must be unsigned, but it's");
      |                     
```

If you think that the error message is not good enough, just write a better one as it's taken from your `static_assert`.

### std::enable_if

Now let's say that we want to support different additions and we want to use the same function signature `T add(T a, T b)`. We can use the `std::enable_if` metafunction from the `<type_traits>` header.

```cpp
#include <iostream>
#include <type_traits>

template <typename T, typename std::enable_if<std::is_unsigned<T>::value, T>::type* = nullptr>
T add(T a, T b) {
    std::cout << "add called with unsigned numbers\n";
    return a + b;
}

template <typename T, typename std::enable_if<std::is_signed<T>::value, T>::type* = nullptr>
T add(T a, T b) {
    std::cout << "add called with signed numbers\n";
    return a + b;
}

int main() {
    add(5U, 6U);
    add(5, 6);
    add(5, -6);
    // add(5U, -6); // error: no matching function for call to 'add(unsigned int, int)'
}
/*
add called with unsigned numbers
add called with signed numbers
add called with signed numbers
*/
```

We can see that we were able to define two functions with the same signature, while only the template parameter list is different. There we used `enable_if` to express that one or the other function should be called in case the `is_signed` or `is_unsigned` trait is evaluated to true.

In case, `std::enable_if` receives `true` as its first argument, then it will have an internal `type` that is taken from the second argument. If its first argument evaluates to `false`, then it doesn't have an internal `type` and the substitution fails. In order not to end up with a compilation error we default these types to `nullptr`.

I know this is still a bit vague, but this part that is often referred to as SFINAE deserves its own article. Something we are going to cover in detail in the coming weeks.

### if constexpr

Since C++17, there is a third way, as we have `if constexpr` at our hands. With `if constepxr` we can evaluate conditions at compile time and we can discard branches from the compilation. With `if constexpr` you can significantly simplify obscure metaprogramming constructs.

Let's see how we can use it to use it to cut down our previous example:

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
T add(T a, T b) {
    if constexpr (std::is_signed<T>::value) {
        std::cout << "add called with signed numbers\n";
        return a + b;
    }
    if constexpr (std::is_unsigned<T>::value) {
        std::cout << "add called with unsigned numbers\n";
        return a + b;
    }
    static_assert(std::is_signed<T>::value || std::is_unsigned<T>::value, "T must be either signed or unsingned!");
}


int main() {
    add(5U, 6U);
    add(5, 6);
    add(5, -6);
    // add(5U, -6); // error: no matching function for call to 'add(unsigned int, int)'
    // add("a", "b"); // error: static assertion failed: T must be either signed or unsingned!
}
/*
add called with unsigned numbers
add called with signed numbers
add called with signed numbers
*/
```

With `if constexpr` we can evaluate conditions at compile-time and as such we can make compile-time decisions based on the type traits. I'm sure I'm not alone considering it much simpler to read than `enable_if`

Could we make it simpler? Yes and that's true for all the previous examples. Since C++17 there is a shortcut I already referred to, you don't have to access `value` in a type_trait, there are metafunctions to return the value directly. They are called the same way as the corresponding type traits, but appended with `_v`:

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
T add(T a, T b) {
    if constexpr (std::is_signed_v<T>) {
        std::cout << "add called with signed numbers\n";
        return a + b;
    }
    if constexpr (std::is_unsigned_v<T>) {
        std::cout << "add called with unsigned numbers\n";
        return a + b;
    }
    static_assert(std::is_signed_v<T> || std::is_unsigned_v<T>, "T must be either signed or unsingned!");
}


int main() {
    add(5U, 6U);
    add(5, 6);
    add(5, -6);
    // add(5U, -6); // error: no matching function for call to 'add(unsigned int, int)'
    // add("a", "b"); // error: static assertion failed: T must be either signed or unsingned!
}
/*
add called with unsigned numbers
add called with signed numbers
add called with signed numbers
*/
```

## Altering types

Now let's have a look at how type traits can alter types. There are templates shipped in the `<type_traits>` header that can
- add or remove `const` and/or `volatile` specifiers from a given type
- add or remove reference or pointer from a given type
- make a type signed or unsigned
- remove dimensions from an array
- etc. (including enable_if, that we already saw briefly)

Let's see three examples.

### Adding/removing the const specifier

With `std::add_const`/`std::remove_const` you can add/remove the topmost const of a type:

```cpp
#include <iostream>
#include <type_traits>
 
int main() {
    using Integer = int;
    
    std::cout << "Integer is " << (std::is_same<int, Integer>::value
        ? "int" : "not an int") << '\n';
    std::cout << "The result of std::add_const<Integer> is " << (std::is_same<const int, std::add_const<Integer>::type>::value
        ? "const int" : "not const int") << '\n';
    std::cout << "The result of std::add_const<Integer> is " << (std::is_same<int, std::add_const<Integer>::type>::value
        ? "a simple int" : "not a simple int") << '\n';        
        
    using ConstInteger = const int;
    
    std::cout << "ConstInteger is " << (std::is_same<const int, ConstInteger>::value
        ? "const int" : "not a const int") << '\n';
    std::cout << "The result of std::remove_const<ConstInteger> is " << (std::is_same<int, std::remove_const<ConstInteger>::type>::value
        ? "int" : "not an int") << '\n';
}
/*
Integer is int
The result of std::add_const<Integer> is const int
The result of std::add_const<Integer> is not a simple int
ConstInteger is const int
The result of std::remove_const<ConstInteger> is int
*/
```
When you make comparisons, make sure that you access the `type` nested member. Since C++17 you can directly get the type by using `std::add_const_t` instead of `std::add_const<T>::type` to keep things shorter and more readable.

But how can this be useful? The above example already sparks an answer. If you want to compare two types regardless of their qualifiers, first you can remove the `const` qualifiers and make the comparison with `std::is_same` only after. Without calling `std::remove_const`, you might compare `T` with `const T` which are different, but after calling it, you'd compare `T` with `T`.

Following the same logic, you can find a use case for removing references or pointers as well.

### Turning an unsigned number into a signed one

You can use type traits to turn a signed type into an unsigned one or the other way around.

```cpp
#include <iostream>
#include <type_traits>
 
int main() {
    
    std::cout << "Making signed to unsigned " << (std::is_same<unsigned int, std::make_unsigned_t<int>>::value
        ? "worked" : "did not work") << '\n';
    std::cout << "Making unsigned to signed " << (std::is_same<int, std::make_signed_t<unsigned int>>::value
        ? "worked" : "did not work") << '\n';
}
/*
Making signed to unsigned worked
Making unsigned to signed worked
*/
```
As you can see, we used the `_t`-style helper functions to get back directly the modified type.

### std::conditional to choose between two types at compile time 

With `std::conditional` you can choose between two types based on a compile time condition. You can imagine it as the compile-time ternary operator though probably it's a bit more difficult to read.

```cpp
#include <iostream>
#include <type_traits>
#include <typeinfo>
 
int main() 
{
    typedef std::conditional<true, int, double>::type Type1;
    typedef std::conditional<false, int, double>::type Type2;
    typedef std::conditional<sizeof(int) >= sizeof(double), int, double>::type Type3;
 
    std::cout << typeid(Type1).name() << '\n';
    std::cout << typeid(Type2).name() << '\n';
    std::cout << typeid(Type3).name() << '\n';
}
/*
i
d
d
*/
```

You can find examples where based the condition is the size of the passed in type. There might be cases, where you want to choose a type based on that for example to have better padding, to fit more the memory layout. How to make a decision based on the size? It's very simple, just use the `sizeof` operator:

```cpp
#include <iostream>
#include <type_traits>
#include <typeinfo>

class SmallSize{};
class BigSize{};

template <class T>
using ContainerType =
typename std::conditional<sizeof(T) == 1, SmallSize, BigSize>::type;
 
int main()
{
    ContainerType<bool> b;
    std::cout << typeid(b).name() << '\n';
    
    ContainerType<int> i;
    std::cout << typeid(i).name() << '\n';
}
/*
9SmallSize
7BigSize
*/
```

## Conclusion

Today we had a look into how to use type traits for conditional compilation and how to use them to alter types. We mentioned SFINAE as well, which will be the topic in a couple of weeks. 

Stay tuned!