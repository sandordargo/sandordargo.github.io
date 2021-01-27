---
layout: post
title: "Can virtual functions have default arguments?"
date: 2021-1-27
category: dev
tags: [cpp, tutorial, oop, inheritance]
excerpt_separator: <!--more-->
---
Yes, they can, but you should not rely on them, as you might not get what you'd expect. 
<!--more-->

If you wonder how this topic came up, the answer is static code analysis! We have been using static code analyzers for years, and little by little, by cleaning up the touching parts, by applying the boy scout rule, we've removed the worst offenders.

What are the worst ones depends highly on how the analyzer. You might not agree with some of the recommendations but if you see even those frequently enough, you'll start fixing them and stop adding them...

Of course, you don't have to be a passenger in this vehicle. You should be the driver as much as you can. On a corporate level, this means that you should customize the profiles used by the analyzers to your needs.

As I spoke about this in [Zuckerberg's gray T-shirt and coding guidelines](https://www.youtube.com/watch?v=CNDejB6Hg5A&t=2s), this mostly means that you should add rules to the industry-standard profile and not remove them.

In my company, we recently applied a new quality profile to our codebase which resulted in thousands of new violations which we started to categorize based on whether we want to fix it in the short term, mid-term or best effort. 

If you wonder why we categorize after the profile is applied, we didn't create the profile, but we want to provide valuable feedback to the creators plus a plan to deal with it to our teammates.

During the coming months, I'll share you a couple of the most interesting rules we found.

## [The problem of default arguments](https://rules.sonarsource.com/cpp/RSPEC-3719)

While it's syntactically perfectly correct to use default argument initializers in virtual functions, there is a fair chance that the code will not be maintained over time. In parallel, the emerging chaos will lead to incorrect polymorphic code and unnecessary complexity in your class hierarchy.

Let's see an example:

```cpp
#include <iostream>

class Base {
public:
  virtual void fun(int p = 42) {
    std::cout << p << std::endl;
  }
};

class DerivedLeft : public Base {
public:
  void fun(int p = 13) override {
    std::cout << p << std::endl;
  }
};

class DerivedRight : public Base {
public:
  void fun(int p) override {
    std::cout << p << std::endl;
  }
};
```

What would you expect from the following `main` function?

```cpp
int main() {
  DerivedLeft *d = new DerivedLeft;
  Base *b = d;
  b->fun();
  d->fun();
}
```

You might expect:
```cpp
42
13
```

If that's the case, congratulations! Especially if was not by chance. If you expected something else, don't worry. It's not evident and that's the problem with using default parameter values for virtual functions. 

`b` points to a derived class, yet `Base`'s default value was used.

Now what about the following possible `main`?

```cpp
int main() {
  Base *b2 = new Base;
  DerivedRight *d2 = new DerivedRight;
  b2->fun();
  d2->fun();
}
```

You might expect 42 twice in a row, but that's incorrect. The code won't compile. The overriding function doesn't _"inherit"_ the default value, so the empty `fun` call on `DerivedRight` fails.

```cpp
/*
main.cpp: In function 'int main()':
main.cpp:28:11: error: no matching function for call to 'DerivedRight::fun()'
   28 |   d2->fun();
      |           ^
main.cpp:19:8: note: candidate: 'virtual void DerivedRight::fun(int)'
   19 |   void fun(int p) override {
      |        ^~~
main.cpp:19:8: note:   candidate expects 1 argument, 0 provided
*/
```

## Static vs dynamic types

In order to understand better what is happening behind the scenes, let's take a step back. Let's modify a bit our original example and let's forget about `DerivedRight`.

```cpp
#include <iostream>

class Base {
public:
  virtual void fun(int p = 42) {
    std::cout << "Base::fun " << p << std::endl;
  }
};

class Derived : public Base {
public:
  void fun(int p = 13) override {
    std::cout << "Derived::fun " << p << std::endl;
  }
};

int main() {
  Derived *derived = new Derived;
  derived->fun();
  Base *base = derived;
  base->fun();
}
```

What output do you expect now?

It is going to be:

```
Derived::fun 13
Derived::fun 42
```

You might find it surprising that in both cases the derived version was called, yet with different default parameters.

The reason is that a virtual function is called on the dynamic type of the object, while the default parameter values are based on the static type. The dynamic type is `Derived` in both cases, but the static type is different, hence the different default values are used.

## Is it really a problem? If so, what to do?

It's definitely not a syntactic issue, after all, it compiles.

The main problem is that it's misleading and easy to misunderstand the code as for determining which function will be executed the dynamic type is used, but for getting the default argument the static type is used.

It's better to avoid such complexities and make the functions that need a default behaviour non-virtual.

A way to achieve this is to use a protected so-called forwarding function:

```cpp
#include <iostream>

class Base {
public:
  void fun(int p = 42) {
    fun_impl(p);
  }
protected:
  virtual void fun_impl(int p) {
    std::cout << "Base::fun " << p << std::endl;
  }
};

class DerivedLeft : public Base {
protected:
  void fun_impl(int p) override {
    std::cout << "DerivedLeft::fun " << p << std::endl;
  }
};

class DerivedRight : public Base {
protected:
  void fun_impl(int p) override {
    std::cout << "DerivedRight::fun " << p << std::endl;
  }
};

int main() {
  DerivedLeft *d = new DerivedLeft;
  Base *b = d;
  DerivedRight *d2 = new DerivedRight;

  b->fun();
  d->fun();
  d2->fun();
}
```

It this case, only the implementation is altered and the behaviour is exactly one would expect:

```cpp
DerivedLeft::fun 42
DerivedLeft::fun 42
DerivedRight::fun 42
```
In case you really need a second default behaviour, you can create another non-virtual `fun` function in the corresponding derived class with the new default argument forward still to `fun_impl`, it will work.

Though it can also be questioned whether it's a good idea to use the same signatures in different classes in the same hierarchy without one overriding the other.

The best is to avoid the need for such varying default arguments.

## Conclusion

Static code analyzers can help us both fixing - potential - bugs in our code and at the same type to educate the team about subtle rules and cases that we might not have considered otherwise.

Today we saw that using default arguments for virtual functions is a bad idea because it mixes static and dynamic types and hence it will become by the time a maintenance burden.

With a simple function forwarding, you can avoid the need.

Given these differences compared to normal polymorphic behaviour, it's best to avoid any default arguments in virtual functions.
