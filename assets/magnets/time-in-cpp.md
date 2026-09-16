# Time in C++

## A Practical Field Guide to `<chrono>`

*By Sandor Dargo*

---

## Foreword

Time is time. We tend to think it's simple to deal with — until clocks jump, time zones shift, or a perfectly good test starts failing on a CI machine that happens to be in a different country.

I started writing this material because I had to. A few months ago I picked up tasks around instrumentation and logging, and I quickly noticed that some parts of `<chrono>` felt cryptic, even though I'd been writing C++ for years. So I dug in, and what came out of that digging was a series of articles on my blog. This guide is the cleaned-up, restructured version of that series — the field guide I wish I'd had before I started.

It is aimed at C++ developers who have used `std::chrono` but have never really thought about what a *clock* is. If you've ever written `auto start = std::chrono::steady_clock::now()` and moved on without thinking, this guide is for you. If you've ever looked at `system_clock`, `steady_clock`, and `high_resolution_clock` and wondered which one to pick, this guide is for you. If you've ever tried to test code that calls `now()` and ended up with a flaky test, this guide is *especially* for you.

A few things about how it's organised:

- The first three chapters set up the mental model: durations, time points, and clocks. Skip them at your own peril — most chrono mistakes are foundation mistakes.
- The next chapters work through each standard clock in turn, then look at conversions, the C++20 additions, and custom clocks.
- The final chapters cover the two topics that bite people in production: time zones, and testing time-dependent code.
- A one-page cheat sheet at the end gives you the *which clock for which job* answer at a glance.

I've also kept a short appendix on the C++23 changes to `<chrono>`, mostly so you know what to expect when you upgrade your toolchain.

You don't need to read the chapters in order, but the cross-references assume you have. Code samples are kept short on purpose — most of them link back to a Compiler Explorer snippet on my blog if you want to play with the full thing. Let's begin.

---

## Table of Contents

1. The Three Pillars: Durations, Time Points, Clocks
2. `system_clock`: The Wall Clock
3. `steady_clock`: The Stopwatch
4. `high_resolution_clock`: Myths and Realities
5. Inter-Clock Conversions, Epochs, and Precision
6. The C++20 Clocks: UTC, TAI, GPS, File, Local
7. Building Your Own Clocks
8. Time Zones in C++20
9. Testing Time-Dependent Code
10. Closing Thoughts: A Map for Time in C++

Appendix A — One-Page Cheat Sheet
Appendix B — C++23 Changes to `<chrono>`

---

## Chapter 1 — The Three Pillars: Durations, Time Points, Clocks

Many people assume that all clocks just return "the current time" and that it doesn't matter whether they use `system_clock` or `steady_clock`. That assumption is the root of an entire family of subtle bugs.

Since C++11, the `<chrono>` library has made working with time simpler, but you can only use it to its full potential if you understand what it is doing under the hood. `<chrono>` was introduced to fix two pain points that plagued time handling in pre-C++11 code:

- **Lack of type safety.** Developers mixed seconds, milliseconds, and microseconds using plain integers, and the bugs that produced were invisible until production.
- **Lack of composability.** It was hard to build higher-level abstractions on top of raw time values.

The library is built around three pillars:

- **Durations** (`std::chrono::duration`) — *how long*
- **Time points** (`std::chrono::time_point`) — *when*
- **Clocks** (`std::chrono::clock`) — *the source of time*

C++20 later added calendar dates and time zones; we'll get to those in chapter 8. For now, let's look at the three pillars one at a time.

### `std::chrono::duration`

Durations represent time intervals. They are defined as:

```cpp
template<
    class Rep,
    class Period = std::ratio<1>
> class duration;
```

`Rep` is a numeric type representing the tick count, and `Period` describes how long one tick is, in seconds, expressed as a compile-time rational fraction (`std::ratio`).

A handful of common ratios show up everywhere:

- `std::ratio<1, 1'000'000'000>` is `std::nano`
- `std::ratio<1, 1'000'000>` is `std::micro`
- `std::ratio<1, 1'000>` is `std::milli`

Combine those with `Rep` and you get the standard duration aliases:

- `std::chrono::nanoseconds` is `std::chrono::duration<long long, std::nano>`
- `std::chrono::milliseconds` is `std::chrono::duration<long long, std::milli>`
- `std::chrono::seconds` is `std::chrono::duration<long long>`

Periods don't have to be smaller than one. Coarser durations work the same way:

- `std::chrono::minutes` is `std::chrono::duration<int, std::ratio<60>>`
- `std::chrono::hours` is `std::chrono::duration<int, std::ratio<3600>>`

C++20 added aliases for `days`, `weeks`, `months`, and `years`.

The big payoff of all this type machinery is that the compiler will not let you compare values with mismatched units:

```cpp
#include <chrono>

int main() {
    using namespace std::chrono_literals;
    constexpr auto d1 = 500ms;   // std::chrono::milliseconds
    constexpr auto d2 = 2s;      // std::chrono::seconds
    static_assert(d1 < d2);
    return 0;
}
```

If `d1` and `d2` were plain integers, `500 < 2` would be `false` and you'd have an embarrassing bug. With durations, `2s` is correctly recognised as the longer interval.

### `std::chrono::time_point`

A `time_point` represents a point in time relative to a clock's epoch — the clock's zero.

```cpp
#include <chrono>
#include <iostream>

int main() {
    std::chrono::time_point now = std::chrono::system_clock::now();
    std::cout << now << '\n';
    // 2025-11-05 04:49:07.802540469
}
```

Subtracting two time points gives you a duration. Adding them, on the other hand, is meaningless — what would `2025-11-05 04:49:07 + 2025-11-07 21:15:51` even mean? The library refuses to let you do it.

```cpp
auto now  = std::chrono::system_clock::now();
auto now2 = std::chrono::system_clock::now();
auto d  = now2 - now;   // duration
auto d2 = now  - now2;  // duration, possibly negative
// d == 565ns, d2 == -565ns
```

You *can* shift a time point by adding or subtracting a duration:

```cpp
using namespace std::chrono_literals;
auto now     = std::chrono::system_clock::now();
auto earlier = now - 10min;
auto later   = now + 10min;
```

But to get a `time_point` in the first place, you need a clock.

### `std::chrono::clock`

Clocks are the source of time points. At their core, they need two things:

- a starting point — an **epoch**
- and a **tick rate**

A typical example is the Unix epoch (1 January 1970) with one tick per second.

For a type to qualify as a clock (`std::chrono::is_clock_v<T> == true`), it must:

