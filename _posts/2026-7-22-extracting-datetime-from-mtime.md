---
layout: post
title: "The Portable Way to Extract Date and Time from a File's mtime in C++"
date: 2026-7-22
category: dev
tags: [cpp, chrono, filesystem, portability]
excerpt_separator: <!--more-->
---
After I finished my [Time in C++ series](https://www.sandordargo.com/blog/2026/02/04/clocks-part-10-conclusion), a reader asked a seemingly simple question:

> *"What is the portable way to extract the year, month, day, hour, minute, and second from a file's modification time? I tried, and I couldn't get it to work reliably on macOS and Windows."*

Fair question. You call `std::filesystem::last_write_time()`, you get back a time point, and you want the calendar components. How hard can it be?

Harder than it should be. The answer touches three layers of portability problems — different epochs, different conversion APIs, and incomplete standard library implementations. Let's dig in.

<!--more-->

## The starting point

`std::filesystem::last_write_time()` returns a `std::filesystem::file_time_type`, which is a `time_point` based on `file_clock` — [a clock introduced in C++20](https://www.sandordargo.com/blog/2026/01/07/clocks-part-6-cpp20) specifically for filesystem timestamps. To extract year/month/day and hour/minute/second, you need to convert this to `system_clock` — a clock whose epoch is known (Unix epoch, January 1, 1970) — and then decompose it into calendar components.

Sounds like two steps. In practice, both steps have portability traps.

## Layer 1: The epoch is implementation-defined

The C++ standard says the epoch of `file_clock` is unspecified. Not even "implementation-defined" — the standard does not require implementations to document it. Each major implementation chose differently:

| Implementation | Epoch | Representation | Period |
|---|---|---|---|
| libc++ (Clang) | 1970-01-01 (Unix) | `__int128_t` | nanoseconds |
| MSVC | 1601-01-01 (Windows FILETIME) | `long long` | 100 nanoseconds |
| libstdc++ (GCC) | **2174-01-01** | `long long` | nanoseconds |

Yes, GCC's epoch is in the future. A signed 64-bit integer with nanosecond resolution covers roughly ±292 years. By centering that range at 2174 instead of 1970, GCC covers the 1882–2466 range — which happens to match what ext4 filesystems need ([GCC patch](https://gcc.gnu.org/pipermail/gcc-patches/2019-January/514334.html)). Current dates simply have negative `time_since_epoch()` values.

The practical consequence: you cannot take `time_since_epoch().count()` and do manual arithmetic. You'd be off by 204 years on GCC and 369 years on MSVC.

## Layer 2: The conversion API split

C++20 introduced `file_clock` and said it must provide conversion functions — but gave implementations a choice. `file_clock` must provide **either** `to_sys()`/`from_sys()` **or** `to_utc()`/`from_utc()`. Not both. Not a common interface.

The implementations made opposite choices:

| | GCC/libstdc++ | Clang/libc++ | MSVC |
|---|:---:|:---:|:---:|
| `file_clock::to_sys()` | yes | yes | **no** |
| `file_clock::to_utc()` | **no** | **no** | yes |

Code calling `file_clock::to_sys()` compiles on GCC and Clang but fails on MSVC. Code calling `file_clock::to_utc()` compiles on MSVC but fails everywhere else. [Microsoft's own documentation](https://learn.microsoft.com/en-us/cpp/standard-library/file-clock-class?view=msvc-170) explicitly warns about this.

## Layer 3: `clock_cast` was supposed to fix this

`std::chrono::clock_cast` is the standard's portable wrapper. It finds a conversion path through whichever intermediate clock is available — exactly what you'd want here.

For a long time, this was the missing piece. [P0355R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0355r7.html) — the paper that added calendars, time zones, and clock conversions to C++20 — was only partially implemented in libc++. While calendar types like `year_month_day` were available and clocks like `utc_clock` and `tai_clock` landed in recent versions, `clock_cast` was simply not there. That meant no single API worked across all three implementations:

- `to_sys()` worked on GCC + Clang/libc++, but not MSVC
- `clock_cast` worked on GCC + MSVC, but not Clang/libc++

The good news: all three major implementations now ship `clock_cast`, so a single line works everywhere:

```cpp
auto sys_time = std::chrono::clock_cast<std::chrono::system_clock>(ftime);
```

That said, the layers above still matter — they explain *why* this was hard and why older code and blog posts are full of `#ifdef` branches and manual conversions. If you're on a recent toolchain, you can skip straight to the clean solution below.

## But `std::format` just works?

The C++20 standard actually allows `file_time_type` to be formatted directly:

```cpp
// https://godbolt.org/z/bneE4xv5s

#include <chrono>
#include <filesystem>
#include <format>
#include <fstream>
#include <iostream>
#include <string>

bool create_file(const std::string& filename) {
    std::ofstream file{filename};
    return file.good();
}

int main() {
    std::string file_name{"some_file.txt"};

    if (!create_file(file_name)) {
        std::cerr << "Failed to create " << file_name << "\n";
        return 1;
    }

    std::error_code ec;
    auto ftime = std::filesystem::last_write_time(file_name, ec);
    if (ec) {
        std::cerr << "Failed to get mtime: " << ec.message() << "\n";
        return 1;
    }

    std::cout << std::format("{:%Y-%m-%d %H:%M:%S}", ftime) << "\n";

    return 0;
}
```

If all you need is a display string, this is elegant — one line, no manual conversion, no `#ifdef`. It works on all three major implementations today. One subtle trap: the `std::formatter` specialization for chrono time points lives in `<chrono>`, not `<filesystem>`. If you forget that include, the compiler rejects the call with a confusing template error — it looks like the formatter doesn't exist, when really it just hasn't been included.

But for the reader's question, there's a more fundamental limitation: `std::format` gives you a string, not integers. If you need the year, month, or day as values you can do arithmetic on, compare, or pass to another API, a formatted string doesn't help. You'd end up parsing your own output, which defeats the purpose.

So the question remains: how do you get from `file_time_type` to calendar components?

## Extracting the components

### Approach 1: `clock_cast` + calendar types (C++20, all platforms)

Now that `clock_cast` works on all three major implementations, the clean solution is straightforward:

```cpp
// https://godbolt.org/z/9vbKTYv1W

#include <chrono>
#include <filesystem>
#include <format>
#include <fstream>
#include <iostream>
#include <string>

bool create_file(const std::string& filename) {
    std::ofstream file{filename};
    return file.good();
}

void print_timestamp(std::chrono::sys_time<std::chrono::system_clock::duration> sys_time) {
    auto dp = std::chrono::floor<std::chrono::days>(sys_time);
    std::chrono::year_month_day ymd{dp};
    std::chrono::hh_mm_ss tod{std::chrono::floor<std::chrono::seconds>(sys_time - dp)};

    std::cout << std::format("{:04}-{:02}-{:02} {:02}:{:02}:{:02}\n",
                             static_cast<int>(ymd.year()),
                             static_cast<unsigned>(ymd.month()),
                             static_cast<unsigned>(ymd.day()),
                             tod.hours().count(),
                             tod.minutes().count(),
                             tod.seconds().count());
}

int main() {
    std::string file_name{"some_file.txt"};

    if (!create_file(file_name)) {
        std::cerr << "Failed to create " << file_name << "\n";
        return 1;
    }

    std::error_code ec;
    auto ftime = std::filesystem::last_write_time(file_name, ec);
    if (ec) {
        std::cerr << "Failed to get mtime: " << ec.message() << "\n";
        return 1;
    }

    auto sys_time = std::chrono::clock_cast<std::chrono::system_clock>(ftime);

    print_timestamp(sys_time);

    return 0;
}
```

`clock_cast` finds a conversion path through whichever intermediate clock is available — on MSVC it routes through `utc_clock`, on GCC and Clang it goes through `to_sys()`. You don't need to know or care. No `#ifdef`, no platform branching. The extraction step with `year_month_day` and `hh_mm_ss` is the same everywhere — these calendar types are well-supported across all three implementations.

### Approach 2: Manual clock correlation (C++20 — older compilers)

If you're on an older toolchain where `clock_cast` isn't available yet, there's a technique that works without it: sample both clocks at roughly the same moment and compute the offset.

```cpp
// https://godbolt.org/z/W7aEKrvfb

#include <chrono>
#include <filesystem>
#include <format>
#include <fstream>
#include <iostream>
#include <string>

bool create_file(const std::string& filename) {
    std::ofstream file{filename};
    return file.good();
}

void print_timestamp(std::chrono::sys_time<std::chrono::system_clock::duration> sys_time) {
    auto dp = std::chrono::floor<std::chrono::days>(sys_time);
    std::chrono::year_month_day ymd{dp};
    std::chrono::hh_mm_ss tod{std::chrono::floor<std::chrono::seconds>(sys_time - dp)};

    std::cout << std::format("{:04}-{:02}-{:02} {:02}:{:02}:{:02}\n",
                             static_cast<int>(ymd.year()),
                             static_cast<unsigned>(ymd.month()),
                             static_cast<unsigned>(ymd.day()),
                             tod.hours().count(),
                             tod.minutes().count(),
                             tod.seconds().count());
}

int main() {
    std::string file_name{"some_file.txt"};

    if (!create_file(file_name)) {
        std::cerr << "Failed to create " << file_name << "\n";
        return 1;
    }

    std::error_code ec;
    auto ftime = std::filesystem::last_write_time(file_name, ec);
    if (ec) {
        std::cerr << "Failed to get mtime: " << ec.message() << "\n";
        return 1;
    }

    auto delta = ftime - decltype(ftime)::clock::now();
    auto sys_approx = std::chrono::system_clock::now() +
        std::chrono::duration_cast<std::chrono::system_clock::duration>(delta);

    print_timestamp(sys_approx);

    return 0;
}
```

The idea: `ftime - file_clock::now()` gives a duration (how far the file's mtime is from "now" on the file clock). Adding that duration to `system_clock::now()` gives the same moment on the system clock. If this pattern looks familiar, it's the [manual correlation technique](https://www.sandordargo.com/blog/2025/12/24/clocks-part-5-conversions) from Part 5 of the series.

The two `now()` calls aren't atomic, so the result is technically approximate. But the error is sub-microsecond — irrelevant for file timestamps.

One gotcha: the `duration_cast` is necessary on libc++, where `file_clock::duration` uses `__int128_t` internally. Without the cast, the resulting duration type can't implicitly convert to `system_clock::duration` and the code won't compile.

## Conclusion

Extracting the year, month, and day from a file's modification time should be simple. For a long time it wasn't — the three major standard library implementations chose different epochs, different conversion APIs, and were at different stages of implementing C++20's clock infrastructure. Today, with `clock_cast` available everywhere, it finally is.

| Approach | What you get |
|---|---|
| `std::format` directly | display string |
| `clock_cast` + calendar types | integer components |
| Manual correlation + calendar types | integer components (approximate) |

For most use cases, use `clock_cast` (Approach 1) — it's exact, portable, and requires no platform branching. If you only need a display string, `std::format` with `file_time_type` directly is even simpler. The manual correlation approach is a fallback for older compilers that don't yet have `clock_cast`.

The deeper lesson: `file_clock` was designed to let each platform represent file timestamps in whatever way the underlying filesystem uses. That's the right design for accuracy. But it means the "easy" question — *give me the year from an mtime* — requires you to think about which clock you're on and how to get off it. If my [Time in C++ series](https://www.sandordargo.com/blog/2025/11/19/clocks-part-1-intro-to-chrono) had one theme, it's this: time is never as simple as it looks.

{% include connect-deeper.html %}
