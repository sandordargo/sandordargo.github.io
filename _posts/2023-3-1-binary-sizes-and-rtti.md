---
layout: post
title: "Binary sizes and RTTI"
date: 2023-3-1
category: dev
tags: [cpp, binarysizes, rtti, cleancode]
excerpt_separator: <!--more-->
---
What is RTTI? What does it have to do with the size of your executables?

Let's start with answering the first one.

RTTI stands for *run-time type information*. It's available for every class that has at least one `virtual` function. With the help of such information, you can determine the type of an object during execution and use it for different purposes.

Let's see the two different ways to use it.

## `typeid()` and `std::type_info`

With the help of `typeid()` we can query some information about a runtime object. In fact, it returns an object of the type `std::type_info` that you can find in the `<type_info>` header.

In most cases, the `type_info` class is used in 2 ways:
- either to use its comparison operator to decide if two objects refer to the same type
- or to query the name of a type. Beware the name is implementation-defined. (`int num; typeid(num).name()` might return `i` as a name)

Nevertheless, you can also query `type_info` for a `hash_code()` that will be identical for each type_info object referring to the same type. Also, you have `before()` which can help you decide whether type precedes the other in the given implementation's collation order. What is a collation order? I've never heard about it before, but [apparently in this case](https://stackoverflow.com/questions/8682582/what-is-type-infobefore-useful-for), it's just a fancy word for `operator<` and it comes in handy when you want to store `type_infos` in a `map`.

As I wrote earlier, RTTI is only available to classes that have at least one `virtual` function.

You can still use `typeid` with non-polymorphic types, but you might not get what you expect if you are looking for the dynamic type of an object. What else would you look for, right?

Well, I can imagine that when you have a function template returning `auto`, you want to debug with `typeid`. That will work.

```cpp
#include <iostream>

auto fun(auto t, auto u) {
  return t + u;
}

int main() {
    std::cout << typeid(fun(4.3, true)).name() << '\n';
};
/*
d
*/
```

But if you are using typeid with a non-polymorphic class hierarchy, it won't work as it normally should. Instead of returning the derived type for `*p` declared as `Base* p = new Derived();` it would return the static type, which is `Base`. Don't forget to dereference the pointer if you want to know what type is behind the polymorphic base.

```cpp
#include <iostream>

class NonPolyBase {};

class NonPolyDerived : public NonPolyBase {};

class PolyBase {
 public:
  virtual ~PolyBase() = default;
};

class PolyDerived : public PolyBase {};

int main() {
    NonPolyBase* p1 = new NonPolyDerived{};
    PolyBase* p2 = new PolyDerived{};
    
    std::cout << typeid(*p1).name() << '\n';
    std::cout << typeid(*p2).name() << '\n';
    
    return 0;
}
/*
11NonPolyBase
11PolyDerived
*/
```

But beware, if you turn RTTI off at compile time, no matter how you use it, you'll get a compilation error: `error: cannot use 'typeid' with '-fno-rtti'`!

## No more `dynamic_cast`s

If you turn RTTI off, there is no more dynamic casting available to you either. In case you attempt to use it, you'll get a similar message: `error: 'dynamic_cast' not permitted with '-fno-rtti'`.

But what is `dynamic_cast` and is it a problem if we have no access to it?

*`dynamic_cast` safely converts pointers and references to classes up, down and sideways along the inheritance hierarchy* - according to [CppReference](https://en.cppreference.com/w/cpp/language/dynamic_cast). Often when you have a collection of pointers to the base class, you'll try to cast it to different derived classes and if the case is successful you do whatever you want with that type.

In other words, when you have no idea what types you have, you can start casting things. You might even have a series of casts like this:

```cpp
OffRoader* offroader = dynamic_cast<OffRoader*>(car);
if (offroader) {
  offroader->turnOnAllWheelDrive();
}

Van* van = dynamic_cast<Van*>(car);
if (van) {
  van->attachThirdSeatRow();
}

Roadster* roadster = dynamic_cast<Roadster*>(car);
if (roadster) {
  roadster->removeRoof();
}
```

Many would say that this is a code smell. I'm among them. Many would go further and say that using `dynamic_cast` in general is a code smell.

For sure, with proper architecture, by using the good patterns of coding, they can be easily avoided.

In a sense, turning RTTI off can make your code much cleaner. It's out of the scope for this article to discuss the different ways.

## What will you gain with RTTI?

Speed? Not necessarily. Getting rid of `dynamic_cast`s does not mean that you don't cast, it just means you don't do it explicitely, the compiler gets to figure out what's the run-time type on its own without you telling it what to try. Probably it's faster.

Let's have a quick example.

```cpp
// Ugly RTTI way using dynamic casts
#include <iostream>
#include <memory>
#include <vector>


class Car {
public:
  virtual ~Car() = default;
};

class OffRoader : public Car {
public:
  void turnOnAllWheelDrive() {
    std::cout << "use all wheel drive on OffRoader\n";
  }
};

class Van : public Car {
public:
  void attachThirdSeatRow() {
    std::cout << "attach third seatrow in a van\n";
  }
};

class Roadster : public Car {
public:
  void removeRoof() {
    std::cout << "remove roadster's roof\n";
  }
};

void prepareCarForFun(Car* car) {
    OffRoader* offroader = dynamic_cast<OffRoader*>(car);
    if (offroader) {
      offroader->turnOnAllWheelDrive();
    }
    
    Van* van = dynamic_cast<Van*>(car);
    if (van) {
      van->attachThirdSeatRow();
    }
    
    Roadster* roadster = dynamic_cast<Roadster*>(car);
    if (roadster) {
      roadster->removeRoof();
    }
}

int main() {
    std::vector<std::unique_ptr<Car>> myCars;
    myCars.push_back(std::make_unique<OffRoader>());
    myCars.push_back(std::make_unique<Van>());
    myCars.push_back(std::make_unique<Roadster>());
    
    for (auto& car: myCars) {
      prepareCarForFun(car.get());
    }
    
    return 0;
}

// using no RTTI, no dynamic casts
#include <iostream>
#include <memory>
#include <vector>


class Car {
public:
  virtual ~Car() = default;
  virtual void doSomeFun() = 0;
};

class OffRoader : public Car {
public:
  void doSomeFun() override {
    std::cout << "use all wheel drive on OffRoader\n";
  }
};

class Van : public Car {
public:
  void doSomeFun() override {
    std::cout << "attach third seatrow in a van\n";
  }
};

class Roadster : public Car {
public:
  void doSomeFun() override {
    std::cout << "remove roadster's roof\n";
  }
};

int main() {
    std::vector<std::unique_ptr<Car>> myCars;
    myCars.push_back(std::make_unique<OffRoader>());
    myCars.push_back(std::make_unique<Van>());
    myCars.push_back(std::make_unique<Roadster>());
    
    for (auto& car: myCars) {
      car->doSomeFun();
    }
    
    return 0;
}
```

For sure you get clarity. Just look at the previous example! The second implementation without `dynamic_cast`s is much more readable and it's also shorter.

There is one more major thing you gain by not having run-time type information available, and that's space.

To get such information during run-time, all the necessary names have to be stored somewhere and that's your binary. By cutting it, your binary becomes smaller and also faster to write.

Let's take the previous example. I separated the classes into their own files and compile both with and without RTTI.

Here are the results.

|     Case      | Binary size  |
|---------------|--------------|
|  Dynamic casts O0   | 88.7k          |
| Dynamic casts O3 | 37.3k          |
|  No-Dynamic casts RTTI on O0   | 88.8k          |
| No-Dynamic casts RTTI on O3 | 37.2k          |
|  No-Dynamic casts RTTI off O0   | 88.6k          |
| No-Dynamic casts RTTI off O3 | 37.0k          |

Getting rid of dynamic casts with optimizations turned on decreased the binary size a little bit and the decrease was a bit more when RTTI was turned off. The differences in run-time and compile-time were a bit flaky, but it seems that they also decreased. 

It seems to me that unless you have a very good reason and limited space, the bytes you can gain might not be motivational enough. But keep in mind that getting rid of those dynamic casts will likely result also in more readable code.

If you can commit to running your project without runtime type info, I think you should definitely do it. Not (only) for the size gain, but more for the cleaner code.

## So how to use it?

That's simple, you need to specify a compiler flag:

- For gcc and clang it's `-fno-rtti`
- For MSVC, it's `/GR-`

## Conclusion

In this article, we saw that run-time type information is needed in order to use some language features such as `typeid` and `dynamic_cast`. We also saw that their usage is not necessarily considered best practice in the community. By forcing you not to rely on RTTI, you might not only get better code, but you'll also end up having a smaller binary.

In my opinion, it's totally worth it -  unless you have such a legacy that you cannot easily get rid of your dependence on RTTI.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!