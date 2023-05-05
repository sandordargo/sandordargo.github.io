---
layout: post
title: "Without RTTI your code will be cleaner"
date: 2023-4-26
category: dev
tags: [cpp, rtti, virtual, dynamic_cast]
excerpt_separator: <!--more-->
---
Recently, in the binary sizes series, we discussed [how run-time type information affects RTTI](https://www.sandordargo.com/blog/2023/03/01/binary-sizes-and-rtti). I also mentioned that in my opinion the lack of RTTI leads to better practices and you'll end up with a more readable, more maintainable code.

It's time to delve into this topic and see why.

## What is RTTI again?

But first of all, let's quickly recap what is run-time type information. RTTI let us have information on the dynamic type of reference/pointer types. It lets us use `dyanamic_cast`s and also call the `typeid()` function and query the returned instance of `std::type_info`.

I think that not having access to these tools will let you write better code. Why so?

Let's start with probably the less controversial tool.

## Don't rely on `typeid()`

Inexperienced programmers might want to automatically use `typeid().name()` to see what's the dynamic type of something during development and testing. It's usually not kept because often the output is not that readable. I mean for `int`, you might simply get `i`. That's something easily recognizable, but not something you'd want to see in the logs.

The bigger problem is that the output is implementation-defined, so you really shouldn't count on it if you want code that is portable. (Even though there is a fair chance that they will be the same on GCC and clang).

Let's see a couple of examples.

```cpp
#include <iostream>

int main() {
    std::cout << typeid(5).name() << ' ';
    std::cout << typeid(5u).name() << ' ';
    std::cout << typeid(5.0).name() << ' ';
    std::cout << typeid(true).name() << ' ';
    std::cout << typeid('5').name() << '\n';
    return 0;
}
/*
gcc:   i j d b c
clang: i j d b c
*/
```

Different outputs among compilers do not mean that you cannot use them consistently. You can still use `typeid().name()` or `typeid.hash()` to compare types against each other in your code and branch the execution based on those comparisons...

On the other hand, you don't have many reasons to do that. If you have to deal with, if you have to compare unrelated types, probably you should extract the type-dependent code and use different overloads.

```cpp
#include <memory>
#include <iostream>
#include <vector>

class Wine {
public:
  virtual ~Wine() = default;
  virtual void print() { std::cout << "this is wine\n"; }
};

class WhiteWine {
public:
  virtual ~WhiteWine() = default;
  virtual void print() { std::cout << "this is white wine\n"; }
};

class RedWine {
public:
  virtual ~RedWine() = default;
  virtual void print() { std::cout << "this is red wine\n"; }
};

class RoseWine {
public:
  virtual ~RoseWine() = default;
  virtual void print() { std::cout << "this is rose wine\n"; }
};

int main() {
    std::vector<std::unique_ptr<Wine>> wines;
    auto wine1 = std::make_unique<WhiteWine>();
    auto wine2 = std::make_unique<RedWine>();
    auto wine3 = std::make_unique<RoseWine>();
}

```

If the types are from the same inheritance tree, then you should either use `dynamic_cast`s or even better, you should benefit from dynamic dispatching.

## `dynamic_cast` is slightly better, but still should be avoided

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

As mentioned earlier, this is almost always better than relying on `typeid()`, but behind `dyanimc_cast`s you'll only find bad inheritance trees and messed up APIs.

Rather than having different public interfaces, we should have a unified one.

There is a convenient reason for that. If the interface is unified, we don't need to cast our objects, we can simply rely on runtime dispatching of our function calls.

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Car {
public:
  virtual ~Car() = default;
  virtual void doSomeFun() = 0;
};

class OffRoader: public Car {
public:
  void doSomeFun() override {
    std::cout << "use all wheel drive on OffRoader\n";
  }
};

class Van: public Car {
public:
  void doSomeFun() override {
    std::cout << "attach third seatrow in a van\n";
  }
};

class Roadster: public Car {
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

## When `typeid` is still better?

As I mentioned earlier, while `dynamic_cast` is not a good solution, it's almost always better than using `typeid()`. Why not always?

There is a not-so-subtle difference between these 2 RTTI tools. `dynamic_cast` checks whether an object "is kind of" another class. But `typeid()` will look for an exact match.

```cpp
#include <iostream>
#include <memory>
#include <vector>

class A {
public:
    virtual ~A() = default;
};

class B: public A {
};

class C: public B {

};

int main() {
    std::unique_ptr<A> p1 = std::make_unique<C>();
    
    B* p2 = dynamic_cast<B*>(p1.get());
    C* p3 = dynamic_cast<C*>(p1.get());
    if (p2) {
        std::cout << "p1 is like B*. it's convertible to B\n";
    } else if (p3) {
        // we won't reach this branch due to bad ordering of if/else branches
        std::cout << "p1 is like C*. it's convertible to C\n";
    }
    
    if (typeid(*p1) == typeid(B{})) {
        std::cout << "*p1 is B\n";
    } else if (typeid(*p1) == typeid(C{})) {
        std::cout << "*p1 is C\n";
    }
    
    return 0;
}

/*
p1 is like B*. it's convertible to B
*p1 is C
*/
```

As such, `typeid()` is both simpler and faster. Even though we had to construct an extra instance of `B` and `C` in the above example to be able to compare the type information. If that's costly and/or we must do this several times, we could create a map of `std::type_index` objects.

```cpp
// ...
const std::map<std::string, std::type_index> typeMap = {
    {"A", std::type_index(typeid(A{}))},
    {"B", std::type_index(typeid(B{}))},
    {"C", std::type_index(typeid(C{}))}
};


if (p2) {
    std::cout << "p1 is like B*. it's convertible to B\n";
} else if (p3) {
    // we won't reach this branch due to bad ordering of if/else branches
    std::cout << "p1 is like C*. it's convertible to C\n";
}

if (typeid(*p1) == typeMap.at("B")) {
    std::cout << "*p1 is B\n";
} else if (typeid(*p1) == typeMap.at("C")) {
    std::cout << "*p1 is C\n";
}
// ...
```

There is a couple of things to note:
~~- `typeMap` is `const`. It's necessary, because `std::type_index` is a copyable wrapper around the non-copyable `std::type_info`. It is not default constructible. In other words, if the `map` is not `const`, then the code wouldn't compile as a mutable map's value type must be default constructible.~~
- `std::type_index` is a copyable wrapper around the non-copyable `std::type_info`. As `std::type_index` does not have a default constructor, you cannot use `operator=()` to add new items to the map. You either initialize it at declaration (which is preferable), or you use the `insert()` method.
- We use `map::at()` and cannot use `map::operator[]`. The reason is that `map::operator[]` is not `const` whereas `map::at()` has a `const` overload.

Overall, the best is still avoiding both using `typeid` and `dynamic_cast`.

## When to still use `dyanmic_cast`?

The C++ core guidelines explain well the differences between casting to a pointer or a reference type and when you should refer which ([C.147](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c147-use-dynamic_cast-to-a-reference-type-when-failure-to-find-the-required-class-is-considered-an-error) and [C.148](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c148-use-dynamic_cast-to-a-pointer-type-when-failure-to-find-the-required-class-is-considered-a-valid-alternative)), but it doesn't mean that you should use any.

In fact, even [C.153](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c153-prefer-virtual-function-to-casting) says that you should prefer using virtual functions to casting. Casting is error-prone and you can make mess it up easily. Virtual functions are safe and when you call virtual functions it's guaranteed that you'll reach the most derived function. On the other hand, we saw with `dynamic_cast` that you might end up calling an intermediary function.

As we saw earlier, with virtual functions your code will be cleaner and speed is not an issue because `dynamic_cast` is (also) slow.

## Conclusion

With Run-time Type Information, we get access to `typeid()` and `dynamic_cast`. They help us query the dynamic types of polymorphic objects. In other words, they help us identify which derived objects are held by base class pointers.

With great power, we also get great responsibility. Sadly, experience shows that responsibility is often abused. These tools are often overused and result in messy code. Using `virtual` functions and relying on dynamic dispatching is almost always better.

Besides having cleaner code, you can also benefit from shorter compile times and smaller binaries as the type information doesn't have to be generated and stored. To me, turning RTTI off is a great option to consider.

Do you rely on it?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!