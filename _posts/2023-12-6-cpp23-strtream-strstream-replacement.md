---
layout: post
title: "C++23: The rise of new streams"
date: 2023-12-6
category: dev
tags: [cpp, cpp23, streams, spanstream]
excerpt_separator: <!--more-->
---
The main goal of this article is to share with you the new `<spanstream>` header, but we are going a bit beyond it. We won't only discuss the motivations behind the proposal introducing this header ([P0448R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0448r4.pdf)), but we'll check out an older proposal also from [Peter Sommerlad](https://sommerlad.ch/) ([P0408R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0408r7.pdf)) which modified `std::basic_stringbuf` back in C++20. The reason for covering both is that the author aims for the same purpose with the two proposals. One completes the other.

## `strstream`s are dead, long live the buffers and spans!

While streams are one of the oldest parts of the C++ standard library they haven't been keeping up with the winds of change since the release of C++11 and they required some updates.

The motivation behind these two proposals is to have a stream that avoids unnecessary copies of buffers. In other words, the goal was to provide a stream that 
- provides an efficient access to the underlying buffer
- uses a fixed-size pre-allocated buffer

One of the problems is that up until [P0408R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0408r7.pdf), there was no non-copying access to the internal buffer of a `std::basic_stringbuf` so getting the results of `ostringstream` always required a copy of the internal buffer, even if one doesn't want to use the stream afterwards. [P0408R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0408r7.pdf) solved this problem.

With the acceptance of the other discussed paper, [P0448R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0448r4.pdf), *spanstreams* provide us a stream whose internal storage can use a fixed-size, pre-allocated buffer. That can be something on the stack, for example, a non-owning array view, `std::span<T>`.

By using `std::span<T>`, we can represent and pass a buffer for a new *spanstream* and avoid any dynamic (re)allocation which might not be acceptable for you depending on your use case.

## The first step towards removing the deprecated stream buffer

In [P0408R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0408r7.pdf), the author shares his belief that `basic_strbuf` should be removed from the *[depr.str.strstreams]* section of the standard as soon as the feature is completely replaced.

> The reason why `strstream` was deprecated in the first place is that it returned a `char*` which was difficult to manage and therefore it was prone to cause memory leaks. This difficulty came from the fact that it was nowhere stated how and where it had been allocated. The only satisfactory deallocation was via the [`std::strstream::freeze()` function](https://en.cppreference.com/w/cpp/io/strstream/freeze), but it was not obvious, hence lots of people got it wrong. On the other hand, `stringstream`s return `std::string`s which manage their own memory allocations.

The first step towards that removal was to extend the API of `basic_stringbuf` (note the difference between `basic_strbuf` and `basic_stringbuf`). `std::string_view` was introduced by C++17 to provide efficient read-only access to continuous sequence of characters. A `basic_stringbuf` has similar characteristics so it was a natural and highly waited-for step forward to provide a `string_view`-like access to its internal buffer.

[P0408R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0408r7.pdf) brought the below changes to `basic_stringbuf` in C++20.

Somewhat accidentally, it introduced allocator-aware construction. I'm using the word "accidentally" because the author of these papers shared with me, that this was not part of his original intentions. But the papers were proposed around when the standard library itself made allocator support more flexible with stateful allocators and `std::pmr`, etc. 

At the same time, `basic_stringbuf` is also benefiting from new constructor overloads taking an initial value by rvalue-reference which again is about avoiding unnecessary copies.

`basic_stringbug::str()` went through several changes. First, we must remind ourselves that `basic_stringbug::str()` is both a getter and a setter depending on its return type and parameters.

The getter `str()` now has an overload that is used when the underlying object is an lvalue and another form rvalues. When `str()` is [called on an rvalue reference](https://www.sandordargo.com/blog/2018/11/25/override-r-and-l0-values#use--for-declaring-rvalue-references), it returns the underlying string by moving it away from the internal buffer. According to the author, this is probably the first ref-qualified member function in the standard library. Moreover, in this case, the standard clearly specifies how the moved-from object should look like. Its buffer becomes empty.

`str()` also received an overload with an allocator which sets how to copy the underlying buffer into the returned string. Of course, this is only for the no-move overload.

There is also a new method called `view()` that is both `const` and `noexcept` and returns a `string_view` so that you can have a no-copy, not-owning, read-only access to the contents of the internal buffer.

```cpp
basic_string<charT, traits, Allocator> str() const &; // The & lvalue qualifier is new!

template<class SAlloc>
basic_string<charT,traits,SAlloc> str(const SAlloc& sa) const; // this is a new overload
basic_string<charT, traits, Allocator> str() &&; // this is a new overload
basic_string_view<charT, traits> view() const noexcept; // this is a new method
```

The setter `str()` methods also received two new overloads. The original one takes a string using the same allocator as the `basic_stringbuf` class and takes the string by `const&`. There is a new overload that still takes the input `string` by `const&` but with a different allocator and another one that takes the `string` by rvalue reference, so it moves it to the internal buffer. 

```cpp
void str(const basic_string<charT, traits, Allocator>& s); // this was already there

template<class SAlloc>
void str(const basic_string<charT, traits, SAlloc>& s);  // this is a new overload
void str(basic_string<charT, traits, Allocator>&& s);  // this is a new overload
```

There are several other changes in which we are not going into details, but it's worth noting that `basic_stringbuf::swap()` became conditionally `noexcept` depending on the used allocator.

Probably the most important change among the above is that now you can get the contents of `basic_stringbuf` without actually having to make a copy of the internal buffer. Either it will be moved if you use `str() &&` or you get a read-only view on it if you use `view`.

```cpp
#include <sstream>
#include <iostream>

int main() {

    std::stringbuf buf;
    std::string temp {"Some content"};
    buf.str(std::move(temp));

    // now temp is moved away from, who knows what's inside
    std::cout << "temp: " << temp << '\n';
    
    // no copy is needed to access the internal buffer
    std::string_view bufview = buf.view();
    std::cout << "bufview: " << bufview << '\n';

    // no copy is needed to access the internal buffer which still contains the data we put in it
    std::string_view anotherView = buf.str();
    std::cout << "anotherView: " << anotherView << '\n';

    // still no copy, buf is used as an rvalue-reference, the internal buffer is moved out
    std::string internalBufferMoved = std::move(buf).str();
    std::cout << "internalBufferMoved: " << internalBufferMoved << '\n';

    // now buf is moved away from, who knows what's inside
    std::string_view viewOnMovedObject = buf.str();
    std::cout << "viewOnMovedObject: " << viewOnMovedObject << '\n';
    
    return 0;
}

/*
temp: 
bufview: Some content
anotherView: Some content
internalBufferMoved: Some content
viewOnMovedObject: 
*/
```

Notice in the above example, how efficiently views and moves are used in order to avoid expensive copy operations.

## Introducing the new `<spanstream>` header

The second proposal, [P0448R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0448r4.pdf), introduces a complete new header `<spanstream>` mainly with 4 class templates:
- `std::basic_spanbuf`
- `std::basic_ispanstream`
- `std::basic_ospanstream`
- `std::basic_spanstream`

Basically what we get are the usual 3 streams and an externally provided memory buffer for them. *spanstreams* do not own the internal buffer, hence the name *span*, which is a non-owning view on an array of items. Therefore re-allocation is also not possible. If you need dynamic reallocation, you need to use `stringstream` et al. A `stringstream` has no-copy access to its contents since C++20.

As a consequence of this hew header and the explained changes to `basic_stringbuf`, there is no more reason to keep the already deprecated `strstream` classes in the standard, and the *[depr.str.strstreams]* section is getting removed.

Not surprisingly a `basic_spanbuf` uses a span of a sort of character (`charT`) as an internal buffer. It's safe and cheap to provide access to it, as it requires no copy of the data. If you need an owning copy of the data, you can always convert the result of `span()` back to `basic_string<charT>` and as such copy it for yourself.

```cpp
#include <iostream>
#include <span>
#include <spanstream>
#include <cassert>

void printSpan(auto spanToPrint) {
    for (size_t i = 0; i < spanToPrint.size(); ++i) {
        std::cout << spanToPrint[i];
    }
}

void useSpanbuf() {
    std::array<char, 16> charArray;
    std::span<char, 16> charArraySpan(charArray);
    std::spanbuf buf;

    char c = 'a';
    for (size_t i = 0; i < 16; ++i) {
        charArraySpan[i] = c;
        ++c;
    }
    
    buf.span(charArraySpan);

    // we can easily print a span got from the buffer
    std::span bufview = buf.span();
    std::cout << "bufview: ";
    for (size_t i = 0; i < 16; ++i) {
        std::cout << bufview[i];
    }
    std::cout << '\n';
}

void useSpanstream() {
    std::array<char, 16> charArray;
    std::ospanstream oss(charArray);

    oss << "Fortytwo is " << 42;
    // copying the contents to a span
    std::string s{oss.span().data(),size_t(oss.span().size())};
    assert(s == "Fortytwo is 42");
}


int main() {
    useSpanbuf();
    useSpanstream();

    return 0;
}
```

## Conclusion

In this post, we had a brief overview of how the world of buffers and streams has changed in C++20 and C++23 thanks to Peter Sommerlad and his two proposals, [P0448R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0448r4.pdf), [P0408R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0408r7.pdf).

With [P0408R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0408r7.pdf) we get non-copying access to the internal buffer of a `std::basic_stringbuf` which is a significant efficiency increase.

With the acceptance of [P0448R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0448r4.pdf) and the introduction of *spanstreams*, we get a stream that as an internal storage can use a fixed-size pre-allocated buffer.

Special thanks to [Peter](https://sommerlad.ch/) who pointed out a couple of missing points and misunderstandings in the draft of this article and helped it become more informative.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!