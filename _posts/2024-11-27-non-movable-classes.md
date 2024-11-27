---
layout: post
title: "How to ensure a class is not copyable or movable"
date: 2024-11-27
category: dev
tags: [career, portfoliojob, watercooler, business]
excerpt_separator: <!--more-->
---
The topic of this post is to show different ways to ensure that a class is either non-moveable or non-copyable.

But first of all, why would you need that?

If we follow the classification proposed by [Sebastian Theophil](https://meetingcpp.com/2024/Talks/items/Classes_Cpp23_style.html), we can talk about 4 different class types:

- value classes
- container classes
- resource classes
- singleton classes

While the first two should be regular classes offering both copy and move semantics, the latter two are different. One shouldn't be able to copy resources and singletons probably shouldn't be moveable.

It's up to us to ensure that a class we create implements the right special member functions (*SMF*s from now on). And the Hinnant table is here to guide us.

![The Hinnant table]({{ site.baseurl }}/assets/img/hinnant-table.jpg "The Hinnant table")
_The Hinnant table (source: https://howardhinnant.github.io/)_

## The simplest and most obvious solution

The simplest and most obvious solution is to implement the needed SMFs and delete the rest respecting the rule of 5.

```cpp
class NonCopyableResource {
public:
	NonCopyableResource() = default;
	
	// deleted copy operations
	NonCopyableResource(const NonCopyableResource&) = delete;
	NonCopyableResource& operator=(const NonCopyableResource&) = delete;
	
	// (default) implemented move operations
	NonCopyableResource(NonCopyableResource&&) = default;
	NonCopyableResource& operator=(NonCopyableResource&&) = default;

	// ...
};


class NonMovableSingleton {
public:
	NonMovableSingleton() = default;
	
	// (default) implemented copy operations
	NonMovableSingleton(const NonMovableSingleton&) = default;
	NonMovableSingleton& operator=(const NonMovableSingleton&) = default;
	
	// deleted move operations
	NonMovableSingleton(NonMovableSingleton&&) = delete;
	NonMovableSingleton& operator=(NonMovableSingleton&&) = delete;
};
```

This is something we probably all know about. It's simple, if you're familiar with C++ it's even readable. It doesn't really document intentions though and provides no insurance. More on the latter later.

How would you document intentions? You might put a comment somewhere but otherwise, you don't know why something was made non-copyable or non-movable. That trait probably doesn't appear in the class name and rightly so.

But with C++26 we can document intentions better, because we can `= delete` something with a reason:

```cpp
// deleted copy operations
NonCopyableResource(const NonCopyableResource&) = delete("resource shouldn't be copyable");
NonCopyableResource& operator=(const NonCopyableResource&) = delete("resource shouldn't be copyable");
```

## Adding some assertions

Now let's talk about the insurance part. 

You might accidentally change this code and make your class copyable or movable. Even though when you `= delete` with a reason, the probabilities are smaller.

Back in 2023, [Kris van Rens gave a talk at C++ On Sea](https://www.sandordargo.com/blog/2023/07/05/trip-report-cpp-on-sea-2023) about special member functions and talked about how to test special member functions. 

You don't necessarily want to test the internals of a copy constructor. You don't necessarily have to test if all the members are copied promptly. Maybe you want to, but you don't have to go as far.

It's already a great step if you can ensure with the help of type traits (or [concepts](https://leanpub.com/cppconcepts)) and `static_cast` that a given class satisfies certain characteristics.

```cpp
static_assert(std::is_trivially_destructible<NonMovableSingleton>{});
static_assert(std::is_trivially_default_constructible<NonMovableSingleton>{});
static_assert(std::is_trivially_copy_constructible<NonMovableSingleton>{});
static_assert(std::is_trivially_copy_assignable<NonMovableSingleton>{});
static_assert(!std::is_trivially_move_constructible<NonMovableSingleton>{});
static_assert(!std::is_trivially_move_assignable<NonMovableSingleton>{});
```

Then even if you modify the class, you make sure that you don't lose its copyablity and you keep it non-movable. Such tests might even enhance your understanding of how certain types of members influence a class. Nevertheless, they also document the code.

But there is no perfect tool, everything has a shortcoming. Let's have a look at the following piece of code.


```cpp
#include <concepts>

class SomeClass {
public:
	SomeClass() = default;

	SomeClass(const SomeClass&) = default;
	SomeClass& operator=(const SomeClass&) = default;
};

static_assert(std::is_destructible<SomeClass>{});
static_assert(std::is_default_constructible<SomeClass>{});
static_assert(std::is_copy_constructible<SomeClass>{});
static_assert(std::is_copy_assignable<SomeClass>{});
static_assert(std::is_move_constructible<SomeClass>{});
static_assert(std::is_move_assignable<SomeClass>{});
```

While this compiles and you might that you have a nice movable class which you even asserted, the move operations will simply fall back silently to a copy because you forgot to define the move operations. The reason behind this is that with the introduction of move semantics in C++11 it had to be avoided that any existing class without move semantics fail to compile whenever a move seems to be invoked.

In any case, in most situations, these assertions work fine and probably you won't write a new class following only the rule of 3. But it's still better to keep this in mind

## Add characteristics with inheritance

Let's move to the next level which is what [Sebastian Theophil explained at Meeting C++ 2024](https://www.sandordargo.com/blog/2024/11/20/trip-report-meeting-cpp2024#my-three-favourite-ideas). Following the rule of 5 is a good best practice, but it's also easy to overlook something and can be a lot to type.

Therefore you might delegate the task of making something non-copyable or non-moveable to a base class.

```cpp
class NonCopyable  {
public:
	NonCopyable() = default;
	~NonCopyable() = default;
	NonCopyable(const NonCopyable& other) = delete;
	NonCopyable& operator=(const NonCopyable& other) = delete;
	NonCopyable(NonCopyable&& other) = default;
	NonCopyable& operator=(NonCopyable&& other) = default;
};

class File : public NonCopyable {
public:
	File() = default;
	File(char const* file);

	File(File&& other) noexcept;
	File& operator=(File&& other) noexcept;
	~File();
private:
	FILE* fp = nullptr;
};
```

If you saw the original example in [Meeting C++ trip report](https://www.sandordargo.com/blog/2024/11/20/trip-report-meeting-cpp2024#my-three-favourite-ideas) or anywhere else, you might recall that it was terser. True, in the original implementation only the necessary is implemented. According to the Hinnant table, if you provide move operations, the copy operations are deleted.

I think it's just easier to follow the rule of five and write down everything. It's more readable and leaves no questions. In addition, you'll only have to do it twice. Once for `class NonCopyable` and once for `class NonMovable` and reuse them.

You might wonder why the destructor of `NonCopyable` is not virtual and whether it's OK to inherit from this class without risking undefined behaviour.

Given `NonCopyable`'s destructor is public, calling code could attempt to destroy the derived class object through a base class pointer and the result would be undefined behaviour.

You might ask who in their right mind would try to use a pointer to the `NonCopyable` or `NonMovable` base class. It's hard to imagine.

Nevertheless, we can follow the [C.35 core guideline](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c35-a-base-class-destructor-should-be-either-public-and-virtual-or-protected-and-non-virtual) and make `NonCopyable`'s destructor protected:

```cpp
class NonCopyable  {
protected:
    ~NonCopyable() = default;
public:
    NonCopyable() = default;

	NonCopyable(const NonCopyable& other) = delete;
	NonCopyable& operator=(const NonCopyable& other) = delete;
	NonCopyable(NonCopyable&& other) = default;
	NonCopyable& operator=(NonCopyable&& other) = default;
};
``` 

As such, we are guaranteed that these base classes won't be used intended ways.

## Conclusion

In this article, we saw different ways to make sure that a class is non-copyable or non-movable. We started with the obvious solution of following the rule of 5 and implementing everything by hand following the Hinnant table. 

As a next step, we added some assertions making sure that our class has the traits we want it to have. Finally, we saw how to delegate the implementation of these traits to a base class in a super-readable way.

How would you do it?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)