- have a nested `duration` type
- have a nested `time_point` type (typically `std::chrono::time_point<Clock, duration>`)
- provide a static `now()` function returning a `time_point`
- have a constant `is_steady` boolean
- define `rep` and `period` types for the underlying representation and tick frequency

> A clock is **steady** if it is guaranteed to be **monotonic** — `now()` can never go backwards as time moves forward. Not every clock is steady: `system_clock`, for instance, can jump when NTP corrects the system time or a user changes the clock manually. C++ also requires that steady clocks tick at a constant, uniform rate.

C++11 introduced three standard clocks:

- `system_clock`
- `steady_clock`
- `high_resolution_clock`

C++20 added four more:

- `utc_clock`
- `tai_clock`
- `gps_clock`
- `file_clock`

Each clock has its own purpose and behaviour. The next three chapters cover the original C++11 trio in detail, and chapter 6 picks up the C++20 additions.

### A mental model to take with you

If you remember nothing else from this chapter, remember this:

- **Clocks define timelines.**
- **Time points locate events on those timelines.**
- **Durations measure distances between events.**

Almost every chrono mistake is a confusion between these three roles. Keep them separate and most of `<chrono>` clicks into place.

---

## Chapter 2 — `system_clock`: The Wall Clock

If you've ever logged a timestamp or printed the current time, there's a good chance you've used `std::chrono::system_clock` — even if you didn't realise it. It is the clock you reach for when you want a number that means something to a human.

### What `system_clock` represents

`std::chrono::system_clock` measures the system-wide **wall-clock** time — the same time you'd see on your watch or in your system tray. It reflects real-world calendar time: hours, minutes, seconds, dates.

Internally it still counts ticks, like every clock in `<chrono>`, but those ticks are anchored to the Unix epoch (1 January 1970, 00:00:00 UTC). Pre-C++20 the epoch was technically implementation-defined; C++20 nailed it down to Unix time. Every mainstream implementation has used Unix time all along.

```cpp
#include <chrono>
#include <iostream>

int main() {
    auto now = std::chrono::system_clock::now();
    std::cout << "now: " << now << '\n';
    // now: 2025-11-10 05:43:45.622822844
}
```

### Converting between `time_point` and `time_t`

Because `system_clock` represents real-world time, it's often useful to convert it to a `time_t` for display or for interop with older C APIs:

```cpp
auto now = std::chrono::system_clock::now();
std::time_t now_c = std::chrono::system_clock::to_time_t(now);
std::cout << "Current time: " << std::ctime(&now_c);
// Current time: Mon Nov 10 05:45:35 2025
```

And the other direction:

```cpp
std::time_t t = std::time(nullptr);
auto tp = std::chrono::system_clock::from_time_t(t);
```

This interoperability is one of the main reasons `system_clock` is so handy in everyday applications: it bridges the modern C++ `<chrono>` world and the old-school C time APIs.

### When to use `system_clock`

Reach for `system_clock` whenever you care about timestamps that make sense to humans:

- log lines (`[2025-11-10 08:34:56] User logged in`)
- file metadata (modification times, creation times)
- saving or displaying dates in a UI

Whenever your program needs to express *what time it is in the real world*, `system_clock` is the right tool.

### When **not** to use it

If your goal is to measure a duration — how long a function took, how much time elapsed between two events — `system_clock` is the wrong choice. Wall-clock time can jump, both forward and backward:

- the user changes the system clock manually
- the Network Time Protocol (NTP) adjusts the system time slightly to stay in sync

A jump like that can turn your elapsed-time calculation into a negative number, or into something far larger than reality.

For accurate timing, use `std::chrono::steady_clock`. We'll cover it in the next chapter.

### Recap

`system_clock` is the wall-clock time source. Use it for timestamps, for anything that has to align with human time, and for interop with C APIs. Don't use it to measure durations, because it can jump.

Next: the clock that *can't* jump.

---

## Chapter 3 — `steady_clock`: The Stopwatch

At first glance, the difference between `steady_clock` and `system_clock` isn't obvious. Both give you a `time_point`, both let you compute durations. But under the hood they behave very differently — and those differences are exactly what make `steady_clock` the right tool for measuring **intervals**.

### Monotonic behaviour

The defining property of `std::chrono::steady_clock` is that it is **monotonic**. Its time never goes backwards. If you query it twice, the second call is guaranteed to give you a time point that is greater than or equal to the first.

That makes it ideal for measuring **elapsed time** or **performance**, because no clock adjustment can ever surprise you.

`system_clock`, by contrast, represents wall-clock time. If the OS synchronises the clock — and it will — `system_clock` may jump forward or backward, and any duration computation suddenly becomes meaningless.

```cpp
auto start = std::chrono::steady_clock::now();
// ... do something ...
auto end = std::chrono::steady_clock::now();

std::chrono::duration<double> elapsed = end - start;
std::cout << "Elapsed time: " << elapsed.count() << " seconds\n";
```

With `steady_clock`, this difference is real elapsed time, even if the system clock is adjusted while your code runs.

### Performance, timeouts, delays

Whenever you want to know how long something took — a function call, a loop iteration, a timeout — `steady_clock` should be your default.

You'll also see it used inside the standard library for the same reason. For example, `std::jthread::wait_for` uses `steady_clock` for timing out, because it has to behave predictably regardless of what the system clock is doing.

The one thing `steady_clock` cannot tell you is *what time it is*. There's no correlation between its time points and wall-clock time — only differences between them matter. If you want to print a human-readable timestamp or log an event, use `system_clock`. If you want to measure how long something took or schedule something to happen after a delay, use `steady_clock`.

### Deterministic simulations and virtual time

Monotonicity also makes `steady_clock` a natural fit for any code that wants to *control* the flow of time rather than passively read it — simulations, replay systems, and game loops, for example. Because the clock is guaranteed to be monotonic and uniform, it composes well with virtual-time abstractions where you decide when time advances. We'll come back to that idea in chapter 7 (custom clocks) and chapter 9 (testing).

### Recap

`steady_clock` is monotonic. That single property is what makes it the right tool for benchmarking, timeouts, and anywhere predictable elapsed time matters. Unlike `system_clock`, it doesn't try to align with the wall clock — there's no concept of "now" in the human sense, just a steady, ever-increasing counter.

In short: `steady_clock` isn't about *when* something happens — it's about *how long* it takes.

---

## Chapter 4 — `high_resolution_clock`: Myths and Realities

If there's one clock in `<chrono>` that causes the most confusion, it's `std::chrono::high_resolution_clock`. The name is irresistible — who wouldn't want "the highest resolution"? But, as so often in C++, the details matter.

### What "high resolution" actually means

