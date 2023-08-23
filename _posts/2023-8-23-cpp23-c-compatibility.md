---
layout: post
title: "C++23: compatibility with C"
date: 2023-8-23
category: dev
tags: [cpp, cpp23, labels, atomics]
excerpt_separator: <!--more-->
---
In this blog post, let's see two papers in C++23 that are being introduced due to compatibility with C.

## The `<stdatomic.h>` header

Originally, atomics for C and C++ were designed together. The aim was to make the non-generic pieces of the C++ atomics library usable as a C library. Due to some design decisions during C++11 and C++14 standardization, this aim was not reached.

As a consequence, if you want to use atomics and want to write code that is portable between C and C++, you'll probably end up using [preprocessor macros](https://www.sandordargo.com/blog/2022/09/07/prepocessive-directive-changes-in-cpp23) to decide which header to include. In case of C++, you'll also import the necessary types to the global namespace. *(The below example is taken from the [proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p0943r6.html))*

```cpp
#ifdef __cplusplus
  #include <atomic>
  using std::atomic_int;
  using std::memory_order;
  using std::memory_order_acquire;
  ...
#else /* not __cplusplus */
  #include <stdatomic.h>
#endif /* __cplusplus */
...
int saturated_fetch_add(atomic_int *loc, int value, memory_order order);
```

Please note that this approach assumes that C and C++ representations of atomics are compatible.

[P0943R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p0943r6.html) proposes the introduction of the header called `<stdatomic.h>` which will *"mirror the identically named C header, and provide comparable functionality"*.

This header does 3 things:
- It includes the `<atomic>` C++ header
- Defines a macro such as `atomic_generic_type(T)` as `std::atomic<T>`. This assumes that C would correspondingly define `atomic_generic_type(T)` as `_Atomic(T)`.
- Introduces types in the `std` namespace starting with `atomic_` and `memory_oder_` to the global namespace through aliases.

The proposal doesn't look for providing equivalence between C and C++ implementations, as it wants to avoid significant language changes. It provides the minimum required to conveniently use atomics in shared headers.

## Labels at the end of compound statements

There is another proposal accepted to C++23 due to compatibility reasons with `C`, [P2324R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2324r2.pdf).

Labels show that compatibility-fueled changes go both ways. Earlier, a change was adopted for C2x allowing placing labels everywhere inside a compound statement, even in front of the declaration. Let's demonstrate it with an example.

```cpp
void foo() {
	first:
		int x;
}
```

The label `first` used to be allowed only in C++, but not in C. It's due to grammar differences between the two languages. In C, declarations and statements are separate production rules and they can both appear as block-items inside compound statements. To fix the above difference, C allowed also labels as independent block items. But that led to another difference. Now in C, you can put a label at the end of a compound statement.

```cpp
void foo() {
	// some code
	/// ...
	last_label:
}
``` 

In C++, labels can be attached to all statements, but they cannot be placed at the end of a compound statement, so the above example is invalid in C++. At least it used to be. The change proposal is accepted for C++23, GCC 13 and Clang 16 already implemented it.

## Conclusion

In this article, we reviewed two changes aiming for higher compatibility with C. The introduction of `<stdatomic.h>` header makes it easier to have shared headers between C anc C++ when atomics are used. The second change allows to have labels at the end of compound statements, so for example at the very of a function. But we also saw that it's not only C++ that introduces compatibility changes but also C when it started to allow having labels in front of variable declarations. By the way, who is using labels in new code?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
