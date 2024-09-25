---
layout: post
title: "C++23: The `<expected>` header; expect the unexpected"
date: 2022-5-4
category: dev
tags: [cpp, cpp23, expected, errorhandling]
excerpt_separator: <!--more-->
---
I was reading recently Klaus Iglberger's book on software design. And when I was studying his guidelines for the visitor pattern, I had an idea. He wrote that each visitable class must implement the `accept()` function. Despite the fact that they are all the same.

While Klaus is certainly right about that, I'm going to make the point that with C++23's expilicit object parameter, a.k.a. thanks to *"deducing `this`"*, the code repetition is not a necessity anymore.

Let's have a bried reminder first on the visitor pattern, than on *[deducing `this`]()* and then let's see the improved version.


## The visitor pattern

The visitor pattern is considered one of the more complex design patterns in C++ and to give a fully detailed explaination is somethign that is beyond the scope of this article. Yet, let me share the basic idea behind the pattern so that we can undertand the implementation and the pattern.

A simple dynamic polymorphic design where we use inheritance can easily solve the problem of frequent new types. Let's say you have a base class with a pure virtual interface and each derived class implements it. Let's say you have an Animal base class, with the pure virtual functions, `eat()` and `speak()`.

Then you add some animals who dervie from `Animal` and implement the two mentioned functions. All goes well. You add more and more animals and you never have to touch the already added animals. You already have like a dozen of them. All goes well. Such a polymorphic model based on inheritance works very wll as long as you frequently have to add new types and not new operations. To demonstrate it, now let's say that after like having created a dozen different animal classes, you realize that your model should supportAt that point you realize that you want to support in addition to `eat()` and `speak()` another operation, `move()`. In order to do that you have to modify the base class and each and every class derived from it - unless you can define a default behaviour in the base class.

In circumstances where the the addition of a new operation is much more likely than the addition of a new type, the visitor pattern performs well. THe reason is that you don't have to touch each class, you can focus on providing the implementation of the operation - at one single place, in a visitor.

Let's have a look at a sample classical implementation -  in this article we won't touch `std::visitor`.

```cpp
#include <iostream>

class AnimalVisitor;

class Animal {
public:
    virtual void accept(const AnimalVisitor&) = 0;
};

class Dog : public Animal {
public:
    void accept(const AnimalVisitor& v) override;
};

class Cat : public Animal {
public:
    void accept(const AnimalVisitor& v) override;
};

class AnimalVisitor {
public:
    virtual void visit(const Dog&) const = 0;
    virtual void visit(const Cat&) const = 0;

};

class Eat : public AnimalVisitor {
public:
    void visit(const Dog& c) const override {
        std::cout << "Eat visiting a Dog\n";
    }

    void visit(const Cat& s) const override {
        std::cout << "Eat visiting a Cat\n";
    }
};


class Speak : public AnimalVisitor {
public:
    void visit(const Dog& c) const override {
        std::cout << "Speak visiting a Dog\n";
    }

    void visit(const Cat& s) const override {
        std::cout << "Speak visiting a Cat\n";
    }
};

void Dog::accept(const AnimalVisitor& v) {
    v.visit(*this);
}

void Cat::accept(const AnimalVisitor& v) {
    v.visit(*this);
}


int main() {
    Dog d1;
    d1.accept(Speak{});
    Dog d2;
    d2.accept(Eat{});

    Cat c1;
    c1.accept(Speak{});
    Cat c2;
    c2.accept(Eat{});
}
```

It's not a short example, but as said, the visitor pattern is one of the more complex ones. If you want to add a new operation, you have to implement `AnimalVisitor` and that has no duplicated code, even though in this example they are quite alike.

On the other hand, we can also observe that the accept functions are the same everywehre! Yet, we have to type that, there is no way around. We cannot simply implement `Animal::accept()`. The line `v.visit(*this)` fails to compile, because the compiler cannot convert `Animal` to `Dog` or `Cat`.

Okay, it's not super incovnient, you have to write the code once, but it's an extra virtual function, an extra indirection and still it is duplicated code that hurts the eyes..

## Deding `this`

Deducing `this`, or on its maiden name, the "Explicit object parameter" is one of the most waited for features of C++23. If you are looking for a deeper explanation, [please check out this article](https://www.sandordargo.com/blog/2022/02/16/deducing-this-cpp23) I posted some time ago.

In short, via a new explicit object parameter, we have access to the exact type of an object. Hmm. Maybe that's something we could use with the visitor pattern?

Here are the possible syntaxes.

```cpp
struct X {
    void foo(this X const& self, int i);

    template <typename Self>
    void bar(this Self&& self);
};
```

## Visitor meets the explicit object parameter

As I mentioned when I was reading Klaus Iglberger's [BOOOOOOOOOOOOOOK](), I thought that hey one of the shortcomings could be overcome by the epxlicit object paramater. Maybe... And it indeed worked!

So first I checked on C++ Reference whether this new feature is supported by any compiler. MSVC supports it, good! Next, I lauched [Compiler Explorer]() selected the latest MSVC and added the `/std:c++latest` compiler option.

I removed the `accept()` from all the derived classes and in `Shape` I implemented it using the new object parameter like this:

```cpp
class Shape {
public:
    template <typename Self>
    void accept(this Self && self, const ShapeVisitor&);// = 0;
};

template <typename Self>
void Shape::accept(this Self && self, const ShapeVisitor& v) {
    v.visit(self);
}
```

This indeed worked, because with the `self` parameter the compiler has access to the exact type dervied from `Shape`.

You can find the working example [here](https://godbolt.org/z/Pfxfadec4), with the original `accept()` functions commented out.


## Conclusion

Writing functions that are supposed to return both a normal return value and a stauts code at the same has been pain for decades. You either had to create your custom data structure or besides return values you had to use out parameters too. Later C++17 introduced both `std::optional` and ` std::variant` offering some alternatives, but they were not designed to deal with error handling.

C++23 will bring as `std::exceptional` that is a standardized data structure with a `std::optional`-like API aiming to hold an expected value or an error code. It's a readable, easy to use solution that will increase the readability of our APIs!

Sounds very promising!


## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!





















not working yet

```cpp
#include <iostream>
struct X
{
    void foo(this X const& self, int i); // same as void foo(int i) const &;
//  void foo(int i) const &; // Error: already declared
 
    void bar(this X self, int i); // pass object by value: makes a copy of `*this`
};

class Shape;
class Circle;
class Square;

class ShapeVisitor {
public:
    virtual void visit(const Circle&) const = 0;
    virtual void visit(const Square&) const = 0;

};

class Rotate : public ShapeVisitor {
public:
    void visit(const Circle& c) const override {
        std::cout << "rotate visiting a circle\n";
    }

    void visit(const Square& s) const override {
        std::cout << "rotate visiting a square\n";
    }
};


class Draw : public ShapeVisitor {
public:
    void visit(const Circle& c) const override {
        std::cout << "draw visiting a circle\n";
    }

    void visit(const Square& s) const override {
        std::cout << "draw visiting a square\n";
    }
};

class Shape {
public:
    virtual void accept(const ShapeVisitor& v) {
        v.visit(*this);
    }
};

class Circle : public Shape {
public:
    void accept(const ShapeVisitor& v) override;
};

class Square : public Shape {
public:
    void accept(const ShapeVisitor& v) override;
};



int main() {
    Circle c1;
    c1.accept(Draw{});
    Circle c2;
    c2.accept(Rotate{});

    Square s1;
    s1.accept(Draw{});
    Square s2;
    s2.accept(Rotate{});
}

```


https://godbolt.org/z/Pfxfadec4