A clock's **resolution** (or precision) is the granularity of its tick period — the smallest representable step in time for that clock's `time_point`. In `<chrono>` it's exposed via `Clock::period`, a `std::ratio`.

Notice the word I didn't use: *accuracy*. A clock can represent nanoseconds and still be wildly inaccurate, because of hardware limitations or OS scheduling. A higher resolution doesn't mean better measurement. For timing, **stability and monotonicity matter much more than how fine-grained the tick is.**

You can inspect a clock's nominal resolution at compile time:

```cpp
// https://godbolt.org/z/WMsWzETd7
#include <chrono>
#include <iostream>

template <typename Clock>
void print_resolution(const char* name) {
    long double ns_per_tick = (1'000'000'000 * Clock::period::num) / Clock::period::den;
    std::cout << name << " tick: ~" << ns_per_tick << " ns\n";
}

int main() {
    print_resolution<std::chrono::system_clock>("system_clock");
    print_resolution<std::chrono::steady_clock>("steady_clock");
    print_resolution<std::chrono::high_resolution_clock>("high_resolution_clock");
}
/*
system_clock tick: ~1 ns
steady_clock tick: ~1 ns
high_resolution_clock tick: ~1 ns
*/
```

That gives you the theoretical granularity. The effective resolution depends on your platform and runtime conditions — so don't assume nanoseconds means nanosecond accuracy.

### The aliasing problem

The standard deliberately leaves `std::chrono::high_resolution_clock` open: it represents *"clocks with the shortest tick period"*. It even says that in practice it *"may be a synonym for `system_clock` or `steady_clock`"*.

You can find out which one *your* implementation aliases:

```cpp
// https://godbolt.org/z/MGM6WWjev
#include <chrono>
#include <iostream>
#include <type_traits>

int main() {
    using std::chrono::high_resolution_clock;
    std::cout << std::boolalpha
              << "high_resolution_clock is steady? "
              << high_resolution_clock::is_steady << "\n"
              << "high_resolution_clock == steady_clock? "
              << std::is_same_v<high_resolution_clock, std::chrono::steady_clock> << "\n"
              << "high_resolution_clock == system_clock? "
              << std::is_same_v<high_resolution_clock, std::chrono::system_clock> << "\n";
}

/*
Possible output:
high_resolution_clock is steady? false
high_resolution_clock == steady_clock? false
high_resolution_clock == system_clock? true
*/
```

If it aliases `system_clock` (as it does on my setup), it can jump. If it aliases `steady_clock`, it's monotonic — but at that point, you might as well use `steady_clock` directly for clarity and portability.

### Why it's not always the best for timing

When measuring durations — timeouts, benchmarks, anything — you generally care about two things:

- **Monotonicity** — time should never go backwards.
- **Stability** — intervals should be consistent and unaffected by clock corrections.

`steady_clock` guarantees both. `high_resolution_clock` makes no such guarantee — it might be steady on one system and wall-clock-based on another. That's enough reason to avoid it in portable timing code.

Stick with the proven pair:

- `std::chrono::steady_clock` for intervals and durations
- `std::chrono::system_clock` for human-readable timestamps

Use `high_resolution_clock` only if you've confirmed (through traits or testing) that it gives you a measurable benefit on your target platform.

### When `high_resolution_clock` might actually pay off

There are rare cases where it does live up to its name. On some platforms, it really is a finer-grained, still-steady timer. Older Windows implementations sometimes mapped it to `QueryPerformanceCounter`, and some Linux libcs use a raw hardware timer (`CLOCK_MONOTONIC_RAW`) under the hood, which can give slightly higher resolution or lower jitter than `steady_clock`.

If you check and find that:

```cpp
std::chrono::high_resolution_clock::is_steady &&
!std::is_same_v<std::chrono::high_resolution_clock,
                std::chrono::steady_clock>
```

then it might offer a measurable benefit — especially for microbenchmarks. But the differences are platform-specific and usually negligible. Unless you've confirmed both steadiness and higher precision through testing, `steady_clock` remains the safer, more portable choice.

### Recap

`high_resolution_clock` sounds impressive, but in practice it's more of a naming illusion than a guarantee. On most systems it's just an alias to `system_clock` or `steady_clock`, offering no real advantage and sometimes even less reliability.

For everything except specialised microbenchmarks, use `steady_clock` for measuring intervals and `system_clock` for wall-clock timestamps, and let `high_resolution_clock` retire quietly into the corner.

---

## Chapter 5 — Inter-Clock Conversions, Epochs, and Precision

Now that we've met the three original clocks, we have to talk about something subtle: how clocks relate to each other, how their epochs differ, and what happens when you convert durations between units.

It sounds simple. A timestamp is a timestamp; a duration is just a number of seconds. But `<chrono>` is type-safe, and it enforces some rules that prevent accidental misuse. Once you understand *why* those rules exist, time handling in C++ becomes much safer — and your tests get more reliable too.

### Clock epochs: zero is not universal

A `time_point` in `<chrono>` is always measured relative to some epoch. The catch is that **each clock defines its own epoch**.

- `std::chrono::system_clock` uses the Unix epoch (1 January 1970 UTC).
- `std::chrono::steady_clock` uses an unspecified monotonic epoch — it might be system boot time, or something entirely different.
- `std::chrono::high_resolution_clock` is usually just an alias to one of the above.

This means:

> **You cannot meaningfully compare time points from different clocks.**

If you try to subtract a `steady_clock::time_point` from a `system_clock::time_point`, you're effectively asking: *"What is the difference between 1970-01-01 and some arbitrary boot-time counter?"* The answer depends on the machine, the OS, and possibly the phase of the moon.

Tests can fall into the same trap. If a test assumes `steady_clock` starts at zero or that its epoch is stable across runs and platforms, it becomes brittle. When you need deterministic behaviour, use a controllable test clock or simply don't expose epochs at all.

### Converting between clocks: what you can and can't do

For years, the standard library offered no built-in mechanism to transform a `time_point` from one clock into another. C++20 changed that with `std::chrono::clock_cast` and custom `clock_time_conversion` specializations — but only for clocks where the conversion is **well-defined**.

A conversion is defined if:

1. the clocks have a known, stable mathematical relationship, and
2. a `clock_time_conversion<FromClock, ToClock>` specialization exists.

The standard library already provides such conversions among:

- `system_clock`
- `utc_clock`
- `tai_clock`
- `gps_clock`
- `file_clock`
- and any custom clocks for which you provide a `clock_time_conversion`.

These clocks share known epochs and offsets — for instance, TAI is currently 37 seconds ahead of UTC — so the library can safely compute conversions between them. (More about the C++20 clocks in chapter 6.)

