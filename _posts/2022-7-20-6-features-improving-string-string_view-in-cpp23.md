---
layout: post
title: "6 C++23 features improving string and string_view"
date: 2022-7-20
category: dev
tags: [cpp, cpp23, string, string_view]
excerpt_separator: <!--more-->
---
In this blog post, let's collect a couple of changes that are going to be shipped with C++23 and are all related to `string`s or `string_view`s.

## `std::string` and `std::string_view` have `contains`

One of C++20's useful addition to maps were the `contains` member function. We could replace the cumbersome to read query of `myMap.find(key) != myMap.end()` with the very easy to understand `myMap.contains(key)`. With C++23, `std::string` and `std::string_view` will have similar capabilities. You can call `contains()` with either a string or a character and it will return `true` or `false` depending on whether the queried `string` or `string_view` contains the input parameter.

```cpp
#include <iostream>
#include <string>
#include <iomanip>

int main() {
    std::string s{"there is a needle in the haystack"};
    std::string_view sv{"acdef"};
    
    if (s.contains("needle")) {
        std::cout << "we found a needle in: " << std::quoted(s) << '\n';
    }
    
    if (!sv.contains('b')) {
        std::cout << "we did not find a 'b' in: " << std::quoted(sv) << '\n';
    }
}
/*
we found a needle in: "there is a needle in the haystack"
we did not find a 'b' in: "acdef"
*/
```

## No more undefined behaviour due to construction from `nullptr`

In an earlier newsletter, we discussed that initializing a `string` from a `nullptr` is undefined behaviour. In practice, this might happen when you convert a `const char *` to a `string`. What happens then? It depends on the compiler, `gcc` for example, throws a runtime exception.

Thanks to [P2166R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p2166r1.html), this is not something to worry about.

Instead of undefined behaviour, the constructor and assignment operator overloaded with `nullptr_t` are deleted and therefore compilation fails when you attempt to construct a new `string` out of a `nullptr`.

```cpp
std::string s(nullptr);
/*
<source>:18:26: error: use of deleted function 'std::__cxx11::basic_string<_CharT, _Traits, _Alloc>::basic_string(std::nullptr_t) [with _CharT = char; _Traits = std::char_traits<char>; _Alloc = std::allocator<char>; std::nullptr_t = std::nullptr_t]'
   18 |     std::string s(nullptr);
      |                          ^
/opt/compiler-explorer/gcc-12.1.0/include/c++/12.1.0/bits/basic_string.h:734:7: note: declared here
  734 |       basic_string(nullptr_t) = delete;
      |       ^~~~~~~~~~~~
*/
```

While this change is good and points in a good direction, not all of our problems disappear with `nullptr`s. Taking a `nullptr` and a size in the constructor (e.g. `std::string s(nullptr, 3)`) is still valid and remains undefined behaviour.

These changes are also valid for `string_view`.

## Build `std::string_view` from ranges

With C++23, our favourite `string_view` doesn't only loses a constructor (the overload with a `nullptr` gets deleted), but also receives a new one. Soon, we'll be able to construct one out of a range directly.

So far, if we wanted to create a `string_view` out of a *"range"*, we had to invoke the constructor with a `begin` and and `end` iterators: `std::string_view sv(myRange.begin(), myRange.end());`. Now we'll be able to directly construct a `string_view` based on a range: `std::string_view sv(myRange);`.

## `basic_string::resize_and_overwrite()`

One of the main reasons to use C++ is its high performance. An area where we often use the language in a non-efficient way is string handling. [C++23 will bring us another `string` member function](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1072r10.html) that will help us to handle strings in a more performant way.

[`std::string::resize_and_overwrite()`](https://en.cppreference.com/w/cpp/string/basic_string/resize_and_overwrite) takes two parameters, a count and an operation and does the following (while returns nothing):
- if the `count` is smaller or equal to the `size()` ofthe string, it erases the last `size() - count` elements
- if `count` is larger than `size()`, appends `n - size()` default-initialized elements
- it also invokes `erase(begin() + op(data(), count), end())`.

In other words, `resize_and_overwrite()` will make sure that the given string has continuous storage containing `count + 1` characters.

If `op()` throws, the behaviour is undefined. It's also undefined if it tries to modify `count`.

But what can be an operation?

An operation is a function or function object to set the new contents of the string and it takes two parameters. The first one is the pointer to the first character in the string's storage and the second one is the same as `count`,     the maximal possible new size of the string. It should return the actual new length of the string.

You have to pay attention that this operation doesn't modify the maximum size, does not try to set a longer string and doesn't modify the address of the first character either. That would mean undefined behaviour.

If correctly used, it'll help add some new content or rewrite the existing one. Or you can actually remove content. To illustrate this latter example, let's have a look at the second example of the original documentation.

```cpp
std::string s { "Food: " };
s.resize_and_overwrite(10, [](char* buf, int n) {
    return std::find(buf, buf + n, ':') - buf;
});
std::cout << "2. " << std::quoted(s) << '\n';
// 2. "Food"
```

Even though `s` is resized to 10, the operation will return the position of `:` in the string meaning that it will be truncated from that point.

A new tool to help us write performant string handling code.

## Require span & basic_string_view to be TriviallyCopyable

[P2251R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2251r1.pdf) updates the requirements the standard has for `std::span` and `std::string_view`. Starting from C++23 they must satisfy the `TriviallyCopyable` concepts.

As both of these objects already have default copy assignment operators and constructs and also destructors and besides they only expose a `size_t` and a raw pointer, it is implied that these types can be trivially copyable and in fact, the major compilers already implemented them as so.

Ensuring this trait for the future makes sure developers can continue depending on these characteristics and less courageous developers can start using them as such for example in heterogeneous computing.

## <spanstream> : string-stream with std::span-based buffer

C++23 is introducing [the `<spanstream>` header](https://en.cppreference.com/w/cpp/header/spanstream). Streams are an old part of the C++ standard library. Nowadays *stringtreams* are widely used. Strings (and vectors) store data *outside* of themselves. When the data to be stored grows, the storage and the already stored data might be automatically and dynamically reallocated. This is often acceptable, but when it's not, we need another option.

`<spanstream>` is going to provide such an option, they provide fixed buffers. You have to take care of the allocation when you create your stream, but you don't have to worry about the costly reallocation of the underlying buffer once it's exhausted. When I wrote that you have to take care of the bugger allocation, I really meant it. The buffer is not owned by the stream object, its life has to be managed by the programmer.

## Conclusion

I hope you enjoyed this article and you also got excited by all these various `string`/`string_view` related features that C++23 is going to bring us. What are you waiting for the most?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
