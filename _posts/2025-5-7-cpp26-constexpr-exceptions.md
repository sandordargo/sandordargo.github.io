---
layout: post
title: "C++26: constexpr exceptions"
date: 2025-5-7
category: dev
tags: [cpp, cpp26, constexpr, standardlibrary]
excerpt_separator: <!--more-->
---
In recent weeks, we’ve explored [language features](https://www.sandordargo.com/blog/2025/04/23/cpp26-constexpr-language-changes) and [library features](https://www.sandordargo.com/blog/2025/04/30/cpp26-constexpr-library-changes) becoming `constexpr` in C++26. Those articles weren’t exhaustive — I deliberately left out one major topic: exceptions.

Starting with C++26, it will become possible to throw exceptions during constant evaluation. This capability is enabled through both language and library changes. Given the significance of this feature, it deserves its own dedicated post.


## [P3068R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3068r6.html): Allowing exception throwing in constant-evaluation

[The proposal for static reflection](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2996r2.html) suggested allowing exceptions in constant-evaluated code, and [P3068R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3068r6.html) brings that feature to life.

`constexpr` exceptions are conceptually similar to `constexpr` allocations. Just as a `constexpr string` can’t escape constant evaluation and reach runtime, `constexpr` exceptions also have to remain within compile-time code. 

Previously, using `throw` in a `constexpr` context caused a compilation error. With C++26, such code can now compile — unless an exception is actually thrown and left uncaught, in which case a compile-time error is still issued. But the error now provides more meaningful diagnostics.

While no compiler supports this at the time of writing, we can walk through an example from the proposal:

```cpp
constexpr unsigned divide(unsigned n, unsigned d) {
	if (d == 0u) {
		throw invalid_argument{"division by zero"};
	}
	return n / d;
}

// 1)
constexpr auto b = divide(5, 0); // BEFORE: compilation error due reaching a throw expression
								 // AFTER: still a compilation error but due the uncaught exception

constexpr std::optional<unsigned> checked_divide(unsigned n, unsigned d) {
	try {
		return divide(n, d);
	} catch (...) {
		return std::nullopt;
	}
}

// 2)
constexpr auto a = checked_divide(5, 0); // BEFORE: compilation error
									     // AFTER: std::nullopt value
```

In example `1)`, `divide(5, 0)` throws at compile time, but since the exception is uncaught, we get a compile-time error. In example `2)`, `checked_divide` catches the exception and returns a valid value — `std::nullopt`. This is now allowed in constant expressions.

This opens up more expressive compile-time code with proper error handling paths. Can’t wait to see this in action!

## [P3378R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3378r2.html): `constexpr` exception types

Allowing exceptions in `constexpr` code is great — but without `constexpr`-enabled exception types, the usefulness would be limited. [P3378R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3378r2.html), authored by Hana Dusíková, brings over a dozen exception types into the constexpr world.

The paper is quite readable once you’ve wrapped your head around `constexpr` exception throwing. Here are some key takeaways:

- It doesn’t yet propose making all exception types `constexpr`, but that is the long-term goal.
- Future proposals involving `constexpr` functionality should ensure any associated exceptions are `constexpr`-friendly.
- Even `std::runtime_error` becomes constexpr, which is notable because it's a common base class for many derived exceptions like `std::out_of_range`.

Here is the list of exception types becoming `constexpr`-friendly:

- `std::logic_error`
- `std::domain_error`
- `std::invalid_argument`
- `std::length_error`
- `std::out_of_range`
- `std::runtime_error`
- `std::range_error`
- `std::overflow_error`
- `std::underflow_error`
- `std::bad_optional_access`
- `std::bad_variant_access`
- `std::bad_expected_access`
- `std::format_error`

## Conclusion

C++26 is taking another big step forward in making compile-time programming more powerful and expressive. With the ability to `throw` and `catch` exceptions during constant evaluation — and with many standard exception types gaining `constexpr` support — developers will be able to write safer, more robust code that’s fully evaluable at compile time.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!  