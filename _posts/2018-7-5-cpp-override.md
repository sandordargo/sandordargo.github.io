---
layout: post
title: "Why to use the 'override' specifier in C++ 11?"
date: 2018-7-5
category: dev
tags: [cpp, tutorial, clean code]
excerpt_separator: <!--more-->
---
If you know Java this might be completely straightforward to you as you are already used to Java's `@Override annotation`. If you've been always coding in C/C++, this might be new. You might ask yourself the question, why should one put there an extra specifier when it's not necessary. Your code will just work the very same way.
<!--more-->

While in most of the cases it's true that your code's behaviour will not change, in some others - when you're actually making a mistake - using `override` will stop you from checking in the buggy code as your compilation will fail. And none of us checks in code that doesn't even compile, right?

The `override` specifier will tell both the compiler and the reader that the function where it is used is actually overriding a method from its base class. 

It tells the reader that "this is a virtual method, that is overriding a virtual method of the base class."

Use it correctly and you see no effect:

```cpp
class Base
{
    virtual void foo();
};
 
class Derived : Base
{
    void foo() override; // OK: Derived::foo overrides Base::foo
};
```

But it will help you _revealing problems with constness_:

```cpp
class Base
{
    virtual void foo();
    void bar();
};
 
class Derived : Base
{
    void foo() const override; // Error: Derived::foo does not override Base::foo
                               // It tries to override Base::foo const that doesn't exist```
};
```

Let's not forget that in C++, methods are non-virtual by default. If we use `override`, we might find that there is nothing to override. Without the `override` specifier we would just simply create a brand new method. _No more base methods forgotten to be declared as virtual._

```cpp
class Base
{
    void foo();
};
 
class Derived : Base
{
    void foo() override; // Error: Base::foo is not virtual
};
```

We should also keep in mind that when we override a method - with or without the `override` specifier - _no conversions are possible_:

```cpp
class Base
{
  public:
    virtual long foo(long x) = 0; 
};


class Derived: public Base
{
   public:
     long foo(int x) override { // error: 'long int Derived::foo(int)' marked override, but does not override
      // ...
     }
};
``` 

In my opinion, using the override specifier from C++11 is part of clean coding principles. It reveals the author's intentions, it makes the code more readable and helps to identify bugs at build time. Use it without moderation!

_If you are looking for more modern C++ tricks, I'd recommend you to check out [Scott Meyers](https://www.aristeia.com/)'s [Effective Modern C++](https://amzn.to/2VZrLec)!_