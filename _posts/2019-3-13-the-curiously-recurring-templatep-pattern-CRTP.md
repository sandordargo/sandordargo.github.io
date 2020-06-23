---
layout: post
title: "The Curiously Recurring Template Pattern (CRTP)"
date: 2019-3-13
category: dev
tags: [cpp, tutorial, crtp]
excerpt_separator: <!--more-->
---
In this article, we are going to discover the pattern that is called the Curiously Recurring Template Pattern. Are you curious? Read on!
<!--more-->

## Introduction

Have you ever wondered about a derived class whose base has access to the derived class' members? Did it seem like something impossible? Impossible? Really? But in this world, nothing is impossible. Not even accessing the members of a derived class. From the base. All you need is some curiosity.

Some curiosity in the form of the Curiously Recurring Template Pattern. The CRTP is an idiom in C++ in which [a class let's call it X derives from a class template instantiation using X itself as template argument](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern). 

```
// The Curiously Recurring Template Pattern (CRTP)
template<class X>
class Base
{
    // methods within Base can use template to access members of Derived
};

class Derived : public Base<Derived>
{
    // ...
};
```

Like so many things in history, the CRTP was discovered around the same time at multiple places in the world.

The technique itself was formalized earlier, but in C++ it was Jim Coplien (author on many [books about advanced C++](https://amzn.to/2BMKr8x) and programming in general) in 1995 who came up with the name and observed it some early C++ template codes.

The other thread leads us to Microsoft, where it was also discovered in the same year and became part of the Active Template Library. Funny enough, one of the first code reviewers thought that such code would not even compile. As we might know from programming history it didn't just compile, but both the ATL and Windows Template Library heavily used this technique.

By the way, you might have heard about this technique as the Upside Down inheritance. 

## How to use the CRTP

There are two broad categories for CRTP usage. You can either add some functionality to your derived class or you can use the technique to implement static polymorphism.

Let's check both categories.

### Adding functionality

The CRTP consists of:
* Inheriting from a template class
* Use the derived class itself as a template parameter of the base class

As mentioned earlier, the main advantage of this technique is that the base class have access to the derived class methods. Why is that?

The base class uses the derived class as a template parameter.

In the base class, we can get the underlying Derived object with a static cast:

```
class Base {
  void foo() {
    X& underlying = static_cast<X&>(*this);  
    // now you can access X's public interface
  }
};
```

In practice, this brings us the possibility of enriching our Derived class' interface through some base classes. In practice, you'd use this technique to add some general functionalities to a class such as some mathematical functions to a sensor class (such as explained by [Johnathan Baccara](https://www.fluentcpp.com/2017/05/16/what-the-crtp-brings-to-code/)). Although these functionalities can be implemented as non-member function or non-member template functions, those are hard to know about when you check the interface of a class. Whereas the public methods of a CRTP's base class are part of the interface.

#### Adding numerical functions to a class (by Johnathan Baccara)

Here is the full example:

```
template <typename T>
struct NumericalFunctions
{
    void scale(double multiplicator);
    void square();
    void setToOpposite();
};

class Sensitivity : public NumericalFunctions<Sensitivity>
{
public:
    double getValue() const;
    void setValue(double value);
    // rest of the sensitivity's rich interface...
};

template <typename T>
struct NumericalFunctions
{
    void scale(double multiplicator)
    {
        T& underlying = static_cast<T&>(*this);
        underlying.setValue(underlying.getValue() * multiplicator);
    }
    void square()
    {
        T& underlying = static_cast<T&>(*this);
        underlying.setValue(underlying.getValue() * underlying.getValue());
    }
    void setToOpposite()
    {
        scale(-1);
    };
};
```


#### Object counter

Another example of adding functionality is implementing object counters.

You can create a counter base class:

```
template <typename T>
class Counter
{
    static int _createdObjects;
    static int _aliveObjects;

    Counter()
    {
        ++_createdObjects;
        ++_aliveObjects;
    }
    
    Counter(const Counter&)
    {
        ++_createdObjects;
        ++_aliveObjects;
    }
protected:
    ~counter() // objects should never be removed through pointers of this type
    {
        --_aliveObjects;
    }
};
template <typename T> int Counter<T>::_createdObjects( 0 );
template <typename T> int Counter<T>::_aliveObjects( 0 );

```

Then even though you have one pair of static counters tracking the created and alive objects, you can have separate counters for separate types. All this, because the Counter is a template base class, that you specialize with the derived class as such:

```
class X : counter<X>
{
    // ...
};

class Y : counter<Y>
{
    // ...
};
```

### Static polymorphism

If you want to use polymorphism in C++, you have to declare functions you want to overload as virtual functions. Right?

Right! But... Not necessarily.

You can avoid virtuals, so you can avoid the runt0time cost of virtual tables in your code by using static interfaces and, surprise, surprise, the CRTP pattern.

Let's take an example. We want to model vehicles and in the example, the interface will have one method, `getNumberOfWheels`:

```
template <typename T>
class Vehicle
{
public:
    double getNumberOfWheels() const
    {
        return static_cast<T const&>(*this).getNumberOfWheels();
    }
};
```

Then we can create the derived class as such:

```
class Bus : public Vehicle<Bus>
{
public:
    explicit Bus(int value) : value_(value) {}
    double getNumberOfWheels() const {return value_;}
private:
    int value_;
};
```

```
class Scooter : public Vehicle<Scooter>
{
public:
    double getNumberOfWheels() const {return 2;}
};
```
If you really want to spare runtime, this technique might help you a bit. Remember, there will be no run-time resolutions of virtual calls because there are no virtual function calls.

## Conclusion

The Curiously Recurring Template Pattern is an interesting technique at least to know and sometimes to use. With the help of the pattern you access the derived class' public interface from the base class which helps you mostly:
* adding functionality to a derived class through the base
* implementing polymorphism without the cost of virtual tables

If you want to get deeper knowledge, I'd recommend you to read the following pages and articles:

* [https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern)
* [https://www.fluentcpp.com/2017/05/12/curiously-recurring-template-pattern/](https://www.fluentcpp.com/2017/05/12/curiously-recurring-template-pattern/)
* [https://stackoverflow.com/questions/4173254/what-is-the-curiously-recurring-template-pattern-crtp](https://stackoverflow.com/questions/4173254/what-is-the-curiously-recurring-template-pattern-crtp)
