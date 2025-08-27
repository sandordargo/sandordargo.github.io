---
layout: post
title: "How to look up values in a map"
date: 2025-8-27
category: dev
tags: [cpp, map, unordered_map, lookup]
excerpt_separator: <!--more-->
---
Whether you're in a coding interview or writing production code, you'll eventually face the question: *What's the right way to look up values in a `std::map` or `std::unordered_map`?* For simplicity, we'll refer to both containers as *maps* in this post.

Let's explore the different options — along with their pros and cons.  

## `operator[]`

Using `operator[]` is the old-fashioned way to access elements of a map. It takes a key and returns a reference to the corresponding value. The complexity is *log(n)* for `std::map` and *average constant time* (with worst-case linear) for `std::unordered_map`.

However, there's a big caveat.

What if the key is not present in the map?

Unlike a `vector` — where accessing an invalid index with `operator[]` leads to undefined behavior — a `map` will instead insert a new entry with the given key and a default-constructed value. This side effect makes `operator[]` unsafe for lookups where insertion is not desired.

That's also why you **can't use `operator[]` on a `const` map**:

```cpp
#include <map>

int main() {
    const std::map<int, int> squares{ {1, 1}, {2, 4}, {3, 3} };
    return squares[2]; // ERROR: passing 'const std::map<int, int>' as 'this' argument discards qualifiers
}
```

Too bad. We need an alternative!

## `at()`

The `at()` method comes in rescue. Like `operator[]`, it provides efficient access, but unlike `operator[]`, it never inserts a new element. Instead, it throws a `std::out_of_range` exception if the key is not found. That makes it suitable for use with const maps:

```cpp
#include <map>

int main() {
    const std::map<int, int> squares{ {1, 1}, {2, 4}, {3, 3} };
    return squares.at(2);
}
```

To use `at()` safely, we either have to ensure that the key is present or wrap the lookup in a try-catch block.

Look at these two wrappers:

```cpp
#include <map>
#include <optional>
#include <stdexcept>

std::optional<int> lookupAtContains(const std::map<int, int> &map, int key) {
    if (map.contains(key)) {
        return map.at(key);
    }
    return std::nullopt;
} 

std::optional<int> lookupAtTryCatch(const std::map<int, int> &map, int key) {
    try {
        return map.at(key);
    } catch (const std::out_of_range& err) {
        return std::nullopt;
    }
}
```

Both are safe to use, and both have some downsides.

The first one, `lookupAtContains`, is not thread-safe. What if the key is removed between the `contains()` and the `at()` calls? But maybe, we don't have such requirements. The other problem is efficiency. The key is looked up twice. By the way, such a lookup wrapper can be used with `operator[]` as well. Though in case of problems, I think an uncaught exception is still better than undefined behaviour.

The downside of the second alternative is the use of exceptions. Maybe you don't allow them in your codebase. Maybe you're worried about the performance penalties. You might say that exceptions are a zero cost abstraction if they are not called. But that's not entirely true. [They have zero cost only at runtime](https://www.sandordargo.com/blog/2023/03/29/binary-size-and-exceptions). At compile-time, you have to pay for generating the necessary information to deal with exceptions. Besides, not finding a key in a map is clearly not an exceptional case. 

## `std::find`

Yet another option is to use the `find()` method.

```cpp
int main() {
    const std::map<int, int> squares{ {1, 1}, {2, 4}, {3, 3} };
    auto maybe_entry = squares.find(2);
    return maybe_entry != squares.end() ? maybe_entry->second : -1;
}
```

It's great that it works on `const` containers, but it's simply ugly. You get back an iterator to the entry with the key if the key is present or an iterator pointing past the end of the container if the key is not present. Then deal with it.

We really need a wrapper with this solution.

```cpp
std::optional<int> lookupFind(const std::map<int, int> &map, int key) {
    auto maybe_entry = map.find(key);
    if (maybe_entry == map.end()) {
        return std::nullopt;
    }
    return maybe_entry->second;
}
```

This still has the problem of thread-safety, but that might not be a problem if you work with a single thread.

On the bright side, now we look up the key only once! But this solution really needs a wrapper if we want to use it in a readable way.

## Is double lookup really an issue?

It depends. There's definitely a performance hit when using `contains()` followed by `at()`. [Benchmarks](https://quick-bench.com/q/WVRxStqOIsYWYlDHuTLixa34bYU) show that `lookupAtContains` is around 60% slower than lookupFind when the key is present:

![Performance comparison when key found]({{ site.baseurl }}/assets/img/lookup_key_found.png)

Even worse, `lookupAtTryCatch` is [orders of magnitude slower](https://quick-bench.com/q/2CtXkOZPw1u6pMb8sLJRPyhNOhE
) when the key is missing, due to exception overhead:

![Performance comparison when key not found]({{ site.baseurl }}/assets/img/lookup_key_not_found.png )

That said, the performance hit of the double lookup only matters when this code runs in a hot path. In most applications, the difference will be negligible compared to other bottlenecks - such as exception handling, I/O, etc. Choose clarity over premature optimization — unless you're in performance - critical territory.

## Conclusion

Looking up values in a `map` isn't as trivial as it seems. Here's a summary of your choices:

- Use `operator[]` if you want insertion on missing keys and you're not dealing with const maps.
- Use `at()` for safe, non-inserting access — especially with `const` maps — but be mindful of exceptions.
- Use `find()` for full control and better performance — at the cost of a more verbose syntax. If you're building a utility or reusable component, wrapping `find()` in a helper like `lookupFind()` strikes the best balance between safety, performance, and clarity.

Ultimately, pick the approach that fits your context — readability, performance, and codebase conventions all matter.

Which one do you use most often — and why?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)

