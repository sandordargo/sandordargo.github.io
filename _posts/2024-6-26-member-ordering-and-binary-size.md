---
layout: post
title: "Member ordering and binary sizes"
date: 2024-6-26
category: dev
tags: [cpp, cpp23, namespace, codeorganization]
excerpt_separator: <!--more-->
---
While I have been preparing my presentation for [C++ On Sea](https://cpponsea.uk/), I realized that something is missing from [How to keep your binaries small](https://cpponsea.uk/2024/session/how-to-keep-your-binaries-small). The importance of member ordering!

I remember learning at a performance tuning workshop that the order of member variables can significantly impact the memory layout and size of objects. Considering this factor, you can make your class more cache-friendly to increase runtime performance. This matters mostly when you plan to store a class in big numbers in a container.

But what about binary sizes?

Before that, let's first discuss what padding means.

## Padding and alignment

When it comes to the memory layout of data structures, padding refers to the extra bytes inserted between member variables of a data structure to satisfy alignment requirements. These requirements depend on your platform.

On most systems, 4-byte integers must start at a memory address that is a multiple of 4. Similarly, 8-byte `double`s have to start at addresses that are a multiple of 8. That's what we call alignment.

Padding is the extra space between the variables that helps satisfy the alignment requirements. Alignment is important for optimizing access speed.

Let's take the following example.

```cpp
struct UnoptimalOrder {
    int i1;     // 4 bytes
    double d1;  // 8 bytes
    int i2;     // 4 bytes
};

static_assert(sizeof(UnoptimalOrder) == 24);
```

The assertion is true. Even though we have two `int`s of 4 bytes and one `double` of 8 bytes, the size of `UnoptimalOrder` is not the sum. It's not 16 bytes, but 24. The reason is what I explained just earlier: 8-byte `double`s have to start at addresses that are a multiple of 8. Therefore `i1` goes to address 0, then it's followed by 4 bytes of padding so that `d1` can be placed on an address that is the multiple of 8. At the end, `i2` follows along and some padding so that the size of the whole struct is a multiple of 8.

If it would contain types that are 4 bytes large at maximum, it could also be a multiple of 4 bytes.

## Order elements by size to reduce padding

To potentially gain some space in terms of binary size, you have to reduce the size of padding bytes.

The easiest and best rule of thumb to follow is to order the members by decreasing size. If we consider the above example, we should start with the biggest member of type `double`, followed by the two integers.

As such we can eliminate all the padding and decrease the size to 16 bytes. Let's also rename the class to `OptimalOrder`.

```cpp
struct OptimalOrder {
    double d1;  // 8 bytes
    int i1;     // 4 bytes
    int i2;     // 4 bytes
};

static_assert(sizeof(OptimalOrder) == 16);
```

## It won't always reduce binary size

If you remember back to the article on object initialization, we saw that often the compiler is smart enough to optimize things and simply use `.zerofill` to reserve a big enough space. Even if you create an array of 10k objects with static storage duration, the order will not change the size of your binary if everything is default initialized.

On the other hand, if one of the members has a value other than (something implicitly convertible to) 0, ordering will matter.

The below two examples are both compiled to a binary of 16,888 bytes on my machine.

With good ordering:

```cpp
#include <array>

class Example {
public:
    double a;// = 4.2;   // 8 bytes
    int b;// = 1;      // 4 bytes
    float c;    // 4 bytes
    char d;     // 1 byte
    bool e;     // 1 byte
    bool f;     // 1 byte
    // Assuming typical alignment, 'a' (8 bytes) should be first,
    // followed by 'b' and 'c' (both 4 bytes), and then 'd' and 'e' (1 byte each).
};

std::array<Example, 10'000> arr {};
static_assert(sizeof(Example) == 24);

int main() {
    return 0;
}
```

With bad ordering:

```cpp
#include <array>

struct Example {
    int b;//=1;      // 4 bytes
    char d;     // 1 byte
    float c;    // 4 bytes
    bool e;     // 1 byte
    double a;// = 4.2;   // 8 bytes
    bool f;
};

std::array<Example, 10'000> arr{};

static_assert(sizeof(Example) == 32);

int main() {
    return 0;
}
```

We can observe that the size of `Example` is 24 bytes in one case, but 32 in the other. Yet the binary size is the same.

But as soon as we add some initial values to the members, for example, `1` to `b` and `4.2` to a, there is a significant size difference. With the good ordering, the size is 264k while with the bad ordering it's 347k.

That's a big diff! But not as big as the difference between the version with 0 initial values and some others.

## Conclusion

Today we discussed member ordering. We saw that in order to minimize padding, we should order members in a decreasing order of their size.

At the same time, even an unoptimal ordering will not change the binary size, if you zero initialize members. So the best thing you can do for binary size is to use default values for members which are implicitly convertible to zero. The second best thing to do is pay attention to member ordering. But that's something to pay attention to anyway for cache friendliness.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!