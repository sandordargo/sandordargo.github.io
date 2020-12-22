---
layout: post
title: "What is virtual inheritance in C++ and when should you use it?"
date: 2020-12-23
category: dev
tags: [cpp, tutorial, oop, inheritance]
excerpt_separator: <!--more-->
---
When we start coding in an object-oriented programming language we often think that it's about building nice inheritance hierarchies. So we do. A bit later we learn that we should rather use composition over inheritance. So we do. But we still need inheritance, and from time to time we run into problems where it seems to be the only way. At those times, we might learn about some more specialized forms of inheritance. In C++, this might mean [private inheritance]() or _virtual inheritance_. Today we speak about the latter.
<!--more-->

## What is virtual inheritance?

### The diamond problem

_Virtual inheritance_ is a C++ technique that ensures that only one copy of a base class's member variables are inherited by second-level derivatives (a.k.a. grandchild derived classes). Without virtual inheritance, if two classes B and C inherit from class A, and class D inherits from both B and C, then D will contain two copies of A's member variables: one via B, and one via C. These will be accessible independently, using scope resolution. 

Instead, if classes B and C inherit virtually from class A, then objects of class D will contain only one set of the member variables from class A.

As you probably guessed, this technique is useful when you have to deal with multiple inheritance and it's a way to solve the infamous diamond inheritance.

![The Diamond Inheritance Problem]({{site.baseurl}}/assets/img/diamon-inheritance.png)

### Multiple base class instances

In practice, virtual base classes are most suitable when the classes that derive from the virtual base, and especially the virtual base itself, are pure abstract classes. This means the classes above the "join class" (the one in the bottom) have very little if any data.

Consider the following class hierarchy to represent the diamond problem, though not with pure abstracts.

```cpp
struct Person {
    virtual ~Person() = default;
    virtual void speak() {}
};

struct Student: Person {
    virtual void learn() {}
};

struct Worker: Person {
    virtual void work() {}
};

// A teaching assistant is both a worker and a student
struct TeachingAssistant: Student, Worker {};

TeachingAssistant ta;
```

As we said above, a call to `aTeachingAssistant.speak()` is ambiguous because there are two `Person` (indirect) base classes in `TeachingAssistant`, so any `TeachingAssistant` object has two different `Person` base class subobjects. 

An attempt to directly bind a reference to the `Person` subobject of a `TeachingAssistant` object would fail, since the binding is inherently ambiguous:

```cpp
TeachingAssistant ta;
Person& a = ta;  // error: which Person subobject should a TeachingAssistant cast into, 
                // a Student::Person or a Worker::Person?
```
To disambiguate, we would need to explicitly convert `ta` to any of the two base class subobjects:

```cpp
TeachingAssistant ta;
Person& student = static_cast<Student&>(ta); 
Person& worker = static_cast<Worker&>(ta);
```
In order to call `speak()`, the same disambiguation, or explicit qualification is needed: `static_cast<Student&>(ta).speak()` or `static_cast<Worker&>(ta).speak()` or alternatively `ta.Student::speak()` and `ta.Worker::speak()`. Explicit qualification not only uses an easier, uniform syntax for both pointers and objects but also allows for static dispatch, so it would arguably be the preferable way to do it.

In this case, the double inheritance of `Person` is probably unwanted, as we want to model that the relation between `TeachingAssistant` and a `Person` exists only once. The fact that a `TeachingAssistant` is a `Student` and is a `Worker` at the same time does not imply that a `TeachingAssistant` is a `Person` twice (unless the `TA` suffers from schizophrenia): a `Person` base class corresponds to a contract that `TeachingAssistant` implements (the "is a" relationship above really means "implements the requirements of"), and a `TeachingAssistant` only implements the `Person` contract once. 

### There should be only one behaviour

The real-world meaning of "exists only once" is that a `TeachingAssistant` should have only one way of implementing `speak`, not two different ways. 

In our degenerate case, `Person::speak()` is not overridden in either `Student` or `Worker`, but that could be different and then we would `TeachingAssistant` would have multiple implementations of the `speak()` method.

If we introduce `virtual` to our inheritance in the following way, our problems disappear:

```cpp
struct Person {
    virtual ~Person() = default;
    virtual void speak() {}
};

// Two classes virtually inheriting Person:
struct Student: virtual Person {
    virtual void learn() {}
};

struct Worker: virtual Person {
    virtual void work() {}
};

// A teaching assistant is still a student and the worker
struct TeachingAssistant: Student, Worker {};
```

Now we can easily call `speak()`.

The `Person` portion of `TeachingAssistant::Worker` is now the same `Person` instance as the one used by `TeachingAssistant::Student`, which is to say that a `TeachingAssistant` has only one - shared - `Person` instance in its representation and so a call to `TeachingAssistant::speak` is unambiguous. Additionally, a direct cast from `TeachingAssistant` to `Person` is also unambiguous, now that there exists only one `Person` instance which `TeachingAssistant` could be converted to.

This can be done through `vtable` pointers. Without going into details, the object size increases by two pointers, but there is only one `Person` object behind and no ambiguity.

You must use the `virtual` keyword in the middle level of the diamond. Using it in the bottom doesn't help.

You can find more detail at the [Core Guidelines](https://isocpp.org/wiki/faq/multiple-inheritance) and [here](https://en.wikipedia.org/wiki/Virtual_inheritance).

## Should we always use virtual inheritance? If yes, why? If not, why not?

The answer is definitely no. The base of an idiomatic answer can be the most fundamental idea of C++: __you only pay for what you use__. And if you don't need virtual inheritance, you should rather not pay for it.

Virtual inheritance is almost never needed. It addresses the diamond inheritance problem that we saw at the beginning of the article. It can only happen if you have multiple inheritance, otherwise, you cannot have this issue.

At the same time, it has some drawbacks.

### More complex dependencies

Virtual inheritance causes troubles with object initialization and copying. Since it is the "most derived" class that is responsible for these operations, it has to be familiar with all the intimate details of the structure of base classes. 

Due to this, a more complex dependency appears between the classes, which complicates the project structure and forces you to make some additional revisions in all those classes during refactoring. All this leads to new bugs and makes the code less readable and thus less maintainable.

### Expensive type conversions

[ISO C++ guidelines](https://isocpp.org/wiki/faq/multiple-inheritance#virtual-inheritance-casts) also suggests that C-style downcasts cannot be used to cast a base class pointer to a derived one.

The problems can be solved by `dynamic_cast`, but it has its performance implications. Using too much `dynamic_cast` in your code can make a big hit, and it also means that your project's architecture is probably very poor. 

You can always implement what you need without multiple inheritance. There is no surprise in that. After all, the feature of virtual inheritance is not present in many other major languages, yet they are used for large and complex projects.

## Conclusion

Today, we discussed the diamond inheritance problem. We understood that when there are multiple paths between a base and a derived class, there are multiple base objects instantiated which is almost never desirable. C++ proposes virtual inheritance to solve this problem and letting such structures to live with only one instance of a base class.

Yet, as you should only pay for what you use, virtual inheritance should not be your default choice. Most projects can be implemented without such a language feature and if you can design your software without multiple inheritance, you don't need to deal with its downsides.

Have you ever used multiple inheritance in your production code? If yes, what was the use-case?


## Connect deeper

If you found interesting this article, please [subscribe to my newsletter](http://eepurl.com/gvcv1j) and let's connect on [Twitter](https://twitter.com/SandorDargo)!