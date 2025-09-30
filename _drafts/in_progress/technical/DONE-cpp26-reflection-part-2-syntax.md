---
layout: post
title: "C++26 Reflection: The Basic Syntax"
date: 2025-X-X
category: dev
tags: [cpp, cpp26, reflection, syntax]
excerpt_separator: <!--more-->
---
Last week, we had a quick intro to C++26's probably most important feature: reflection. We saw that reflection in general is both about introspection and code injection. C++26 will offer both, maybe with a bit more emphasis on introspection. Which makes sense if you think about it, as it's often the basis of meaningful code generation.

We also saw that C++ reflection is going to be compile-time, meaning that it has no runtime performance overhead unlike in some other languages. The compile-time nature of C++ reflection has some consequences on the syntax as well. So without further ado, let's jump into some practical details.

## How to compile reflection examples?

At the moment of writing this series (summer of 2025), the easiest is if you use [Compiler Explorer](). There are two compilers which you can use:
- EDG (experimental reflection)
- x86-64 clang (reflection) or x86-64 clang (experimental P2996)

If you wish so, you can also [install clang on your local thanks to Bloomberg](https://github.com/bloomberg/clang-p2996), but using Compiler Explorer is the most comfortable choice. You'll also always get a reasonably fresh version - reflection is not yet fully or perfectly implemented, but it's already good enough to play around it with.

I'll mostly use the EDG version in the examples.

## How to enter the reflection domain?

First, what is the reflection domain? We can talk about the reflection domain when we introspect code or manipulate the data that we get from introspection. Then there is the C++ domain which is the usual C++ world. (I'm borrowing these term's from [Inbal Levi's talk at ACCU 2024](https://www.youtube.com/watch?v=G4i45R7sX8I))

So how do we enter the reflection domain?

You can ask the compiler to retrieve you the reflection value of a type with the help of the reflection operator: prefix `^^`, e.g. `auto reflection_value = ^^MyTpe;`

I know what you think, `^^` looks horrible, right? And it's just the first element of the syntax of this whole new (sub)language. Why can't it be `^` I've heard people asking. There are two main reasons. A single caret (`^`) is ambiguous with an operator in Objective-C, besides, in C++ `^` is the binary XOR operator and it's better to have a clear distinction.

> What the reflection operator would be wasn't set in stone until recently, you'll find still relevant conference videos about reflection where a single `^` is used.

Let's see some code!

```cpp
// https://godbolt.org/z/Mjacz1nz4
struct MyStruct {
    int num;
    bool flag;
};

int main() {
    [[maybe_unused]] auto reflection_value = ^^MyStruct;
}
```

## How to leave the reflection world?

We can leave the reflection world with the help of a splicer using the syntax of `[: reflection_value :]`. The easiest way to demonstrate this is by using the reflection operator and the splicer hand in hand.

```cpp
https://godbolt.org/z/xrTGzKo1r
struct MyStruct {
    int num;
    bool flag;
};

int main() {
    [[maybe_unused]] typename[:^^MyStruct:] instance_of_my_struct{.num = 42, .flag = true};
}
```

Of course, this code doesn't make much sense, it's for demonstrative purposes.

Note the keyword `typename`. We have to use it to show that we are getting out a concrete type from the splicer. (TODO verify this!)

The `typename` prefix can be omitted in the same contexts as with dependent qualified names (i.e., in what the standard calls type-only contexts). For example:
```cpp
using MyType = [:sizeof(int)<sizeof(long)? ^^long : ^^int:];  // No explicit "typename" prefix
```

Wouldn't it be better to use the variable `reflection_value` created in the previous example?

Let's have a look at it.

```cpp
auto reflection_value = ^^MyStruct; 
// error: expression must have a constant value
[[maybe_unused]] typename [: reflection_value :] instace_of_my_struct{.num = 42, .flag = true};
```

This doesn't compile, because `reflection_value` cannot be used as a constant value!

And that makes sense! Let's not forget, that C++ offers compile-time (a.k.a. static) reflection. `reflection_value` must be declared as `constexpr`

```cpp
// https://godbolt.org/z/GsPvM9jPx
struct MyStruct {
    int num;
    bool flag;
};

int main() {
    constexpr auto reflection_value = ^^MyStruct;
    [[maybe_unused]] typename [: reflection_value :] instace_of_my_struct{.num = 42, .flag = true};
}
```

## What is between the reflection operator and the splicer?

So far, we only used `auto` to represent what comes out of a reflection operator and goes into a splicer.

`auto` represents here `std::meta::info` and you can find it in the `<meta>` header. Well, for the time being, it's rather in the `<experimental/meta>` header.

> We will find our the types and tools needed for working in the reflection domain within `namespace std::meta` and (probably) in the `<meta>` header. 


While there were (quite) some critiques proposing that different language elements should be covered by different meta types, such as `std::meta::variable`, `std::meta::type`, etc., the committee decided else.

There is one single opaque reflection type. The reasons is to avoid codifying the language design into the type system. Once types are standardized, it's almost impossible to change their semantics. At the same time, the language evolves. For example, up until C++11, references were not considered types. A future similar change would bring great difficulties to maintain proper reflection types.

Besides, `std::meta::info` is designed to be easily extensible and it's very easy to use them for collections.

Next week, we'll get deeper into this type and see how to work with it.

## Conclusion

This week, we started to learn about the basics of C++ reflection syntax. We learned that we can enter the reflection world with the `^^` operator and leave that domain with the help of splicers (`[: meta_info :]`).

We also briefly looked into what is between the two, `std::meta::info` being a single opaque type representing reflection information. Next week, we'll dig deeper into what's inside and how to work with it.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
