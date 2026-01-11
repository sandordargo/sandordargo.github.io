---
layout: post
title: "Time in C++: Understanding &lt;chrono&gt; and the Concept of Clocks"
date: 2025-11-19
category: dev
tags: [cpp, chrono, clocks, time]
excerpt_separator: <!--more-->
---
Let's start a new series!  

A few months ago, I began working on tasks related to instrumentation and logging, and I had to deal with timestamps, different clock types, and more. Some parts felt a bit cryptic at first, so I decided to dig deeper into the topic.

As I've shared many times during the past eight years, the main goal of this blog is to document what I learn — hence this new series on clocks and time.

<!--more-->

In this first part, I'll set the stage for the whole series, which is aimed at C++ developers who've used `std::chrono` but never really thought deeply about what a *clock* is.  

Many people assume that all clocks just return "the current time" or that it doesn't matter whether they use `system_clock` or `steady_clock`. Such assumptions can lead to subtle bugs.

You’ve probably written something like this a hundred times:

```cpp
auto start = std::chrono::steady_clock::now();
```

Even if you have, time handling can still surprise you. Ignoring time zones helps, but daylight saving changes can still wreak havoc!

And let's not forget about testing or choosing the wrong clock type — both can cause headaches.

Since C++11, the `<chrono>` library has made working with time simpler, but we can only use its features to their full potential if we *understand them well*.

`<chrono>` was introduced to fix two major pain points:
- **Lack of type safety**: Developers mixed seconds, milliseconds, etc., using plain integers.
- **Lack of composability**: It was hard to build higher-level abstractions on top of raw time values.

The `<chrono>` library was built around 3 pillars:
- Durations (`std::chrono::duration`)
- Timepoints (`std::chrono::time_point`)
- Clocks (`std::chrono::clock`)

Later, C++20 introduced calendar dates and time zones, but we won't cover those here — this series focuses on clocks.

## `std::chrono::duration`

Durations — unsurprisingly — represent time intervals.
They are defined as:

```cpp
template<
    class Rep,
    class Period = std::ratio<1>
> class duration;
```

`Rep` is a numeric type representing the tick count, while `Period` represents how long one tick is, in seconds, expressed as a compile-time rational fraction (`std::ratio`).

Let's look into some other ratios to understand better their meaning:

- `std::ratio<1, 1'000'000'000>` means nano, you can actually replace it with `std::nano`
- `std::ratio<1, 1'000'000>` is the same as `std::micro`
- `std::ratio<1, 1'000>` is the same as `std::milli`

Now let's look at some common durations:

- `std::duration<long long, std::nano>` (or `std::duration<long long, std::ratio<1, 1'000'000'000>>`) represents durations in nanoseconds, it's aliased as `std::chrono::nanoseconds`
- `std::duration<long long, std::milli>` represents durations in milliseconds (`std::chrono::milliseconds`)
- `std::duration<long long>` represents durations in seconds (`std::chrono::seconds`)

As a fraction doesn't have to be smaller than one, we can represent durations in coarser granularitly.

- `std::chrono::duration<int, std::ratio<60>>` is `std::chrono::minutes`
- `std::chrono::duration<int, std::ratio<3600>>` is `std::chrono::hours`

C++20 even added aliases for `days`, `weeks`, `months` and `years`.

Also note that the type for `Rep` changed in the above examples depending on the number of bytes you need to represent durations.

A great thing about `std::chrono::duration` is that it prevents comparing values with different units by accident:

```cpp
#include <chrono>

int main() {
    using namespace std::chrono_literals;
    constexpr auto d1 = 500ms;   // std::chrono::milliseconds
    constexpr auto d2 = 2s;     // std::chrono::seconds
    static_assert(d1 < d2);
    return 0;
}
```

If `d1` and `d2` were plain integers, `500 < 2` would be `false` — and you'd have an embarrassing bug.

## `std::chrono::time_point`

A `time_point` represents a point in time relative to a clock's epoch (its zero time point).

```cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono_literals;
    std::chrono::time_point now = std::chrono::system_clock::now();
    std::cout << now << '\n';
}
/*
2025-11-05 04:49:07.802540469
*/
```

You can subtract two `time_points` to get a duration, but adding them doesn't make sense — what would `2025-11-05 04:49:07 + 2025-11-07 21:15:51` even mean?

```cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono_literals;
    std::chrono::time_point now = std::chrono::system_clock::now();
    std::chrono::time_point now2 = std::chrono::system_clock::now();
    auto d = now2 - now;
    auto d2 = now - now2;
    std::cout << d << ' ' << d2 << '\n';
    return 0;
}
/*
565ns -565ns
*/
```

You can also shift a `time_point` by adding or subtracting a duration:

```cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono_literals;
    std::chrono::time_point now = std::chrono::system_clock::now();
    std::chrono::time_point earlier = now - 10min;
    std::chrono::time_point later = now + 10min;
    std::cout << earlier << ' ' << now << ' ' << later << '\n';
    return 0;
}
/*
2025-11-05 04:49:53.388934987 2025-11-05 04:59:53.388934987 2025-11-05 05:09:53.388934987
*/
```

But to get a `time_point`, we first need a clock...

## `std::chrono::clock`

Last but not least, let's talk about the third main building block of `<chrono>`: *clocks*.

Clocks are the source of `time_point`s. At their core, they need two things:
- a starting point, a.k.a. an **epoch**
- and a **tick rate**

A typical example is the Unix epoch (1st of January 1970) with one tick per second.

For a type to qualify as a clock (`std::chrono::is_clock_v<T> == true`), it must:
- have a nested `duration` type
- have a nested `time_point` type (typically `std::chrono::time_point<Clock, duration>`)
- provide a static `now()` function that returns a `time_point`
- have a constant `is_steady` boolean
- define `rep` and `period` types for the underlying representation and tick frequency (just like `duration` does)

> A clock is *steady* if its guaranteed to be *monotonic*, in other words, `now()` can never go backwards as the time moves forward. Given that we’ve just switched back to winter time in many countries, it's easy to see that not every clock is steady! C++ also requires that steady clocks tick at a constant, uniform rate.

C++11 introduced three standard clocks:
- `system_clock`
- `steady_clock`
- `high_resolution_clock`

Then C++20 brought us 4 more:
- `utc_clock`
- `tai_clock`
- `gps_clock`
- `file_clock`

Each clock has its own purpose and behavior — we’ll explore them in the coming articles.

## Conclusion

`<chrono>` brought much-needed clarity, safety, and composability to time handling in C++. Understanding durations, time points, and clocks is the foundation for writing robust and testable time-related code.

In the next articles, we'll dive deeper into the standard clocks and how to choose the right one for your use case. Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!