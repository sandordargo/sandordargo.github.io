---
layout: post
title: "C++26: std::constant_wrapper"
date: 2026-11-4
category: dev
tags: [cpp, cpp26, constexpr, metaprogramming]
excerpt_separator: <!--more-->
---
There's a fundamental gap in C++ that trips up everyone who reaches for it: you can't pass a `constexpr` value through a function boundary and have it remain `constexpr` on the other side. The value is known at compile time, the function is `constexpr`, but the parameter inside the function body is not a constant expression. This forces workarounds - template parameters, tag types, `std::integral_constant` - all of which are verbose and limited.

C++26 adds `std::constant_wrapper` ([P2781R9](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2781r9.html) by Hana Dusíková, Matthias Kretz, Zach Laine; refined by [P3978R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3978r3.pdf) by Kretz, Kamiński) and its shorthand `std::cw<V>` to close this gap. It wraps *any* structural type — not just integers — in a type that carries the value as a non-type template parameter, preserving its compile-time nature across function calls.

<!--more-->

## The problem

Consider this:

```cpp
constexpr int square(int x) {
    return x * x;
}

constexpr int result = square(5); // fine — full expression is constant-evaluated
```

That works because the entire call is evaluated at compile time. But the moment you try to *use* the parameter as a constant expression *inside* the function, things fall apart:

```cpp
// https://godbolt.org/z/hEWPP1cdE

constexpr int get_element(int index) {
    std::array<int, 3> arr = {10, 20, 30};
    return std::get<index>(arr); // error: index is not a constant expression
}
```

The parameter `index` is not a constant expression inside the function body, even though the caller passed a literal. First it might seem a compiler limitation — but it's by design. A `constexpr` function can also be called at runtime, so the compiler can't assume parameters are compile-time constants.

The traditional workaround is to lift the value into the type system:

```cpp
// https://godbolt.org/z/3eEzqajnT

template<int Index>
constexpr int get_element() {
    std::array<int, 3> arr = {10, 20, 30};
    return std::get<Index>(arr); // fine — Index is a template parameter
}
```

But this changes the calling convention, doesn't compose with other function arguments, and scales poorly when you need multiple compile-time values alongside runtime ones.

`std::integral_constant` was the library-side answer — encode the value as a type, then recover it via `::value`:

```cpp
// https://godbolt.org/z/KoEq19Wa4

template<typename T>
constexpr int get_element(T) {
    std::array<int, 3> arr = {10, 20, 30};
    return std::get<T::value>(arr);
}

get_element(std::integral_constant<int, 1>{}); // works, but verbose
```

This approach has two problems. First, `std::integral_constant<int, 1>{}` is painful to write, especially when you need several. Second, it only works for integral and enumeration types. If you need to pass a `constexpr` floating-point value, a pointer, or a user-defined structural type, `integral_constant` can't help you.

## `std::constant_wrapper` and `std::cw`

`std::constant_wrapper<X>` wraps a value `X` of any structural type as a non-type template parameter. `std::cw<X>` is a variable template that gives you a `constant_wrapper` instance without spelling out the type:

```cpp
// https://godbolt.org/z/GKnoK8E1a

constexpr auto a = std::cw<42>;      // constant_wrapper wrapping int 42
constexpr auto b = std::cw<3.14>;    // wrapping double
constexpr auto c = std::cw<'x'>;     // wrapping char

// the verbose form — same types, same values
constexpr std::constant_wrapper<42>   a2{};
constexpr std::constant_wrapper<3.14> b2{};
constexpr std::constant_wrapper<'x'>  c2{};

static_assert(std::is_same_v<decltype(a), decltype(a2)>);
static_assert(std::is_same_v<decltype(b), decltype(b2)>);
static_assert(std::is_same_v<decltype(c), decltype(c2)>);
```

It lives in `<utility>`. P2781 originally placed it in `<type_traits>` next to `integral_constant`, but P3978R3 moved it — `constant_wrapper` is not a trait, and the unwrapping `operator()` / `operator[]` overloads added by P3978 use *INVOKE*, which isn't available in `<type_traits>`. Think of it as `integral_constant` generalized to all structural types, with a concise spelling and operator support.

A `constant_wrapper` implicitly converts to the wrapped value, so it works anywhere the underlying type is expected:

```cpp
void takes_int(int n);
takes_int(std::cw<42>); // implicit conversion to 42
```

But the key property is that when a function receives a `constant_wrapper` by value, it can extract `T::value` as a constant expression:

```cpp
template<typename T>
constexpr int get_element(T) {
    std::array<int, 3> arr = {10, 20, 30};
    return std::get<T::value>(arr);
}

constexpr int x = get_element(std::cw<1>); // x == 20
```

The value survives the function boundary because it's baked into the type.

## Beyond integers

`integral_constant` is limited to integral and enumeration types — you can't write `std::integral_constant<double, 3.14>{}`. `constant_wrapper` accepts any structural type: floating-point values, pointers, and user-defined class types that qualify as non-type template parameters.

Where this matters is the same scenario from the opening section: you need the value as a constant expression *inside* the function body, not just at the call site. Consider a function that dispatches to a type parameterized by a floating-point constant:

```cpp
// https://godbolt.org/z/7nd8TPYP5

template<double Factor>
struct ScaledArray {
    std::array<double, 100> data{};
    double get(int i) const { return data[i] * Factor; }
};

template<typename F>
auto make_scaled_array(F) {
    return ScaledArray<F::value>{};
}

auto arr = make_scaled_array(std::cw<2.5>); // ScaledArray<2.5>
```

