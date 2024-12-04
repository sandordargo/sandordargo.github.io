---
layout: post
title: "Replace CRTP with concepts?"
date: 2024-12-4
category: dev
tags: [cpp, crtp, concepts]
excerpt_separator: <!--more-->
---
In my Meeting C++ 2024 trip report, among my favourite ideas I mentioned [Klaus Iglberger's talk](https://meetingcpp.com/2024/Talks/items/Design_Patterns___The_Most_Common_Misconceptions__2_of_N_.html) where he mentioned the possibility of replacing the curiously returning template pattern with the help of class tagging and concepts.

> Class tagging might mean different things in different contexts, or at least might be implemented in different ways. The end goal is to mark, in other words, tag classes or [functions to be used in certain contexts, with certain algorithms](https://www.fluentcpp.com/2018/04/27/tag-dispatching/). As you'll see, in our case it'll also be a tool to prevent duck typing.

We are going to see an example implementation of a static interface with CRTP with a couple of different derived classes, then we'll see the implementation without CRTP.

## The CRTP solution

With the static interface, we are creating a static family of types. There is no need for dynamic polymorphism to share the same interface. It's still granted through a base class, which is a template taking the deriving class as a parameter.

Let's use animals making sounds for a sample implementation.

```cpp
// CRTP version
#include <iostream>

template<typename Derived>
class Animal {
public:
    void make_sound() const {
        const Derived& underlying = static_cast<const Derived&>(*this);
        underlying.make_sound();
     }

};

class Cow: public Animal<Cow> {
    public:
    void make_sound() const { std::cout << "moo\n"; }
};

class Sheep: public Animal<Sheep> {
    public:
    void make_sound() const { std::cout << "baa\n"; }
};

class Dog: public Animal<Dog> {
    public:
    void make_sound() const { std::cout << "wouf\n"; }
};

template<typename Derived>
void print(Animal<Derived> const& animal) {
    animal.make_sound();
}

int main() {
    Cow cow;
    print(cow);
    Sheep sheep;
    print(sheep);
    Dog dog;
    print(dog);
}
```

## The non-CRTP solution

In this case, we don't use the CRTP pattern (the base class template) to add functionality but to ensure having a common interface without the costs of dynamic polymorphism.

We can achieve that with a concept.

```cpp
template<typename T>
concept Animal = 
    requires(T animal) { animal.make_sound();};
```

The problem with the above concept is that now every class that has a `make_sound()` method will be accepted as an animal. Even if the author of the `Animal` concept or the author of those _fake_ animal classes didn't want that. That's why we are also going to require the `AnimalTag`.

```cpp
class AnimalTag {};

template<typename T>
concept Animal = 
    requires(T animal) { animal.make_sound();} &&
    std::derived_from<T, AnimalTag>;
```

We can be pretty sure that nobody will accidentally inherit from the `AnimalTag`

Now let's see the non-CRTP version.

```cpp
// non CRTP version

#include <concepts>
#include <iostream>

class AnimalTag {};

template<typename T>
concept Animal = 
    requires(T animal) { animal.make_sound();} &&
    std::derived_from<T, AnimalTag>;

void print(Animal auto const& animal) {
    animal.make_sound();
}

class Sheep: public AnimalTag {
    public:
    void make_sound() const { std::cout << "baa\n"; }
};

class Cow: public AnimalTag {
    public:
    void make_sound() const { std::cout << "moo\n"; }
};

class Dog: public AnimalTag {
    public:
    void make_sound() const { std::cout << "wouf\n"; }
};

int main() {
    Cow cow;
    print(cow);
    Sheep sheep;
    print(sheep);
    Dog dog;
    print(dog);
}
```

## In comparison

In my opinion, the non-CRTP solution is more readable and less error-prone. With the CRTP you might accidentally pass in a wrong template argument. It's true that there is a solution to that. You can make the base class constructor private and make `Derived` a friend of `Base`. But you need to think about this.

Also, for those who are not familiar with the pattern, seeing the CRTP inheritance plus the `static_cast` to the derived class is not necessarily easy to understand.

The non-CRTP solution is more readable if you are familiar with concepts. While CRTP is a not-so-well-known design pattern, concepts are part of the main language, so you'll have to get familiar with them sooner rather than later.

> *If you want to learn more about concepts, you can find [a series on this blog](https://www.sandordargo.com/tags/concepts/) and I also have [a book on concepts](https://leanpub.com/cppconcepts)*

At the same time, you need to compile using C++20, which might not be available to you at the moment.

I expected the non-CRTP solution to result in a significantly smaller binary, but I was proven wrong. With these small examples, I didn't find a consistent difference. Depending on the optimization level even one was a bit smaller or the other.

I still want to try it in a bigger example and I'll report back the results, but it might take some time.

## Conclusion

In this article, we covered how we can replace CRTP when we use it to have static interfaces for a family of classes. We saw that with C++20's concepts, we could replace CRTP and have less error-prone and more readable code. The only question is whether you can already use C++20.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
