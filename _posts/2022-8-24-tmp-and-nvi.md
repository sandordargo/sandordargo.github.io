---
layout: post
title: "The Template Method Pattern and the Non-Virtual Idiom"
date: 2022-8-24
category: dev
tags: [cpp, design patterns, tmp, nvi]
excerpt_separator: <!--more-->
---
The above title is also one of the chapter titles from [Hands-On Design Patterns with C++ by Fedor Pikus](https://devreads.sandordargo.com/hands-on-design-patterns-by-fedor-pikus/). I liked the idea so much that I quickly started to use it and I wanted to share some more details about this pattern and idiom.
<!--more-->

But first, let's briefly discuss what is the difference between a pattern and an idiom. In short, patterns are language-agnostic and relate to design, while idioms are language-specific and relate to the code. For more details, [check this out](https://wiki.c2.com/?IdiomOrPattern).

## The Template Method Pattern

After having read the title, you might ask why we speak both about The Template Method Pattern (*TMP* from now on) and Non-Virtual Idiom (*NVI* from now on). The *TMP* is a classic design pattern from the [Gang Of Four book](https://amzn.to/36VKyO2) and *NVI* is an idiom specific to C++.

*TMP* is the go-to pattern when you have to implement an algorithm with a given structure but where some of the details have to be customized. Let's take the example of refueling a car. No matter whether you use a petrol or an electric car, first, you have to follow an algorithm like this:

```
stopTheCar();
plugTheFeed();
waitUntilEnoughFuelTransmitted();
unplugTheFeed();
```

The parts of the algorithms are following each other always in the same order, but the parts, or at least some of them will differ. Stopping the car and waiting, might be very similar. They might not even differ - depending on the level of abstraction, we have.

How are we going to involve C++ templates in this solution? The answer is simple. We won't. In the *Template Method Pattern*, *template* doesn't refer to this generic programming concept. It simply means that we are going to have a template for our algorithm.

```cpp
class BaseCar {
public:
	void fuelUpCar() {
		stopTheCar();
		plugTheFeed();
		waitUntilEnoughFuelTransmitted();
		unplugTheFeed();
	}

	// ...
};
```

The steps of the algorithm can be implemented directly in the base class, or at least it might provide a default implementation and the rest would be pure virtual making it mandatory for all the derived classes to implement them.

```cpp
class BaseCar {
public:
	void fuelUpCar() {
		stopTheCar();
		plugTheFeed();
		waitUntilEnoughFuelTransmitted();
		unplugTheFeed();
	}

private:
	virtual void stopTheCar() { /* ... */ };
	virtual void plugTheFeed() = 0;
	virtual void waitUntilEnoughFuelTransmitted() { /* ... */ };
	virtual void unplugTheFeed() = 0;

	// ...
};
```

There are several advantages of using the *TMP*.
- We can control which parts of the algorithm can be modified by a subclass
- We decrease code duplication by keeping the common parts in the base class
- We increase maintainability as new common logic doesn't have to be added at multiple places

## The Non-Virtual Interface idiom

It's time to discuss the *Non-Virtual Interface* idiom. 

You might have noticed that the virtual functions we created are listed after a `private` access specifier. Software development is about breaking down complexities. Programming is about making the complex simple. Just think about the first [SOLID](https://en.wikipedia.org/wiki/SOLID) principle. An entity should be responsible for one thing, no more. Or in a better interpretation, we would say that an entity should change only for one single reason. Still, the first interpretation shows our inherent longing for simplicity.

Non-virtual interfaces are about simplicity. Let's think about what public virtual functions represent?!

It represents both a customization point for the implementation and a public interface.

With *NVI*, we separate those roles and what is part of the public interface becomes non-virtual. The public interface won't be restated in derived classes. At the same time, with *NVI*, the customization points (i.e. the virtual functions) become non-public, preferably private.

Combining the *NVI* with *TMP* means that your public interface will always be non-virtual and it's basically one function that runs the whole algorithm. Let's expand our previous example.

```cpp
class BaseCar {
public:
	void fuelUpCar() {
		stopTheCar();
		plugTheFeed();
		waitUntilEnoughFuelTransmitted();
		unplugTheFeed();
	}

private:
	virtual void stopTheCar() { /* ... */ };
	virtual void plugTheFeed() = 0;
	virtual void waitUntilEnoughFuelTransmitted() { /* ... */ };
	virtual void unplugTheFeed() = 0;

	// ...
};

class ElectricCar : public BaseCar {
private:
	void plugTheFeed() override { /* ... */}
	void unplugTheFeed() override { /* ... */}
};

class FossilFuelCar : public BaseCar {
private:
	void plugTheFeed() override { /* ... */}
	void unplugTheFeed() override { /* ... */}
};
```

In this example, we can easily observe how we managed to separate the public interface and all the customization points. The customization does not happen through the public interface, but it's done in non-public virtual methods. The control of the public interface stays completely with the base class.

There is one public method though that should still be virtual. The destructor. We probably all know that deleting a polymorphic object, [deleting a derived class through a base class pointer without having a virtual destructor results in *undefined behaviour*](https://github.com/isocpp/CppCoreGuidelines/blob/036324/CppCoreGuidelines.md#c35-a-base-class-destructor-should-be-either-public-and-virtual-or-protected-and-nonvirtual).

```cpp
BaseCar* car = new ElectricCar{};
delete car; // this is UB!
```

If you don't delete objects like that, there is nothing to be afraid of. The problem is that you cannot make such assumptions, even if you avoid deleting through base class pointers, you cannot be sure that someone will not come and do so. And sometimes it would be quite limiting. Better be safe, the destructor is not part of the *NVI* idiom and we should make our base class destructors virtuals.

Using *TMP* and *NVI* is widely accepted as it doesn't really have any specific drawbacks. It's not a silver bullet, your base class might be a bit fragile and composability is questionable but these problems have nothing to do with having private virtuals, it's more about the problems of object-oriented design - therefore we're not going into details here. *NVI* doesn't make these problems worse.

## Conclusion

The Template Method Pattern can be used with any object-oriented language and despite its name, it has nothing to do with generics. The Non-Virtual Interface is a way of implementation specific to C++. It decouples the public interface by making it non-virtual, from functions that are providing customization points. It's all about making complex things simpler - that is our job as developers.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!