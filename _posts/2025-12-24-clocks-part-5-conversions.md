---
layout: post
title: "Time in C++: Inter-clock Conversions, Epochs, and Durations"
date: 2025-12-24
category: dev
tags: [cpp, chrono, clocks, time]
excerpt_separator: <!--more-->
---
By now in this series, we've spent time looking at the major standard clocks and their behavior. We've talked about [wall-clock time](https://www.sandordargo.com/blog/2025/11/26/clocks-part-2-system_clock), [monotonic clocks](https://www.sandordargo.com/blog/2025/12/03/clocks-part-3-steady_clock), and the myths around "[high resolution](https://www.sandordargo.com/blog/2025/12/10/clocks-part-4-high_resolution_clock)". Today, we are going to talk about a subtle area: how clocks relate to each other, how epochs differ, and what happens when you - need to - convert durations.

It sounds simple, right? A timestamp is a timestamp, and a duration is just a number of seconds - or other time units. But `<chrono>` is designed to be type-safe, and it enforces some rules that prevent accidental misuse. Once you understand why those rules exist, time handling in C++ starts to feel much safer — and your tests get more reliable too. And 

It sounds simple, right? A timestamp is a timestamp, and a duration is just a number of seconds - or other time units. But when it comes to converting between clocks, the reality is much messier. Different clocks have different epochs, different guarantees, and sometimes completely different purposes. `<chrono>` doesn’t stop you from doing conversions just to be pedantic — it discourages them because many of those conversions simply don't make sense or can silently introduce subtle bugs.

Once you understand why these conversions are tricky, and why the library forces you to be explicit about them, the design choices in `<chrono>` click into place. And more importantly, you start avoiding whole classes of timing bugs — both in production and in tests.

Let's jump right into the details.


## Clock Epochs: Why "Zero" Isn’t Universal

A time_point in `<chrono>` is always measured relative to some epoch. But here's the catch: each clock defines its own epoch.

- `std::chrono::system_clock` uses the Unix epoch (1 January 1970 UTC).
- `std::chrono::steady_clock` uses an unspecified monotonic epoch. Its zero might be system boot time, or something entirely different.
- `std::chrono::high_resolution_clock` is usually just an alias to either `system_clock` or `steady_clock`.

This means:

> **You cannot meaningfully compare time_points from different clocks.**

If you try to subtract a `steady_clock::time_point` from a `system_clock::time_point`, you’re effectively asking: *"What is the difference between 1970-01-01 and some arbitrary boot-time counter?"*

The answer, of course, is: it depends on the machine, the OS, and maybe even the phase of the moon... :)

Even tests can fall into this trap. If a test assumes that `steady_clock` starts at zero or that its epoch is stable across runs or platforms, that test becomes brittle. When you need deterministic behavior, it's best to use controllable test clocks — or simply avoid exposing epochs at all.

## Converting Between Clocks: What You Can (and Can’t) Do

Converting between clocks is tricky because their epochs differ—sometimes radically.  
For years, the standard library offered **no built-in mechanism** to transform a `time_point` from one clock into another. That changed with C++20: we now have **`clock_cast`** and **custom `clock_time_conversion` specializations**, which allow *well-defined* conversions when clocks have a meaningful relationship.

At the same time, many clocks (such as `system_clock` and `steady_clock`) still **cannot be safely converted** because they measure fundamentally different notions of time. For those cases, you must fall back to a **manual correlation technique**, understanding that the result is only an approximation.

Let’s look at both.

### **Standard-Supported Conversions (`clock_cast` and `clock_time_conversion`)**

C++20 introduced `std::chrono::clock_cast`, which allows converting a `time_point` from one clock to another **when the conversion is defined**.  

A conversion is defined if:

1. The clocks have a known, stable mathematical relationship  
2. A `clock_time_conversion<FromClock, ToClock>` specialization exists  

The standard library already provides such conversions between the following pairs:

- `system_clock`
- `utc_clock`
- `tai_clock`
- `gps_clock`
- `file_clock`
- Custom clocks if `clock_time_conversion` is soecified

These clocks share known epochs and offsets (e.g., TAI is always 37 seconds ahead of UTC at the moment of writing), so the library can safely compute conversions.


*More on some C++20 clocks next week.*

**Example: converting `utc_clock` to `tai_clock`:**

```cpp
using namespace std::chrono;

auto utc_now = utc_clock::now();
auto tai_now = clock_cast<tai_clock>(utc_now);
```

