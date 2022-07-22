---
layout: post
title: "The infamous bug of range-based for loops"
date: 2022-4-20
category: dev
tags: [cpp, bug, undefinedbehaviour]
excerpt_separator: <!--more-->
---
In this article, let's discuss a C++ design pattern that often referred to in modern talks, that is probably more and more in the mainstream and that [is used in the standard library too](SMART_PTRS).

We are going to learn about type erasure, what it erases after all, how to implement it and whether it's worth it.

While, type erasure as a design pattern is considered modern, it's not *that* new. The idea was introduced by Kevlin Henney in his paper [From Mechanism To Method]() back in 2020. But type erasure itself is an older idea.

But first, let's understand what type erasure really is in a nutshell.

Type erasure is the idea of removing the need of dealing with concrete types and custom interfaces.
it hides the type in it's signature, unlike templates.

This is an important requirement when you think about how to store objects in a container, for the sake of simplicity, let's say, in a vector. If you want to store objects of different types, you might build a class hierachy and you can store pointers to the base class. 

Let's say that you want to avoid runtime polymorphism and you decide to go with templates classes. You cannot store different specializations in a vector. You cannnot declare a vector that takes X with whatever type T. Not even if you use a constrained template type. At least, not yet. Right now, to make it work, we'd need to erase the type from the signature.

There is a quite old technique that achieves type erasure in C++. There is a type that removes the concrete type of the parameters, `void*`. The problem with taking `void*` parameters that you don't just erase the types but it removes any type safety from the solution.  It's a dangerous relic from the past.

>But why is it dangerous? Think about the `qsort`. It takes a `void*` as a pointer to the data, it takes the count of the data and the size of each element. It also takes a comparator, which itself takes two items to compare, both by `void*`. Here is an example for the comparator:
>
>```cpp
>int int_greater (const void * a, const void * b)
>{
>  return *(int*)b - *(int*)a;
>}
>```
>
>From an erased type we have to deliberately cast to the type we expect. But there is no compile time check that it's valid. We can easily mix it up and use it with the wrong container. We'll only learn about it at run-time.


- Talk about polymorphism with interfaces
- Poly with templates
- The idea of TE (add examples)

Imagine that you don't want to rely a common base class, or that you can't! Maybe the implementation of those base classes are not in your control.

You need to define your own interface through a kind of a wrapper. Having wrappers for several classes can be kind of a boilerplate and while different techniques, such as templates do help, expsoing them to the caller is not very clean solution.

If we can hide all that and remvoe the burden of interacting with custom interfaces and templates, we end up with something we call modern type erasure.

- transfer the idea to something more generic from Klaus!


How is a type erased solution look like.

There is a so called concept with pure virutal methods. The API of the concept is the API you want to support. It really represents the concept behind what is represented.

```cpp
struct ShapeConcept
{
 virtual ~ShapeConcept() {}
 virtual void serialize( /*...*/ ) const = 0;
 virtual void draw( /*...*/ ) const = 0;
 // ...
};
```

Then there is the the so called Model, that inherits from the Concept. It's responsability is to model a concrete type of a concept. As it has to represent many types, it's a class template and it's responibility is to forward call's to the underlying type. Or the free functions! 

!!! 
We can have two version. Try two versions.
!!!

https://meetingcpp.com/mcpp/slides/2021/Type%20Erasure%20-%20A%20Design%20Analysis9268.pdf
61
this better than internalizing it, like in 
https://davekilian.com/cpp-type-erasure.html

Then we just have to wrap everything into a class

use the greeter example?

use cryprto (encrypt, decrypt)




watch: Inheritance is the base class of evil

from mechanism to method, valued conversion, C++ Report, kevlin henney

TE is not a void*
not a pointer to base
not a variant

TE is a templated ctor
non-virtual interface
mix of external poly, bridge, proto

we have simple, very basic classes (Circle, Square)

we have a base class (virt dtor), shapeconcept
and as well, abstracts for serialize, draw.

we have a templated model, taking a shape

template<typename GeomShape, typename DrawStrategy>
struct ShapeModel: ShapeConcept {
  ShapeModel(GeomShape const& shape, DrawStrategy strategy) : 
    shape_ (shape), strategy_(strategy) {}
  GeomShape shape_;
  DrawStrategy strategy_;

  void serialize () const override { // forwards to free functions!
    serialize(shape_, /* ... */)
  }

  void draw () const override {
    strategy_(shape_, /* ... */)
  }
};

so far external polymor, with concept and model
  1996 Cleeland

void serialize(Cirtcle const& ...);
void serialize(Square const& ...);

drawAllShapes(vector<unique_ptr<ShapeConcepts>> shapes) {
  for (auto const& shape: shapes)
  {
    shape->draw;
  }
}

int main() {
  using Shapes = vector<unique_ptr<ShapeConcepts>>;
  auto drawRedShape = [color = Color::Red] (auto const& shape, ...) {
    draw(circle, color, /* .. */);
  };
  using DrawStrat = decltype(drawRedShape);

  Shapes shapes;
  shapes.emplace_back(make_unique<ShapeModel<Circle, DrawStrat>>(Circle{...}, drawRed));

  drawAllShapes(shapes);
}

So far, only ext poly, now add more:


class Shape {
  private:
    struct ShapeConcept {
      // ...
      virtual unique_ptr<ShapeConcept> clone() const = 0; // proto DP
    };

    template<typename GeomShape, typename DrawStrategy>
      struct ShapeModel: ShapeConcept {
      ShapeModel(GeomShape const& shape, DrawStrategy strategy) : 
        shape_ (shape), strategy_(strategy) {}
      GeomShape shape_;
      DrawStrategy strategy_;
      // ...

      unique_ptr<ShapeConcept> clone() const override {
        return make_unique<ShapeModel>(*this);
      }

      void serialize () const override { // forwards to free functions!
        serialize(shape_, /* ... */)
      }

      void draw () const override {
        strategy_(shape_, /* ... */)
      }
    };

    friend void draw(Shape const& shape, ...) {
      shape.pimpl->serialize(...);
    }

   unique_ptr<ShapeConcept> pimpl; // the brdige DP, the type is erased! we don't know what's the type we have!

  public:
   template <typename GeomShape, typename DrawStrategy>
    Shape(GeomShape const& shape, DrawStrategy strategy):
    pimp(new ShapeModel<GeomShape, DrawStrategy>(shape, strat)) {}
    //the ctor creates a bridge,takes dependencies togeather

    // default dtor and moves
    // copyies? with clone!

};


void serialize(Cirtcle const& ...);
void serialize(Square const& ...);

drawAllShapes(vector<Shape> shapes) {
  for (auto const& shape: shapes)
  {
    draw(shape);
  }
}

int main() {
  using Shapes = vector<Shapes>;
  auto drawRedShape = [color = Color::Red] (auto const& shape, ...) {
    draw(circle, color, /* .. */);
  };

  Shapes shapes;
  shapes.emplace_back(Circle{...}, drawRed);

  drawAllShapes(shapes);
}


SParent



## Conclusion


## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!


## References
https://aherrmann.github.io/programming/2014/10/19/type-erasure-with-merged-concepts/
https://akrzemi1.wordpress.com/2013/11/18/type-erasure-part-i/
https://davekilian.com/cpp-type-erasure.html

https://aherrmann.github.io/programming/2014/10/19/type-erasure-with-merged-concepts/