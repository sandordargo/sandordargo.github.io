---
layout: post
title: "C++26: pack indexing"
date: 2025-1-22
category: dev
tags: [cpp, cpp26, templates, packindexing]
excerpt_separator: <!--more-->
---
C++11 introduced parameter packs to provide a safer way to pass an undefined number of parameters to functions [instead of relying on variadic functions](https://www.sandordargo.com/blog/2023/05/03/variadic-functions-vs-variadic-templates).

While packs are a useful feature, and since C++17 it's [so easy to use them in fold expressions](https://www.fluentcpp.com/2021/03/12/cpp-fold-expressions/), extracting a specific element of a pack is somewhat cumbersome.

You either have to [rely on some standard functions not made for the purpose](https://devblogs.microsoft.com/oldnewthing/20240516-00/?p=109771) or use [*"awkward boolean expression crafting or recursive templates"*](https://medium.com/@simontoth/daily-bit-e-of-c-pack-indexing-28e52d69c2d7). None of them is unbearable, but it might be error-prone or simply expensive regarding compile-time performance. Nevertheless, they are not the most readable solutions.

C++26 brings us pack indexing as a core language feature thanks to the proposal of Corentin Jabot and Pablo Halpern, [P2662R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2662r3.pdf).

Before discussing some interesting points, first, let's look at an example:

```cpp
template <typename... T>
constexpr auto first(T... values) -> T...[0] {
  return values...[0];
}

template <typename... T>
constexpr auto last(T... values) -> T...[sizeof...(values)-1] {
  return values...[sizeof...(values)-1];
}

int main() {
  // first(); // ill-formed: invalid index 0 for pack 'T' of size 0
  static_assert(first(1, 2, 3, 4, 5) == 1);
  static_assert(last(1, 2, 3, 4, 5) == 5);
  static_assert(first(1, 2, 3, 4, 5) + last(1, 2, 3, 4, 5) == 6);
}
```

We can observe that we can use indexes both with the types and with the values and we can use anything that can be evaluated at compile time. With that, I refer to the fact, that we can get the type or value of the last element by using `sizeof...(expr)-1`.

[P2662R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2662r3.pdf) is a long but interesting proposal. It discusses several alternative syntaxes for pack indexing and mentions that in fact several people independently came to the same conclusion regarding syntax, which is also proposed by this paper.

While reading the above example one might have the feeling that it uses way too many triple dots, but it still perfectly fits in and just by looking at it, you probably know exactly what's going on. Maybe we could even say that it reads like well-written prose.

Moreover, the beauty of this syntax and proposal is that it's open for extension. The proposal doesn't try to push for everything at once. I met many people who think that we often get half-baked solutions in the standard. To get a new feature completely usable, we usually have to wait for one more standard at least, think about ranges, coroutines, not to mention modules... 

In this, case, I don't think that the feature is half-baked. While indexing from the back is not solved, it's still just as usable as any indexing in C++. On the other hand, it's educating to read about the considerations. Did you think about using negative indexes just like in Python? It would also kick in if an expression was accidentally evaluated as negative and therefore it's undesirable.

But maybe an alternative syntax like `T...[^1]` or `T...[$-1]` will be accepted in the future. Until then, we'll have to live with `T...[sizeof...(T)-1`. Providing a syntax for slicing is also yet to be standardized.

## Conclusion

C++26 brings us pack indexing as a language feature. We'll be able to extract individual elements from a pack using the subscript operator: `T...[0]`. Thanks for the work of those who contributed to the proposal!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)