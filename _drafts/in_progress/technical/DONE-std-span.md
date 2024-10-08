---
layout: post
title: ""
date: 2024-X-X
category: dev
tags: [cpp, builds, staticlinking, dynamiclinking]
excerpt_separator: <!--more-->
---
While reading the awesome book C++ Brain Teasers by Anders Schau Knatten, I realized it might be worth to write about spans.

`std::span` is a class template that was added to the standard library in C++20 and you'll find it in the `<span>` header. A `span` is a non-owning object that can refer to a contiguous sequence of objects with the first element of the sequence at position zero.

In its goal, a `span` is quite similar to a `string_view`. While a `string_view` is a non-owning view of string like objects, a `span` is also a non-owning view for array-like objects where the stored elements occupy a contiguous places in memory.

While it's possible to use `span`s with `vector`s and `array`s, most frequently it will be used with C-style arrays because a span gives you safe access to its elements annd also to the size of the view, something that you don't get with C-style arrays.

When and why does it come really handy?

Let me steal an example from C++ Brain Teasers, but we'll go with another solution compared to the one in the book.

```cpp
#include <iostream>

void serialize(char characters[]) {
    std::cout << sizeof(characters) << "\n";
}

int main() {
    char characters[] = {'a', 'b', 'c'};
    std::cout << sizeof(characters) << "\n";
    std::cout << sizeof(characters) / sizeof(characters[0]) << "\n";
    serialize(characters);
}
```

In the above piece of code, `serialize` takes an array of characters. When we define the array of characters in `main()`, we can use `sizeof` to prints the size of array. Well, we actually print how many bytes the `characters[]` array occupy. Let me demonstrate.

```cpp
char characters[] = {'a', 'b', 'c'};
std::cout << sizeof(characters) << "\n";
/*
3
*/
```

When we try to print the size of a `char` array all seems fine. We expect 3 and the output is three. But use another type, like an `int` and we see there is a problem:

```cpp
int ints[] = {1, 2, 3};
std::cout << sizeof(ints) << "\n";
/*
12
*/
```

The output is 12, because we printed the memory size the array needs and that's 3 times of a size of an `int` in this case. As an `int` on my system is 4 bytes, the output is 3 * 4 bytes, that is 12. As the size of a `char` is 1 byte the memory size of the array and the number of elements in it are the same.

If you want to know how many elements are there in a `c`-style array of any type, you have to use this good old verbose and cumbersome pattern:

```cpp
std::cout << sizeof(characters) / sizeof(characters[0]) << "\n";
std::cout << sizeof(ints) / sizeof(ints[0]) << "\n";
/*
3
3
*/
```

Dividing the size of the array with the size of the first item will always work.

Well, not always.

In the above examples we had the arrays declared in the same scope - or at least we assumed that they are declared there.

But if the array is a function parameter, our assumptions break down. Let's have a look at the following example.

```cpp
#include <iostream>
#include <span>

void serialize(char characters[]) {
    std::cout << sizeof(characters) << "\n";
    std::cout << sizeof(characters) / sizeof(characters[0]) << "\n";
}

void serialize(int ints[]) {
    std::cout << sizeof(ints) << "\n";
    std::cout << sizeof(ints) / sizeof(ints[0]) << "\n";
}

int main() {
    int ints[] = {1, 2, 3};
    char characters[] = {'a', 'b', 'c'};
    serialize(characters);
    serialize(ints);
}
/*
8
8
2
2
*/
```

The outputs are broken both the size of the arrays and the number of items in them. The reason is that when a function takes a C-style array as an argument, the array is implicitly converted into a pointer. This is also called array decay.

From a usage persective, you still means that you can still access individual elements, but we lost any means to compute its size because the size of the parameter is not the size of the array anymore, simply the size of a pointer point to the first element of the array.

That's why we can often observe in C-style APIs that along an array its size is also passed.

With `std::span` we don't need that anymore.

As a `std::span` is a proper (non-owning) object, it doesn't decay to a pointer. On the other hand, a C-style array can be implicitly converted into a `span`. A span gives you access to the number of elements in it (without having to do a verbose and error-prone calculation), it gives you an easy way to access the items in the span and it's also iterable.

```cpp
#include <iostream>
#include <span>

void serialize(std::span<char> characters) {
    std::cout << characters.size() << "\n";
    for(size_t i = 0; i < characters.size(); ++i) {
        std::cout << characters[i] << " ";
    }
    std::cout << '\n';
    for (const auto c: characters) {
        std::cout << c << " ";
    }
    std::cout << '\n';
}

int main() {
    char characters[] = {'a', 'b', 'c'};
    serialize(characters);
}
```

As a general rule of thumb, I'd recommend not using C-style arrays, but if you have no choice, use spans as function parameters to make it easier and safer to work with them.

## Conclusion

C-style array are still used, mostly when you have to deal with C-libraries. They come with significant limitations, particularly when passed to functions where array decay occurs, leading to loss of size information.

`std::span`, introduced in C++20, solves this issue by providing a safe, non-owning view of contiguous data, retaining the size and offering easy access to elements. It simplifies working with arrays in functions without needing additional parameters for size, making code safer and more concise.

Whenever possible, it's advisable to replace C-style arrays with spans for more robust and maintainable code.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!