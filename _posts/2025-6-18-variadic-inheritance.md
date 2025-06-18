---
layout: post
title: "Variadic class template arguments"
date: 2025-6-18
category: dev
tags: [cpp, templates, variadic]
excerpt_separator: <!--more-->
---
Let's talk about class templates and variadic parameters. How to use them in combination?

But first of all, what are variadic templates?

Variadic template don't accept a fixed size of parameters, but anything from one to more. The set of parameters is called the parameter pack.

In the template parameter list, we can see the three dots (`...`) signaling the pack after the `typename` or `class` keywords, or after the constraints. Then later in the function / constructor parameter list, you can observe it right after the actual type name, and finally once the pack is expanded, it comes after the packed variable name.

Variadic arguments can be used in so many different ways both with function and class templates. Today, we are focusing on class templates.

When you are using variadic templates with class templates, the expansion can happen at different places. You might expand the parameter pack
- within the constructor or any other member function
- when declaring a member variable
- at the inheritance list
- in a using declaration
- in friend declarations

## Expansion in the constructor and in variable declaration

Let's start with the first option and expand the argument pack in the constructor. To make it a bit more meaningful, we'll use our variadic template type with one of our member variables, a tuple. As such, we make a storage where we can store elements from different types.

```cpp
#include <tuple>

template <typename... Ts>
class Storage {
   public:
    Storage(Ts&&... args) : elements(args...) {}

    // some getters here

   private:
    std::tuple<Ts...> elements;
};

template <typename... Ts>
Storage(Ts&&... ts) -> Storage<Ts...>;

int main() {
    auto myStore = Storage(41, 4.2, "forty two");
}
```

It's worth noting that in order to be able to use this `Storage` type with variadic templates without listing the exact types at instantiation, we might have to provide a simple deduction guide. Otherwise we used the same `Ts` type(s) following the three dots (`...`) both in the constructor and in the tuple declaration to expand the types. In the constructor's member initializer list, the constructor parameter itself was followed by `...` to expand the pack and initialize the `elements` tuple.

But we can use the variadic template parameter at other places too! We can even inherit variadically.

## Expansion at the inheritance list

Let's start with an example where we'll call our class `MultiInheritance`, before we'd switch over for a more descriptive name.

```cpp
template<typename... Ts>
struct MultiInheritance : Ts... { };
```

So that means that we have a class that inherits from an arbitrary number of classes. That's nice! What can we do with that?

We can for example combine it with the using keyword!

```cpp
template<typename... Ts>
struct MultiInheritance : Ts... {
	using Ts::operator()...;
};
```

This means that we have included each inherited class' `operator()` into the scope of `MultiInheritance`. If you've seen [C++ Weekly - Episode 440](https://www.youtube.com/watch?v=et1fjd8X1ho), you probably know what comes next.

Thanks to *Class Template Argument Deduction (CTAD)*, a class can easily inherit from lambdas without saving their generated types anywhere!

Let's rename our class and see how we can benefit from this.

```cpp
template<typename... Ts>
struct Visitor : Ts... {
	using Ts::operator()...;
};

auto v = Visitor{
    [](int i){ std::cout << i << '\n'; },
    [](float f){ std::cout << f << '\n'; },
    [](const std::string& s){ std::cout << s << '\n'; }
};

v(42); // prints 42
```

By passing lambdas to inherit from and using their `operator()`, we can implement a visitor where the different bases classes (lambdas) take care of providing the logic. That's a very sleek implementation. 

## Variadic friend declarations

Since C++26, we will also be able to expand a variadic template argument pack in a friend declaration. In other words, we can pass in as template arguments the friends a class should have.

```cpp
template<class... Ts>
class FriendlyClass {
    friend Ts...;  // before C++26 this is not valid code
};
```

Before C++26 this was only possible via several class template arguments, like this:

```cpp
template<class T, class U>
class FriendlyClass {
    friend T;
    friend U;
};
```

While using friends is discouraged in general, there are some idioms where this new feature is particularly useful. In [C++26: variadic friends](https://www.sandordargo.com/blog/2025/04/02/cpp26-variadic-friends), I shared a bit more details on this topic.


## Conclusion

In this article, we've seen how we can use variadic template arguments with class templates. The variadic pack can be expanded in various places. It can be expanded in the constructor or any member function, it can expanded in member variable declarations, in the inheritance list, using declarations or friend declarations.

We also saw a neat implementation of a visitor with the help of CTAD and inheriting from several lambdas.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)