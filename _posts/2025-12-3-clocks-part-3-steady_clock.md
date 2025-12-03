---
layout: post
title: "Time in C++: Understanding std::chrono::steady_clock"
date: 2025-12-3
category: dev
tags: [cpp, chrono, clocks, time]
excerpt_separator: <!--more-->
---
In the previous articles, we [explored what clocks are in general](https://www.sandordargo.com/blog/2025/11/19/clocks-part-1-intro-to-chrono) and [took a closer look at `std::chrono::system_clock`](https://www.sandordargo.com/blog/2025/11/26/clocks-part-2-system_clock), the one that represents the wall-clock or calendar time.  
Now let’s move to another important one: `std::chrono::steady_clock`.

At first sight, the difference between `steady_clock` and `system_clock` might not be obvious. They both give you a `time_point`, and you can use both to measure durations. But under the hood, they behave quite differently — and those differences make `steady_clock` the right choice for measuring **intervals** and **durations**.

## Monotonic behavior — and why it matters

A key property of `std::chrono::steady_clock` is that it is **monotonic**. That means its time never goes backwards — ever. If you query it twice, the second call will always give you a time point greater than or equal to the first.

This makes it ideal for measuring **elapsed time** or **performance** because you’ll never be surprised by sudden jumps or adjustments.

On the contrary, `system_clock` represents the "real-world" wall clock. If your operating system synchronizes time, `system_clock` might jump forward or backward — and your elapsed-time calculations would instantly become meaningless.

```cpp
auto start = std::chrono::steady_clock::now();
// ... run something ...
auto end = std::chrono::steady_clock::now();

std::chrono::duration<double> elapsed = end - start;
std::cout << "Elapsed time: " << elapsed.count() << " seconds\n";
```

With `steady_clock`, you can safely rely on this difference to measure real elapsed time, even if the system clock is adjusted while your code runs.

## Measuring performance, timeouts, and delays

Whenever your goal is to measure how long something takes — for example, a function call, a loop, or a timeout — `steady_clock` should be your default choice.

That's also why you'll see it used in many standard library facilities.For example, `std::jthread::wait_for` use `steady_clock` for timing out, because it must behave predictably even if the system time changes.

The only thing `steady_clock` cannot tell you is what time it is. There's no correlation between its time points and wall-clock time — only differences between them matter. If you want to print a human-readable timestamp or log an event, use `system_clock`. If you want to measure how long something took or schedule something to happen after a certain delay, use `steady_clock`.

## Testing angle — deterministic simulations and virtual time

The monotonic nature of `steady_clock` also makes it a great fit for testing, especially when you want deterministic control over time.

Imagine you're testing a retry mechanism or a timeout. You don't really want your test to sleep for 5 seconds — that would inacceptably slow for your suite. Instead, you can abstract time behind an interface and simulate or mock a clock.

Here is a simple sketch:

```cpp
struct Clock {
    using time_point = std::chrono::steady_clock::time_point;
    virtual time_point now() const = 0;
    virtual ~Clock() = default;
};

struct RealClock : Clock {
    time_point now() const override {
        return std::chrono::steady_clock::now();
    }
};

struct FakeClock : Clock {
    time_point current{};
    time_point now() const override { return current; }
    void advance(std::chrono::milliseconds delta) { current += delta; }
};
```

In your production code, - unsurprisingly - you use `RealClock`. In tests, you use `FakeClock` and manually advance the "time" which means that no real waiting involved to deterministicly forwarding the time.

```cpp
FakeClock clock;
auto start = clock.now();
clock.advance(1s);
auto end = clock.now();
assert(end - start == 1s);
```

This pattern is particularly helpful in **simulations**, **retry logic**, or **event scheduling** where reproducibility matters.

You can *"fast-forward"* time without making your tests actually take time.

## Conclusion

Today, we talked about `std::chrono::steady_clock` which a tool that it essential for reliable time measurement. It's main characteristic is its monotonic behaviour, meaning it never goes backwards. This makes it the perfect choice when you care about *how much time has passed*, not *what the current time is*. Such cases are benchmarking code, waiting on a timeout, or simulating delays.

Unlike `system_clock`, it doesn't try to align with the wall clock. There's no concept of "now" in the human sense — just a steady, ever-increasing counter of time. 

In tests, this predictability opens up great opportunities. By wrapping or mocking `steady_clock`, you can advance "time" however you like, making your tests both fast and deterministic. 

In short, `steady_clock` isn't about *when* something happens — it's about *how long* it takes.

Next week, we'll talk about the `high_resolution_clock`. Stay tuned.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!