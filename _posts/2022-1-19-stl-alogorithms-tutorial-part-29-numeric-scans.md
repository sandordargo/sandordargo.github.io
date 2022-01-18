---
layout: post
title: "The big STL Algorithms tutorial: \"numeric\" scans"
date: 2022-1-19
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we are going to talk about the 4 algorithms in the `<numeric>` header that we haven't discussed yet:
<!--more-->

- `exclusive_scan`
- `inclusive_scan`
- `transform_exclusive_scan`
- `transform_inclusive_scan`

They all end with `_scan`? But what do they scan? Let's have a closer look.

## `exclusive_scan`

`std::exclusive_scan` resembles a lot to `std::partial_sum` that we discussed [in the previous episode](). It takes an input range denoted by its beginning and its end, an output range defined by its beginning and an initial value for the summing.

*Exclusive* in the name means that the given *i*th element is excluded from the partial sum. To see this perfectly, we can have a look at the first element of the output which is the initial value instead of the first element of the input.

```cpp
#include <numeric>
#include <iostream>
#include <vector>

int main() {
    std::vector v{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    std::vector<int> partial_sums{};
    partial_sums.reserve(v.size());
    std::vector<int> exclusion_scan_results{};
    exclusion_scan_results.reserve(v.size());
    std::partial_sum(v.begin(), v.end(), std::back_inserter(partial_sums));
    std::exclusive_scan(v.begin(), v.end(), std::back_inserter(exclusion_scan_results), 0, std::plus<int>());
    std::cout << "partial_sum results   :";
    for (auto ps: partial_sums) {
        std::cout << ps << " ";;
    }
    std::cout << std::endl;
    std::cout << "exclusive_scan results:";
    for (auto ps: exclusion_scan_results) {
        std::cout << ps << " ";;
    }
    std::cout << std::endl;
}
/*
partial_sum results   :1 3 6 10 15 21 28 36 45 55 
exclusive_scan results:0 1 3 6 10 15 21 28 36 45 
*/
```

It's worth mentioning that before all the other parameters, `exclusive_scan` can take an execution policy.

## `inclusive_scan`
*Exclusive* meant that the given *i*th element is excluded from the partial sum, following this logic inclusive should mean that the element is included in the partial sum and that's right!

You might suspect it well, `partial_sum` and `inclusive_scan` often end up with the same results. Let's have a look!

```cpp
#include <numeric>
#include <iostream>
#include <vector>

int main() {
    std::vector v{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    std::vector<int> partial_sums{};
    partial_sums.reserve(v.size());
    std::vector<int> exclusion_scan_results{};
    exclusion_scan_results.reserve(v.size());
    std::vector<int> inclusive_scan_results{};
    inclusive_scan_results.reserve(v.size());
    std::partial_sum(v.begin(), v.end(), std::back_inserter(partial_sums));
    std::exclusive_scan(v.begin(), v.end(), std::back_inserter(exclusion_scan_results), 0, std::plus<int>());
    std::inclusive_scan(v.begin(), v.end(), std::back_inserter(inclusive_scan_results), std::plus<int>(), 0);
    std::cout << "partial_sum results   :";
    for (auto ps: partial_sums) {
        std::cout << ps << " ";
    }
    std::cout << std::endl;
    std::cout << "exclusive_scan results:";
    for (auto ps: exclusion_scan_results) {
        std::cout << ps << " ";
    }
    std::cout << std::endl;
    std::cout << "inclusive_scan results:";
    for (auto ps: inclusive_scan_results) {
        std::cout << ps << " ";
    }
    std::cout << std::endl;
}
/*
partial_sum results   :1 3 6 10 15 21 28 36 45 55 
exclusive_scan results:0 1 3 6 10 15 21 28 36 45 
inclusive_scan results:1 3 6 10 15 21 28 36 45 55 
*/
```

I find how `exclusive_scan` and `inclusive_scan` are defined is a bit misleading. Better to say, they don't follow the same logic.

They both have overloads when they take the input range defined by their beginning and end, plus the output range defined by their beginning. They both can take an execution policy in the **0th** position. So far, so good.

But while `exclusive_scan` can optionally take an initial value and a binary operation in this order, `inclusive_scan` takes these optional values in the other order, first the binary operation and then the initial value.

Is this on purpose to make sure that you call the algorithm you really intended or by accident, that is unknown to me.

## `transform_exclusive_scan`

`std::transform_exclusive_scan` is easy to understand once `std::exclusive_scan` is understood. It "sums" up all the elements of the input range and write the results to the output range. Exclusive means that the *i*th element is not included in the *i*th sum.

The main difference compared to `std::exclusive_scan` is that before the sum operation happens, all elements are transformed with a unary operation.

Another difference is that `std::transform_exclusive_scan` cannot default the initial value nor the binary operation of the summing. They must be defined.

In the following example, we are going to sum up all the elements after they were multiplied by 10.

```cpp
#include <numeric>
#include <iostream>
#include <vector>

int main() {
    std::vector v{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    std::vector<int> results{};
    results.reserve(v.size());
    std::transform_exclusive_scan(v.begin(), v.end(), std::back_inserter(results), 0, std::plus<int>(), [](int i) {return i*10;});
    for (auto r: results) {
        std::cout << r << " ";;
    }
    std::cout << std::endl;
}
/*
0 10 30 60 100 150 210 280 360 450 
*/
```

## `transform_inclusive_scan`

Based on `inclusive_scan` and `transform_exclusive_scan`, I'm sure we can deduce what `std::transform_inclusive_scan` does. It "sums" up all the elements of the input range after performing a transformation on them and write the results to the output range. Inclusive means that the *i*th element is also included in the *i*th sum.

On the other hand, after having seen the differences between `inclusive_scan` and `exclusive_scan`, I cannot assume anything about `transform_inclusive_scan`'s signature.

After the optional execution policy and the three iterators denoting the input and the output ranges, this algorithm takes a binary operation for the summing and a unary operation for the transformation and at the very end, an optional initial value.

`transform_inclusive_scan` is also constexpr.

Let's have a look at the same example as we used for `transform_exclusive_scan`, let's sum up integers after multiplying them by 10.

```cpp
#include <numeric>
#include <iostream>
#include <vector>

int main() {
    std::vector v{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    std::vector<int> results{};
    results.reserve(v.size());
    std::transform_inclusive_scan(v.begin(), v.end(), std::back_inserter(results), std::plus<int>(), [](int i) {return i*10;}, 0);
    for (auto r: results) {
        std::cout << r << " ";;
    }
    std::cout << std::endl;
}
/*
10 30 60 100 150 210 280 360 450 550 
*/
```

We can observe that the results are different as the *i*th elements are included in the results and that the order of parameters changed. For sure, you cannot accidentally mix up the two algorithms.

## Conclusion

This time, we learned about the different scan algorithms in the `<numeric>` header. With them, we can sum up the items of a container and have the rolling results in many different ways.

We finished discussing all the algorithms defined in the `<numeric>` header, next time we will discuss the `<memory>` header.

Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
