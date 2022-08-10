---
layout: post
title: "The copy and swap idiom in C++"
date: 2022-8-10
category: dev
tags: [cpp, designpatterns, architecture, copyandswap]
excerpt_separator: <!--more-->
---
Last year, as the usage of our services grew sometimes by 20 times, we had to spend significant efforts on optimizing our application. Although these are C++-backed services, our focus was not on optimizing the code. We had to change certain things, but removing not needed database connections I wouldn't call performance optimization. It was rather fixing a bug.
<!--more-->

In my experience, while performance optimization is an important thing, often the bottleneck is about latency. It's either about the network or the database.

Checking some of our metrics, we saw some frontend queueing every hour.

Long story short, it was about a [materialized view](https://docs.oracle.com/cd/B19306_01/server.102/b14200/statements_6002.htm#:~:text=A%20materialized%20view%20is%20a,(a%20data%20warehousing%20term).). We introduced it for better performance, but seemingly it didn't help enough.

What could we do?

The view was refreshed every hour. A refresh meant that the view was dropped, then in a few seconds, a new one was built. The few seconds of downtime were enough to build up a queue.

We found a setting to have an out-of-place refresh. With that, the new view was built up while the old one was still in use. Then once ready, Oracle started to use the new view and drop the old.

The queueing vanished.

We traded some space for time.

The idea is not exclusive to databases obviously. In C++, there is a similar concept, an idiom, called *copy-and-swap*.

## The motivations

But are the motivations the same?

Not exactly.

Even though I can imagine a situation where there is a global variable that can be used by different threads and it's crucial limiting the time spent updating that variable.

There is something more important.

It's about the safety of copy assignments. What is a copy assignment about? You create a new object and assign it to an already existing variable. The object that was held by the existing variable gets destroyed.

So there is construction and destruction. The first might fail, but destruction must not.

Is that really the case in practice?

Not necessarily.

What often happens is that the assignment is performed from member to member.

```cpp
class MyClass {
 public:
  MyClass(int x, int y) : m_x(x), m_y(y) {}

  MyClass& operator=(const MyClass& other) {

    if (this != &other)
    {
      //Copy member variables
      m_x = other.m_x;
      m_y = other.m_y;
    }

    return *this;
  }

  // ...

 private:
  //Member variables
  int m_x;
  int m_y;
};
```

The problem is that what if the copy assignment fails? Here we deal with simple POD members, but it could easily be something more complex. Something more error-prone. If the copy fails, if constructing any of those members fails, our object which we wanted to assign to remains in an inconsistent state. 

That is basic exception safety at best. Even if all the values remain valid, they might differ from the original.

> [The C++ standard library provides several levels of exception safety (in decreasing order of safety)](https://en.wikipedia.org/wiki/Exception_safety#Classification):
> 1) No-throw guarantee, also known as failure transparency: Operations are guaranteed to succeed and satisfy all requirements even in exceptional situations. If an exception occurs, it will be handled internally and not observed by clients.
> 2) Strong exception safety, also known as commit or rollback semantics: Operations can fail, but failed operations are guaranteed to have no side effects, leaving the original values intact.[9]
Basic exception safety: Partial execution of failed operations can result in side effects, but all invariants are preserved. Any stored data will contain valid values which may differ from the original values. Resource leaks (including memory leaks) are commonly ruled out by an invariant stating that all resources are accounted for and managed.
> 3) No exception safety: No guarantees are made.

If we want strong exception safety, the copy-and-swap idiom will help us achieve that.

## The building blocks

The constructions might fail, but destruction must not. Therefore, first, we should create a new object on its own and then swap it with the old one. If the construction fails, the original object is not modified at all. We are on the safe side. Then we should switch the handles and we know that the destruction of the temporary object with the old data will not fail.

Let's see it in practice.

We need three things to implement the copy and swap idiom. We need a copy constructor and a destructor which are not very big requirements and we also need a swap function. The swap function has to be able to swap two objects of the same class, do it, member, by member, and **without** throwing any exception.

We want our copy assignment operator to look like this:

```cpp
MyClass& MyClass::operator=(const MyClass& other) {

  if (this != &other)
  {
    MyClass temp(other);
    swap(*this, temp);
  }

  return *this;
}
```

