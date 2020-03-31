---
layout: post
title: "The quest of private inheritance in C++"
date: 2020-4-1
category: dev
tags: [cpp, oop, mentoring, inheritance]
excerpt_separator: <!--more-->
---
I love mentoring.

It requires a huge quantity of humility, and if you possess it, it will bring you tremendous benefits on a human as well as on a technical level.
<!--more-->

A few weeks ago, I met with one of my mentees who told me that she finally started to work on interesting tasks. In the team, they have been doing pair programming, but they don't always have the time to go into deeper explanations. I asked Cathy if she faced some issues she would like to discuss and she came up with private inheritance that they tried to use with more-or-less success.

We talked a little bit about it, but I had to tell the truth that I had never used it since school probably, so I didn't remember exactly how it works.

Have you ever had teachers who returned questions as homework when he didn't know the answer?

I wanted to play. We opened up my laptop, connected to an online IDE/Compiler and started to have some fun.

## Experimenting with non-public inheritance

We started to with a [simple example](http://coliru.stacked-crooked.com/a/a34a8683841c5aea) of the usual public inheritance which worked as expected.

```cpp
#include <iostream>

class Base {
public:
    Base() = default;
    virtual ~Base() = default;
    virtual int x() { 
        std::cout << "Base::x()\n";
        return 41; 
    }

protected:
    virtual int y() { 
        std::cout << "Base::y()\n";
        return 42; 
    }
};

class Derived : public Base {
public:
    int x() override { 
        std::cout << "Derived::x()\n";
        return Base::y(); 
    }
};

int main() {
    Base* p = new Derived();
    std::cout << p->x() << std::endl;
}
```

_In this very example we take advantage of being able to access Derived::x(), through a pointer to `Base`. We call `Base::y()` from `Derived::x()` just to make a call from a function that is public in both `Base` and `Derived` to a protected function in Base._

Then we decided to take the experimental way combining with the methodology of _Compiler Driven Development_. We changed the public keyword in the inheritance to protected and recompiled waiting for the compilation errors.

This line didn't compile anymore.
```cpp
Base* p = new Derived();
// main.cpp:25:27: error: 'Base' is an inaccessible base of 'Derived'
//   25 |     Base* p = new Derived();
```
Seemed reasonable, no big surprise at first sight. So I just changed that line and [it compiled](http://coliru.stacked-crooked.com/a/1278c6cefa4d0e0a).

```cpp
Derived* p = new Derived();
```

As the next step, we changed the inheritance to private and clicked on the compile button. It expected the compilation to fail, I expected that `Base::y()` would be handled as private to `Derived` and as such in `Derived::x()` would fail to compile. But. It. Compiled.

This meant that something about non-public inheritance we didn't remember well or got completely misunderstood.

Let's stop for a second. Is this embarrassing? 

It is. 

I could start enumerating some excuses. But who cares? Nobody. And those excuses wouldn't matter anyway. What is important is that I realized I didn't know something well and I used the situation to learn something.

It was high time to open up some pages about non-public inheritance and re-read them carefully. 

> _The access specifier of the inheritance doesn't affect the inheritance of the implementation. The implementation is always inherited based on the function's access level. The inheritance's access specifier only affects the accessibility of the class interface._

This means that all the public and protected variables and functions will be useable from the derived class even when you use private inheritance.

On the other hand, those public and protected elements of the base class will not be accessible from the outside through the derived class.

When does this matter?

It counts when the next generation is born.

A grandchild of a base class, if its parent inherited privately from the base (the grandparent...), it won't have any access to the base's members and functions. Not even if they were originally protected or even public.

Just to make the point here is another example. You can play with it on [coliru](http://coliru.stacked-crooked.com/a/656fbfb81aa5c7a2).

```cpp
#include <iostream>

class Base {
public:
    Base() = default;
    virtual ~Base() = default;
    virtual int x() { 
        std::cout << "Base::x()\n";
        return 41; 
    }

protected:
    virtual int y() { 
        std::cout << "Base::y()\n";
        return 42; 
    }

};

class Derived : private Base {
public:
    int x() override { 
        std::cout << "Derived::x()\n";
        return Base::y(); 
    }
};

class SoDerived : public Derived {
public:
    int x() override { 
        std::cout << "SoDerived::x()\n";
        return Base::y(); 
    }
};

int main() {
    SoDerived* p = new SoDerived();
    std::cout << p->x() << std::endl;
}
```

## What is private inheritance for?

We probably all learnt that inheritance is there for expressing is-a relationships, right?

If there is `Car` class inheriting from `Vehicle`, we can all say that a `Car` is a `Vehicle`. Then `Roadster` class inherits from `Car`, it's still a `Vehicle` having access to all `Vehicle` member( function)s.

But what if that inheritance between `Vehicle` and `Car` was private? Then that little shiny red `Roadster` will not have access to the interface of `Vehicle`, even if it publicly inherits from `Car` in the middle.

We simply cannot call it an is-a relationship anymore.

It's a has-a relationship. `Derived` class, in this specific example `Car`, will have access to the `Base`(=> `Vehicle`) and exposes it based on the access level, protected or private. Well, this latter means that it's not exposed. It serves as a private member.

In the case of protected, you might argue that well, `Roadster` still have access to `Vehicle`, that is true.

But you cannot create a `Roadster` as a `Vehicle`, in case of non-public inheritance this line will not compile.

```cpp
Vehicle* p = new Roadster();
```

Just to repeat it, non-public inheritance in C++ expresses a has-a relationship.

Just like composition. So if we want to keep the analogy of cars, we can say that a `Car` can privately inherit from the hypothetical `Engine` class - while it still publicly inherits from `Vehicle`. And with this small latter addition of multiple inheritance, you probably got the point, why composition is easier to maintain than private inheritance.

But even if you have no intention of introducing an inheritance tree, I think private inheritance is not intuitive and it's so different from most of the other languages that it's simply disturbing to use it. It's not evil at all, it'll be just more expensive to maintain.

That's exactly what you can find on the [ISO C++ page](https://isocpp.org/wiki/faq/private-inheritance).

> Use composition when you can, private inheritance when you have to.

But when do you have to use private inheritance?

According to the above reference ISO C++ page, you have a valid use-case when the following conditions apply:

- The derived class has to make calls to (non-virtual) functions of the base
- The base has to invoke (usually pure-virtual) functions from the derived

## Conclusion

Today, I made the point that if the humble and more difficult road is taken, mentoring will pay off with great benefits to both parties. Recently, that's how I (re)discovered non-public inheritance in C++.

Non-public inheritance is - to me - a syntactically more complicated way to express a _has-a_ relationship compared to composition. Even though from time to time you might encounter use-cases, when it [provides some benefits](https://isocpp.org/wiki/faq/private-inheritance#priv-inherit-vs-compos), most often it just results in code that is more difficult to understand and maintain.

Hence, do as the C++ Standard FAQ says: _Use composition when you can, private inheritance when you have to._

Happy coding!