---
layout: post
title: "Two cases when forward declaring is not enough"
date: 2024-5-15
category: dev
tags: [cpp, forwarddeclaration, covariant, oop]
excerpt_separator: <!--more-->
---
Let me share two cases when I had to include some header files instead of just using forward declarations. I was surprised by both at first. As you will see, one was a simple overlook, but the other wasn't, the class definition was indeed needed.

Let's start with the first one.

## The case of the "missing" destructor

Here is a class definition.

```cpp
// foo.h

#include <memory>
#include <string>

class Bar;

class FooBase {
public:
    virtual ~FooBase() = default;
    const Bar& bar() const = 0;

};

class Foo: public FooBase {
public:
	Foo(std::string text, int num);

	const Bar& bar() const override;

private:
	std::unique_ptr<Bar> _bar;
};

// foo.cpp

#include <mylib/include/bar.h>
#include <mylib/include/foo.h>

Foo::Foo(std::string text, int num) : _bar(std::make_unique<Bar>(text, num)) {}
const Bar& Foo::bar() const { return *_bar; }
```
Do you already see what's wrong? It's easier here than it was in real production code because the original header was rather big.

When I built the library exposing this header, it was all fine.

On the other hand, when I built another library that instantiated `Foo` and owned its lifetime, the compiler started to complain.

The compiler said that it cannot destruct `Foo` because `Bar` is an incomplete type. To solve that I either had to include `#include <mylib/include/bar.h>` in `foo.h` or at the client using `Foo`. I was reluctant to do either, but to solve the immediate build issue I went with the latter.

Obviously, I received a comment on my pull request indicating that if that header is needed I should include it in `foo.h`. But I didn't want to do that, because I thought it was needed there. That's when I looked at it again.

Needless to say, in real life, `Foo` is not called *Foo* and it's much bigger with plenty of constructor parameters and members. So when I looked at it once again, I realized that I forgot to declare the destructor and define it out of line. As such, the compiler generated it as if it was part of the header file. Given that, users calling `~Foo`, needed `Bar`'s definition within `foo.h`. Adding `~Foo() override;` to the header and `Foo::~Foo() = default` to the implementation solved the problem.

It's also worth noting that if you want to get a hold on the `Bar` object that is returned from `Foo::bar()`, you need to include `bar.h`, but if you just want to pass around the reference, or `Foo` itself, you'll be fine.

## Covariant return types

Here is another example of when an extra header inclusion was needed.

In 2020, [we already talked about covariant return types](https://www.sandordargo.com/blog/2020/08/19/covariant-return-types). In short, you can override a function in such a way that it doesn't return the same type as the base virtual function, but a derived class of the original return type.

Let me share the same example as a few years ago.

```cpp
#include <iostream>

class Car {
public:
 virtual ~Car() = default;
};

class SUV : public Car {};

class CarFactoryLine {
public:
	virtual Car* produce() {
		return new Car{};
	}
};

class SUVFactoryLine : public CarFactoryLine {
public:
	virtual SUV* produce() override {
		return new SUV{};
	}
};


int main () {
    SUVFactoryLine sf;
    SUV* car = sf.produce();
}
```

As you can see, while the base `CarFactoryLine::produce` returns a `Car*`, the override, `SUVFactoryLine::produce` returns a `SUV*` and that's perfectly fine.

What I didn't write about in this example is what this situation requires in terms of `Car` and `SUV` definition availability.

In real life, you'd hardly have all that code in one file.

The following is much more realistic.

```cpp
// car.h
class Car {
public:
 virtual ~Car() = default;
};

// suv.h
#include <car.h>
class SUV : public Car {};


// car_factory_line.h

class Car;

class CarFactoryLine {
public:
	virtual Car* produce();
};

// car_factory_line.cpp
#include <car_factory_line.h>
#include <car.h>

Car* CarFactoryLine::produce() {
	return new Car{};
}


// suv_factory_line.h

#include <car_factory_line.h>

class SUV;

class SUVFactoryLine : public CarFactoryLine {
public:
	virtual SUV* produce() override;
};

// suv_factory_line.cpp
#include <suv_factory_line.h>
#include <suv.h>

SUV* SUVFactoryLine::produce() {
	return new SUV{};
}

// main.cpp
#include <suv_factory_line.h>
#include <suv.h>

int main () {
    SUVFactoryLine sf;
    SUV* car = sf.produce();
}
```

Note the factories are separated into header and implementation files and even `Car` and `SUV` would be. Also, notice that `Car` and `SUV` are forward declared in their factories.

The problem is that the above example wouldn't work. While we know that `SUV` is a covariant of `Car` and therefore it's OK to return it, the compiler doesn't know. All that it sees is that there is a `Car` forward-declared in `car_factory_line.h` and `SUV` in `suv_factory_line.h`. To the compiler at that point, those two classes are unrelated and the compilation fails.

In order to fix that, instead of forward declaring `SUV`  in `suv_factory_line.h`, we must properly include `suv.h`. Even though you don't need to have the full definition available of a class that only appears as a return type, when the compiler needs more information on class relationships, you have no choice.

If you wonder, no, you cannot forward declare a class with its base class.

I'm not saying that this should prevent you from using the technique of returning covariant types in overrides, but you must be aware of this problem.

## Conclusion

Today, I shared with you two small stories about when forward declarations were not enough and I had to include the header files instead. We saw that the first case was about an overlook on my side, but it's still a reminder that if you let the compiler implicitly declare and define the destructor then depending on the class, forward declarations of members might not be enough to properly destruct an instance.

The second case was about returning covariant types in overrides. It's not enough that we know that two types are related, the compiler also has to be able to infer it. Solely from forward declarations, that is impossible to do, you have to include the declaration of the derived type (which inevitably brings in the declaration of the base class as well).

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