The swap function should swap, or in other words, exchange the content of two objects, member by member. For that, we cannot use `std::swap`, because that needs both a copy-assignment and a copy-constructor, something we try to build up ourselves. Here is what we can do instead.


```cpp
friend void swap(MyClass& iLhs, MyClass& iRhs) {
    using std::swap;
    swap(iLhs.m_x, iRhs.m_x);
    swap(iLhs.m_y, iRhs.m_y);
}
```

There are probably three things to note here.
1) We call `swap` member-by-member.
2) We call `swap` unqualified, while we also use `using std::swap`. By importing `std::swap` to our namespace, the compiler can decide whether a custom `swap` or the standard one will be called.
3) We made `swap` a friend function. [Find out here about the reasons!](https://stackoverflow.com/questions/5695548/public-friend-swap-member-function)

At this point, whether you need to explicitly write the copy constructor and the destructor depends on what kind of data your class manages. Have a look at the "Hinnant table"! As we wrote a constructor and a copy assignment, the copy constructor and the destructor are defaulted. But who can memorize the table?

![The Hinnant table]({{ site.baseurl }}/assets/img/hinnant-table.jpg "The Hinnant table")
_The Hinnant table (source: https://howardhinnant.github.io/)_

It's better to follow the rule of five and simply write all the special functions if we wrote one. Though we can default the missing ones. So let's have the solution right here.

```cpp
#include <utility>

class MyClass {
 public:
  MyClass(int x, int y) : m_x(x), m_y(y) {}
  
  ~MyClass() = default;
  MyClass(const MyClass&) = default;
  MyClass(MyClass&&) = default;
  MyClass& operator=(MyClass&& other) = default;

  MyClass& operator=(const MyClass& other) {

    if (this != &other)
    {
      MyClass temp(other);
      swap(*this, temp);
    }

    return *this;
  }
  
  friend void swap(MyClass& iLhs, MyClass& iRhs) {
      using std::swap;
      swap(iLhs.m_x, iRhs.m_x);
      swap(iLhs.m_y, iRhs.m_y);
  }

  
 private:
  int m_x;
  int m_y;
};
```

## What about pointer members?

If our class has a pointer member, the copy constructor has to be properly implemented to perform a deep copy and of course, the destructor also must be correct so that we can avoid having leaks. At the same time, the assignment operator doesn't have to be changed, swapping is still correct.

Let's have a small example here, I simply changed the `int` members to `unique_ptr`s.

```cpp
class MyClass {
 public:
  MyClass(int x, int y) : m_x(std::make_unique<int>(x)), m_y(std::make_unique<int>(y)) {}
  
  ~MyClass() = default;
  MyClass(const MyClass& other) : m_x(std::make_unique<int>(*other.m_x)), m_y(std::make_unique<int>(*other.m_y)) {}
  MyClass(MyClass&&) = default;
  MyClass& operator=(MyClass&& other) = default;

  MyClass& operator=(const MyClass& other) {

    if (this != &other)
    {
      MyClass temp(other);
      swap(*this, temp);
    }

    return *this;
  }
  
  friend void swap(MyClass& iLhs, MyClass& iRhs) {
      using std::swap;
      swap(iLhs.m_x, iRhs.m_x);
      swap(iLhs.m_y, iRhs.m_y);
  }

  
 private:
  std::unique_ptr<int> m_x;
  std::unique_ptr<int> m_y;
};
```

## Any drawbacks?

By implementing the copy-and-swap idiom we get less code repetition as in the copy assignment we call the copy constructor. We also get strong exception safety. Is there a catch?

You might get a performance hit. After all, we have to make an extra allocation in the copy assignment where we create the temporary. This might or might not be relevant depending on your case. The more complex your class is and the more you use it in a container, the more significant the problem gets.

In simpler cases, the differences might even be optimized away, as happened with the above classes. You cannot simply assume. Before you commit to a decision, measure, measure, and measure!

## Conclusion

Copy and swap is an idiom in C++ that brings strong exception safety for copying objects. It also removes a bit of code duplication, though it might seem a bit of overkill sometimes.

Keep in mind that the extra safety might cost you a bit of performance. Nothing is ever black and white, there are tradeoffs to be made.

I'd go with the extra safety by default, otherwise measure, measure and measure so that you can make an informed decision.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!