`ScaledArray` is parameterized on a `double` template argument — something only possible since C++20 allowed class-type and floating-point NTTPs. To construct a `ScaledArray<2.5>` from a function, the factor must be a constant expression inside the function body. A plain `double` parameter wouldn't work: `ScaledArray<factor>` would fail to compile. `integral_constant` can't carry a `double` at all. `constant_wrapper` bridges both gaps.

## Arithmetic operators

This is where things get interesting. `constant_wrapper` overloads all the standard arithmetic, comparison, logical, and bitwise operators. The operations happen at compile time and the results are themselves `constant_wrapper` values:

```cpp
// https://godbolt.org/z/1arsazcnG

constexpr auto a = std::cw<10>;
constexpr auto b = std::cw<3>;

constexpr auto sum  = a + b;   // constant_wrapper<13>
constexpr auto diff = a - b;   // constant_wrapper<7>
constexpr auto prod = a * b;   // constant_wrapper<30>
constexpr auto quot = a / b;   // constant_wrapper<3>
constexpr auto mod  = a % b;   // constant_wrapper<1>

static_assert(sum == std::cw<13>);
```

The comparison operators also return `constant_wrapper` rather than a plain `bool`:

```cpp
constexpr auto cmp = std::cw<5> < std::cw<10>; // constant_wrapper<true>
static_assert(cmp); // implicit conversion to bool
```

This means comparisons compose — you can pass them into further compile-time computations without losing the "constant" property. The unary operators (`-`, `~`, `!`, `++`, `--`) work the same way:

```cpp
constexpr auto neg = -std::cw<42>;  // constant_wrapper<-42>
constexpr auto inc = ++std::cw<0>;  // constant_wrapper<1>
```

Note that `++` and `--` don't mutate — they can't, since the value is a template parameter. They return a new `constant_wrapper` with the incremented/decremented value.

One subtlety: the short-circuit operators `&&` and `||` are overloaded only when at least one operand's value type is not convertible to `bool`. For ordinary boolean-like values, the built-in operators with their short-circuit semantics apply. This avoids surprising behavior where `std::cw<false> && some_expensive_cw` would evaluate both sides.

## Call and subscript: "it's the thing it holds, unless it can stay wrapped"

You can also wrap a callable or an array and call or index into it through the wrapper. This is where [P3978R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3978r3.pdf) comes in — it added unwrapping `operator()` and `operator[]` overloads, with a design principle borrowed from `std::reference_wrapper`: the wrapper should transparently become the thing it holds, *unless* both the wrapper and all arguments are `constant_wrapper`s, in which case the result stays wrapped too.

```cpp
// https://godbolt.org/z/PrYE9M963

constexpr int iota[4] = {0, 1, 2, 3};

auto a = std::cw<iota>[1];           // int 1 — unwraps ("the thing it holds")
auto b = std::cw<iota>[std::cw<1>];  // constant_wrapper<1> — stays wrapped
```

The same logic applies to function calls. If you wrap a function pointer:

```cpp
// https://godbolt.org/z/Po4ccTG9q

constexpr int fun(int x) { return x + 1; }

auto a = std::cw<fun>(1);           // int 2 — unwraps
auto b = std::cw<fun>(std::cw<1>); // constant_wrapper<2> — stays wrapped
```

When all operands are `constant_wrapper`s, the operation can be evaluated entirely at compile time, so the result is re-wrapped into a `constant_wrapper`. When any argument is a plain runtime value, the wrapper unwraps to the underlying callable/array and calls/indexes it normally. This follows the same pattern as the arithmetic operators: `cw<1> + cw<2>` yields `cw<3>`, but `cw<1> + 2` yields plain `3`.

## The `std::submdspan` motivation

The [paper's](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2630r4.html) opening example comes straight from `std::submdspan`. `submdspan` takes slice specifiers that are sometimes compile-time indices, sometimes runtime values. Before `constant_wrapper`, passing a compile-time index looked like this:

```cpp
auto sir = std::strided_index_range{
    std::integral_constant<size_t, 0>{},
    std::integral_constant<size_t, 10>{},
    3};
auto sub = submdspan(x, sir);
```

With `cw`:

```cpp
auto sub = submdspan(x, std::strided_index_range{std::cw<0uz>, std::cw<10uz>, 3});
```

Both pass compile-time values for the first two arguments and a runtime value for the stride. The difference is readability.

## String literals

Since C++20 made class types with `constexpr` constructors usable as non-type template parameters (if they're structural), and `constant_wrapper` wraps its value through a helper that supports arrays, you can wrap string literals:

```cpp
constexpr auto greeting = std::cw<"hello">;
std::cout << greeting; // prints: hello
```

The value is stored as a fixed-size character array inside the template parameter. This is different from a `const char*` — it's a value, not a pointer, so it remains a constant expression.

## Conclusion

`std::constant_wrapper` is a small type with an outsized effect. It takes a problem every C++ programmer has hit - "this value is `constexpr`, why can't I use it as one inside this function?" - and provides a clean, general, composable answer. The operator overloads make it more than a transport mechanism: compile-time arithmetic just works, without dropping out of the type system. [P3948R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3948r1.pdf) even replaced the separate `std::nontype`/`constant_arg` mechanism in `function_ref` with `constant_wrapper`, unifying two features into one. And `std::cw<V>` makes it terse enough that you'll actually use it.

If you've been writing `std::integral_constant<size_t, N>{}` and wishing there was a better way - there is now.

{% include connect-deeper.html %}
