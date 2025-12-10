---
layout: post
title: "Time in C++: std::chrono::high_resolution_clock — Myths and Realities"
date: 2025-12-10
category: dev
tags: [cpp, chrono, clocks, time]
excerpt_separator: <!--more-->
---
If there’s one clock in `<chrono>` that causes the most confusion, it's `std::chrono::high_resolution_clock`. The name sounds too tempting — who wouldn't want "the highest resolution"? But like many things in C++, the details matter.

In the earlier parts of this series, we looked at [`system_clock` as the wall-clock time source](https://www.sandordargo.com/blog/2025/11/26/clocks-part-2-system_clock), and at [`steady_clock` as the reliable choice for measuring intervals](https://www.sandordargo.com/blog/2025/12/03/clocks-part-3-steady_clock). This time, we'll tackle the so-called "high-resolution" clock, separate fact from myth, and see why it's not always the right choice — even when you think you need precision.

## What "high resolution" actually means

A clock's resolution (or precision) is the granularity of its tick period — i.e., the smallest representable step in time for that clock's `time_point`. In `<chrono>` it's exposed via `Clock::period`, a `std::ratio`.

Notice an important difference. I didn't mention accuracy, only precision. A clock might represent nanoseconds, but still be inaccurate due to hardware or OS scheduling. A higher resolution doesn't necessarily mean better measurement. For timing, stability and monotonicity matter much more than how fine-grained the tick is.

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
Output on compiler explorer:
system_clock tick: ~1 ns
steady_clock tick: ~1 ns
high_resolution_clock tick: ~1 ns 
*/
```

With the above piece of code, you can get the theoretical granularity. The effective resolution depends on your platform and runtime conditions — so don't assume nanoseconds mean nanosecond accuracy.

## The platform-dependent nature (aliasing)

The C++ standard deliberately leaves `std::chrono::high_resolution_clock` open: it represents "clocks with the shortest tick period."

It also says, that in practice, it "may be a synonym for `system_clock` or `steady_clock`".

To find out which one you're dealing with, you can use this piece of code:

```cpp
// https://godbolt.org/z/MGM6WWjev
#include <chrono>
#include <iostream>
#include <type_traits>

int main() {
    using std::chrono::high_resolution_clock;
    std::cout << std::boolalpha << "high_resolution_clock is steady? "
              << high_resolution_clock::is_steady << "\n"
              << "high_resolution_clock == steady_clock? "
              << std::is_same_v<high_resolution_clock,
                                std::chrono::steady_clock> << "\n"
              << "high_resolution_clock == system_clock? "
              << std::is_same_v<high_resolution_clock,
                                std::chrono::system_clock> << "\n";
}

/* 
Possible output:
high_resolution_clock is steady? false
high_resolution_clock == steady_clock? false
high_resolution_clock == system_clock? true
*/
```

If it aliases `system_clock` - as for me -, it may jump forward or backward when the system time changes (for example, due to daylight savings adjustments).

If it aliases `steady_clock`, then it's monotonic — but at that point, you might as well use `steady_clock` directly for clarity and portability.

## Why it’s not always the best for timing

When measuring durations (timeouts, benchmarks, etc.), you generally care about two things:

- **Monotonicity** – time should never go backwards.
- **Stability** – intervals should be consistent and unaffected by clock corrections.

`steady_clock` guarantees both with its steadiness. `high_resolution_clock`, however, makes no such guarantee — it might be steady on one system and wall-clock based on another.

That alone is enough reason to avoid it in portable timing code.
Stick with:
- `std::chrono::steady_clock` for intervals and durations,
- `std::chrono::system_clock` for human-readable timestamps.

Only use `high_resolution_clock` if you've confirmed (through traits or testing) that it's stable and gives you a measurable benefit on your target platform.

## When `high_resolution_clock` might actually be better

There are a few rare cases where `std::chrono::high_resolution_clock` does live up to its name.
On some platforms, it's not just an alias, but a truly finer-grained and still steady timer.

For example, older Windows implementations sometimes mapped it to `QueryPerformanceCounter`,
and some Linux libcs use a raw hardware timer (`CLOCK_MONOTONIC_RAW`) under the hood,
which can give slightly higher resolution or lower jitter than `steady_clock`.

If you check and find that:

```cpp
std::chrono::high_resolution_clock::is_steady &&
!std::is_same_v<std::chrono::high_resolution_clock,
                std::chrono::steady_clock>
```

then it might offer a measurable benefit — especially for microbenchmarks or other extremely fine-grained measurements.

Just keep in mind that these differences are platform-specific and often negligible in real code. Unless you've confirmed both steadiness and higher precision through testing, `steady_clock` remains the safer and more portable choice.

## Conclusion

`std::chrono::high_resolution_clock` sounds impressive, but in practice it's more of a naming illusion than a guarantee. On most systems, it's just an alias to either `system_clock` or `steady_clock`, offering no real advantage — and sometimes even less reliability.

That said, there are cases where it genuinely wraps a finer-grained hardware timer. If you've verified that it's steady and distinct from `steady_clock`, it can be handy for very short performance measurements or specialized use cases.

For everything else, stick to the proven pair: *use `steady_clock` for measuring intervals and `system_clock` for wall-clock timestamps.* 

Next time, we'll look into whether it's possible to convert time between different clocks. Stay tuned. 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!