---
layout: post
title: "What are string_views and why should we use them?"
date: 2022-7-13
category: dev
tags: [cpp, string_view, bestpractices, string]
excerpt_separator: <!--more-->
---
`std::string_view` has been introduced by C++17 and its purpose is to provide read-only access to character sequences. It potentially replaces `const string&` parameters and offers a significant performance gain. Let's delve into some details. 

## How is it implemented?

A typical implementation of a `string_view` needs two pieces of information. A pointer to the character sequence and its length. The character sequence can be both a C++ or a C-string. After all, `std::string_view` is a non-owning reference to a string.

If we check the major implementations, we can observe that indeed all of them implemented `string_view` by storing a pointer to the string data and the size of the string. You can have a look at the implementations here:

- [gcc](https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/std/string_view)
- [clang](https://github.com/llvm/llvm-project/blob/main/libcxx/include/string_view)
- [Microsoft](https://github.com/microsoft/STL/blob/main/stl/inc/xstring)

## Why is it useful?

This type is particularly useful! It's quite cheap to copy it as it only needs the above-mentioned copy and its length. It's so cheap to copy it that you should never see a `string_view` passed around by reference. It's so cheap to copy that makes `const string&` parameters superfluous in the vast majority of the cases.

If a function doesn't need to take ownership of its `string` argument and it only performs read operations (plus some modifications, to be discussed later) then you can use a `string_view` instead.

When you need to own a character sequence, you should use a `std::string` [as the Core Guidelines reminds us](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#slstr1-use-stdstring-to-own-character-sequences). Otherwise, `string_view`s provide an easy way to get a view of strings no matter how they are allocated or stored. By that I mean that it doesn't matter whether the underlying string has an implicit null termination (`std::string`) or not (`const char *`), `string_view` will be useable.

If for some reason though you need that implicit null termination, you still must stick with a `const string&`.

If you want to get a bit more information about the performance of `std::string_view` against `std::string`, I highly recommend checking out [this article from ModernesC++](https://www.modernescpp.com/index.php/c-17-avoid-copying-with-std-string-view). In the last sections, Rainer Grimm shows the time difference it takes to create substrings either with `std::string::substr` or with `std::string_view::substr` and the results are just amazing.

The difference depends a lot on the size of the substring which is due to the cost allocation a `string` needs and also due to small string optimization eliminating this need. All in all, the bigger substrings we create the more we save. Having `-O3` turned on for smaller strings, Rainer achieved an improvement of almost 10x at least, but for big enough strings it was beyond an astonishing x7500 improvement.

### What API does `string_view` offers?

As mentioned ealier, even though `string_view` is not owning the underlying string, it offers some modifying operations. I'd say 
- `std::string_view::swap` is obvious, it simply exchanges views between two `string_views`.
- `remove_prefix` and `remove_suffix` are more interesting, how is that possible?

These modifiers take a number (`size_type`) `n` to be removed. As we discussed earlier, a `string_view` usually has two data members. A pointer to the underlying character list and its size. In order to remove the suffix, so the end of the string, it's enough to decrease the size data member by `n`. And in order to remove the prefix, besides decreasing the size, the pointer pointing at the character list should also be increased. It's as easy, assuming that the characters are stored in a contiguous memory area.

```cpp
#include <iostream>
#include <string_view>


int main() {
    std::string_view sv{"here this is a string_view example"};
    std::cout << sv << '\n';
    sv.remove_prefix(5);
    std::cout << sv << '\n';
    sv.remove_suffix(8);
    std::cout << sv << '\n';
}
/*
here this is a string_view example
this is a string_view example
this is a string_view
*/
```

Apart from these, the `string_view` offered from the beginning the following functionalities:
* `copy`
* `substr`
* `compare`
* a bit set of `find` methods

Let's have a look at `copy` and `compare`!

### std::string_view::copy

I wanted to zoom in on this method because when I first saw, I asked myself the question what do we copy there? And from there?

`std::string_view::copy` takes three parameters with the last one having a default value. The first parameter is the destination, the second one is the length of the substring you want to copy and the third one is the starting point. If you don't specify the last one, it's the beginning of the string by default.

So with `std::string_view::copy` we copy from the underlying view to somewhere else.

Where can we `copy`? It can be any container of characters. Here are a few examples.

```cpp
#include <array>
#include <iostream>
#include <iomanip>
#include <string_view>


int main() {
    std::string_view sv{"here this is a string_view example"};
    std::array<char, 8> destinationArray{};
    
    
    sv.copy(destinationArray.data(), 4);
    for (auto c: destinationArray) {
        std::cout << c;
    }
    std::cout << '\n';
    
    std::string destinationStringNoSpace;
    sv.copy(destinationStringNoSpace.data(), 9);
    std::cout << destinationStringNoSpace << '\n';
    
    std::string destinationStringWithSpace(' ', 9);
    sv.copy(destinationStringWithSpace.data(), 9);
    std::cout << destinationStringWithSpace << '\n';
}
```

It's worth noting that we can copy to `char*`, therefore we always pass in the result of the `data()` accessor. It's also worth nothing that we have to make sure that a `string` is big enough. And `reserve` is not good enough as it only makes sure that there is enough space to grow, not that there is space initialized.

### `std::string_view::compare`

I wanted to zoom in on `std::string_view::compare` as it's always worth having a look at comparisons that return an integer value? What do they mean?

But having a look at the [available signatures](https://en.cppreference.com/w/cpp/string/basic_string_view/compare) poses some other questions.

There are two straightforward ones. The `compare` member method can be called with either another `string_view` or with a `const char*`. But that is not all! You don't have to compare the full `string_view`. You might pass in a starting position and a count for the underlying `script_view`, they precede the other character sequence.

In addition, if you compare with another `string_view`, you can pass in the starting position and the size for the other view too. If you compare with a `const char*`, you cannot define the starting position, but you can still pass in the size.

And what are the available return values?
- `0` if both are equal.
- You get a positive value if the underlying string is greater.
- You get a negative value if the other string is greater.

Let's have a look at some examples.

```cpp
#include <string_view>

int main() {
    using std::operator""sv;

    static_assert( "abc"sv.compare("abcd"sv) < 0 ); // Other is greater
    static_assert( "abcd"sv.compare(0, 3, "abcd"sv) < 0 ); // Other is greater
    static_assert( "abcd"sv.compare(1, 3, "abcd"sv) > 0 ); // This is greater
    static_assert( "abcd"sv.compare(1, 3, "abcd"sv, 1, 3) == 0 ); // Both are equal
    static_assert( "abcd"sv.compare(1, 3, "bcde", 3) == 0 ); // Both are equal
    static_assert( "abcd"sv.compare("abc"sv) > 0 ); // This is greater
    static_assert( "abc"sv.compare("abc"sv) == 0 ); // Both are equal
    static_assert( ""sv.compare(""sv) == 0 );// Both are equal
}
```

## Novelties of string_view in C++23/C++20

But since its introduction in C++17, `string_view` has received some new functionalities in both C++20 and 23.

### `starts_with` / `ends_with` added in C++20

These two queries were added to `string_view` in C++20. They help us to write more expressive code. We can simply call them to check whether a string starts or ends with a given substring. Look at the below example to see how it simplifies life.

```cpp
#include <iostream>
#include <iomanip>
#include <string_view>


int main() {
    std::string_view sv{"here this is a string_view example"};
    
    if (sv.starts_with("here")) {
        std::cout << std::quoted(sv) << " starts with \"here\"\n";
    }
    
    if (!sv.ends_with("view")) {
        std::cout << std::quoted(sv) << " does not end with \"view\"\n";
    }
}
```

How much does it simplify life? Just check out [this](https://www.techiedelight.com/check-if-a-string-starts-with-a-certain-string-in-cpp/) or [this](https://stackoverflow.com/questions/1878001/how-do-i-check-if-a-c-stdstring-starts-with-a-certain-string-and-convert-a) article and you'll see! This is just a super addition!

### `std::string_view` now have `contains`

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
### Build `std::string_view` from ranges

With C++23, our favourite `string_view` doesn't only loses a constructor (the overload with a `nullptr` gets deleted), but also receives a new one. Soon, we'll be able to construct one out of a range directly.

So far, if we wanted to create a `string_view` out of a *"range"*, we had to invoke the constructor with a `begin` and and `end` iterators: `std::string_view sv(myRange.begin(), myRange.end());`. Now we'll be able to directly construct a `string_view` based on a range: `std::string_view sv(myRange);`.

### Require span & basic_string_view to be TriviallyCopyable

[P2251R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2251r1.pdf) updates the requirements the standard has for `std::span` and `std::string_view`. Starting from C++23 they must satisfy the `TriviallyCopyable` concepts.

As both of these objects already have default copy assignment operators and constructs and also destructors and besides they only expose a `size_t` and a raw pointer, it is implied that these types can be trivially copyable and in fact, the major compilers already implemented them as so.

Ensuring this trait for the future makes sure developers can continue depending on these characteristics and less courageous developers can start using them as such for example in heterogeneous computing.

## Conclusion

In this post, we discussed what `string_view`s are and howthey simplify our lives. We saw that they don't just offer superior performance due to fewer copies but they also provide an easy-to-use interface that gets better with each version.

Have you started using more and more the `string_view` instead of `const string&` in your projects?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
