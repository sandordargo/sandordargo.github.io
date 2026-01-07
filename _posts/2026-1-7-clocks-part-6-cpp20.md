---
layout: post
title: "Time in C++: Additional clocks in C++20"
date: 2026-1-7
category: dev
tags: [cpp, chrono, clocks, time]
excerpt_separator: <!--more-->
---
In this series, we've already talked about [the main pillars behind `<chrono>`](https://www.sandordargo.com/blog/2025/11/19/clocks-part-1-intro-to-chrono), the most widely used clocks, and even [inter-clock conversions](https://www.sandordargo.com/blog/2025/12/24/clocks-part-5-conversions).

Those clocks — [`system_clock`](https://www.sandordargo.com/blog/2025/11/26/clocks-part-2-system_clock), [`steady_clock`](https://www.sandordargo.com/blog/2025/12/03/clocks-part-3-steady_clock), and [`high_resolution_clock`](https://www.sandordargo.com/blog/2025/12/10/clocks-part-4-high_resolution_clock) — all arrived in C++11, the first standard shipping the `<chrono>` library.

C++20 expanded the picture quite a bit by introducing **five additional clocks**.  

Let’s briefly explore them today.

## Why More Clocks?

If you've only used `system_clock` and `steady_clock` so far, the newer clocks may feel obscure.  
Do you really need a UTC clock? Or a TAI one? Isn't a timestamp just a timestamp?

Unfortunately... no.

The world runs on **multiple time scales**, and the mapping between them isn't fixed. Leap seconds, epoch differences, and OS-specific behavior often complicate what seems like a simple task: *"what time is it?"*

C++20 exposes these time scales explicitly so you can choose the right one — and avoid silently mixing incompatible concepts.

## `std::chrono::utc_clock`: The Leap-Second-Aware Clock

`utc_clock` is C++20's answer to *"give me real-world wall-clock time, including leap seconds."*

It explicitly models leap seconds, meaning conversions can fail or yield non-linear results if a timestamp lands inside a leap-second insertion.

C++20 also introduced `std::chrono::leap_second` and related utilities. You cannot construct a leap second manually — the implementation provides these objects based through the time zone database. They allow the program to reason about known leap-second insertions.

`utc_clock` helps with logging and tracing when timestamps must align with real-world UTC or when you have to interact with external systems using leap seconds.

## `std::chrono::tai_clock`: International Atomic Time (TAI)

TAI comes from the French *Temps Atomique International*. TAI is an abbreviation of the French name: *Temps Atomique International*. It has an earlier epoch than other clocks, measuring time since **00:00:00, 1 January 1958**.

TAI does **not** include leap seconds.  
This means:

- It runs uniformly.
- UTC gradually falls behind it.
- The UTC–TAI offset grows whenever a leap second is inserted into UTC.

At its birth, TAI was 10 seconds ahead of UTC. Today that offset is 37 seconds — and will keep increasing.

`tai_clock` meets the `Clock` requirements. It does not meet the `TrivialClock` requirements unless the implementation can guarantee that `now()` does not throw an exception.

Use it for scientific computing, simulation and satellite calculations or anytime leap-second-free arithmetic is needed. It's also useful as a reference scale for custom clocks if you want stable arithmetics and interoperability with civil time.

## `std::chrono::gps_clock`: Time for Satellites

As you probably guessed, `gps_clock` represents **Global Positioning System (GPS) time**.

Its epoch is 6th January 1980, and it has no leap seconds. Therefore, just like `tai_clock`, `gps_clock` drifts from `utc_clock` as new leap seconds are added. Since 2018, the difference between `utc_clock` and `gps_clock` is 18 seconds. The constant difference betweem `tai_clock` and `utc_clock` is 19 seconds (GPS is behind TAI),

Like `tai_clock`, `gps_clock` satisfies the `Clock` requirements, but it's short of `TrivialClock` with a non-throwing `now()` implementation.

It is indispensable for navigation, mapping, telemetry, and systems consuming GNSS timestamps.

## `std::chrono::file_clock`: A Practical Clock for Filesystem Timestamps

`file_clock` is the clock underlying `std::filesystem::file_time_type`.

Its epoch is implementation-defined — different operating systems store file times differently — but it **must** satisfy the `TrivialClock` requirements.

Before C++20, conversions between file timestamps and `system_clock` were also implementation-defined, which made round trips lossy or surprising.

C++20 fixed that:

- `file_clock` models exactly the timestamp format used by the filesystem.
- Conversions to `system_clock` or `utc_clock` must be **well-defined**.
- Round-tripping file times becomes predictable.

It’s not intended for scheduling or measurement — it's purely for **interoperability** with the filesystem.


## `std::chrono::locat_t`: Civil Time Without a Time Zone

C++20 also introduced a pseudo-clock named `local_t`, used to indicate that a `time_point` represents **local civil time**, but without a specified time zone.

```cpp
auto local = std::chrono::local_time<std::chrono::minutes>{ ... };
```

A `local_time` only becomes meaningful when paired with a time zone:

```cpp
auto tz = std::chrono::locate_zone("Europe/Berlin");
auto sys_time = tz->to_sys(local);
```

This design forces you to handle time-zone rules, DST transitions, and ambiguities explicitly.

## Conclusion

The additional clocks introduced in C++20 expand `<chrono>` with an expressive model of real-world time.

They let you choose the right time scale for your needs — whether you care about leap seconds, scientific precision, satellite signals, filesystem compatibility, or human-readable local time.

In future articles, we'll look at how these clocks interact with time zones, calendars, and custom clock designs — and how to test all of these reliably.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!