Here the result is **well-defined and stable**, because the relationship between TAI and UTC is part of the standard, not dependent on your system’s wall clock or boot time.

You can also define your own conversions by providing a `clock_time_conversion` specialization for your custom clocks. This is particularly useful for:

- virtual/test clocks
- simulated clocks
- domain-specific clocks (e.g., frame counters, monotonic-but-shifted clocks)

As soon as the specialization exists, `clock_cast` becomes available.

### Manual Correlation (When No Meaningful Conversion Exists)

For clocks that don’t have a fixed mathematical relationship — like `system_clock` and `steady_clock` — the standard cannot give you a correct conversion. Their epochs differ and their behavior differs; one jumps, one doesn’t.

In these cases, you can only estimate a conversion using a manual correlation pattern:

```cpp
auto system_now = std::chrono::system_clock::now();
auto steady_now = std::chrono::steady_clock::now();

// The offset lets you map steady to system later on.
auto offset = system_now - steady_now;
```

From here, you can convert from steady time to **approximate** system time:

```cpp
auto estimated_system_time = some_steady_tp + offset;
```

This is useful, for example, when you measure an event duration with `steady_clock` (which is good for accuracy), but still want to log a human-readable timestamps.

However, there's a big caveat! If `system_clock` jumps (e.g. due to NTP sync or to manual clock change), your offset becomes invalid.

If you rely on this relationship for long-running processes, you're essentially betting that the wall clock won't move. That's a dangerous bet.

Testing-wise, this is exactly the kind of logic that benefits from dependency injection: give the code two controllable clocks and make the conversion behavior explicit. Tests shouldn't rely on the real-world relationship between clocks; they should verify your conversion math.

## Duration Casting and Precision

Durations seem simple — just a number plus a unit. But converting between units introduces subtle precision issues.

On the one hand, casting to a coarser unit truncates:

```cpp
using namespace std::chrono_literals;
auto ns = 1500ns;            // 1500 nanoseconds
auto us = std::chrono::duration_cast<std::chrono::microseconds>(ns);
// us == 1 microsecond (the remaining 500ns are lost)
```

On the other hand, casting to a finer unit introduces "imaginary precision":

```cpp
std::chrono::milliseconds ms{1};
auto ns2 = std::chrono::duration_cast<std::chrono::nanoseconds>(ms);
// ns2 == 1'000'000ns, but we didn't *measure* at nanosecond precision
```

C++20 gives us `chrono::floor`, `ceil`, and `round`, which make intent clear and help you see where you’re losing information.

We are explicit in this lossy cast: we **choose** to lose 499ns, and that’s fine as long as it’s intentional.

```cpp
// https://godbolt.org/z/K7GPdKMn9
using namespace std::chrono_literals;
auto original = 1499ns;

// Round to the *nearest* microsecond
auto rounded_us = std::chrono::round<std::chrono::microseconds>(original);
// rounded_us == 1us

// We've effectively decided that 1499ns ≈ 1µs
// The remaining 499ns are gone by design.
std::cout << original.count() << "ns\n";
std::cout << rounded_us.count() << "us\n";
/*
1499ns
1us
*/
```

## Overflow, Underflow, and Representation Limits

Another subtle danger lies in subtraction or conversion involving very large durations.

Imagine you accidentally subtract two `system_clock` `time_points` taken decades apart, or you add a huge duration that exceeds 64-bit limits. Normally, modern platforms give you plenty of room, but it's not infinite.

As there is almost always an integer type behind `std::chrono::duration<Rep, Period>`'s `Rep`, there is a risk of signed integer overflow and therefore undefined behaviour.

Yet, it's useful to use signed integer types as `Rep`, because negative durations can happen legitimetly or they can show a logic error that would be otherwise hard to spot.

## Conclusion

Inter-clock conversions turn out to be one of the trickiest parts of working with `<chrono>`. Clocks have different epochs, some jump while others don't, and durations behave differently depending on how you convert them. But with the right mental model — and a few best practices — you can write robust, portable, and testable time-handling code.

- Always measure intervals using one clock, preferably use `steady_clock`
- Convert to human-readable preresentations only at the boundary
- Never assume epochs are related. Even if two timestamps "look close", you can't rely on any stable relationship between clocks.
- Keep rounding and precision choices explicit. Use `floor`, `ceil` or `round` to do so.
- Don't validate clocks, validate your logic.

Get those right, and suddenly time in C++ becomes a lot less mysterious.

Next week, we'll talk about some additional clocks introduced by C++20. Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!