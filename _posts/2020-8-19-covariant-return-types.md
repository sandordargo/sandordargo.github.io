---
layout: post
title: "Covariant return types"
date: 2020-8-19
category: dev
tags: [cpp, covariant, tutorial, oop]
excerpt_separator: <!--more-->
---
Even after spending years in software development, you will find expressions that you simply don't understand. Even if you are considered somewhat a senior. Those terms might express an advanced concept or something that is more basic, it doesn't matter. You should always be humble enough to accept that you don't understand them and hungry enough to seek for comprehension.
<!--more-->

I spent quite some time reading about test contravariance and even though I didn't understand the word _contravariance_, by devoting some time to the topic I understood the concept without understanding the word. Then I came through _"covariant return types"_ in the boost documentation, then on other blogs and it became crystal clear that I'm missing something important.

In this post, I attempt to provide a summary of my understanding on covariant return types.

The most simple explanation is that when you use covariant return types for a virtual function and for all its overriden versions, you can replace the original return type with something narrower, in other words, with something more specialized.

Let's take a concrete example in the realms of automobiles.

Let's say you have a `CarFactoryLine` producing `Car`s. The specialization of these factory lines might produce `SUV`s, `SportsCar`s, etc.

How do you represent it in code?

The obvious way is still having the return type as a Car pointer, right?

```cpp
class CarFactoryLine {
public:
	virtual Car* produce() {
		return new Car{};
	}
};

class SUVFactoryLine : public CarFactoryLine {
public:	
	virtual Car* produce() override {
		return new SUV{};
	}
};
```

This will work as long as a `SUV` is a derived class of `Car`.

But working like this is cumbersome because if you directly try to get a SUV out of your SUVFactory line, you will get a compilation error:

```cpp
int main () {
    SUVFactoryLine sf;
    SUV* c = sf.produce();
}
/*
output:
main.cpp: In function 'int main()':
main.cpp:27:20: error: invalid conversion from 'Car*' to 'SUV*' [-fpermissive]
   27 | SUV* c = sf.produce();
      |          ~~~~~~~~~~^~
      |                    |
      |                    Car*
*/

```

So it means you have to apply a dynamic cast, somehow like this:
```cpp
// ...
int main () {
    SUVFactoryLine sf;
    Car* car = sf.produce();
    SUV* suv = dynamic_cast<SUV*>(car);
    if (suv) {
        std::cout << "We indeed got a SUV\n";
    } else {
        std::cout << "Car is not a SUV\n";
    }
}
/*
output:
We indeed got a SUV
*/
```

For the sake of brevity, I didn't delete the pointers. It's already too long.

So ideally, `SUVFactoryLine::produce` should be able to change its return type fixed into `SUV*` while still keeping the [override specifier](http://sandordargo.com/blog/2018/07/05/cpp-override). Is that possible?

It is!

This below example works like a charm:

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

But you could also directly get a `Car*` from `SUVFactoryLine::produce()`, this would be also valid:
```cpp
Car* car = sf.produce();
```

## Conclusion

What we have seen in `SUVFactoryLine` is that in C++, in a derived class, in an overriden function you don't have to return the same type as in the base class, but you must return a covariant type. In other words, you can replace the original type with a "narrower" one, i.e. with a more specified data type.

As you could see, this helps a lot. There is no need for casting at all. But you must not forget to use [override specifier](http://sandordargo.com/blog/2018/07/05/cpp-override) because if you don't use it, it's easy to overlook and you might think that `SUV* SUVFactoryLine::produce()` doesn't override `Car* CarFactoryLine::produce()` while actually it does.

So in the end, when can we speak about covariant return types? When in a derived class' overriden method a narrower, a more specialized type can replace the other wider type from the base implementation. It's as simple as that.  