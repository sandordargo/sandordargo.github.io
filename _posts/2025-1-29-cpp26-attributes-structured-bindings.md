---
layout: post
title: "C++26: attributes in structured bindings"
date: 2025-1-29
category: dev
tags: [cpp, cpp26, structuredbindings, attributes]
excerpt_separator: <!--more-->
---
We recently talked about [C++26's unnamed placeholder](https://www.sandordargo.com/blog/2025/01/08/cpp26-unnamed-placeholders) and how useful it will be with structured bindings. Before unnamed placeholders, one of our problems was that in structured bindings we could not mark only one of the decomposed variables `[[maybe_unused]]`, only the whole set of variables.

This is going to change with C++26. Thanks to Aaron Ballman's proposal, [P0609R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p0609r3.pdf), we will be able to tag individual structured bindings with an attribute.

This means that the code below will become valid:

```cpp
std::map<int, std::string> m{
        {1, "one"}, {2, "two"} {3, "three"}};

for(const auto& [ [[maybe_unused]] k, v]: m) {
        DEBUG(k) // only used in debug builds
    std::cout << v << '\n';
}
```

This comes in handy, I've seen several examples where some structured bindings are only used for debug logs, so the release builds throw warning for unused variables. So far one possible solution was to mark the entire structured binding as `[[maybe_unused]]` which usually does not reflect the intentions of the programmer.

Once again, C++ becomes a bit more expressive.

Maybe it's a bit too many square brackets, but it perfectly fits into the existing syntactic rules of the language.

I only mentioned one attribute, `[[maybe_unused]]`. The other current standard attributes do not really make sense in such a context. However, as the author explained, several vendor-specific attributes in different implementations will highly benefit from this new language feature.

This change has already been implemented in GCC 15 and Clang 19.

## Conclusion

Starting from C++26, we'll be able to mark individual structured bindings with attributes, meaning that `const ato& [ [[maybe_unused]] x, y ] = someVar;` will become valid code.

You can already try this feature with a fresh version of GCC and Clang. 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)