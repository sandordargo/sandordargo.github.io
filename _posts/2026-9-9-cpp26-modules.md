---
layout: post
title: "C++26: Module improvements"
date: 2026-9-9
category: dev
tags: [cpp, cpp26, modules]
excerpt_separator: <!--more-->
---
C++20 introduced modules as a - potential - fundamental shift in how we organize and consume C++ code. Sadly, the adoption has been slow — build system support lagged, compiler implementations were incomplete and diverging, and some of the original design decisions created unnecessary friction. C++26 addresses some of these pain points with three targeted fixes.

<!--more-->

## P3034R1: Module declarations shouldn't be macros

One of the practical problems with C++20 modules is that build systems need to figure out which module a source file belongs to *before* compiling it. That's the dependency discovery problem: to build files in the right order, you need to know what each file imports and exports.

The C++20 grammar allows macros to appear in module declarations. You could write something like:

```cpp
#define MODULE_NAME my.module
export module MODULE_NAME;
```

This means that to determine the module name, a build tool must run the preprocessor — or at least simulate enough of it to expand macros. That's slow, fragile, and forces build systems into doing work they shouldn't have to do.

[P3034R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3034r1.html) fixes this by requiring that module declarations must not involve macros. The module name must be spelled out directly in the source:

```cpp
export module my.module; // OK
```

This is technically a breaking change. But given the low adoption rate of modules so far and the rarity of macro-based module declarations in practice, the committee judged this acceptable. The benefit is concrete: build systems can now determine module dependencies with a simple lexical scan, without invoking the preprocessor.

## P3618R0: Attaching `main` to a module

In C++20, `main` must be in the global module. You cannot define it inside a module unit. This seems like a minor restriction, but it has a practical consequence: if you want to unit-test non-exported entities of a module, you're out of luck. The test harness — which typically needs a `main` — can't be part of the module, and non-exported entities aren't visible outside it.

[P3422](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3422r0.html) attempted to solve this by *implicitly* attaching `main` to the global module even when defined in a module unit. But implicit rules are the wrong tool here — they create a special case that's easy to forget and hard to reason about.

[P3618R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3618r0.html) takes the cleaner approach: it simply allows `main` to be explicitly attached to the global module from within a module unit. Instead of an implicit exception, you get a straightforward rule: `main` can live in a module, and it has the same linkage and visibility as if defined in the global module.

```cpp
export module my.module;

// non-exported implementation details
int internal_helper() { return 42; }

// main explicitly in the global module, but can see module internals
extern "C++" int main() {
    return internal_helper() != 42;
}
```

This is a non-breaking change — code that was previously ill-formed now compiles.

## P3868R1: Allowing `#line` before module declarations

Code generators and tools that produce C++ source files commonly insert `#line` directives at the top. These directives tell the compiler which file and line number the generated code corresponds to, improving error messages and debugging.

The problem is that the C++20 grammar does not allow `#line` directives to appear before a module declaration. If a code generator emits a `#line` directive before the `export module` line, the resulting code is ill-formed.

[P3868R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3868r1.html) fixes this by allowing `#line` directives before module declarations:

```cpp
#line 1 "generated_module.cpp"
export module generated; // now OK
```

The fix is straightforward and non-breaking — it simply makes previously ill-formed code valid. Tools that generate module source files no longer have to work around this restriction.

## Conclusion

None of these changes are revolutionary on their own. They're small, targeted fixes that remove pain points from the module system. That's exactly what modules need right now and whether it will help speed up adoption, that's another question. Let's hope so. The grand vision of C++20 modules was always ahead of the tooling and ecosystem. C++26 isn't adding new module features — it's clearing the obstacles that have been slowing adoption. Faster dependency scanning, testing non-exported code, and better tool support all contribute to making modules practical.

{% include connect-deeper.html %}
