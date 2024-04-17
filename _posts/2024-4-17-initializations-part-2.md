---
layout: post
title: "When and how variables are initialized? - Part 2"
date: 2024-4-17
category: dev
tags: [cpp, fundamental, initialization, undefinedbehaviour]
excerpt_separator: <!--more-->
---
During the last two weeks, we saw [a bug related to uninitialized values and undefined behaviour](https://www.sandordargo.com/blog/2024/04/03/upgrading-the-compiler-and-undefined-behaviour), [we listed the different kinds of initializations in C++ and we started to more detailed discovery with copy-initialization](https://www.sandordargo.com/blog/2024/04/10/initializations-part-1).

This week, we continue this discovery with direct-, list- and aggregate-initialization.

## Direct-initalization

Direct-initialization initializes an object from an explicit set of constructor arguments. Different syntaxes invoke direct initialization such as `T object(<at least one arg>);`, `T(<at least one arg>);` or `new T(<at least one arg>);` but it might also happen when you use curly braces (`T object{oneArg};`). Additional use cases are static casts, constructor initializer lists and values taken by value in lambda captures.

While at first glance this might obvious there are some catches.

Take this expression: `T object{ arg };`. In this case, `object` is directly initialized only if it's a non-class type, otherwise, we talk about list-initialization. But if you use the parentheses syntax (`T object(arg)` then there is no such distinction between class and non-class types, in both cases, direct-initialization is performed. Also, `T object{ arg1, arg2 };` would never be direct-initialized, that's always an aggregate-initialization.

In the expression `[arg]() {}`, the generated lambda members will not be copy-initialized, but they will be directly initialized.

Another notable catch, or maybe better to say, a change introduced by C++17 is represented by the following snippet. Up until C++17, the following piece of code was ill-formed.

```cpp
struct A {
    explicit A(int i = 0) {}
};

A a[2](A(1));
``` 

But with C++20 it became valid and it initializes the first variable with `A(1)` and the second with `A()` - for elements without an initializer *value-initialization* is performed. With braces, it's considered a list-initialization and it's still invalid as it wouldn't initialize all the array items.

Direct initialization has quite a few small rules. What is worth noting is that since C++17 copy elision is guaranteed, there is no temporary object constructed when the initializer is a *prvalue* expression with the same type. The destination object is directly initialized.

If the target is an aggregate class, it is initialized as it would happen with aggregate initialization except that it also accepts narrowing conversions and you cannot use designated initializers. Similarly to initializing an array, any elements without a value are value-initialized.

For non-class types, all user-provided and standard conversion functions are examined and used if necessary.

## List-initialization

First of all, list initialisation always involves braces.

Not that we don't have enough different kinds of initializations, we even have to separate two different kinds of list initializations. There is direct-list-initialization and copy-list-initialization.

In general, we can talk about direct-list-initialization where there is no `=` between the object to be initialized and the opening brace (`T object{arg1, arg2}`;) and copy-list-initialization when there is a `=` (`T object = { arg1, arg2 }`;).

Besides these cases, direct-list-initialization can happen when a possibly empty pair of braces follow `T`, `new T` or even member variables in the constructor-initializer-list.

Further cases for copy-list-initialization involve also a possible empty pair of braces in function or constructor calls, return statements, subscript expressions (`[{...}]`) and the right side of assignments.  

There are a couple of different rules to consider if we want to understand what actually happens during a list-initialization.

If `T` is an aggregate class, you don't use designated initializers and there is one single parameter then either copy-initialization or direct-initialization will be performed depending on whether you're in the case of copy-list-initialization or direct-list-initialization.

```cpp
struct Aggregate {
    int num;
    char c;
};

int main() {
    Aggregate a{4, 'c'}; // direct-list-initialization => direct-initialization
    Aggregate b = {4, 'c'}; // copy-list-initialization => copy-initialization 
    return 0;
}
```

Otherwise, if `T` is an aggregate type, aggregate-initialization is performed.

If `T` is a `char` array and between the braces there is only a string literal the array will be initialization from the string literal. If the size of the array is not declared then the size of the literal + 1 for the `\0` will be the size, otherwise, if the declared size is longer than the literal's size + 1, the rest will be filled with `\0` characters. If the literal is too long, the program is ill-formed.

```cpp
int main() {
    char arr1[4] = {"abc"}; // 'a', 'b', 'c', '\0'
    char arr2[5] = {"abc"}; // 'a', 'b', 'c', '\0', '\0'
    char arr3[] = {"abc"}; // arr3[4], 'a', 'b', 'c', '\0'
    // char arr4[3] = {"abc"}; // initializer is too long
    return 0;
}
```

If `T` is a class type and the braces are empty and `T` has a default constructor or T is non-class type, then value-initialization will be performed.

If `T` is a specialization of `std::ininitializer_list`, then the brace-initializer-list is considered a `std::initializer_list` and each element will be copy-initialized from the list.

If `T` has a constructor that takes only a `std::initializer_list` or it's the first argument with others that have default values, those are considered first. If not, all other constructors are considered.

```cpp
#include <iostream>
#include <vector>

class MyClass {
public:
  MyClass(std::initializer_list<int> l): m_v(l) {
    std::cout << "MyClass::MyClass(std::initializer_list)\n";
  }

  MyClass(std::vector<int> l): m_v(l) {
    std::cout << "MyClass::MyClass(std::vector)\n";
  }
private:
  std::vector<int> m_v;
};

int main() {
  MyClass mc1{};
  MyClass mc2{1, 2, 3};
  MyClass mc3 = {1, 2, 3};
  MyClass mc4 = std::vector{1, 2, 3};
  MyClass mc5 {std::vector{1, 2, 3}};

  return 0;
}

/*
MyClass::MyClass(std::initializer_list)
MyClass::MyClass(std::initializer_list)
MyClass::MyClass(std::initializer_list)
MyClass::MyClass(std::vector)
MyClass::MyClass(std::vector)
*/
```  

If `T` is a non-class type and there is only one item between the braces then either copy-initialization or direct-initialization will be performed depending on whether you're in the case of copy-list-initialization or direct-list-initialization.

It's worth noting that narrowing conversion cannot happen with list-initialization.

## Aggregate-initialization

Aggregate-initialization was mentioned several times while we discussed list-initialization, so let's continue with that. It was introduced by C++11 as a form of list-initialization. It can happen only when braces are used as an initializer sequence. Since C++20, it might also include designated initializers.

But what is aggregate?

First of all, it might be an array type.

Or it can also be a class type that adheres to a couple of requirements. An aggregate class cannot have any of the following (as of C++20):
- user-declared or inherited constructors
- private or protected direct non-static data members
- virtual base classes
- private or protected direct base classes
- virtual member functions

If you use aggregate initialization, array elements will be initialized in subscript order and for classes, direct base classes and non-static data members will be initialized in declaration order.

It's worth noting that it's possible to nest initializer lists, to initialize both an object and its member. Even though in many situations, the braces around the nested initializer list can be omitted.

```cpp
struct Foo {
    int i;
    int j;
    int a[3];
};
struct Bar {
    int x;
    Foo b;
};

int main() {
    // These are the same initializations
    Bar s1 = {1, {2, 3, {4, 5, 6}}};
    Bar s2 = {1, 2, 3, 4, 5, 6};
    return 0;
}
```

When aggregate-initialisation is performed, you cannot pass more items than expected, it won't compile.

```cpp
struct Foo {
    int i;
    int j;
    int a[3];
};

int main() {
    // error: too many initializers for 'Foo'
    Foo f = {1, 2, 3, 4, 5, 6};
    return 0;
}
```

When aggregate initialization is performed of an object, on the items there will be either direct-, copy- or list-initialization performed.

## Conclusion

After having discussed the importance of initializations two weeks ago and exploring the different initializer syntaxes and copy-initialization last week, we discussed 3 other kinds of initialization this week.

We saw how direct initialization is used to initialize objets from its constructor arguments and then we deeped down into the world of brace initialization with list- and argument-initialization.

TODO Next week, we are going to cover default-, value- and zero-initialization.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!