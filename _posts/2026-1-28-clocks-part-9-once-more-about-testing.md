---
layout: post
title: "Time in C++: Once More About Testing"
date: 2026-1-28
category: dev
tags: [cpp, chrono, clocks, time]
excerpt_separator: <!--more-->
---
I planned to finish this series this week. But then I realized that there are still a couple of important things about testing that I haven't written about yet. We already touched on the problem of testing when we discussed [`system_clock`](https://www.sandordargo.com/blog/2025/11/26/clocks-part-2-system_clock), but it definitely deserves a bit more attention.

We said that as it's not really possible to mock `system_clock`, it’s a good idea to wrap your clock and pass that wrapper into your functions. That way, in tests, you can substitute it with a fake implementation where you control the returned time.

Let's first look at the problem in a bit more detail, and then walk through a few possible solutions.

## The problem of time and testing

Let's say you have a function that takes some values, combines them into a record, and stores that record in a database. Oh, and besides the values, it also adds the time of creation.

For the sake of simplicity, our "database" will just be a `std::vector`, and the record will contain only one value besides a timestamp:

```cpp
struct Record {
    int value;
    std::chrono::system_clock::time_point created_at;
};

std::vector<Record> db;

void store(int value) {
  db.emplace_back(value, std::chrono::system_clock::now());
}
```
Now the problem is... how do you test that the database contains a record with the expected properties?

You can check the value field easily, but validating `created_at` is tricky. You could approximate it (for example, assert that it's within a certain range), but that approach is brittle, inaccurate, and often leads to flaky tests.

What we really want is control. We want to decide what "now" means during the test.

## A hiearchy of clocks

In one of the earlier articles of this series — where we talked about `system_clock` — I mentioned that wrapping your clock improves testability. One straightforward solution is to create a hierarchy of clocks.

First, let's define an interface (an abstract base class in C++):

```cpp
struct Clock {
    virtual ~Clock() = default;
    virtual std::chrono::system_clock::time_point now() const = 0;
};
```

Next, we can define a real clock and a fake one. The fake clock will return a predefined timestamp, while the real clock simply forwards the call to `std::chrono::system_clock`.

```cpp
struct RealClock : Clock {
    std::chrono::system_clock::time_point now() const override {
        return std::chrono::system_clock::now();
    }
};

struct FakeClock : Clock {
    std::chrono::system_clock::time_point provided{};
    std::chrono::system_clock::time_point now() const override { return provided; }
};
```

With this in place, testing becomes straightforward. We only need to inject the clock into store:

```cpp
void store(int value, const Clock& clock) {
    db.emplace_back(value, clock.now());
}
```

Now your test can fully control time by passing in a `FakeClock`. (You can find a complete example [linked here](https://godbolt.org/z/qcMdvMKM5
).)

## Use the trait, use concepts

You might not like the idea of inheritance - and that's fair. You might ask: why not just pass in a clock-like type *directly*? After all, the standard library even defines a trait for clock-like types.

And since we're in modern C++, we should try to use concepts, right?

> If you're not familiar with concepts, [check out my series on them](https://www.sandordargo.com/tags/concepts/).

It would be great to declare store like this:

```cpp
void store(int value, ClockLike auto clock);
```

That's absolutely possible. We just need to define our own `concept`, because we can't directly use the type trait itself.

A first attempt could look like this:

```cpp
template <class C>
concept ClockLike = std::chrono::is_clock_v<C>;
```

This `concept` is both more and less permissive than the interface-based solution from the previous section.

It's less permissive, because it requires all the characteristics of a standard clock — not just the existence of a `now()` function.

But it's also more permissive, because it allows passing a `steady_clock` into store. If we do that, the code won't compile, because this line will fail:

```cpp
db.emplace_back(value, clock.now());
```

The compiler cannot assign a `steady_clock::time_point` to a `std::chrono::system_clock::time_point`. Still, store() itself happily accepted the clock.

So let's tighten the concept:

```cpp
template <class C>
concept SystemClockLike = std::chrono::is_clock_v<C> && requires {
    { C::now() } -> std::same_as<std::chrono::system_clock::time_point>;
};
```

That does the trick.

Now we need to update our fake clock — and we can completely remove the class hierarchy:

```cpp
struct FakeClock {
    using rep = std::chrono::system_clock::rep;
    using period = std::chrono::system_clock::period;
    using duration = std::chrono::system_clock::duration;
    using time_point = std::chrono::system_clock::time_point;
    static constexpr bool is_steady = false;

    static inline time_point provided{};

    static time_point now() { return provided; }
};
```

Now our test looks like this (and [here is the full example](https://godbolt.org/z/h68M5rGdv)):

```cpp
// Full example is here: https://godbolt.org/z/h68M5rGdv
void testStore() {
    FakeClock clock;
    clock.provided = std::chrono::system_clock::now();
    store(42, clock);
    assert(db[0].value == 42);
    assert(db[0].created_at == clock.provided);
}
```

## A time provider lambda

You might not have access to C++20 — or you might simply not want to emulate concepts with SFINAE, which is completely understandable. There's another, much simpler option.

If you think about it, all we really need is *something* that returns a `std::chrono::system_clock::time_point`.

We could use a function pointer or a template, but probably the most readable and flexible solution is a `std::function` (you can [read about the trade-offs here](https://www.sandordargo.com/blog/2023/04/05/binary-size-and-templates))

```cpp
using TimeProvider = std::function<std::chrono::system_clock::time_point()>;
```

Now we can remove `FakeClock` entirely. All we need to do is pass a lambda that returns a predefined timestamp.

Here's what the test looks like ([full example is here](https://godbolt.org/z/WYTxKMaYs
)):

```cpp
// Full example: https://godbolt.org/z/WYTxKMaYs
void testStore() {
    auto expected_created_at = std::chrono::system_clock::now();
    TimeProvider time_provider = [&expected_created_at]() {
        return expected_created_at;
    };
    store(42, time_provider);
    assert(db[0].value == 42);
    assert(db[0].created_at == expected_created_at);
}
```

## Which one to choose?

As usual, the answer is: it depends.

- **Inheritance-based clocks** are explicit and easy to understand, especially in object-oriented codebases. They work well when you already rely on runtime polymorphism and dependency injection.

- **Concept-based solutions** are zero-overhead, expressive, and scale nicely in template-heavy or header-only code. They shine when you want compile-time guarantees and flexibility.

- **Time provider functions** are the simplest option. They require no custom types, no inheritance, and no concepts. If all you need is "give me the current time", this approach is often more than enough.

In practice, I tend to start with the time provider approach. It's minimal, readable, and easy to refactor later if the design grows more complex.

## Conclusion

Time is one of those dependencies that silently makes code harder to test. As soon as you call `now()`, you give up control and tests suffer.

The good news is that C++ gives us plenty of tools to deal with this problem. Whether you prefer classic polymorphism, modern concepts, or simple callables, the key idea is the same: don't hardcode the source of time.

Once you treat time as an injectable dependency, your tests become deterministic, expressive, and far less fragile.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!