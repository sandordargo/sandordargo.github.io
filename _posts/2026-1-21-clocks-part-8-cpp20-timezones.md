---
layout: post
title: "Time in C++: C++20 Brought Us Time Zones"
date: 2026-1-21
category: dev
tags: [cpp, chrono, clocks, time]
excerpt_separator: <!--more-->
---
In the earlier parts of this series, we explored the foundations of `<chrono>`: durations, time points, clock selection, inter-clock conversions, and even custom clocks. [Click here to access all articles from the series.](https://www.sandordargo.com/tags/clocks/)

But for many real-world applications, none of that solves the hardest time problem developers face:
> *"What does this timestamp mean to a human, in a specific place, at a specific moment?"*

Until C++20, the standard library simply couldn't answer that question as it provided no support for timezones.

C++20 changed that - and in doing so, it finally made `<chrono>` suitable for serious, user-facing time handling.

## Why Time Zones Are a Big Deal

If you've ever scheduled a meeting across regions, processed user activity logs, or worked on billing systems, you already know: time zones are where bugs go to hide. I remember having worked in a team that had to deal with timezone data. The weeks around winter/summer time switches were always particularly stressful.

The difficulty isn't just offsets. It's that:
- daylight saving rules change
- offsets depend on historical and political decisions
- "local time" is not a stable concept

Before C++20, your options were:
- ignore the problem and hope UTC is "good enough", or
- pull in a third-party library (most commonly [Howard Hinnant's date library](https://github.com/HowardHinnant/date) - the basis of `std::chrono`), or
- provide in-house support for timezones

The standard library offered nothing between `std::tm` and "good luck".

## What C++20 Introduced

C++20 standardized a full time-zone model, building on years of real-world experience.

At the core of this model is system time, represented by `sys_time`, which is tied to `system_clock` and Unix time.

In practice, this means time-zone support in `<chrono>` revolves around conversions between:
- system time (`sys_time`), and
- local civil time (`local_time`)

This matches how operating systems and time-zone databases actually work.

## System Time as the Anchor

A typical entry point looks like this:

```cpp
auto now = std::chrono::system_clock::now();
```

`system_clock` counts seconds since the Unix epoch. Unix time closely tracks UTC, which is why developers often informally describe these conversions as "UTC <-> local time". Even [C++ Reference uses utc in variable names](https://en.cppreference.com/w/cpp/chrono/time_zone/to_local.html).

Strictly speaking, though, Unix time ignores leap seconds, and that distinction is reflected in the standard library’s design.

For most applications, this is exactly the right trade-off: practical, interoperable, and predictable.

> **Unix Time vs UTC**
> 
> Unix time is defined as the number of seconds since **1970-01-01 00:00:00**, *ignoring leap seconds*.
> 
> UTC, on the other hand, occasionally inserts leap seconds to keep civil time aligned with Earth's rotation.
> 
> The difference between Unix time and UTC is the accumulated number of leap seconds.
> 
> Most operating systems expose itme as Unix time and it's close enough to UTC that the difference and distinction doesn't matter.
> 
> If your domain does care about leap seconds (astronomy, time synchronization, some scientific systems), [C++20 provides utc_clock explicitly](https://www.sandordargo.com/blog/2026/01/07/clocks-part-6-cpp20#stdchronoutc_clock-the-leap-second-aware-clock) — but time-zone conversions intentionally operate on system time instead.

## Converting to Local Time

Once you have system time, converting it to a local representation is straightforward:

```cpp
// https://godbolt.org/z/rj3jvPxGh
#include <chrono>
#include <iostream>

int main() {
    const auto& timezone_db = std::chrono::get_tzdb();
    const auto* riviera_tz = timezone_db.locate_zone("Europe/Paris");

    const std::chrono::sys_time now = std::chrono::system_clock::now();
    const std::chrono::local_time local_now = riviera_tz->to_local(now);
    const std::chrono::sys_time reconverted_sys_now =
        riviera_tz->to_sys(local_now);

    std::cout << "sys_time now: " << now << '\n';
    std::cout << "local now: " << local_now << '\n';
    std::cout << "reconverted sys_time now: " << reconverted_sys_now << '\n';
    return 0;
}

/*
sys_time now: 2025-12-26 06:04:48.812719821
local now: 2025-12-26 07:04:48.812719821
reconverted sys_time now: 2025-12-26 06:04:48.812719821
*/
```

First, we take a `const&` of the timezone database using [`get_tzdb()`](https://en.cppreference.com/w/cpp/chrono/tzdb_functions.html). Then we locate the timezone representation based on its name with [`locate_zone`](https://en.cppreference.com/w/cpp/chrono/locate_zone.html). Note that `locate_zone` throws a `runtime_error` if we pass in a string that doesn't represent any valid timezone. Then we simply have to pass a `sys_time` to `time_zone::to_local` function. With `to_sys` it also works the other way around.

This conversion applies:

- the correct offset for the zone
- daylight saving rules
- historical and future transitions

All without manual calculations or platform-specific code.

## Zoned Time: Making Time and Place Explicit

Passing around raw time points can be error-prone, especially across API boundaries.

C++20 addresses this with `zoned_time`, which combines a time point with the time zone used to interpret it:

```cpp
std::chrono::zoned_time meeting{"Europe/Paris", std::chrono::system_clock::now()};
```

Just like the `locate_zone` function, `zoned_time` constructor will throw if we pass in a location that is not in the time zone database. 

`zoned_time` makes intent explicit and helps prevent accidental misuse — particularly in larger codebases.

## Working with Dates and Civil Time

A few weeks ago, we discussed about a pseudo-clock introduced also by C++20, `local_t`. Its usage indicates that a `time_point` represents **local civil time**, but without a specified time zone.

```cpp
std::chrono::local_time meeting = std::chrono::local_days{2025y/3/30} + 9h;
```

It becomes meaningul when it's paired with a time zone. `zoned_time` constructor takes it along with a time zone name or pointer, also with a time zone object, you can convert it into a `sys_time`.

```cpp
// https://godbolt.org/z/KnW5zoTrs
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono_literals;

    std::chrono::local_time meeting =
        std::chrono::local_days{2025y / 3 / 30} + 9h;

    const auto& timezone_db = std::chrono::get_tzdb();
    const auto* riviera_tz = timezone_db.locate_zone("Europe/Paris");

    const std::chrono::zoned_time zoned_meeting{riviera_tz, meeting};

    std::cout << "local time: " << meeting << '\n';
    std::cout << "zoned time: " << zoned_meeting << '\n';
    std::cout << "system time: " << riviera_tz->to_sys(meeting) << '\n';

    return 0;
}

/*
local time: 2025-03-30 09:00:00
zoned time: 2025-03-30 09:00:00 CEST
system time: 2025-03-30 07:00:00
*/
```

When this local time is associated with a time zone, the library can correctly handle:

- ambiguous times during DST fall-back
- skipped times during spring-forward

These are edge cases that are notoriously difficult to handle correctly without a proper model.

## Practical Implications

With time zones available, some best practices become much clearer:

- store and transmit system time
- convert to local time at the edges
- treat "local time" as a presentation concern, not a storage format

This aligns well with the guidance we discussed earlier when choosing clocks and designing time-aware APIs.

## How This Fits with Custom Clocks

[In the article on custom and user-defined clocks](https://www.sandordargo.com/blog/2026/01/14/clocks-part-7-custom-clocks), we talked about simulations and virtual timelines.

Time zones complement those designs nicely:
- custom clocks define how time progresses
- time zones define how humans interpret a moment

Keeping these responsibilities separate leads to cleaner abstractions and fewer surprises.

## Conclusion

Time zones were the missing piece of `<chrono>`.

C++20 doesn't pretend time is simple — but it gives us tools that reflect how systems and humans actually use it.

By standardizing time-zone support, C++ finally makes correct civil time handling possible without external libraries or fragile assumptions.

If your code interacts with humans, schedules events, or displays dates, time zones are no longer optional — and with C++20, they’re finally a first-class part of the standard library.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!