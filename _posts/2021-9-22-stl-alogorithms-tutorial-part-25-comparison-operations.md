---
layout: post
title: "The big STL Algorithms tutorial: comparison operations"
date: 2021-9-22
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we are going to talk about three comparison operations:
<!--more-->

* `equal`
* `lexicographical_compare`
* `lexicographical_compare_three_way`

## `equal`

`std::equal` compares two ranges to each other and returns `true` if the ranges are equal, `false` otherwise.

There are mainly two different overloads of `std::equal`, but as each of them can be `constexpr` (since C++20), and all of them can be parallelized by passing an `ExecutionPolicy` as a "0th" parameter (since C++17) and a binary predicate as the last parameter in order to replace the default `operator==`, there are many different overloads.

So what are the different overloads?

The first one accepts three iterators. The first two iterators define the first range by its first and last element and the third iterator is to show where the second range starts.

In this case, the caller must make sure that after the third iterator there are least as many elements as there are in the range defined by the first two iterators. Otherwise, it's undefined behaviour.

The other overload takes four iterators, where the second pair fully defines the second range used in a comparison, and it's available since C++14.

It seems like a nice idea, first I was thinking that I might check whether the second range is the same size as the first one. But it's not the case.

On the other hand, let's say you have a larger range that you want to pass in the second position. With the 3 parameter version, `std::equal` will check if the first range is the subrange of the second one, which might mean equality. With the *"full"* version where you define both ranges by their beginning and end, you really check for the equality of two ranges.

Let's see it in an example.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector nums {1, 2, 3, 4, 5};
    std::vector fewerNums {1, 2, 3, 4};
    
    std::cout << std::boolalpha;
    std::cout << std::equal(nums.begin(), nums.end(), fewerNums.begin()) << '\n';
    std::cout << std::equal(fewerNums.begin(), fewerNums.end(), nums.begin()) << '\n';
    std::cout << std::equal(nums.begin(), nums.end(), fewerNums.begin(), fewerNums.end()) << '\n';
    std::cout << std::equal(fewerNums.begin(), fewerNums.end(), nums.begin(), nums.end()) << '\n';
}
/*
false
true
false
false
*/

```

## `lexicographical_compare`

`std::lexicographical_compare` checks whether the first range is lexicographically less, smaller than the second range using the `operator<` unless the caller passes in a different comparison function.

Both of the two ranges are defined by their `begin()` and `end()` iterators and as mentioned you can pass in custom comparator and of course an execution policy.

But what is lexicographical comparison?

A [lexicographical comparison](https://en.wikipedia.org/wiki/Lexicographic_order) is basically an alphabetical ordering where two ranges are compared sequentially, element by element:

- if there is any mismatch, that defines the result
- if one range is a subrange of the other, the shorter range is *"less"* than the other
- an empty range is always *"less"* than the other

The returned value is `true` if the first range is "less" than the other, otherwise, we get `false`.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector nums {1, 2, 3, 4, 5};
    std::vector fewerNums {1, 2, 3, 4};
    std::vector<int> empty{};
    
    std::cout << std::boolalpha;
    std::cout << std::lexicographical_compare(nums.begin(), nums.end(), fewerNums.begin(), fewerNums.end()) << '\n';
    std::cout << std::lexicographical_compare(fewerNums.begin(), fewerNums.end(), nums.begin(), nums.end()) << '\n';
    std::cout << std::lexicographical_compare(nums.begin(), nums.end(), nums.begin(), nums.end()) << '\n';
    std::cout << std::lexicographical_compare(empty.begin(), empty.end(), nums.begin(), nums.end()) << '\n';
    std::cout << std::lexicographical_compare(empty.begin(), empty.end(), empty.begin(), empty.end()) << '\n';
}
/*
false
true
false
true
false
*/
```

## `lexicographical_compare_three_way`

If you feel that it's impractical to get a `true`/`false` result for a comparison whereas there could be 3 outcomes (less, greater or equal), you should use `std::lexicographical_compare_three_way`- given that you work with a compiler supporting C++20.

By default it returns one of the constants of [`std::strong_ordering`](https://en.cppreference.com/w/cpp/utility/compare/strong_ordering), but it can also return `std::weak_ordering` or `std::partial_ordering` depending on the return type of the custom comparator that you can also define. The default comparator is [std::compare_three_way](https://en.cppreference.com/w/cpp/utility/compare/compare_three_way).

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

std::ostream& operator<<(std::ostream& out, std::strong_ordering ordering) {
    if (ordering == std::strong_ordering::less) {
        out << "less than";
    } else if (ordering == std::strong_ordering::equal) {
        out << "equal";
    } else if (ordering == std::strong_ordering::greater) {
        out << "greater than";
    }
    return out;
}

int main() {
    std::vector nums {1, 2, 3, 4, 5};
    std::vector fewerNums {1, 2, 3, 4};
    std::vector<int> empty{};
    
    std::cout << std::boolalpha;
    std::cout << std::lexicographical_compare_three_way(nums.begin(), nums.end(), fewerNums.begin(), fewerNums.end()) << '\n';
    std::cout << std::lexicographical_compare_three_way(fewerNums.begin(), fewerNums.end(), nums.begin(), nums.end()) << '\n';
    std::cout << std::lexicographical_compare_three_way(nums.begin(), nums.end(), nums.begin(), nums.end()) << '\n';
    std::cout << std::lexicographical_compare_three_way(empty.begin(), empty.end(), nums.begin(), nums.end()) << '\n';
    std::cout << std::lexicographical_compare_three_way(empty.begin(), empty.end(), empty.begin(), empty.end()) << '\n';
}
```

As you can see, the possible outcomes are not printable, you have to convert them manually into something that can be streamed to the output stream.

When you think about non-equal results, they are always relative to the first range. The first is *greater* or *less* than the second.

## Conclusion

This time, we learned about comparisons algorithms. They help us to compare ranges of elements. With `std::equal` we can compare if two ranges are equal or not and with `std::lexicographical_compare` or `std::lexicographical_compare_three_way` we can perform lexicographical comparison.

Next time we will discover permutation operations.

Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
