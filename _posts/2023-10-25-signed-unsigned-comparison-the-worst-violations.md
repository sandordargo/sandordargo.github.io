---
layout: post
title: "My battle against signed/unsigned comparison: the worst violations"
date: 2023-10-25
category: dev
tags: [cpp, cpp20, cleancode, integercomparison]
excerpt_separator: <!--more-->
---
We spent the last two weeks discussing the dangers of signed/unsigned comparisons. First, we discussed why it's a problem in the first place, and [how we can safely compare signed and unsigned types to each other](https://www.sandordargo.com/blog/2023/10/11/cpp20-intcmp-utilities), then I shared with you which are [the most common ways `-Wsign-compare` is violated](https://www.sandordargo.com/blog/2023/10/18/signed-unsigned-comparison-the-most-usual-violations).

This week, let me share with you the strangest violations. The worst offenders, the things I didn't want to see.

*(Obviously I simplified and anonymized the examples)*

## Comparing enumerations with the wrong sign

The following example is bleeding from several wounds.

```cpp
#include <iostream>
#include <random>

enum TheChoice {
    A,
    B,
    C,
    D,
    EMPTY_CHOICE = -1
};


unsigned int choose() {
    std::random_device dev;
    std::mt19937 rng(dev());
    std::uniform_int_distribution<std::mt19937::result_type> dist4(0,3);
    return dist4(rng);
}

int main() {
    if (choose() == EMPTY_CHOICE) {
        std::cout << "Oh boy, do we have some problem?\n";
    }
    // ...
    return 0;
}
```

We have an `enum` which is not a modern `enum class` and one of the enumerators has the value `-1` to represent a kind of error state.

Then we have a function that returns an `unsigned int`. 

And then the `enum` is compared to the `unsigned int`. Let alone that they can never be equal, that also generates a compiler warning.

Ideally, you would fix this by `choose()` returning an enumerator and it'd be also nice to turn the `TheChoice` to an `enum class`.

If that doesn't work, you could still match the types, either by modifying `TheChoice` so that its underlying value is an unsigned int and with that `EMPTY_CHOICE` must get a new value or by updating the return type of `choose()` to an `int`.

If none of those is an option, you can fall back to the new integer comparison utilities. But as you cannot use `std::cmp_*` to compare an `int` and an `enum`, you must cast the enum to its underlying type.


```cpp
if (std::cmp_equal(choose(), 
                   static_cast<std::underlying_type_t<TheChoice>>(EMPTY_CHOICE))) 
```

You could also simply `static_cast` to `int` without accessing the underlying type with `std::underlying_type_t`.

## Relying on narrowing an unsigned value

The following one took me some time even understand why it can actually work.

```cpp
#include <iostream>
#include <memory>
#include <utility>


class Resource {
 public:
    long /* originally ssize_t */ getSize() const {
        return 42;
    } 
 private:
    // ...
};

std::unique_ptr<Resource> getResource() {
    return std::make_unique<Resource>();
}


int main() {
    auto resource = getResource();
    if (!resource) {
        return 0;
    }
    if (const auto size = resource->getSize(); size != static_cast<size_t>(-1)) {
        // perform something        
    }
    // ...
    return 0;
}
```

Let's concentrate on `main()`. What on Earth goes on here?! First, we get a resource that is a - unique - pointer. Then within the `if` statement's condition we first get the size of the resource and save it in the `size` variable (we can do this since C++17), and then we compare it to `-1` after it's statically casted to an unsigned value.

First I thought that this comparison would always be true as the two sides cannot be equal but I was wrong. 

Let's break this down.

The variable `size` has the type of `const long` - that's usually the type behind `ssize_t`. On the other side, we have `-1` that is casted to a `size_t`. That is `std::numeric_limits<size_t>::max()` which is the same as `std::numeric_limits<unsigned long>::max()`.

The compiler will implicitly cast the `long` on the left side to an `unsigned long` (`size_t`). `long` and `unsigned long` have the same ranks when it comes to integer promotions, but the rule says that when "the unsigned type has conversion rank greater than or equal to the rank of the signed type, then the operand with the signed type is implicitly converted to the unsigned type."

Overall, it's possible that on both sides we have -1 converted to `unsigned long`.

Quite overcomplicated.

Well, that's life.

Now how can we get rid of this mayhem?

The simplest answer is to just compare a signed value to another signed value!

```cpp
if (const auto size = resource->getSize(); size != -1)
```

This will be just good enough.

But I was wondering why someone wrote such code?

Originally, the code looked like this - I omit the class and function definitions:

```cpp
int main() {
    auto resource = getResource();
    auto size = resource ? resource->getSize() : -1;
    if (size != static_cast<size_t>(-1)) {
        // perform something        
    }
    // ...
    return 0;
}
```

Before C++17, `size` had to be declared and initialized before the `if` statement. It was the result of a ternary and its type was `long` as both `-1` and `resource->getSize()` return a signed value and `ssize_t` has a higher rank so `-1` would be promoted to a `long`.

`size` could be set to `-1` in case the resource is not available. In other words, we use `-1` to handle an error case, the unavailability of a resource. Not the best solution, but it's nothing horrendous.

After C++17, the declaration of size was pulled into the conditional and a guard clause was added to return early if `resource` is not available.

The reason why `-1` was left there is because `getSize()` returned a `ssize_t`. But after further analysis, it turned out that it never returned a signed value, it actually always returned something that would fit `size_t`.

The correct solution was to change the return type of `getSize()` and remove the condition completely as the guard clause already covered the error case.


```cpp
#include <iostream>
#include <memory>
#include <utility>


class Resource {
 public:
    long /* originally ssize_t */ getSize() const {
        return 42;
    } 
 private:
    // ...
};

std::unique_ptr<Resource> getResource() {
    return std::make_unique<Resource>();
}


int main() {
    auto resource = getResource();
    if (!resource) {
        return 0;
    }
    // perform something that was previously conditional
    // ...
    return 0;
}
```

The moral of the story? When you modernize code, don't just blindly do it, but have a deeper look, because you might be able to further simplify it.

## Explicit C-cast to the wrong type

This last one is very easy to fix and the code becomes more readable.

Check this piece of code:

```cpp
#include <iostream>

class Container {
    public:
    int length() const {
        return 42;
    }
};

int main() {
   Container c;
   int someOffset = 39;
   for (int i = 0; i < c.length(); ++i) {
     if ((unsigned int)(i + someOffset) < c.length()) {
        std::cout << "We are still below c.length()\n";
     }
   }
   return 0;
}
```

In the body of the `for`-loop there is a conditional where `i + someOffset` is casted to an `unsigned int`. While it's problematic in 2023 to use a C-style cast, that's not our biggest issue here. The type of that expression is `int` and on the other side of the comparison there is a function call that also returns an `int`. There is no reason for that cast so the fix is as easy as removing it.

```cpp
// ...
     if ((i + someOffset) < c.length()) {
// ...
```

As you can see, I didn't even bother to remove the parentheses around `i + someOffset` because I think it enhances readability.

I was thinking for a while why some code like that might have been introduced. My best guess is that `Container::length()` returned `unsigned int` or `size_t` one day but when it was updated for whatever reason, the usages were not checked.

## Conclusion

In this third and last article of the signed/unsigned comparison mini-series, I shared with you the three strangest cases of `-Wsign-compare` violations.

While `std::cmp_*` often comes in handy, most of the time there is a better solution. Often we can simplify and correct the code which is always preferable as long as the solution is correct.

Do you care about `-Wsign-compare`? If so, how do you deal with it?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!