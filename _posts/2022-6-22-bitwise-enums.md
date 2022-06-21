---
layout: post
title: "Bitwise enumerations"
date: 2022-6-22
category: dev
tags: [cpp, enums, bitflags, bitwise]
excerpt_separator: <!--more-->
---
In C++ the size of a `bool` is 1 byte. That's the case despite that it can only have two values, `true` or `false` which can be represented on one single bit. This might not be a concern all the time, but it's for sure not optimal. There are different techniques in C++ to use that one byte better.
<!--more-->

## The idea of bitwise `enum`s

The idea of bit flags is to use each bit in a clever, yet relatively readable way. When the flags are encapsulated with an `enum`, they are called bitwise `enum`s.

What is behind the idea?

On one byte, we can store 256 different values. One byte is not only the size of a `bool`, but it's also the size of a `char`. The idea is to store 8 related boolean values on the 8 bits of a byte.

Technically, we could do this just by using a `char`.

```cpp
// flag 1: is automatic
// flag 2: is electric
// flag 4: is 4x4
// flag 8: has rooftop
// flag 16: GPS
char flags = 10; // 2 + 8 = 10

std::cout << std::boolalpha;
std::cout << static_cast<bool>(flags & 1) << '\n';
std::cout << static_cast<bool>(flags & 2) << '\n';
std::cout << static_cast<bool>(flags & 4) << '\n';
std::cout << static_cast<bool>(flags & 8) << '\n';
std::cout << static_cast<bool>(flags & 16) << '\n';
```

In this example, we see that we initialized our `flags` bitset with the combination of 2 and 8, so it represents an electric car with a rooftop. By using the *bitwise and operator* (`operator&`) we could check what is turned on. Of course, there are lots of magic values here, let's make it a bit better (pun intended).

```cpp
constexpr char isAutomaticFlag = 1;
constexpr char isElectricFlag = 2;
constexpr char is4x4Flag = 4;
constexpr char hasRooftopFlag = 8;
constexpr char hasGPSFlag = 16;
char flags = 10;

std::cout << std::boolalpha;
std::cout << static_cast<bool>(flags & isAutomaticFlag) << '\n';
std::cout << static_cast<bool>(flags & isElectricFlag) << '\n';
std::cout << static_cast<bool>(flags & is4x4Flag) << '\n';
std::cout << static_cast<bool>(flags & hasRooftopFlag) << '\n';
std::cout << static_cast<bool>(flags & hasGPSFlag) << '\n';
```

Now we use each flag by its name instead of its value. The initialization is still problematic. We can either use an addition there or it would be more idiomatic with the checking part (`operator&`) to use the *bitwise or operator* (`operator|`).

```cpp
char flags = isElectricFlag | hasRooftopFlag;
```
The problem that we should still solve is that while all these values are related, we don't communicate that well. Having meaningful names, pre- or postfixes are nice things, but it would be even better to encapsulate them. For encapsulating related values our best option is an `enum`!

## How to implement the scoped bitwise `enum`

As this article's first published in 2022, we should go with a *scoped `enum`* (a.k.a `enum class`)!

```cpp
enum class CarOptions : char {
    isAutomaticFlag = 1,
    isElectricFlag = 2,
    is4x4Flag = 4,
    hasRooftopFlag = 8,
    hasGPSFlag = 16
};
```

But there is a problem! Our code breaks because of two reasons. First, our flags need to be prepended with their scopes and we also have to change the type of the `flags` variable to `CarOptions`. That's easy peasy.

```cpp
CarOptions flags = CarOptions::isElectricFlag | CarOptions::hasRooftopFlag;
```

The problem is that it does not compile because there is no match for `operator|`. To fix that we have to get the underlying values of each option, apply the bitwise operation to them and use the obtained value to construct another value.

```cpp
CarOptions flags = CarOptions(static_cast<std::underlying_type<CarOptions>::type>(CarOptions::isElectricFlag) | static_cast<std::underlying_type<CarOptions>::type>(CarOptions::hasRooftopFlag));
```

That is long and ugly. Let's break it into two statements.

```cpp
using CarOptionsType = std::underlying_type<CarOptions>::type;
CarOptions flags = CarOptions(static_cast<CarOptionsType>(CarOptions::isElectricFlag) | static_cast<CarOptionsType>(CarOptions::hasRooftopFlag));
```

