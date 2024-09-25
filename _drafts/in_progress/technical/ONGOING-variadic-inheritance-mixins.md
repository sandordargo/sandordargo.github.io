---
layout: post
title: "Variadic class template arguments"
date: 2024-X-X
category: dev
tags: [career, portfoliojob, watercooler, business]
excerpt_separator: <!--more-->
---
Last week, we were talking about variadic class template arguments and we saw a couple of interesting examples of usage. We saw how to implement easily a visitor, we saw how we can create a storage that can take elements of different types. We also mentioned that we can implement mixings with the help of variadic template arguments.

## What are mixins?

Mixins are a design pattern used to "mix" additional functionality into a class through inheritance or composition. The primary goal of a mixin is to provide reusable and modular functionality that can be added to multiple classes without forming deep or complex inheritance hierarchies.

Mixins are commonly used to achieve multiple inheritance-like behavior, but with more focused and reusable pieces of code. The key idea is to separate out small, reusable components of functionality into classes (mixins) and then combine them as needed to create more complex behavior in derived classes.

## An old-school mixin implementation

## Mixins with variadic class template arguments

We can use our template parameters as mixins!

```cpp
template<typename... Ts>
struct MultiInheritance : Ts... {
	MultiInheritance(const Ts& ... ts) : Ts(ts)... {}

};

MultiInheritance mh0;
MultiInheritance mh1a(S1{2});
MultiInheritance mh1b(S2{"two"});
MultiInheritance mh2(S1{}, S2{"two"});
```

With the pack expansion, we call all the specific class copy or move constructors. 



## Conclusion

In this article, we've seen how we can use variadic template arguments with class templates. The variadic pack can be expanded in various places. It can be expanded in the constructor or any member function, it can expanded in member variable declarations, in the inheritance list, using declarations or friend declarations.

We also saw a neat implementation of a visitor with the help of CTAD and inheriting from several lambdas.

Next week, we are going to have a deeper look into how to implement mixins with the help of variadic class template arguments. 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)





https://godbolt.org/z/Gr9f3Ej9b


With the pack expansion, we'll call all the base class constructors. With this pattern we can...

We can even go further than that and extend a CRTP pattern.
https://www.fluentcpp.com/2018/06/22/variadic-crtp-opt-in-for-class-features-at-compile-time/

But we can go into a different direction as well.

We can simpy declare that we want to use a certain symbol from all derived classes:


In this posst, we are going to talk about a certain form of multiple inheritance, variadic inheritance.

Since C++11, we can have variadic templates in C++. What does it mean to use variadic templates?


```cpp
template<typename... Ts>
struct MultiInheritance : Ts... {
	using Ts::foo...;

};
```





A great example of the usage of such pattern can be found in C++ Weekly, where the classes we inherit from are lambdas. Yes, lambdas. We cannot know their exact types and we don't even have to. Class Template Argument Deduction (CTAD) will help us! So with the help of variadic templates and CTAD, we can implement a nice visitor as such:

```cpp

```

For details, I recommend watching this episode on C++ weekly.

Yet another usage is to declare friends.

## Conclusion

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)

further details.

Variadic arguments can be used in diferent ways. Without wanting to be exahustive, let's see a couple of usages.

https://en.cppreference.com/w/cpp/language/parameter_pack

0 + ... pack // fold expressions


siezof...

the recursive way

friend declaration...

mixins

visitor pattern 
https://www.fluentcpp.com/2018/06/22/variadic-crtp-opt-in-for-class-features-at-compile-time/


https://www.youtube.com/watch?v=et1fjd8X1ho 6  

https://www.youtube.com/watch?v=t6hFPKiOS-Q

maybe write abaout how to use with constructors?