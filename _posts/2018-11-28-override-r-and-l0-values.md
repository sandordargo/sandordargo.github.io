---
layout: post
title: "How to use ampersands in C++"
date: 2018-11-25
category: dev
tags: [cpp, tutorial, c++11]
excerpt_separator: <!--more-->
---
In one of my previous articles, I wrote about Scott Meyer's [Effective Modern C++](https://amzn.to/2Rbh5pI) and that with its focus on C++11/14 it's like discovering a completely new language. I already wrote about [trailing return type declaration](/blog/2018/11/07/trailing-return-type). Now it's time to review what usages you might have in C++ for ampersands (`&`).
<!--more-->

Let's start with the good old, better-known usages:

* `&` to declare a reference to a type
* `&` to get the address of a variable
* `&` as a bit-wise operator
* `&&` in a conditional expression

These are not new, but "repetition is the mother of learning".

## Use `&` to declare a reference to a type

If you use `&` in the left-hand side of a variable declaration, it means that you expect to have a [reference](https://www.tutorialspoint.com/cplusplus/cpp_references.htm) to the declared type. It can be used in any type of declarations (local variables, class members, method parameters).

```
std::string mrSamberg("Andy");
std::string& theBoss = mrSamberg;
```

This doesn't just mean that both `mrSamberg` and `theBoss` will have the same value, but they will actually point to the same place in the memory. You can read more about references [here](https://www.tutorialspoint.com/cplusplus/cpp_references.htm).

## Use `&` to get the address of a variable

The meaning of `&` changes if you use it in the right-hand side of an expression. In fact, if you use it on the left-hand side, it must be used in a variable declaration, on the right-hand side, it can be used in assignments too.

When using it on the right-hand side of a variable, it's also known as the "address-of operator". Not surprisingly if you put it in front of a variable, it'll return its address in the memory instead of the variable's value itself. It is useful for pointer declarations.


```
std::string mrSamberg("Andy");
std::string* theBoss;

theBoss = &mrSamberg;

```

The end result of the previous snippet is the same as previously. Although the type of `theBoss` is different. Previously it was a reference, now it's a pointer. The main difference is that a pointer can be null, while a reference must point to a valid value. (Well... There are shortcuts... But that's beyond our scope in this article.). More on this topic [here](https://stackoverflow.com/questions/57483/what-are-the-differences-between-a-pointer-variable-and-a-reference-variable-in).

## Use `&` as a bitwise operator

It is the bitwise AND. Its an infix operator taking two numbers as inputs and doing an `AND` on each of the bit pairs of the inputs. Here is an example. 14 is represented as _1110_ as a binary number and 42 can be written as _101010_. So _1110_ (14) will be zero filed from the left and then the operation goes like this.

|           | 32 | 16 | 8 | 4 | 2 | 1 |
|----------:|:--:|:--:|:-:|:-:|:-:|:-:|
| 14        |  0 |  0 | 1 | 1 | 1 | 0 |
| 42        |  1 |  0 | 1 | 0 | 1 | 0 |
| 14&42=10  |  0 |  0 | 1 | 0 | 1 | 0 |

## Use `&&` in a logical expression

`&&` in a (logical) expression is just the C-style way to say `and`. That's it.


## Use `&&` for declaring rvalue references

Declaring a what? - you might ask. Okay, so let's clarify first what are lvalues and rvalues and what are the differences.

According to [Eli Bendersky](https://eli.thegreenplace.net/):

> An lvalue (locator value) represents an object that occupies some identifiable location in memory (i.e. has an address).
> 
> rvalues are defined by exclusion, by saying that every expression is either an lvalue or an rvalue. Therefore, from the above definition of lvalue, an rvalue is an expression that does not represent an object occupying some identifiable location in memory.

Let's take one example to show both an lvalue and an rvalue.

```
auto mrSamberg = std::string{"Andy"};
```

`mrSamberg` represents an lvalue. It points to a specific place in the memory which identifies an object. On the other hand, what you can find on the right side `std::string{"Andy"}` is actually an rvalue. It's an expression that can't have a value assigned to, that's already the value itself. It can be only on the right-hand side of an assignment operator.

For a better and deeper explanation, please read [Eli's article](https://eli.thegreenplace.net/).

Although rvalues can only appear on the right-hand side, still one can capture references to them. Those "captures" are called _rvalue references_ and such variables have to be declared with double ampersands (`&&`). Binding such temporaries are needed to implement move semantics and perfect forwarding. (I'll explain perfect forwarding and move semantics in a later article.)

## Use `&&` for declaring universal references

The bad news is that `&&` after a type might or might not mean that you are declaring an rvalue reference. In certain circumstances, it only means something that [Scott Meyers] calls a universal reference in his [Effective Modern C++](https://amzn.to/2Rbh5pI).

What are those circumstances? Briefly, if type deduction takes place, you declare a universal reference, if not an rvalue reference.

```
Vehicle car;
auto&& car2 = car; // type deduction! this is a universal reference!
Vehicle&& car3 = car; // no type deduction, so it's an rvalue reference
```

There is another possibility, it's in case of templates. Taking the example from [Effective Modern C++](https://amzn.to/2Rbh5pI):

```
template<typename T>
void f(std::vector<T>&& param);     // rvalue reference

template<typename T>
void f(T&& param); // type deduction occurs, so this is a universal reference!
```

There are more subtilities in the case of templates, but again, it's beyond the scope. Read Item 24 from [Effective Modern C++](https://amzn.to/2Rbh5pI) in case you want to learn more about how to distinguish universal references from rvalue references.

## Use `&` or `&&` for function overloading

We are not finished yet.

Since C++11 you can use both the single and double ampersands as part of the function signature, but not part of the parameter list. If I'm not clear enough, let me give the examples:

```
void doSomething() &;
void doSomething() &&;
auto doSomethingElse() & -> int;
auto doSomethingElse() && -> int;
```

What this means is that you can limit the use of a member function based on whether `*this` is a lvalue or an rvalue.  So you can only use this feature within classes, of course. Let's expand our example.


```
class Tool {
public:
  // ...
  void doSomething() &; // used when *this is a lvalue
  void doSomething() &&; // used when *this is a rvalue
};

Tool makeTool(); //a factory function returning an rvalue

Tool t; // t is an lvalue

t.doSomething(); // Tool::doSomething & is called

makeTool().doSomething(); // Tool::doSomething && is called
```

When would you use this kind of differentiation? Mostly when you'd like to optimize your memory footprint by taking advantage of move semantics. In a later post, I'll go more deeply on that.


## Conslusion

In this post, you saw 7 different type of usages the ampersands in C++. They can be used in single or double form, in variable declarations, function declarations and conditional expressions.

I didn't intend to give you a full explanation of each. Move semantics and perfect forwarding can fill multiple chapters of good books, like in [Effective Modern C++](https://amzn.to/2Rbh5pI). On the other hand, I'll try to give a deeper explanation on those topics in a later post.