So first we get the underlying type of our enumeration. Though we could use simply `char` instead, this will always keep working, even if we change the underlying type of `CarOptions`. Then on the second line, we explicitly cast the flags we want to combine to their underlying types, we use `operator|` on them and then we initialize a new `CarOptions` with the obtained value. Just as before, but probably in a more readable way.

Lots of hassle and we are not done.

The checks with `operator&` do not work either!

Following a similar logic, at the end, we'd end up with checks like this:

```cpp
std::cout << static_cast<bool>(static_cast<CarOptionsType>(flags) & static_cast<CarOptionsType>(CarOptions::isAutomaticFlag)) << '\n';
```

This is definitely not acceptable. One option is to go with an unscoped `enum` where implicit conversions are allowed and we don't have to change anything in our code, it'd just work.

```cpp
#include <iostream>

enum CarOptions : char {
    isAutomaticFlag = 1,
    isElectricFlag = 2,
    is4x4Flag = 4,
    hasRooftopFlag = 8,
    hasGPSFlag = 16
};

int main() {
    char flags = CarOptions::isElectricFlag | CarOptions::hasRooftopFlag;
    
    std::cout << std::boolalpha;
    std::cout << static_cast<bool>(flags & CarOptions::isAutomaticFlag) << '\n';
    std::cout << static_cast<bool>(flags & CarOptions::isElectricFlag) << '\n';
    std::cout << static_cast<bool>(flags & CarOptions::is4x4Flag) << '\n';
    std::cout << static_cast<bool>(flags & CarOptions::hasRooftopFlag) << '\n';
    std::cout << static_cast<bool>(flags & CarOptions::hasGPSFlag) << '\n';
    
}
```

Given the dangers of unscoped `enums` let's think about another solution. What if we overloaded the necessary operators?

```cpp
#include <iostream>

enum class CarOptions : char {
    isAutomaticFlag = 1,
    isElectricFlag = 2,
    is4x4Flag = 4,
    hasRooftopFlag = 8,
    hasGPSFlag = 16
};

CarOptions operator|(CarOptions lhs, CarOptions rhs) {
    using CarOptionsType = std::underlying_type<CarOptions>::type;
    return CarOptions(static_cast<CarOptionsType>(lhs) | static_cast<CarOptionsType>(rhs));
}

CarOptions operator&(CarOptions lhs, CarOptions rhs) {
    using CarOptionsType = std::underlying_type<CarOptions>::type;
    return CarOptions(static_cast<CarOptionsType>(lhs) & static_cast<CarOptionsType>(rhs));
}

int main() {
    // flag 32: mutually exclusive with 8, has skibox
    CarOptions flags = CarOptions::isElectricFlag | CarOptions::hasRooftopFlag;
    
    std::cout << std::boolalpha;
    std::cout << static_cast<bool>(flags & CarOptions::isAutomaticFlag) << '\n';
    std::cout << static_cast<bool>(flags & CarOptions::isElectricFlag) << '\n';
    std::cout << static_cast<bool>(flags & CarOptions::is4x4Flag) << '\n';
    std::cout << static_cast<bool>(flags & CarOptions::hasRooftopFlag) << '\n';
    std::cout << static_cast<bool>(flags & CarOptions::hasGPSFlag) << '\n';
    
}
```
With a little bit of boilerplate, we managed to keep the original code. The only additional change is the scoping that is necessary due to the `enum` class.

It's worth noting that you don't necessarily want to use integer values in the enum. Since C++14, you might go with a binary format.

```cpp
enum class CarOptions : char {
    isAutomaticFlag = 0b1,
    isElectricFlag = 0b10,
    is4x4Flag = 0b100,
    hasRooftopFlag = 0b100,
    hasGPSFlag = 0b10000,
};
```

First I thought that maybe it's more difficult to introduce a typo as such, but I realized that I was mistaken. Indeed, you only have to pay attention that in each value there is only one `1`, but you can accidentally use the same values for multiple constants as I just did with `is4x4Flag` and `hasRooftopFlag`. Even `-Wall -pedantic -Wextra` didn't warn about that. So I'd argue that it's still easier to keep it correct with decimal values.

