---
layout: post
title: "C++26: std::indirect"
date: 2026-8-12
category: dev
tags: [cpp, cpp26, valuesemantics, smartpointers]
excerpt_separator: <!--more-->
---
C++26 adds two new vocabulary types in `<memory>`, introduced by [P3019R14](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3019r14.html) (Coe, Peacock, Parent). From the abstract:

> *The class template `indirect` confers value-like semantics on a dynamically-allocated object. An `indirect` may hold an object of a class `T`. Copying the `indirect` will copy the object `T`. When an `indirect<T>` is accessed through a const access path, constness will propagate to the owned object.*
>
> *The class template `polymorphic` confers value-like semantics on a dynamically-allocated object. A `polymorphic<T>` may hold an object of a class publicly derived from `T`. Copying the `polymorphic<T>` will copy the object of the derived type. When a `polymorphic<T>` is accessed through a const access path, constness will propagate to the owned object.*

As you can tell, these two types are very close in spirit. They used to be two separate proposals — P1950 for `indirect` and P0201 for `polymorphic` — before being merged into one paper. Likewise, I originally planned to cover both in a single article, but it grew long enough that I decided to split it. This post covers `std::indirect`; the next one will cover `std::polymorphic`.

<!--more-->

## The problem with `unique_ptr`

`std::unique_ptr` has two fundamental issues when used as a member of a value-type class.

**First, it breaks const propagation.** `unique_ptr::operator*() const` returns a non-const `T&`. A `const` object can mutate its indirectly-stored members:

```cpp
// https://godbolt.org/z/P7zdodhsd

struct Settings {
    int volume = 50;
    bool muted = false;
};

class Player {
    std::unique_ptr<Settings> settings_;
public:
    Player() : settings_(std::make_unique<Settings>()) {}

    void mute() const {
        settings_->muted = true;  // compiles — mutates through const!
    }
};

const Player p;
p.mute();  // const-correctness is broken
```

**Second, it deletes copy operations.** If `Car` should be copyable, you must write all five special member functions yourself. This is the tedious [Rule of Five](https://www.sandordargo.com/blog/2024/07/31/rule-of-5-once-again) boilerplate that every C++ developer knows too well.

## `std::indirect` — value semantics for heap-allocated objects

`std::indirect<T>` is what `std::unique_ptr<T>` would be if it had been designed for composite class members rather than ownership transfer. It owns a heap-allocated `T` and provides deep copies, const propagation, value-based comparison, and hashing — all the things you'd expect from a value type.

```cpp
// https://godbolt.org/z/ePxb8E9Ko

struct Settings {
    int volume = 50;
    bool muted = false;

    bool operator==(const Settings&) const = default;
    auto operator<=>(const Settings&) const = default;
};

class Player {
    std::indirect<Settings> settings_;
public:
    Player() : settings_(std::in_place) {}

    void mute() { settings_->muted = true; }
    void set_volume(int v) { settings_->volume = v; }

    int volume() const { return settings_->volume; }
    bool is_muted() const { return settings_->muted; }

    const Settings& settings() const { return *settings_; }

    // ALL special member functions are compiler-generated.
    // Copying deep-copies the Settings. Moving transfers it.
};
```

Note that `mute()` and `set_volume()` are non-const now — as they should be. If you tried to make them `const`, the compiler would stop you: `indirect::operator->() const` returns a `const Settings*`, so `settings_->muted = true` in a `const` method is a compile error:

```
error: assignment of member 'Settings::muted' in read-only object
         settings_->muted = true; // this wouldn't compile!
```

The exact bug from the `unique_ptr` version is structurally impossible.

Let's walk through what else `indirect` gives us.

### Const propagation

Unlike `unique_ptr`, `indirect::operator*() const` returns a `const T&`. When you have a `const Player`, `settings_->` gives you a `const Settings&`, so attempting to mutate any member is a compile error. This is how member subobjects behave, and `indirect` simply extends that to the heap.

### Deep copies

Copying an `indirect<T>` copies the owned `T`. Your class becomes copyable without writing a single line of boilerplate:

```cpp
// https://godbolt.org/z/z6aEE4oP4

Player a;
a.set_volume(80);
a.mute();

Player b = a;  // deep copies the Settings
b.set_volume(30);

assert(a.volume() == 80);  // a is unchanged
assert(b.volume() == 30);  // b has its own copy
```

With `unique_ptr` this would require a hand-written copy constructor.

### Value-based comparison

If `T` supports `==` and `<=>`, then `indirect<T>` does too — by comparing the owned objects, not pointers:

```cpp
// https://godbolt.org/z/znMWdPvjr

Player a;
Player b;
assert(a.settings() == b.settings());  // true — both have volume=50, muted=false

a.set_volume(80);
assert(a.settings() != b.settings());  // true — different volume now
```

### The valueless state

`indirect` has no null or empty state by design. There is no default `operator bool()`, no `has_value()`. An `indirect` always owns an object — except after being moved from. In that case, `valueless_after_move()` returns `true`, and accessing the object is undefined behaviour.

If you need nullable indirection, use `std::optional<std::indirect<T>>`.

### When to reach for it

`std::indirect` is the right tool when you need heap allocation for structural reasons but want your class to behave like a value:

- **PIMPL**: `indirect<Impl>` replaces the usual `unique_ptr<Impl>` — no more hand-written copy/move/destructor. Marius Bancila has a [detailed walkthrough](https://mariusbancila.ro/blog/2026/07/23/the-pimpl-idiom-and-the-cpp26-stdindirect-type/) of this.
- **Recursive types**: a `struct Node { int value; std::indirect<Node> next; };` just works.
- **Large members**: moving a big member to the heap to shrink `sizeof(YourClass)` while keeping value semantics.

## Conclusion

`std::indirect` fills a gap that has existed since C++11 introduced move semantics and smart pointers. `unique_ptr` solved ownership, but it never solved *value semantics* for indirectly-stored objects. With `indirect`, PIMPL implementations lose their boilerplate, composite classes get correct const propagation, and deep copies, comparison, and hashing all work without writing a single special member function.

In the next article, we'll look at its sibling `std::polymorphic`, which extends the same idea to class hierarchies — giving you polymorphic containers with value semantics and no `clone()` methods.

{% include connect-deeper.html %}
