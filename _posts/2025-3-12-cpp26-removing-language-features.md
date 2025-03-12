---
layout: post
title: "C++26: Removing language features"
date: 2025-3-12
category: dev
tags: [cpp, cpp26, remove, deprecate]
excerpt_separator: <!--more-->
---
Probably you all heard that C++ is an ever-growing language - I wrote so many times as well. Each standard indeed comes with a great bunch of highly-anticipated features. At the same time, due to binary compatibility considerations, very few old features are removed. This has several implications:
- we have probably more than one way to do something
- the standard keeps growing

This is true, but let's not forget that each new standard removes some features. In this post, let's review what are the language features that are removed in C++26 and in a later post, we'll have a look at the removed library features.

> At this point, it's worth mentioning that a removal from the language usually happens in two steps. First a feature gets deprecated, meaning that its users would face compiler warnings for using deprecated features. As a next step, which in some cases never comes, the compiler support is finally removed.

## Remove Deprecated Arithmetic Conversion on Enumerations From C++26

[P2864R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2864r2.pdf) removes the ability to implicitly convert enumeration values in arithmetic conversions.

If you check sections 7, 8 and 9, you'll see how difficult it is to get any kind of consensus to make the language leaner by removing something old. Sometimes after deprecation and bad experience, you even have to consider reinstating deprecated features.

In this case, the features are finally removed. From C++26, expressions are ill-formed where one operand is of an enumeration type and the other operand is of a different enumeration or a floating-point type.

I can hardly imagine writing any of the following lines, but surely, in some situation certain people felt inclined to do so.


```cpp
int main() {
    enum E1 { e };
    enum E2 { f };
    bool b = e <= 3.7; // no more language support
    int k = f - e; // no more language support
}
```

Well, if you still want to make such expressions compile (probably you shouldn't), you can promote an enum value to an integer with the unary `operator+` and there you go...

```cpp
int main() {
    enum E1 { e };
    enum E2 { f };
    bool b = +e <= 3.7; // a not so nice quick fix
    int k = +f - e; // please do not do that
}
```

Clang 18 and GCC 14 implement this removal.

## Remove Deprecated Array Comparisons from C++26

[P2865R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2865r6.pdf) removes C-style array comparisons from C++ that were already deprecated in C++20 with the arrival of the spaceship operator.

While with the spaceship operator we can correctly compare - among others - arrays and their contents, with the comparison operator, due to some array-to-pointer decay, memory addresses have been compared. 

```cpp
#include <iostream>

int main() {
    int a[5] = {0, 1, 2, 3, 4};
    int b[5] = {0, 1, 2, 3, 4};

    if ( a == b ) {  // deprecated since C++20, ill-formed since C++26 
        std::cout << "a and b have the same address\n";        
    } else {
        std::cout << "a and b might have the same values, but their contents are different\n";
    }
    
}
/*
a and b might have the same values, but their contents are different
*/
```

If this isn't/wasn't bad enough, for greater or less than comparisons the results are often unspecified. But even if it's defined for a specific implementation, there is a fair chance that it's not what the code author wanted to do.

These comparisons are ill-formed since C++26, but array-to-pointer conversions are still possible if one of the two operands is a pointer. Meaning that the following is not great, but valid code:

```cpp
#include <iostream>

int main() {
    int a[5];
    int* b = a;

    if ( a == b ) {
        std::cout << "a and b have the same address\n";        
    } else {
        std::cout << "a nd b might have the same values, but their contents are different\n";
    }
}
```

Besides, this also means that you can compare arrays directly against nullptrs.

## Conclusion

Even though C++ is growing, from almost every new standard there are a couple of features that are removed. In C++26, two language features are being removed. One is arithmetic conversion from enumerations and the other is comparisons of C-style arrays. I think both these do not simply simplify the language, but also make affected code bases cleaner.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
