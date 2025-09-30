---
layout: post
title: "Pragma unroll"
date: 2025-X-X
category: dev
tags: [cpp, optimization, loops, pragmas]
excerpt_separator: <!--more-->
---
After conferences, I often write a couple of short articles about features or techniques I learned about, even if I don't always find them particularly useful in my own work.

At a recent talk, [Andrei Alexandrescu](https://erdani.org/) mentioned the `unroll` *pragma*. I knew what loop unrolling was, but I had never used it and had never seen the pragma before. So let’s dive into it.

## What is loop unrolling?

Loop unrolling is a performance optimization technique that reduces the overhead of loop control instructions and can improve opportunities for parallel execution.

Normally, when a loop runs, three things happen in addition to executing the loop body:
- the loop counter is incremented,
- the loop condition is checked,
- and the program jumps back to the start of the loop.

With loop unrolling, multiple iterations of the loop body are combined into one. In some cases, the loop can even be completely unrolled.

```cpp
std::array<int, 8> arr{1, 2, 3, 4, 5, 6, 7, 8};

// loop without unroll
for (int i = 0; i < 8; i++) {
    arr[i] *= 2;
}

// loop unrolled by factor of 2
for (int i = 0; i < 8; i+=2) {
    arr[i] *= 2;
    arr[i+1] *= 2;
}
// loop fully unrolled
arr[0] *= 2;
arr[1] *= 2;
arr[2] *= 2;
arr[3] *= 2;
arr[4] *= 2;
arr[5] *= 2;
arr[6] *= 2;
arr[7] *= 2;
arr[8] *= 2;
```

The benefits of loop unrolling include:
- less time spent on increments and condition checks,
- potentially better instruction pipelining on modern CPUs,
- and easier vectorization, since compilers can take advantage of SIMD instructions more effectively.

There are three main ways loop unrolling can happen:
- **Manual unrolling**: as shown above. In practice, you should almost never do this, unless you are writing extremely performance-critical code and know exactly what you’re doing.
- **Compiler-driven unrolling**: the compiler decides when and how much to unroll, often guided by optimization flags. As [Steve Downey said at CppCon2025](https://www.sandordargo.com/blog/2025/09/24/trip-report-cppcon-2025#my-favourite-ideas), most days the compiler is smarter than us.
- **Compiler-directed unrolling**: the developer can hint to the compiler—through pragmas—that unrolling is preferable in certain cases.

When you do loop unrolling, there are a couple of benefits:
- Less time is spent on increment and conditions checks
- arguably modern CPUs can take benefit of longer bodies and pipeline instruction
- unrolling also enables using vectorization, compilers can take advantage of SIMD instructions more easily

## What are pragmas?

A pragma stands for “pragmatic information”. It is a special instruction for the compiler. Pragmas don't change the logic of your program but can influence the compiler's behavior, usually for optimization or platform-specific adjustments.

They follow the syntax:

```cpp
#pragma <instruction>
```

Pragmas are compiler-specific and not standardized. If a compiler doesn't support a `pragma`, it simply ignores it. This means your code will still compile, though the `pragma` may have no effect.

In practice, pragmas give developers fine-grained control over compiler behavior, especially for performance tuning. They are powerful, but using them too heavily can reduce portability.

## What is `pragma unroll`?

`#pragma unroll` is a compiler directive that hints to the compiler that a loop should be unrolled a certain number of times, or as much as possible. Its exact syntax and behavior depend on the compiler.

From what I've found, it is most impactful in GPU programming, and the exact syntax `#pragma unroll` is primarily associated with NVIDIA’s CUDA compiler (NVCC). The example I saw at CppCon in Alexandrescu's talk also used this specific syntax, which makes sense since he works at NVIDIA.

### NVCC

In CUDA, you can write:

```cpp
#pragma unroll 2
for (int i = 0; i < 8; i++) {
    arr[i] *= 2;
}
```

The above would produce something equivalent to:

```cpp
for (int i = 0; i < 8; i += 2) {
    arr[i] *= 2;
    arr[i+1] *= 2;
}

```

`#pragma unroll 8` would fully unroll the loop.

`#pragma unroll` (without a number) leaves it up to the compiler to decide the level of unrolling.

### Clang

Clang offers more fine-grained control with several options:
- `#pragma clang loop unroll(disable|enable)` – disable or enable loop unrolling,
- `#pragma clang loop unroll(full)` – request full unrolling,
- `#pragma clang loop unroll_count(N)` – request unrolling by `N`.

Even without these pragmas, Clang already performs aggressive unrolling at `-O2` and especially at `-O3`.

### GCC

GCC provides `#pragma GCC unroll N.`

If `N` is 0 or 1, no unrolling happens.

Like Clang, GCC enables some unrolling with `-O2` and `-O3`.

The `-funroll-loops` compiler flag can explicitly force more unrolling.

### MSVC

MSVC does not provide a `pragma` for loop unrolling. It has other loop-related pragmas (such as `#pragma loop` for hints on parallelization and vectorization), but loop unrolling must be left to the compiler's own heuristics.

## Conclusion

Loop unrolling is a well-known optimization technique that reduces loop overhead and can unlock better performance through pipelining and vectorization. While you can always unroll loops manually, it's almost always better to leave this job to the compiler, which can make smarter, context-aware decisions.

Pragmas like `#pragma unroll` (and their compiler-specific variants) allow developers to guide the compiler in cases where performance is critical—especially in GPU programming. However, overuse can lead to code bloat and reduced portability.

As with most optimizations, start by trusting the compiler. Only reach for pragmas like `unroll` if you've measured a performance bottleneck and are confident that the hint will help.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!