```cpp
using namespace std::chrono;

auto utc_now = utc_clock::now();
auto tai_now = clock_cast<tai_clock>(utc_now);
```

This result is well-defined and stable, because the relationship between TAI and UTC is part of the standard, not dependent on your system's wall clock or boot time.

You can also define your own conversions by specialising `clock_time_conversion` for your custom clocks. That's particularly useful for:

- virtual or test clocks
- simulated clocks
- domain-specific clocks (frame counters, monotonic-but-shifted clocks, and so on)

As soon as the specialization exists, `clock_cast` becomes available.

### Manual correlation when no conversion exists

For clocks that don't have a fixed relationship — `system_clock` and `steady_clock`, for example — the standard cannot give you a correct conversion. Their epochs differ; one jumps, one doesn't. You're left with manual correlation.

The trick is to take both `now()` readings together, then convert *durations* (which are clock-agnostic) rather than time points (which aren't):

```cpp
auto system_now = std::chrono::system_clock::now();
auto steady_now = std::chrono::steady_clock::now();

// Later, given some_steady_tp from the same steady_clock:
auto delta = some_steady_tp - steady_now;          // duration on the steady clock
auto estimated_system_time = system_now + delta;   // approximate system_clock::time_point
```

You can't subtract a `steady_clock::time_point` from a `system_clock::time_point` directly — the language won't let you, for the same epoch reasons we just discussed. But subtracting two `steady_clock::time_point`s gives a duration, and adding that duration to a `system_clock::time_point` is well-defined.

This is useful when, say, you measure an event duration with `steady_clock` (for accuracy) but still want to log it with a human-readable timestamp.

Big caveat: if `system_clock` jumps after you took the two `now()` readings — NTP sync, manual change — the relationship is now wrong. If you rely on this for long-running processes, you're betting the wall clock won't move. That's a dangerous bet. Re-correlate periodically, or accept that the result is only approximate.

This is exactly the kind of logic that benefits from dependency injection: hand the code two controllable clocks and make the conversion behaviour explicit. Tests shouldn't rely on the real-world relationship between clocks; they should verify your conversion math.

### Duration casting and precision

Durations look simple — a number plus a unit. But conversions between units introduce subtle precision issues.

Casting to a coarser unit truncates:

```cpp
using namespace std::chrono_literals;
auto ns = 1500ns;
auto us = std::chrono::duration_cast<std::chrono::microseconds>(ns);
// us == 1µs — the remaining 500ns are lost
```

Casting to a finer unit introduces *imaginary* precision:

```cpp
std::chrono::milliseconds ms{1};
auto ns2 = std::chrono::duration_cast<std::chrono::nanoseconds>(ms);
// ns2 == 1'000'000ns — but you didn't actually measure at nanosecond precision
```

C++20 gave us `chrono::floor`, `ceil`, and `round`, which make intent clear and let the reader see where you're losing information.

```cpp
// https://godbolt.org/z/K7GPdKMn9
auto original = 1499ns;
auto rounded_us = std::chrono::round<std::chrono::microseconds>(original);
// rounded_us == 1µs
// We've decided that 1499ns ≈ 1µs. The remaining 499ns are gone by design.
```

The point is to be explicit about lossy casts. *We choose* to lose 499ns, and that's fine — as long as it's intentional.

### Overflow, underflow, and representation limits

There is one more subtle danger: subtraction or conversion involving very large durations.

If you accidentally subtract two `system_clock` time points taken decades apart, or you add a huge duration that exceeds 64-bit limits, you can run into signed integer overflow — which, for a signed `Rep`, is undefined behaviour. Modern platforms give you plenty of headroom, but it isn't infinite.

It's still worth using a signed integer for `Rep`. Negative durations can happen legitimately, and they often surface logic errors that an unsigned representation would silently hide.

### A short list of best practices

- Measure intervals using one clock — preferably `steady_clock`.
- Convert to human-readable representations only at the boundary.
- Never assume epochs are related. Even if two timestamps "look close", you can't rely on any stable relationship between clocks.
- Keep rounding and precision choices explicit. Use `floor`, `ceil`, or `round`.
- Don't validate clocks — validate your logic.

Get those right and `<chrono>` stops feeling mysterious.

---

## Chapter 6 — The C++20 Clocks: UTC, TAI, GPS, File, Local

If you've only used `system_clock` and `steady_clock` so far, the newer clocks might feel obscure. Do you really need a UTC clock? A TAI one? Isn't a timestamp just a timestamp?

Unfortunately, no. The world runs on **multiple time scales**, and the mapping between them isn't fixed. Leap seconds, epoch differences, and OS-specific behaviour complicate what looks like a simple question: *what time is it?* C++20 exposes these time scales explicitly so you can choose the right one and stop silently mixing incompatible concepts.

### `utc_clock`: leap-second-aware

`utc_clock` is C++20's answer to "give me real-world wall-clock time, *including* leap seconds."

It explicitly models leap seconds, which means conversions can fail or yield non-linear results when a timestamp lands inside a leap-second insertion. C++20 also introduced `std::chrono::leap_second` and related utilities. You can't construct a leap second yourself — the implementation provides them, based on the time-zone database — but you can reason about known leap-second insertions in your own code.

Use it for logging and tracing when timestamps must align with real-world UTC, or when you have to interact with external systems that include leap seconds.

### `tai_clock`: International Atomic Time

`tai_clock` represents International Atomic Time (TAI is an abbreviation of the French *Temps Atomique International*). It has an earlier epoch than the other clocks, measuring time since 00:00:00 on 1 January 1958.

TAI does **not** include leap seconds. The implications:

- It runs uniformly.
- UTC gradually falls behind it.
- The UTC–TAI offset grows whenever a leap second is inserted into UTC.

At its birth, TAI was 10 seconds ahead of UTC. Today the offset is 37 seconds, and it will keep increasing.

`tai_clock` meets the `Clock` requirements. It does not necessarily meet the stricter `TrivialClock` requirements unless the implementation can guarantee that `now()` doesn't throw.

Reach for `tai_clock` for scientific computing, simulation, satellite calculations, or anywhere leap-second-free arithmetic is needed. It also works well as a reference scale for custom clocks, when you want stable arithmetic plus interoperability with civil time.

### `gps_clock`: time for satellites

As you'd expect, `gps_clock` represents Global Positioning System time. Its epoch is 6 January 1980, and like TAI it has no leap seconds. So `gps_clock` drifts from `utc_clock` as new leap seconds are added. Since the last leap second of December 2016 took effect, GPS time has been 18 seconds ahead of UTC. The constant difference between GPS time and TAI is 19 seconds (GPS is behind TAI).

Like `tai_clock`, `gps_clock` satisfies the `Clock` requirements but falls short of `TrivialClock` if its `now()` can throw. It's indispensable for navigation, mapping, telemetry, and systems consuming GNSS timestamps.

### `file_clock`: a clock for filesystem timestamps

`file_clock` is the clock underlying `std::filesystem::file_time_type`. Its epoch is implementation-defined — different operating systems store file times differently — but it must satisfy the `TrivialClock` requirements.

Before C++20, conversions between file timestamps and `system_clock` were also implementation-defined, which made round trips lossy or surprising. C++20 fixed that:

- `file_clock` models exactly the timestamp format used by the filesystem.
- Conversions to `system_clock` or `utc_clock` must be well-defined.
- Round-tripping file times is now predictable.

It is not intended for scheduling or measurement — it's purely for **interoperability** with the filesystem.

### `local_t`: civil time without a time zone

C++20 also introduced a pseudo-clock named `local_t`. It indicates that a `time_point` represents **local civil time**, but without a specified time zone:

```cpp
auto local = std::chrono::local_time<std::chrono::minutes>{ /* ... */ };
```

A `local_time` only becomes meaningful when paired with a time zone:

```cpp
auto tz = std::chrono::locate_zone("Europe/Berlin");
auto sys_time = tz->to_sys(local);
```

This design forces you to handle time-zone rules, DST transitions, and ambiguities explicitly — which is the topic of chapter 8.

### Recap

The C++20 clocks expand `<chrono>` into an expressive model of real-world time. They let you choose the right time scale for the job — leap seconds, scientific precision, satellite signals, filesystem compatibility, or human-readable local time.

Most application code will keep using `system_clock` and `steady_clock`. The new clocks are there for the cases where those two aren't enough.

---

## Chapter 7 — Building Your Own Clocks

Most applications happily rely on `system_clock` or `steady_clock`. But sometimes the clock you need doesn't exist — and C++ lets you build your own. It is surprisingly approachable.

Custom clocks aren't just a fun experiment. They are genuinely useful in simulations, testing, deterministic workflows, and anywhere you're not satisfied with how time is provided by the operating system.

### Why a custom clock?

A few classic reasons:

**Testing.** You don't want a unit test sleeping for 200 ms or relying on the host system's clock. A fake clock lets you freeze, fast-forward, or otherwise control time deterministically.

**Simulations and virtual time.** Game engines, physics simulators, backtesting systems, and distributed simulators often operate on virtual time, where you choose when time advances.

**Replaying events.** Maybe you're analysing logs or reproducing a bug. A replay clock can step through events using their original timestamps.

**External time sources.** Some systems pull time from hardware sensors, an external server, a GPS unit, or a custom synchronisation source. Wrapping that source in a custom clock makes the rest of your code oblivious to where time comes from.

In short: **if your program's notion of time isn't the computer's notion of time, you probably want a custom clock.**

### What makes a clock a "clock"

To be a proper *Clock* in the C++ sense, a type must satisfy a small interface. You've already seen these in chapter 1, but here they are explicitly:

- `rep` — the underlying representation type (e.g. `std::int64_t`)
- `period` — a `std::ratio` describing the tick duration
- `duration` — alias for `std::chrono::duration<rep, period>`
- `time_point` — alias for `std::chrono::time_point<Clock>`
- `static time_point now() noexcept;` — returning the current time
- `is_steady` — `true` if the clock never goes backwards

That's it. A clock in C++ is just a thin struct with a few typedefs and one static function. You can confirm it satisfies the requirements with `std::chrono::is_clock_v<MyClock>`.

### A manually advanced clock

Let's build the simplest meaningful custom clock — one whose time only moves when we tell it to. This is the kind of thing you'd use for testing or simulation:

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

Each typedef and static maps to a requirement above. Now use it:

```cpp
int main() {
    using namespace std::chrono_literals;

    ManualClock::advance(500ms);
    auto t1 = ManualClock::now();

    ManualClock::advance(200ms);
    auto t2 = ManualClock::now();

    std::cout << (t2 - t1).count() << " ns\n";  // 200000000 ns
}
```

Time only flows because we said so. Notice how custom clocks integrate seamlessly with `<chrono>` types: they follow the same contract as the built-in ones, so the rest of the library Just Works.

### Interoperability with standard clocks

In chapter 5 we saw that the standard deliberately avoids implicit conversions between clock epochs. The same rule applies to your clocks: **there is no automatic conversion between standard clocks and your custom clock.**

If you want a meaningful conversion, you have to define how your clock relates to another. Does it follow `system_clock` with some offset? Does it map to real time at all? Is its epoch aligned with the Unix epoch?

The pattern is the same as the one we used in chapter 5 to bridge `system_clock` and `steady_clock`: take both `now()` readings together and use durations — which are clock-agnostic — to bridge the gap.

```cpp
auto system_now = std::chrono::system_clock::now();
auto manual_now = ManualClock::now();

// Later, given some_manual_tp from the same ManualClock:
auto delta = some_manual_tp - manual_now;          // duration on ManualClock
auto estimated_system_time = system_now + delta;   // system_clock::time_point
```

The same caveat as in chapter 5 applies: this only works as long as both clocks behave as you expect, and one of them — usually `system_clock` — doesn't jump. With your own clocks, *you* control the behaviour, so you get to define the rules.

### A scaling clock

Here's a slightly playful one — a clock that runs faster or slower than real time:

```cpp
// https://godbolt.org/z/639zjsbd8
struct ScaledClock {
    using rep        = std::int64_t;
    using period     = std::nano;
    using duration   = std::chrono::duration<rep, period>;
    using time_point = std::chrono::time_point<ScaledClock>;

    static time_point now() noexcept {
        auto real = std::chrono::steady_clock::now().time_since_epoch();
        auto real_in_duration = std::chrono::duration_cast<duration>(real);
        double scaled_count = static_cast<double>(real_in_duration.count()) * multiplier;
        return time_point{duration{static_cast<rep>(scaled_count)}};
    }

    static constexpr bool is_steady = false;

    static void set_speed(double mul) { multiplier = mul; }

private:
    static inline double multiplier{1.0};
};

static_assert(std::chrono::is_clock_v<ScaledClock>);
```

Useful for slow-motion or time-dilation effects in games, animations, or stress-test scenarios.

### Practical advice for designing custom clocks

- **Keep epochs consistent.** If your clock relates to real time at all, decide explicitly how its epoch aligns with `system_clock`.
- **Think about thread safety.** Clocks are accessed globally, so make sure your `now()` behaves under concurrency. (I tend to skip this in blog examples for simplicity, but production code should not.)
- **Set `is_steady` honestly.** If your clock can jump, set `is_steady = false`.
- **Don't reinvent the wheel.** If you only need to mock time in tests, GoogleTest and friends offer helpers. Custom clocks shine when you need flexibility beyond what those helpers offer.

### Recap

Standard clocks take you far. But sometimes real time isn't the right time. Custom clocks let you simulate, test, control, and bend time to your will — and they plug into the `<chrono>` ecosystem with almost no plumbing. If a chapter convinces you that `<chrono>` is more flexible than you thought, this is the one.

---

## Chapter 8 — Time Zones in C++20

The previous chapters covered the foundations: durations, time points, clock selection, conversions, custom clocks. But for many real-world applications, none of that solves the hardest time problem developers face:

> *"What does this timestamp mean to a human, in a specific place, at a specific moment?"*

Until C++20, the standard library couldn't answer that question — there was no time-zone support at all. C++20 fixed that, and in doing so it finally made `<chrono>` suitable for serious, user-facing time handling.

### Why time zones are a big deal

If you've ever scheduled a meeting across regions, processed user-activity logs, or worked on a billing system, you already know: time zones are where bugs go to hide. I once worked in a team that had to deal with time-zone data, and the weeks around the winter/summer time switches were always particularly stressful.

The difficulty isn't just offsets. It's that:

- daylight saving rules change
- offsets depend on historical and political decisions
- "local time" is not a stable concept

Before C++20, your options were:

- ignore the problem and hope UTC is good enough,
- pull in a third-party library — most commonly [Howard Hinnant's date library](https://github.com/HowardHinnant/date), which became the basis of `std::chrono`'s C++20 additions, or
- roll your own, which is somewhere between *unwise* and *career-limiting*.

The standard library offered nothing between `std::tm` and "good luck."

### What C++20 introduced

C++20 standardised a full time-zone model, building on years of real-world experience. At its core is **system time**, represented by `sys_time`, which is tied to `system_clock` and Unix time.

In practice, time-zone support in `<chrono>` revolves around conversions between:

- system time (`sys_time`), and
- local civil time (`local_time`)

That mirrors how operating systems and time-zone databases actually work.

### System time as the anchor

A typical entry point:

```cpp
auto now = std::chrono::system_clock::now();
```

`system_clock` measures time since the Unix epoch. Unix time closely tracks UTC, which is why developers often informally describe these conversions as "UTC ↔ local time" — even cppreference uses `utc` in variable names. Strictly speaking, Unix time ignores leap seconds, and that distinction is reflected in the standard library's design.

For most applications, that's exactly the right trade-off: practical, interoperable, predictable.

> **Unix time vs UTC**
>
> Unix time is defined as the number of seconds since 1970-01-01 00:00:00, *ignoring leap seconds*.
>
> UTC, by contrast, occasionally inserts leap seconds to keep civil time aligned with Earth's rotation.
>
> The difference between Unix time and UTC is the accumulated number of leap seconds.
>
> Most operating systems expose time as Unix time, and it's close enough to UTC that the distinction usually doesn't matter. If your domain *does* care about leap seconds — astronomy, time synchronisation, some scientific systems — C++20 gives you `utc_clock` explicitly. But time-zone conversions intentionally operate on system time instead.

### Converting to local time

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
    const std::chrono::sys_time reconverted = riviera_tz->to_sys(local_now);

    std::cout << "sys_time now: " << now << '\n';
    std::cout << "local now: "    << local_now << '\n';
    std::cout << "reconverted: "  << reconverted << '\n';
}
/*
sys_time now: 2025-12-26 06:04:48.812719821
local now:    2025-12-26 07:04:48.812719821
reconverted:  2025-12-26 06:04:48.812719821
*/
```

You take a `const&` of the time-zone database via `get_tzdb()`, locate the zone by name with `locate_zone`, and then call `to_local` (or `to_sys`) on the resulting pointer. `locate_zone` throws a `runtime_error` if the name is invalid.

This conversion applies the correct offset, daylight saving rules, and historical and future transitions automatically — no manual calculations, no platform-specific code.

### Zoned time: making time and place explicit

Passing raw time points around is error-prone, especially across API boundaries. C++20 addresses this with `zoned_time`, which combines a time point with the time zone used to interpret it:

```cpp
std::chrono::zoned_time meeting{"Europe/Paris", std::chrono::system_clock::now()};
```

Just like `locate_zone`, the `zoned_time` constructor will throw if the time-zone name doesn't exist. `zoned_time` makes intent explicit and prevents accidental misuse — particularly in larger codebases.

### Working with civil time and dates

Recall `local_t` from chapter 6: it represents local civil time without a specified time zone.

```cpp
std::chrono::local_time meeting = std::chrono::local_days{2025y/3/30} + 9h;
```

It only becomes meaningful when paired with a time zone. The `zoned_time` constructor will accept a local time plus a zone name, pointer, or object, and the `time_zone` itself can convert it into a `sys_time`:

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

    std::cout << "local time: "  << meeting << '\n';
    std::cout << "zoned time: "  << zoned_meeting << '\n';
    std::cout << "system time: " << riviera_tz->to_sys(meeting) << '\n';
}
/*
local time:  2025-03-30 09:00:00
zoned time:  2025-03-30 09:00:00 CEST
system time: 2025-03-30 07:00:00
*/
```

Once a local time is associated with a zone, the library can correctly handle the painful edge cases:

- ambiguous times during DST fall-back
- skipped times during spring-forward

These are exactly the cases that any homegrown date library gets wrong.

### Practical implications

Once you have time zones, a few best practices fall out naturally:

- **Store and transmit system time.**
- **Convert to local time at the edges.**
- **Treat "local time" as a presentation concern, not a storage format.**

This aligns with the broader chapter-5 advice on conversions: keep your data in one timeline as long as possible, and convert at the boundary.

### How this fits with custom clocks

Custom clocks (chapter 7) and time zones complement each other:

- custom clocks define **how time progresses**
- time zones define **how humans interpret a moment**

Keeping these responsibilities separate leads to cleaner abstractions and far fewer surprises.

### Recap

Time zones were the missing piece of `<chrono>`. C++20 doesn't pretend time is simple — it gives you tools that reflect how systems and humans actually use it. If your code interacts with humans, schedules events, or displays dates, time zones are no longer optional, and they're finally a first-class part of the standard library.

---

## Chapter 9 — Testing Time-Dependent Code

Time is one of those dependencies that quietly makes code harder to test. As soon as you call `now()`, you give up control, and your tests suffer for it. This chapter walks through the problem and three ways to fix it.

### The problem

Suppose you have a function that takes a value, combines it with a timestamp into a record, and stores the record in a database. For simplicity, the "database" is a `std::vector` and the record holds a single value:

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

Now: how do you test that the database contains a record with the expected properties?

The `value` field is easy. Validating `created_at` is not. You can approximate it — assert that it's within some range — but that approach is brittle, inaccurate, and a recipe for flaky tests.

What you really want is **control**. You want to decide what "now" means during the test.

The fix in every case is the same idea: don't hardcode the source of time. Below are three ways to do it, in increasing order of "modern C++."

### Option 1: a hierarchy of clocks

The classic OO approach. Define an interface, a real implementation, and a fake one:

```cpp
struct Clock {
    virtual ~Clock() = default;
    virtual std::chrono::system_clock::time_point now() const = 0;
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

Then inject the clock:

```cpp
void store(int value, const Clock& clock) {
    db.emplace_back(value, clock.now());
}
```

In tests, pass a `FakeClock` and control time directly. The full example is on [Compiler Explorer](https://godbolt.org/z/qcMdvMKM5).

### Option 2: a concept

You might not love runtime polymorphism — fair enough. The standard library even has a trait for clock-like types, so let's use a concept instead. We can't use the trait directly as a concept, but we can build one:

```cpp
template <class C>
concept ClockLike = std::chrono::is_clock_v<C>;
```

Then declare `store` as:

```cpp
void store(int value, ClockLike auto clock);
```

This concept is both more and less permissive than the inheritance version. **More permissive,** because it would let you pass `steady_clock` into `store` — the call would compile, but the body wouldn't, because the compiler can't assign a `steady_clock::time_point` to a `system_clock::time_point`. **Less permissive,** because it requires all the formal characteristics of a standard clock, not just a `now()`.

Tighten it:

```cpp
template <class C>
concept SystemClockLike = std::chrono::is_clock_v<C> && requires {
    { C::now() } -> std::same_as<std::chrono::system_clock::time_point>;
};
```

That does the trick. The `FakeClock` itself can shed the inheritance:

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

The test:

```cpp
// https://godbolt.org/z/h68M5rGdv
void testStore() {
    FakeClock clock;
    clock.provided = std::chrono::system_clock::now();
    store(42, clock);
    assert(db[0].value == 42);
    assert(db[0].created_at == clock.provided);
}
```

If you have C++20, this approach is elegant: zero overhead, expressive, and it scales nicely in template-heavy or header-only code.

### Option 3: a time-provider lambda

You may not have C++20, or you may not want to emulate concepts with SFINAE. There's a much simpler option.

If you think about it, all you really need is *something that returns a `std::chrono::system_clock::time_point`*. You could use a function pointer or a template; the most readable option is `std::function`:

```cpp
using TimeProvider = std::function<std::chrono::system_clock::time_point()>;
```

Now you can drop `FakeClock` entirely. The test passes a lambda:

```cpp
// https://godbolt.org/z/WYTxKMaYs
void testStore() {
    auto expected = std::chrono::system_clock::now();
    TimeProvider time_provider = [&] { return expected; };
    store(42, time_provider);
    assert(db[0].value == 42);
    assert(db[0].created_at == expected);
}
```

(Note: `std::function` does have its own trade-offs — heap allocation, type erasure, indirection. [I've written about those.](https://www.sandordargo.com/blog/2023/04/05/binary-size-and-templates))

### Which one to choose?

As usual: it depends.

- **Inheritance-based clocks** are explicit and easy to understand, especially in OO codebases. They work well when you already use runtime polymorphism and dependency injection elsewhere.
- **Concept-based solutions** are zero-overhead, expressive, and scale nicely in template-heavy or header-only code. They shine when you want compile-time guarantees.
- **Time-provider functions** are the simplest. No custom types, no inheritance, no concepts. If all you need is "give me the current time", this is often more than enough.

In practice, I tend to start with the time-provider approach. It's minimal, readable, and easy to refactor later if the design grows more complex.

### Recap

The key idea is the same across all three options: **don't hardcode the source of time.** Once you treat time as an injectable dependency, your tests become deterministic, expressive, and far less fragile. Whether you reach for classic polymorphism, modern concepts, or a simple callable is a style question, not a correctness one.

---

## Chapter 10 — Closing Thoughts: A Map for Time in C++

Time is one of those topics that never quite stops being tricky. Clocks jump, calendars change, and even something as simple as "now" depends on more assumptions than it first appears.

The goal of this guide wasn't to make `<chrono>` simple, but to make it *understandable*. Modern C++ doesn't try to hide the complexity of time — it exposes it in a structured way. Clocks, time points, durations, calendars, and time zones all exist to make different aspects of time explicit, and using them well starts with knowing which question you're asking.

### The three pillars, revisited

Almost every chrono mistake is a confusion between three roles:

- A **time point** answers *when did something happen?* It locates an event on a timeline. A `time_point` is incomplete without knowing which clock it belongs to — the same numeric value can mean entirely different moments depending on the clock behind it.
- A **duration** answers *how long did something take?* Durations are independent of clocks and epochs. They represent an amount of time, not a position on a timeline. They compose well, they make units explicit, they're cheap.
- A **clock** defines *how time flows*. A clock specifies an epoch, a tick rate, and the guarantees it provides. When you choose a clock, you're choosing a set of assumptions about how that time behaves.

Seen together, these form a simple but powerful model. Clocks define timelines; time points locate events on those timelines; durations measure distances between events. Keeping these roles separate — and being explicit about which one you're working with — is the foundation for writing correct, maintainable time-related code.

### The clock landscape

Most mistakes with time in C++ are really mistakes about *which clock is being used*. Memorising APIs doesn't help; understanding why each clock exists does.

- `steady_clock` is monotonic and never goes backwards. The default for elapsed time, timeouts, and deadlines.
- `system_clock` represents wall-clock time. It can jump, but it's the right tool for timestamps, logging, and anything that has to align with human time.
- `high_resolution_clock` is most often just an alias with no strong guarantees. In nearly every case, using `steady_clock` or `system_clock` directly makes intent clearer.
- `utc_clock` models UTC explicitly, leap seconds and all. Use it when leap-second correctness matters.
- `tai_clock` represents a continuous atomic time scale. Rare in application code, but underpins conversions between time scales.
- `gps_clock` represents GPS time. Mostly relevant in navigation, synchronisation, and some embedded or scientific domains.
- `file_clock` is for filesystem timestamps. Implementation-defined epoch, well-defined conversions to `system_clock`.

The cheat sheet at the end of this guide gives you the *use case → clock* mapping in one page.

### Conversions, briefly

Once you work with multiple clocks, conversions become unavoidable. Durations are the simple case: cheap, explicit, and safe — as long as you accept possible truncation. Time points are different. A time point only makes sense in the context of its clock, and not all clocks share a common timeline. Only a limited set of conversions is supported, typically where there's a well-defined relationship between clocks.

The verbosity of `<chrono>` conversions is intentional. When time crosses clock boundaries, the code should make it obvious.

### Custom clocks

Standard clocks observe the real world. That's fine in production but quickly becomes a problem in tests, simulations, or anywhere you need deterministic behaviour. Custom clocks let you control what "now" means and how time flows. More importantly, using a custom clock turns time into an *explicit dependency*. Instead of hiding calls to `system_clock::now()`, your design clearly communicates where and how time matters.

Custom clocks aren't exotic — they're often the simplest, cleanest solution when real time gets in the way.

### Time zones

C++20's time-zone support isn't about convenience. It's about *correctness*. The library separates concerns clearly: `sys_time` anchors a moment on the system clock's timeline, while `local_time` and `zoned_time` handle civil time. Conversions are explicit, and ambiguous situations — DST transitions especially — are surfaced rather than silently guessed.

In practice, time zones live at the edges of a system: parsing input, displaying output, communicating with users and external services. Keep them there, anchor everything else to a well-defined clock, and your design stays clean.

### One takeaway

If there's one thing worth keeping from this whole guide, it's this:

> **Be deliberate about time.**
>
> Choose your clocks carefully. Prefer durations for measurement. Keep civil time and time zones at the edges. Don't be afraid to introduce custom clocks when real time gets in the way.

You don't need to memorise every type in `<chrono>`. What matters is knowing which questions to ask and recognising when time deserves a second thought. With that mindset, the standard library gives you the tools you need — without pretending that time is simpler than it really is.

---

## Appendix A — One-Page Cheat Sheet

### Which clock for which job?

| Use case | Clock | Why |
|---|---|---|
| Measuring elapsed time | `steady_clock` | Monotonic. Never jumps. |
| Timeouts, deadlines, retries | `steady_clock` | Same — predictable elapsed time. |
| Performance benchmarks | `steady_clock` | Stability matters more than precision. |
| Log timestamps | `system_clock` | Aligns with wall-clock time. |
| File metadata (display) | `system_clock` | Or convert from `file_clock`. |
| User-facing dates and times | `system_clock` + time zones | Display via `zoned_time`. |
| Filesystem APIs | `file_clock` | Implementation-defined, but round-trips cleanly. |
| Leap-second-aware logic | `utc_clock` | Models leap seconds explicitly. |
| Astronomy, satellite, scientific | `tai_clock` | Continuous, no leap seconds. |
| GNSS, mapping, telemetry | `gps_clock` | GPS time scale. |
| Tests and simulations | Custom clock or injected provider | Deterministic by construction. |

### When **not** to use `high_resolution_clock`

Almost always. On most platforms it's an alias for `system_clock` or `steady_clock`. Pick the one whose semantics you actually want.

### Choosing between `system_clock` and `steady_clock`

| Question | Use |
|---|---|
| "What time is it right now?" | `system_clock` |
| "How long did this take?" | `steady_clock` |
| "When does this expire (in real-world time)?" | `system_clock` |
| "Wake me up in 500 ms." | `steady_clock` |
| "Has 30 seconds passed?" | `steady_clock` |
| "What did the user write into the log?" | `system_clock` |

### Duration casting

- Coarser → finer (e.g. `ms` → `ns`): exact, but the extra digits are *imaginary precision* — you didn't actually measure that finely.
- Finer → coarser with `duration_cast`: silently truncates.
- Use `chrono::floor`, `chrono::ceil`, `chrono::round` to make rounding intent explicit.

### Time-zone rules of thumb

- Store and transmit `sys_time`.
- Convert to `local_time` / `zoned_time` only at the edges.
- Locate zones by IANA name: `"Europe/Paris"`, `"America/New_York"`, etc.
- `locate_zone` throws on unknown names — wrap accordingly.

### Testing time

Three options, in order of simplicity:

1. **Time-provider lambda** — `std::function<system_clock::time_point()>`. Start here.
2. **Concept** — `SystemClockLike auto clock`. Zero overhead; great for templates.
3. **Inheritance** — `Clock` interface with `RealClock`/`FakeClock`. Familiar to OO codebases.

Whichever you pick, the rule is the same: **never call `now()` directly from production code that you intend to test.**

### The four common pitfalls

1. Comparing `time_point`s from different clocks. (You can't.)
2. Trusting `system_clock` for elapsed-time measurement. (It can jump.)
3. Reaching for `high_resolution_clock` because of the name. (It probably isn't.)
4. Hardcoding `now()` in a function you'll later have to test. (Don't.)

---

## Appendix B — C++23 Changes to `<chrono>`

A short note for readers upgrading their toolchain. C++23 brought three changes worth flagging.

### Locale handling in chrono formatters

[P2372R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2372r3.html) fixed a defect: `std::format`'s chrono specialisations were *automatically* localised, with no way to opt out via format specifiers — even though `std::format` itself is locale-independent by default. The fix flips the behaviour so it matches the rest of `std::format`:

```cpp
std::locale::global(std::locale("ru_RU"));
using sec = std::chrono::duration<double>;
auto a = std::format("{:%S}",  sec(4.2)); // "04.200" (not localised)
auto b = std::format("{:L%S}", sec(4.2)); // "04,200" (localised on demand)
```

What used to be automatically localised is no longer; localisation is now opt-in via `L`.

### Encoding rules for localised chrono formatting

[P2419R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2419r2.html) clarifies what happens when a UTF-8 literal interacts with a non-UTF-8 locale during chrono formatting. Without the fix, mixing them could produce *Mojibake* — garbled characters that result from an encoding mismatch. C++23 specifies that, for an implementation-defined set of locales, replacements that depend on the locale are converted to UTF-8.

### Relaxed requirements for `time_point<>::clock`

[P2212R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p2212r2.html) recognised that the original `Cpp17Clock` requirements were too strict to model some legitimate cases:

- `local_t` is a pseudo-clock and never satisfied the full requirements anyway.
- Sometimes you need a stateful clock with a non-static `now()`.
- Sometimes you want a `time_point` that represents *time of day* without a date.

The change: the standard no longer imposes `Cpp17Clock` requirements on the `Clock` template parameter of `time_point`. Where the requirements *do* still apply (notably in threading utilities), the standard now spells that out explicitly.

