---
layout: post
title: "Time in C++: std::chrono::system_clock"
date: 2025-11-26
category: dev
tags: [cpp, chrono, clocks, time]
excerpt_separator: <!--more-->
---
Last week, we started a series on clocks, by introducing the `<chrono>` library in general. We saw the three pillars of the library: `time_point`s, `duration`s and clocks from a birds-eye view. As a next step, let's talk about one of its most commonly used clocks: `std::chrono::system_clock`. 

If you've ever logged timestamps or printed the current time, there is a fair chance that you've already used it. But let's take a closer look at what this clock really represents and where its limitations lie.

## What `system_clock` representes

`std::chrono::system_clock` measures the system-wide **wall-clock** time, in other words, the same time you'd see on your watch or system tray. It reflects real-world calendar time — hours, minutes, seconds, and dates that we humans care about.

Internally, it still counts ticks (just like every clock in `<chrono>`), but those ticks are anchored to the Unix epoch (January 1, 1970, 00:00:00 UTC). Even that's not mandated by the standard.

When you ask it for the current time you get a time_point representing that instant:

```cpp
#include <chrono>
#include <iostream>

int main() {
    auto now = std::chrono::system_clock::now();
    std::cout << "now: " << now << '\n';
    return 0;
}
/*
now: 2025-11-10 05:43:45.622822844
*/
```

## Converting Between time_point and time_t

Because `system_clock` represents real-world time, it’s often useful or simply needed to convert it to a `time_t` for display or interop with older APIs:

```cpp
#include <chrono>
#include <iostream>

int main() {
    auto now = std::chrono::system_clock::now();
    std::time_t now_c = std::chrono::system_clock::to_time_t(now);
    std::cout << "Current time: " << std::ctime(&now_c);
    return 0;
}
/*
Current time: Mon Nov 10 05:45:35 2025
*/
```

And if you ever need to go the other way:

```cpp
#include <chrono>
#include <iostream>

int main() {
    std::time_t t = std::time(nullptr);
    auto time_point = std::chrono::system_clock::from_time_t(t);
    std::cout << "time_point: " << time_point << '\n';
    return 0;
}
/*
time_point: 2025-11-10 05:47:33.000000000
*/
```

This interoperability is one reason `system_clock` is so handy in everyday applications — it bridges the modern C++ `<chrono>` world and the old-school C time APIs.

## When to use `system_clock`

Use `system_clock` whenever you care about timestamps that make sense to humans:
- Logging messages (e.g. `[2025-11-10 08:34:56] User logged in`)
- File metadata (modification times, creation times)
- Saving or displaying dates in a user interface

Basically, when your program needs to express *"what time it is in the real world"*, `system_clock` is the right tool.

## When not to use it

If your goal is to measure durations — like how long a function call takes, or how much time elapsed between two events — then `system_clock` is not a good choice.

The reasong behind is that wall-clock time can jump backwards or forwards:
- The user changes the system clock manually.
- The Network Time Protocol (NTP) adjusts the system time slightly to stay in sync.

*As Ingo Van Lil pointed out at the comments section, DST changes do not affect `system_clock`*

In such cases, your measurements won't be precise, they might suddenly show negative durations or too big results. For accurate timing, you should use `std::chrono::steady_clock` instead — it never goes backward. More about that in the next part of this series.

## What about testing

Tests that depend on real-world time are often flaky. If your code calls `system_clock::now()` directly, it ties the behavior of your tests to the passage of real time — and that makes them unreliable and slow.

For example:

```cpp
EXPECT_TRUE(order.expires_at > std::chrono::system_clock::now());
```

This might pass on your machine but fail on CI if the system clock drifts, or if execution timing slightly changes.

A better approach is to wrap or abstract the clock:

```cpp
struct Clock {
    virtual std::chrono::system_clock::time_point now() const = 0;
    virtual ~Clock() = default;
};

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

Now your production code depends on `Clock`, not on `system_clock` directly, and in tests you can inject `FakeClock` and control time deterministically.

This kind of abstraction makes your tests stable, reproducible, and fast — no waiting, no flakiness, no surprises.

## Conclusion

Today, we looked into probably the most widely used standard clock, `std::chrono::system_clock`. It returns the *real-world* time. It's the best for timestamps and also to interact with old C APIs. On the other hand, it's not good for measurements because this clock can jump due to manual adjustments and for NTP syncs. For testing purposes, it's the best wrap it so that we can fake the *current* time.

Next week, we will explore `steady_clock` which is more sutable for measuring durations.


## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!