---
layout: post
title: "Why to use the override specifier in C++ 11?"
date: 2018-7-5
category: dev
tags: [cpp, tutorial, cleancode, override, virtual]
excerpt_separator: <!--more-->
---
The `override` specifier was introduced to the language with C++11 and it is one of the easiest tool to significantly improve the maintainability of our codebases.
<!--more-->

`override` tells both the reader and the compiler that a given function is not simply `virtual` but it overrides a `virtual` method from its base class(es).

Hopefully it you replace the right `virtual` keywords in your codebase your compilation will not break, but if it does it means that you just identified some bugs and now you have a way to fix them.

If you are correctly overriding a `virtual` method of a base class, you will see no effect:

```cpp
class Base
{
    virtual void foo();
    virtual void bar();
};
 
class Derived : public Base
{
    void foo() override; // OK: Derived::foo overrides Base::foo
};

class Derived2 : public Derived
{
    void bar() override; // OK: Derived2::bar overrides Base::bar
};
```

Now let's see the different kind of errors it can help catch.

## Catch `const`/non-`const` mismatch with `override`

`override`  will help you revealing *problems with constness*. Meaning that if you try to override a `const` method with a non-`const` method, or if you try to override a non-`const` method with a `const` one, it's not going to work:

```cpp
class Base
{
    virtual void foo();
    virtual void bar() const;
};
 
class Derived : Base
{
    void foo() const override; // error: Derived::foo does not override Base::foo

    void bar() override;    // error: 'void Derived::bar()' marked 'override', but does not override              
};
```

What we've just see would work also with `volatile` and `noexcept`. All these qualifiers must exactly be the same in the base and dervided classes in order to correctly override a base class method.

## Find when you `override` a non-`virtual` method

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

## Find mismatching signatures with `override`

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

## Conclusion

In my opinion, using the override specifier from C++11 is part of clean coding principles. It reveals the author's intentions, it makes the code more readable and helps to identify bugs at build time. Use it without moderation!

_If you are looking for more modern C++ tricks, I'd recommend you to check out [Scott Meyers](https://www.aristeia.com/)'s [Effective Modern C++](https://amzn.to/2VZrLec)!_

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
