---
layout: post
title: "C++23: bitwise operations"
date: 2024-1-17
category: dev
tags: [cpp, cpp23, bitset, byteswap]
excerpt_separator: <!--more-->
---
While C++ is getting increasingly expressive with each new standard, we must not forget its origins. It is inherently a low-level language which operates close to the hardware level and allows operations that languages such as Javascript cannot even express.

A part of providing low-level functionalities is to be able to work on a byte or even on a bit level. In this post, we are going to see what related features C++23 brings or modifies.

## `constexpr std::bitset`

To be fair, we already covered this earlier last year in [C++23: Even more constexpr](https://www.sandordargo.com/blog/2023/05/24/cpp23-constexpr). But as I'm using this feature to demonstrate the next one, I decided to mention it again.

[P2417R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2417r2.pdf) extends the `constexpr` interface of `std::bitset`. So far, only one of the constructors and `operator[]` was marked as `constexpr`. However, [since `std::string` can be `constexpr`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0980r1.pdf), all the internals - and therefore the full API - of `std::bitset` can be `constexpr`.

If you're unfamiliar with `std::bitset`, it represents the object you pass in as a sequence of bits. You have to pass the number of bits as a non-type template argument.

```cpp
#include <bitset>
#include <iostream>

int main()
{
    constexpr short i = 15;
    constexpr int numberOfBitsInInt = sizeof(i) * 8;
    std::cout << "i:" << i << ", i as binary: " << std::bitset<numberOfBitsInInt>(i) << '\n';
}
/*
i:15, i as binary: 0000000000001111
*/
```

It's more capable than that, it also offers several ways to query the individual bits or set them. These features are out of the scope of this article, but [you can check them out on C++ Reference](https://en.cppreference.com/w/cpp/utility/bitset).

## `std::byteswap`

`std::byteswap` is a new library function introduced by [P1272R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1272r4.html).

```cpp
namespace std {
  constexpr auto byteswap (integral auto value) noexcept;
}
```

As you can see from its signature, it only accepts integral values.

Personally, I don't find the name *"byteswap"* very informative, but maybe it's just me. I'm not working in the world of byte-level operations. Probably, I would have called it `reverse_bytes`. By writing that, you probably understood what this function does. It takes an integral and returns another instance of the same type, where the bytes are in the reverse order compared to the input.

The following code snippet ([godbolt](https://godbolt.org/z/hbr4eaKfq)) represents what is happening.

```cpp
#include <bit>
#include <bitset>
#include <iostream>

int main()
{
    int i = 1;
    int j = std::byteswap(i);
    constexpr int numberOfBitsInInt = sizeof(i) * 8;
    std::cout << "i:        " << i << ", i as binary: " << std::bitset<numberOfBitsInInt>(i) << '\n';
    std::cout << "j: " << j << ", j as binary: " << std::bitset<numberOfBitsInInt>(j) << '\n';
}
/*
i:        1, i as binary: 00000000000000000000000000000001
j: 16777216, j as binary: 00000001000000000000000000000000
*/
```

I think it's worth emphasizing that the reverse is not happening on the level of bits but on the levels of bytes. It's even clearer in the following example.

```cpp
#include <bit>
#include <bitset>
#include <iostream>

int main()
{
    constexpr char i = 1;
    constexpr char j = std::byteswap(i);
    constexpr int numberOfBitsInChar = sizeof(i) * 8;
    static_assert(numberOfBitsInChar == 8);
    std::cout << "i as binary: " << std::bitset<numberOfBitsInChar>(i) << '\n';
    std::cout << "j as binary: " << std::bitset<numberOfBitsInChar>(j) << '\n';
    static_assert(i == j);
}
```

## Conclusion

In this article, we had a look at C++23 changes affecting bit and byte manipulations. We had a reminder on `std::bitset` that received a full `constexpr` API. Then we learned about the new library function, `std::byteswap` that helps reversing the bytes - and not the bits - in an object.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!