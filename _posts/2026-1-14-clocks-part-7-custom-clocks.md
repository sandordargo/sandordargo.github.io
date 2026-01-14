---
layout: post
title: "Time in C++: Creating Your Own Clocks with &lt;chrono&gt;"
date: 2025-1-14
category: dev
tags: [cpp, chrono, clocks, time]
excerpt_separator: <!--more-->
---
In earlier articles of this series, [we walked through the foundations of `<chrono>`](https://www.sandordargo.com/blog/2025/11/19/clocks-part-1-intro-to-chrono), explored the essential clocks ([system_clock](https://www.sandordargo.com/blog/2025/11/26/clocks-part-2-system_clock), [steady_clock](https://www.sandordargo.com/blog/2025/12/03/clocks-part-3-steady_clock), [high_resolution_clock](https://www.sandordargo.com/blog/2025/12/10/clocks-part-4-high_resolution_clock)), and even looked at [the extra clocks introduced in C++20](https://www.sandordargo.com/blog/2025/12/10/clocks-part-4-high_resolution_clock) — not to mention why [converting between clocks](https://www.sandordargo.com/blog/2025/12/24/clocks-part-5-conversions) is trickier than it seems.

Today, we'll shift gears a bit: what if the clock you need... doesn't exist?

C++ lets you build your own clocks. And it's surprisingly approachable.

Custom clocks aren't just a fun experiment — they're genuinely useful in simulations, testing, deterministic workflows, and anywhere you're not fully satisfied with how time is provided by the operating system.

Let's dive in.

## Why Would You Want a Custom Clock?

Most applications happily rely on `system_clock` or `steady_clock`. But sometimes, real time is the wrong tool:

**Testing**

You don't want your unit tests sleeping for 200 ms or relying on the host system's clock. A fake clock lets you explicitly control time, freeze it, fast-forward it, or make it deterministic.

**Simulations and Virtual Time**

Game engines, physics simulations, backtesting systems, and distributed systems often operate on virtual time where you choose when time increases.

**Replaying Events**

Maybe you're analyzing logs or reproducing a bug. A replay clock can step through events using their original timestamps.

**External Time Sources**

Some systems pull time from hardware sensors, an external server, a GPS unit, or a custom synchronization source. Wrapping that in a custom clock makes the rest of your system oblivious to how time is obtained.

In short: **if your program's notion of time isn't the computer's notion of time, you probably want a custom clock.**

## What Makes a Clock a "Clock" in `<chrono>`?

To be a proper *Clock* according to the C++ standard, a type must satisfy a very small interface. You've already met these in earlier posts, but here they are explicitly:

A Clock must define:
- `rep` — the underlying representation type (e.g., `std::int64_t`)
- `period` — a `std::ratio` describing tick duration
- `duration` — an alias for `std::chrono::duration<rep, period>`
- `time_point` — an alias for `std::chrono::time_point<Clock>`
- `static time_point now() noexcept`; — returning the current time
- `is_steady` — `true` if the clock never goes backwards

That's it! A clock in C++ is just a thin struct with a few typedefs and one static function. You can always check if a type satisfies the clock requirements with the help of [the `std::chrono::is_clock` type trait](https://en.cppreference.com/w/cpp/chrono/is_clock.html).

## A Simple Example: A Manually-Advanced Clock

Let's build the simplest meaningful custom clock: a clock whose time only moves when we tell it to. A clock you might use for testing of simulations:

```cpp
// https://godbolt.org/z/Keq1xxv37
#include <chrono>
#include <cstdint>
#include <iostream>

class ManualClock {
   public:
    using rep = std::int64_t;
    using period = std::nano;
    using duration = std::chrono::duration<rep, period>;
    using time_point = std::chrono::time_point<ManualClock>;

    static time_point now() noexcept { return time_point(current_time); }

    static constexpr bool is_steady = true;

    static void advance(duration d) noexcept { current_time += d; }

   private:
    static inline time_point current_time{duration{0}};
};

static_assert(std::chrono::is_clock_v<ManualClock>);
```

Notice how one by one we satisfy the requirements listed above. 

And now let's use it.

```cpp
// https://godbolt.org/z/Keq1xxv37
// ...

int main() {
    using namespace std::chrono_literals;

    ManualClock::advance(500ms);
    auto t1 = ManualClock::now();

    ManualClock::advance(200ms);
    auto t2 = ManualClock::now();

    std::cout << (t2 - t1).count() << " ns\n";
    return 0;
}
```

This prints *200000000 ns* — because you decided how time flows.

Also notice that custom clocks integrate seamlessly with `<chrono>` types because they follow the same contract as the built-in ones.

You can [check the full example on Compiler Exlorer](https://godbolt.org/z/Keq1xxv37).

## Interoperability with Standard Clocks

Earlier in this series, [we talked about inter-clock conversions](https://www.sandordargo.com/blog/2025/12/24/clocks-part-5-conversions) and how the C++ standard deliberately avoids providing implicit conversions between clock epochs.

That same rule applies to your clocks.

**There is no automatic conversion between standard clocks and your custom clock.**

If you want meaningful conversions, you must define how your clock relates to another:

- Does it follow system_clock with some offset?
- Does it map to real time at all?
- Is its epoch aligned with Unix epoch or something arbitrary?

You might define a relationship like this:

```cpp
auto system_now = std::chrono::system_clock::now();
auto manual_now = manual_clock::now();

auto offset = system_now.time_since_epoch() 
            - manual_now.time_since_epoch();

// later...
auto estimated_system_time = manual_t + offset;
```

But as discussed in the earlier article on converting between clocks:
**this only works as long as both clocks behave as you expect**, and one of them (usually `system_clock`) doesn’t jump.

With your own clocks, you control the behavior — so you get to define the rules.

## A More Realistic Example: A Scaling Clock

Here's a playful clock that runs faster or slower than real time:

```cpp
// https://godbolt.org/z/639zjsbd8
#include <chrono>
#include <cstdint>

struct ScaledClock {
  using rep        = std::int64_t;
  using period     = std::nano;
  using duration   = std::chrono::duration<rep, period>;
  using time_point = std::chrono::time_point<ScaledClock>;

  static time_point now() noexcept {
    auto real = std::chrono::steady_clock::now().time_since_epoch();
    // Convert to this clock's duration type (nanoseconds with int64_t)
    auto real_in_duration = std::chrono::duration_cast<duration>(real);

    // Scale the count as double, then cast back to rep
    double scaled_count = static_cast<double>(real_in_duration.count()) * multiplier;

    return time_point{duration{static_cast<rep>(scaled_count)}};
  }

  static constexpr bool is_steady = false;

  static void set_speed(double mul) {
    multiplier = mul;
  }

private:
  static inline double multiplier{1.0};
};

static_assert(std::chrono::is_clock_v<ScaledClock>);

int main() {
    return 0;
}
```

A clock like this can simulate slow-motion or time dilation effects in games, animations, or test scenarios.

## Practical Advice for Designing Custom Clocks

**Keep epochs consistent**:
If your clock relates to real time at all, decide clearly how its epoch aligns with `system_clock`.

**Think about thread safety**:
Clocks are accessed globally — ensure your clock behaves well under concurrency. Something I usually ignore on the blog to keep examples fairly simple.

**Consider `is_steady` carefully**:
If your clock can jump, set `is_steady = false`.

**Don't reinvent the wheel**:
If you're only trying to mock time in tests, libraries like GoogleTest offer helpers — but custom clocks offer the ultimate flexibility.

## Conclusion

Standard clocks take you far, and in earlier articles we've seen how powerful they are for measuring durations, producing timestamps, and handling real-world time scales like UTC and TAI.

But sometimes real time isn't the right time. Custom clocks let you:
- simulate,
- test,
- control,
- and bend time to your will.

They're surprisingly simple to implement and plug neatly into the `<chrono>` ecosystem with almost no plumbing. In the next part of this series, we'll continue exploring how to build robust time abstractions on top of `<chrono>`, including adapters, wrappers, and patterns for safer time handling. Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!