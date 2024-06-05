---
layout: post
title: "The limits of `[[maybe_unused]]`"
date: 2024-6-5
category: dev
tags: [cpp, attributes, macros, maybe_unused]
excerpt_separator: <!--more-->
---
The codebase I work with defines a utility macro called `UNUSED`. Its implementation is straightforward, it takes a variable (let's call it `x`) as a parameter and expands it as `((void)x)`. The reason we use it is to avoid compiler warnings for unused variables.

Why don't we delete those variables you might ask? We usually end up with the need for `UNUSED` when we use preprocessor macros to include certain pieces of code only in debug builds, for example, debug logs. This is a simple enough example, right?

```cpp
auto result = doSomething(param);
#ifdef DEBUG
    std::cout << "Result: " << result << '\n';
#endif
UNUSED(result);
```

In this case, if we compile in release mode, `result` is only used by `UNUSED`. If we haven't had that, the compilation would fail in release mode.

But we don't like macros, do we? [They are error-prone due to their limited readability and complicated debugging.](https://arne-mertz.de/2019/03/macro-evil/)

Can we do something better?

First of all, even though I agree with Arne and I dislike macros, I think that the macro is still more readable in this case than what it hides: `(void)x;`

But since C++17 we also have the `[[maybe_unused]]` attribute at our hands. If an entity is declared with this label, any lack of usage emitted warning will be suppressed.

Can it replace our `UNUSED` macro?

The answer is sadly no.

It's true that `[[maybe_unused]]` can be used in a lot of places.

Starting from C++26 it can even mark attributes as potentially unused ones. Until then, we have the following possibilities.

Any `class` / `struct` or `union` can be declared as such: `class [[maybe_unused]] Wrapper`. Though I've barely seen a compiler complaining that a class is unused...

`typedef`s or alias declarations using the `using` keyword can also be declared with `[[maybe_unused]]`.

```cpp
using Squad [[maybe_unused]] = std::vector<Player>;
[[maybe_unused]] typedef std::vector<Player> Squad;
```

Local and non-static data members can also be `[[maybe_unused]]`, just like functions, enumerators and enumerations. `[[maybe_unused]]` can even be used with structured bindings.

```cpp
enum [[maybe_unused]] E {
	A [[maybe_unused]],
	B [[maybe_unused]] = 42
};


[[maybe_unused]] void foo([[maybe_unused]] int param) {
	[[maybe_unused]] bar = 3 * param;
	assert(bar); // only compiled in debug mode
}
```

Though, with structured bindings, we are reaching the limits of `[[maybe_unused]]`. If you use `[[maybe_unused]]`, then all the subobjects are declared as *maybe unused*. You cannot simply mark specific subobjects. It's one or nothing.

```cpp
// both 'a' and 'b' might be unused
// you cannot have only one of them [[maybe_unused]]
[[maybe_unused]] auto [a, b] = std::make_pair(42, 0.23);
```
So why did I say that it cannot replace the `UNUSED` macro?

Well, there is one thing it cannot mark maybe unused. Lambda captures.

If you have a lambda capture that will be only part of the debug build, the release build will complain. And you have no way to use `[[maybe_unused]]` with a lambda capture. When this question came up at a mailing list, [the answer of a committee member](https://lists.isocpp.org/std-proposals/2019/12/0827.php) was that you should use `(void)x;`, as it means less clutter and it's easier to read and maintain. Quite ironic as this solution can be always used, yet `[[maybe_unused]]` seems superior in terms of readability.

```cpp
auto foo = doSomething(param);
auto callback = [&foo] () {
	#ifdef DEBUG
	    std::cout << "foo: " << foo << '\n';
	#endif
    UNUSED(result); // or (void)result;
};
```

Too bad.

What can we do?	

We can keep using our good old `UNUSED` macro.

## Conclusion

In this article, we've seen that the `[[maybe_unused]]` label can help us suppress compiler warnings for variables (and other entities) that are only used in certain builds. Sadly, it doesn't work in all situations, you cannot use it with lambda captures. In those situations, we still need other solutions, such as plain cast to void or a macro.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!

