---
layout: post
title: "User-defined literals in C++"
date: 2020-10-21
category: dev
tags: [cpp, tutorial, strongtypes]
excerpt_separator: <!--more-->
---
[Last time](https://www.sandordargo.com/blog/2020/10/14/strong-types-for-containers) we discussed strong types and in particular, strongly typed containers. We introduced the idea through a constructor that takes two integers and two boolean values and we saw how easy it is to mess them up.
<!--more-->

## A little recap of the problem

There is not much difference between the two below instantiations of the `Car` constructor
```cpp
Car::Car(unit32_t horsepower, unit32_t numberOfDoors, bool isAutomatic, bool isElectric);
//...
auto myCar{Car(96, 4, false, true)};
auto myCar{Car(4, 96, true, false)};
```

Yet one doesn't make much sense, while the other is something meaningful. Then we ended up with the following constructor and instantiations:

```cpp
Car::Car(Horsepower hp, DoorsNumber numberOfDoors, Transmission transmission, Fuel fuel);
auto myCar = Car{Horsepower{98u}, DoorsNumber{4u}, Transmission::Automatic, Fuel::Gasoline};
auto myCar = Car{DoorsNumber{98u}, Horsepower{4u}, Transmission::Automatic, Fuel::Gasoline}; // Really?
```

Here we could, we can already see the value of strong typing, it's much more difficult to make a mistake. Not only the - sometimes hardcoded - numbers and the variable names represent values, but the types as well. One more checkpoint.

Though that's not the last step if you want to increase safety and readability, especially in unit tests, where most of the hardcoded values reside.


## User-defined literals to the rescue

User-defined literals allow integer, floating-point, character, and string literals to produce objects of user-defined type by defining a user-defined suffix.

Ok, what does it mean in practice?

It means that still keeping the strong types of `Horsepower` and `DoorsNumber`, you can declare a `Car` object as such:

```cpp
auto myCar = Car{98_hp, 4_doors, Transmission::Automatic, Fuel::Gasoline};
```

Just like in the previous version, you have to write the type or something similar, yet if you look at it, it seems more natural to write `98_hp` or `4_doors` than `Horsepower(98u)` or `DoorsNumber(4u)`. We are closer to the ideal state of code when it reads like a well-written prose as Grady Booch wrote in [Object
Oriented Analysis and Design with Applications](https://amzn.to/3jR0mXn).

All that you need for that is a user-defined literal for both types. For the sake of brevity, let's omit `Transmission` and `Fuel`.

```cpp
#include <iostream>

class Horsepower {
public:
  Horsepower(unsigned int performance) : m_performance(performance) {}
private:
 unsigned int m_performance;
};

Horsepower operator"" _hp(unsigned long long int horsepower) { //1
    return Horsepower(horsepower); //2
}

class DoorsNumber {
public:
  DoorsNumber(unsigned int numberOfDoors) : m_numbeOfDoors(numberOfDoors) {}
private:
 unsigned int m_numbeOfDoors;
};

DoorsNumber operator"" _doors(unsigned long long int numberOfDoors) { //3
    return DoorsNumber{static_cast<unsigned int>(numberOfDoors)}; //4
}

class Car {
public:
  Car(Horsepower performance, DoorsNumber doorsNumber) : m_performance(performance), m_doorsNumber(doorsNumber) {}
private:
  Horsepower m_performance;
  DoorsNumber m_doorsNumber;
};

int main() {
  auto car = Car{98_hp, 4_doors};
}
```

There are a couple of things to notice here. On lines 1) and 3) we use `unsigned long long int`. Either we envision extremely powerful cars with a door for everyone in the world, or there is something else going on.

It's something else.

For a reason that I haven't found myself, only [about a dozen types are allowed on literal operators](https://en.cppreference.com/w/cpp/language/user_literal) and this seemed to be the best available option.

This doesn't mean that we should change the types wrapped by `Horsepower` or `DoorsNumber` . There is no reason to change them, so in the literal operators, we must narrow from an `unsigned long long int` to an `unsigned int`. 

We could of course fall back an implicit narrowing as we did on line 2), but implicit conversions are barely a good idea, and [narrowing conversions are even worse - even according to the Core Guidelines](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-narrowing). If you really must perform one, [be explicit about it](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-casts-named), like we were on line 4). Please note, that probably `gsl::narrow_cast` is a better idea, given that you have access to `gsl`.

`static_cast` has no performance overhead like `dynamic_cast` has, so that cannot be a concern. And besides, the above usage is mostly to increase the readability of unit tests, and their performance is not a big concern.

But I don't want to imply that user-defined literals can only be useful when you write unit tests. Even with the above usage, you might increase the readability of your production code when you define some constants, but more importantly there can be other usages.

Imagine that it makes come conversions, such as you could use it for converting between Celsius and Fahrenheit.

```cpp
#include <iostream>


long double operator"" _celsius_to_fahrenheit(long double celsius) {
    return celsius * 9 / 5 +32;
}

int main() {
  std::cout << "100 Celsius is " << 100.0_celsius_to_fahrenheit << std::endl;
}
```

## Conclusion

Today, we have learned about user-defined literals, a powerful way to boost the readability of your code. Whether you want to perform some conversions on certain primitive types or you want to improve the instantiation of your strongly-typed primitives, user-defined literals will help you.

Have you already used them? Please share your use-cases!