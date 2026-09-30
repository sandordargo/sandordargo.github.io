---
layout: post
title: "C++26: the new debugging header"
date: 2026-9-30
category: dev
tags: [cpp, cpp26, debugging]
excerpt_separator: <!--more-->
---
If you've ever needed to hit a breakpoint programmatically in C++, you've probably written something like this:

```cpp
#if defined(_MSC_VER)
    __debugbreak();
#elif defined(__clang__)
    __builtin_debugtrap();
#elif defined(__GNUC__) && (defined(__i386__) || defined(__x86_64__))
    __asm__ volatile("int3");
#else
    raise(SIGTRAP);
#endif
```

Every major C++ project reinvents this wheel. Boost.Test has `debugger_break()`, Catch2 has `CATCH_BREAK_INTO_DEBUGGER`, Dear ImGui has `IM_DEBUG_BREAK()` and Unreal has `UE_DEBUG_BREAK`. The platform knowledge required is arcane but well-known to tooling implementers — it just wasn't standardized.

C++26 fixes this with [P2546R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2546r5.html) and its companion [P2810R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2810r4.html) thanks to Rene Ferdinand Rivera Morell. A new `<debugging>` header provides three functions.

<!--more-->

## The three functions

```cpp
#include <debugging>

namespace std {
    void breakpoint() noexcept;
    bool is_debugger_present() noexcept;
    void breakpoint_if_debugging() noexcept;
}
```

All three are `noexcept` and all three are freestanding. None are `constexpr` — debugging is a runtime concern.

### `std::breakpoint()`

An unconditional breakpoint. When a debugger is attached, execution halts and control is handed to the debugger. On x86, this typically compiles to a single `int3` instruction.

When no debugger is present, the behaviour is implementation-defined — on most platforms the process receives a signal (such as `SIGTRAP`) and will terminate unless the signal is caught.

### `std::is_debugger_present()`

Returns `true` if the program is currently being traced by a debugger. The check is performed immediately on each call — the result is not cached because a debugger can attach or detach at any time during execution.

How this works under the hood varies by platform. On Windows, for example, an implementation might call `::IsDebuggerPresent()`. On Linux, GCC's libstdc++ reads `/proc/self/status` and checks the `TracerPid` field. The standard doesn't mandate any particular mechanism — the semantics are implementation-defined, with the intended behaviour described in non-normative notes.

What makes this function interesting is that it's **replaceable**, in the same sense as `operator new` and `operator delete`. You can define your own version:

```cpp
bool std::is_debugger_present() noexcept {
    return my_hardware_debug_flag;
}
```

This was introduced by [P2810R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2810r4.html) to solve the freestanding problem: on bare-metal platforms, there may be no OS-level way to detect a debugger. Without replaceability, the function would either have to be excluded from freestanding (forcing `#ifdef` guards everywhere) or return a useless default. Replaceability lets you wire it to whatever your platform provides.

### `std::breakpoint_if_debugging()`

The convenience function most users will reach for. It's equivalent to:

```cpp
if (std::is_debugger_present())
    std::breakpoint();
```

When a debugger is attached, you hit a breakpoint. When it's not, nothing happens. This is the pattern that Catch2, Boost.Test, and every other framework implements manually.

## Beyond breakpoints

Having `is_debugger_present()` as a separate function enables more than just conditional breakpoints. You might use it to:

- Produce extra diagnostic output when debugging
- Disable timeouts that would fire during a stepping session
- Switch to a more debugger-friendly allocator
- Skip optimizations that make debugging harder

The separation of concerns is deliberate — `breakpoint()` and `is_debugger_present()` are orthogonal building blocks, and `breakpoint_if_debugging()` is the convenience composition.

## Conclusion

`<debugging>` is another one of those C++26 additions that makes you wonder why it took this long. The platform knowledge is well-understood, every major project already has its own version, and the API is three functions. Sometimes the most useful standard library additions are the simplest ones.

{% include connect-deeper.html %}
