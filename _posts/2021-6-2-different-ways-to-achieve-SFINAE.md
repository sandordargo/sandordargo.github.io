---
layout: post
title: "Different ways to achieve SFINAE"
date: 2021-6-2
category: other
tags: [cpp, typetraits, templates, SFINAE]
excerpt_separator: <!--more-->
---
Life is a chain of opportunities. Each task you take on will lead you to more doors hiding other opportunities. Some are worth opening, some are not.
<!--more-->

Proofreading [C++20: Get the Details by Rainer Grimm](https://www.sandordargo.com/blog/2021/04/03/cpp20-get-the-details-rainer-grimm) led me to [concepts](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations). Concepts led me to type traits and type traits led me to the door of the dreaded word that I passed on by many times. I looked at it, I tried to have a peek inside, but I never put my feet in.

That word is SFINAE.

Repeat with me:

**Substitution Failure Is Not An Error**

SFINAE came up when [we introduced `std::enable_if`](https://www.sandordargo.com/blog/2021/04/14/how-to-use-type-traits). It helps to have different overloads for templates.

Let's say that a template has several overloads and you make a call. The compiler will start substituting the template parameters with the provided types or values. If the substitution leads to invalid code, the compilation will not fail, it'll not be reported as an error because a *substitution failure is not an error*. Instead, the substitution will continue with the other available overloads as long as there is any left.

I won't share with you the old tricks to do SFINAE, in 2021 I don't really find them relevant. Instead, I want to share with you different possibilities we have at our hands since C++11 - which is considered the first modern C++ standard.

# Basic SFINEA with function parameter list

Probably the simplest example to demonstrate SFINEA is when we use only the template parameter list and the function parameter list without calling any template metafunctions.

We provide 2 overloads for `foo()`, both take one template parameter `T` and an instance of `T`. As a second parameter, one of the overloads takes `T::type` while the other `T::other_type`. 

In case `T` doesn't have a member type `type`, the substitution fails, but we receive no immediate compiler error. Instead, it will try to match `T` with the other overload just as we are going to see in the example below.

On the other hand, if all the available substitutions fail, the compiler cannot do anything else then throw an error.

```cpp
#include <iostream>

class MyType {
public:
    using type = char;
};

class MyOtherType {
public:
    using other_type = int;
};

template<typename T>
void foo(T bar, typename T::type baz)
{
    std::cout << "void foo(T bar, typename T::type baz) is called\n";
}

template<typename T>
void foo(T bar, typename T::other_type baz)
{
    std::cout << "void foo(T bar, typename T::other_type baz) is called\n";
}


int main()
{
    MyType m;
    MyOtherType mo;
    foo(m, 'a');
    foo(mo, 42);
    // error: no matching function for call to 'foo(MyOtherType&, const char [3])'
    // foo(mo, "42");
}
/*
void foo(T bar, typename T::type baz) is called
void foo(T bar, typename T::other_type baz) is called
*/
```

# SFINAE with `decltype`

In the previous example, we used the parameter list for having SFINAE. It might not be very convenient, especially if we don't plan to use those values passed in for the different substitutions.

Another way is to use the return type for SFINAE.

First, let's see the code.

```cpp
#include <iostream>

class MyType {
public:
    using type = char;
};

class MyOtherType {
public:
    using other_type = int;
};

template<typename T>
decltype(typename T::type(), void()) foo(T bar)
{
    std::cout << "decltype(typename T::type(), void()) foo(T bar) is called\n";
}

template<typename T>
decltype(typename T::other_type(), void()) foo(T bar)
{
    std::cout << "decltype(typename T::other_type(), void()) is called\n";
}


int main()
{
    MyType m;
    MyOtherType mo;
    foo(m);
    foo(mo);
    // error: no matching function for call to 'foo(MyOtherType&, const char [3])'
    // foo(mo, "42");
}
```
We are using `decltype` and as a first argument, we pass in what we want to use for the substitution.

In case `decltype` gets multiple arguments separated by commas, each of them will be evaluated, but only the last one will be considered as a type. Hence as the first argument, we pass in the type for substitution, if the substitution succeeds, the next parameter gets evaluated that is for the actual return type of the function.

We put parentheses after each parameter because we need an expression that decltype can take the type of.

In the above case, we SFINAE-d based on an inner type. In case we need to check that a function exists we might need as well `std::declval`. `std::declval` converts any type `T` to a reference type, making it possible to use member functions in decltype expressions without the need to go through constructors.

In case our `T` should have a function `fun()`, we could have written such a decltype expression: `decltype(std::declval<T>().fun(), void())`.

I like this way of SFINAE because it's not polluting the parameter list, but at the same time, it's true that the return type is quite a bit obfuscated. 

# SFINAE with `std::enable_if`

We can use `std::enable_if` for activating a piece of code and for using SFINAE since C++11, though it was part of `boost` even before.

`enable_if` takes two parameters, the first is a boolean expression and the second is a type. If the boolean expression evaluates to `true` then then `enable_if` has an inner type `type` that is taken from the parameter. Otherwise, if the boolean expression is false, then there is no inner type.

Speaking of boolean expressions, we can easily use `enable_if` with type traits and specialize our functions based on type characteristics.

Let's say that we have a function `add()` that takes two parameters and adds them up. Let's suppose you want to implement two versions based on whether the parameters are integral or floating-point numbers.

```cpp
template<typename T>
std::enable_if_t<std::is_integral<T>::value> f(T t){
    //integral version
}
template<typename T>
std::enable_if_t<std::is_floating_point<T>::value> f(T t){
    //floating point version
}
```

As we omitted the second parameter of `std::enable_if`, the return type is automatically `void`. Let's fix that:

```cpp
template<typename T>
std::enable_if<std::is_integral<T>::value, T>::type f(T t){
    //integral version
}
template<typename T>
std::enable_if<std::is_floating_point<T>::value, T>::type f(T t){
    //floating point version
}
```

And if we want to avoid putting `::type` at the end, we have the `std::enable_if_t` helper at our hands:

```cpp
template<typename T>
std::enable_if_t<std::is_integral<T>::value, T> f(T t){
    //integral version
}
template<typename T>
std::enable_if_t<std::is_floating_point<T>::value, T> f(T t){
    //floating point version
}
```

Another possibility is that you have a template class where you have a generic implementation for a function, but you also want an overload based on the characteristics of the template argument.

It's not going to be something very nice.

```cpp
template<typename T>
class MyClass {
public:
    void f(T x) {
        std::cout << "generic\n"; 
    }

    template<typename T_ = T>
    void f(T x,
           typename std::enable_if<std::is_floating_point<T_>::value,
           std::nullptr_t>::type = nullptr) {
        std::cout << "with enable_if\n"; 
    }
};

```

I warned you.

You might wonder about `template<typename T_ = T>`. `T` is the template type of the class, not the type of the method. Using SFINAE requires a template context, hence we have to turn the function into a template itself and in order to keep the caller side as simple as possible we make default `T_`'s type to `T`. You can [read more on this example on Fluent C++](https://www.fluentcpp.com/2018/05/15/make-sfinae-pretty-1-what-value-sfinae-brings-to-code/).

The other fishy thing is all those `nullptr`s. It'd be simpler to set the second function parameter simply `void`, but as a function parameter cannot be void and we are lazy to define a separate empty type for this purpose, the easiest thing is to use `nullptr`.

This solution has some drawbacks. It's complex, verbose and therefore not easily maintainable.

The future is luckily brighter.

# The future with concepts

We already saw in previous articles techniques that can be used to achieve the same goals and they are much easier to read and write.

With [`if constexpr`](https://www.sandordargo.com/blog/2021/04/14/how-to-use-type-traits) we can achieve the same without all the verbosity of `enable_if`. We can even spare turning `f()` into a template.

```cpp
template<typename T>
class MyClass {
public:
  void f(T x) {
    if constexpr (std::is_floating_point<T>::value) {
      std::cout << "with enable_if\n"; 
    } else {
      std::cout << "generic\n"; 
    }
  }
};
```

More details in [this article](https://www.sandordargo.com/blog/2021/04/14/how-to-use-type-traits).

Another way - if you already use C++20 is to use [concepts](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations).

```cpp
#include <concepts>

template<typename T>
class MyClass {
public:
  void f(T x) {
    std::cout << "generic\n"; 
  }
  
  void f(T x) requires std::floating_point<T> {
    std::cout << "with enable_if\n"; 
  }
};
```

With this solution, you have to separate the different functions, the different implementations, but that's fine. One might consider it as an advantage. As long as it's expressive, it's not a problem. At least, it's enough to check the signatures and you don't have to read the implementations.

You can read more about concepts [in this series](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations) and you should also check out [my book on concepts](https://leanpub.com/cppconcepts).

# Conclusion

Today we learnt about SFINAE. First, we discussed what does *Substitution Failure Is Not An Error* mean in practice and we saw 3 different ways to benefit from it. We used the function parameter list, we used the return type with `std::decltype` and last but not least `std::enable_if`.

I didn't go into the most complex examples, because I think that while it's worth knowing about SFINAE, but soon it should be the relics of the past. Since C++ we have `if constexpr` to replace many usages of SFINAE and C++20 gave something even better: [concepts](https://leanpub.com/cppconcepts).