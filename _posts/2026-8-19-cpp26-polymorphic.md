---
layout: post
title: "C++26: std::polymorphic"
date: 2026-8-19
category: dev
tags: [cpp, cpp26, valuesemantics, polymorphism]
excerpt_separator: <!--more-->
---
In the [previous article](https://www.sandordargo.com/blog/2026/08/12/cpp26-indirect), we looked at `std::indirect` — C++26's answer to the "I need a heap-allocated member with value semantics" problem. `indirect<T>` always stores exactly a `T`. But what if you need to store a `Circle` or a `Rectangle` through a `Shape` base class — and copy it without slicing?

That's `std::polymorphic<T>`, the other half of [P3019R14](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3019r14.pdf). It owns a heap-allocated object whose dynamic type may be `T` or any type derived from `T`. Copying performs a type-erased deep copy that preserves the dynamic type. No virtual `clone()` needed.

<!--more-->

## The `clone()` tax

To have a copyable collection of polymorphic objects, every derived class needs a virtual `clone()` method. Let's see what that looks like:

```cpp
class Shape {
public:
    virtual ~Shape() = default;
    virtual std::unique_ptr<Shape> clone() const = 0;
    virtual double area() const = 0;
};

class Circle : public Shape {
    double radius_;
public:
    explicit Circle(double r) : radius_(r) {}
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Circle>(*this);
    }
    double area() const override { return 3.14159 * radius_ * radius_; }
};

class Rectangle : public Shape {
    double w_, h_;
public:
    Rectangle(double w, double h) : w_(w), h_(h) {}
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Rectangle>(*this);
    }
    double area() const override { return w_ * h_; }
};
```

Now you want a `Picture` that owns a collection of shapes with value semantics:

```cpp
// full example at https://godbolt.org/z/Evz8b7r8o
class Picture {
    std::vector<std::unique_ptr<Shape>> shapes_;
public:
    Picture(const Picture& other) {
        shapes_.reserve(other.shapes_.size());
        for (const auto& s : other.shapes_)
            shapes_.push_back(s->clone());
    }
    Picture& operator=(const Picture& other) {
        auto tmp = other;          // copy-and-swap
        swap(shapes_, tmp.shapes_);
        return *this;
    }
    // ... move constructor, move assignment, destructor
};
```

Every new shape has to implement `clone()`. Forget it once and you get slicing. The `Picture` class needs all five special member functions written by hand. This is a lot of ceremony for something conceptually simple: "I want a copyable collection of polymorphic objects."

## Dropping the boilerplate with `std::polymorphic`

`std::polymorphic<T>` eliminates all of that. The shapes don't need `clone()`, `Picture` doesn't need any special member functions, and — perhaps surprisingly — `Shape` doesn't even need a virtual destructor:

```cpp
#include <memory>

class Shape {
protected:
    ~Shape() = default;  // no virtual destructor needed!
public:
    virtual double area() const = 0;
};

class Circle : public Shape {
    double radius_;
public:
    explicit Circle(double r) : radius_(r) {}
    double area() const override { return 3.14159 * radius_ * radius_; }
};

class Rectangle : public Shape {
    double w_, h_;
public:
    Rectangle(double w, double h) : w_(w), h_(h) {}
    double area() const override { return w_ * h_; }
};

class Picture {
    std::vector<std::polymorphic<Shape>> shapes_;
public:
    void add_circle(double r) {
        shapes_.emplace_back(std::in_place_type<Circle>, r);
    }
    void add_rectangle(double w, double h) {
        shapes_.emplace_back(std::in_place_type<Rectangle>, w, h);
    }

    double total_area() const {
        double sum = 0;
        for (const auto& s : shapes_)
            sum += s->area();
        return sum;
    }

    size_t size() const { return shapes_.size(); }

    // ALL special member functions are compiler-generated.
    // Copying deep-copies every shape, preserving its dynamic type.
};
```

No `clone()`. No Rule of Five. Copying a `Picture` copies every shape — a `Circle` is copied as a `Circle`, a `Rectangle` as a `Rectangle`. The type-erasure machinery inside `polymorphic` handles this automatically. The only requirement is that every stored type must be copy-constructible.

## Deep copies just work

```cpp
Picture a;
a.add_circle(5.0);
a.add_rectangle(3.0, 4.0);

Picture b = a;  // deep copies both shapes, preserving their dynamic types
a.add_circle(1.0);

assert(a.size() == 3);  // a has three shapes
assert(b.size() == 2);  // b is unchanged — it's an independent copy
assert(a.total_area() != b.total_area());
```

With `unique_ptr` this would require the hand-written copy constructor we saw above.

## No virtual destructor needed

Notice that `Shape`'s destructor is `protected` and non-virtual in the `polymorphic` version. Because `polymorphic` uses type erasure for both destruction and copying, it knows the actual dynamic type and can call the right destructor directly — no vtable dispatch needed. This is a genuine safety improvement: you can't accidentally `delete` a `Shape*` and get undefined behaviour, because nobody ever holds a raw `Shape*`.

## Const propagation

Just like `std::indirect`, `polymorphic` propagates const correctly. Accessing through a `const polymorphic<Shape>&` yields a `const Shape&`. In the `Picture` example, `total_area()` is a `const` method, and `s->area()` correctly resolves to the `const` overload of `area()`. If `Shape` had a non-const mutating method, calling it through a `const Picture` would be a compile error — the same improvement over `unique_ptr` that we discussed in the [previous article](https://www.sandordargo.com/blog/2026/08/12/cpp26-indirect).

## What `polymorphic` does *not* provide

Unlike `indirect`, `polymorphic` has **no comparison operators and no hash support**. The reason is straightforward: the dynamic type is erased. There is no way to forward `==` or `<=>` to the actual stored object without requiring virtual comparison methods on the base class, which the committee considered out of scope.

There is also no perfect-forwarded assignment (`operator=(U&&)`) — again because the type information is erased at runtime.

## The valueless state

Like `indirect`, `polymorphic` has no null state. A `polymorphic<T>` always owns an object — except after being moved from, where `valueless_after_move()` returns `true`. If you need nullable polymorphic indirection, use `std::optional<std::polymorphic<T>>`.

## Choosing between `indirect` and `polymorphic`

Now that we've seen both types, here's a quick summary of when to use which:

| Scenario | Type |
|----------|------|
| PIMPL, recursive types, large members | `std::indirect<T>` |
| Need comparison/hash on the owned object | `std::indirect<T>` |
| Open set of derived types, polymorphic collections | `std::polymorphic<T>` |
| Closed set of known types | `std::variant<A, B, C>` |
| Nullable indirection | `std::optional<indirect<T>>` or `std::optional<polymorphic<T>>` |
| Shared ownership | `std::shared_ptr<T>` (as before) |

The simplest mental model: `indirect` is for when you know the exact type but need it on the heap. `polymorphic` is for when you don't.

## Conclusion

`std::polymorphic` eliminates the most tedious pattern in object-oriented C++: writing `clone()` methods and Rule of Five boilerplate just to have a copyable collection of polymorphic objects. The type-erasure approach means no virtual destructor, no virtual `clone()`, and no hand-written special member functions. Together with `std::indirect`, it completes the value-semantic indirection story that has been missing from the standard library since smart pointers were introduced.

{% include connect-deeper.html %}