## How to have mutually exclusive flags?

So far we've seen how to handle many flags in one single byte. We can combine them and we can check what is turned on.

But what if we wanted to have mutually exclusive values. For example, it's hard to imagine a car that can have both a manual and an automatic air conditioner at the same time.

Of course, one could say that let's just not include it in the `CarOptions` and we could have a separate enum for that purpose and that would not be composed of bitflags. But let's say we really want to extend our `CarOptions` with mutually exclusive options. What can we do?

We already overloaded `operator|`, let's modify that.

```cpp
CarOptions operator|(CarOptions lhs, CarOptions rhs) {
    using CarOptionsType = std::underlying_type<CarOptions>::type;
    if ((static_cast<bool>(lhs & CarOptions::hasManualACFlag)) && (static_cast<bool>(rhs & CarOptions::hasAutomaticACFlag))) {
        throw std::invalid_argument("mutually exclusive values");
    }
    
    return CarOptions(static_cast<CarOptionsType>(lhs) | static_cast<CarOptionsType>(rhs));
}
```

The problem is that while this would throw an exception for `CarOptions mxFlags = CarOptions::hasManualACFlag | CarOptions::hasAutomaticACFlag;` it would pass for `CarOptions mxFlags2 = CarOptions::hasAutomaticACFlag | CarOptions::hasManualACFlag;`. 

The brute force approach is to add one more condition with the reversed logic.

```cpp
CarOptions operator|(CarOptions lhs, CarOptions rhs) {
    using CarOptionsType = std::underlying_type<CarOptions>::type;
    if ((static_cast<bool>(lhs & CarOptions::hasManualACFlag)) && (static_cast<bool>(rhs & CarOptions::hasAutomaticACFlag))) {
        throw std::invalid_argument("mutually exclusive values");
    }
    if ((static_cast<bool>(lhs & CarOptions::hasAutomaticACFlag)) && (static_cast<bool>(rhs & CarOptions::hasManualACFlag))) {
        throw std::invalid_argument("mutually exclusive values");
    }
    
    return CarOptions(static_cast<CarOptionsType>(lhs) | static_cast<CarOptionsType>(rhs));
}
```

While this works, it's repetitive, error-prone and doesn't scale. Imagine what would happen if we had 3 mutually exclusive fields. That would mean 6 different `if` statements to throw from!

We need a smarter solution!

For that, the best thing we can do is to rephrase what we want. We have a list of mutually exclusive flags. `opreator|` combines two options. We make sure that if they are different and both of them have mutually exclusive options, then we throw an exception. That's something easier to grasp.

```cpp
CarOptions operator|(CarOptions lhs, CarOptions rhs) {
    if (lhs == rhs) {
        return lhs;
    }
    using CarOptionsType = std::underlying_type<CarOptions>::type;
    std::array<CarOptions, 2> mxs {CarOptions::hasAutomaticACFlag, CarOptions::hasManualACFlag};
    const bool isLhsSet = std::any_of(mxs.begin(), mxs.end(), [lhs](CarOptions option) {
        return static_cast<bool>(lhs & option);
    });
    const bool isRhsSet = std::any_of(mxs.begin(), mxs.end(), [lhs](CarOptions option) {
        return static_cast<bool>(lhs & option);
    });
    if (isLhsSet && isRhsSet) {
        throw std::invalid_argument("mutually exclusive values");
    }
        
    return CarOptions(static_cast<CarOptionsType>(lhs) | static_cast<CarOptionsType>(rhs));
}
```

So we start with a guard statement making sure that if the two options are the same, we don't throw an exception. As a next step, we have the array of mutually exclusive options and then we check if both `lhs` and `rhs` have them turned on.

If we make the list of mutually exclusive fields an external dependency to `operator|`, we could even make it more dynamically configurable. But I let you implement it if you're interested.

## Conclusion

Today we saw how to use bit flags and how to implement bit flag enumerations. We also saw that if we want to keep up with the winds of change and we want to go with scoped enums (a.k.a. `enum class`es) then we'd better overload `operator|` and `operator&`. And that actually opens up more possibilities to go further and define mutually exclusive flags in the same `enum`.

Now it's over to you! Do you sometimes use bit flag enums? If so, what are your preferences?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!