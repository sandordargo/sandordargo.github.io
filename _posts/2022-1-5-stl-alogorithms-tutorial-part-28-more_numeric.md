---
layout: post
title: "The big STL Algorithms tutorial: more numeric algorithms"
date: 2022-1-5
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
It's high time to continue [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), and in this next part we are going to talk about 4 operations that are part of the `<numeric>` header: 
<!--more-->

- `iota`
- `inner_product`
- `partial_sum`
- `adjacent_difference`


## `iota`

`std::iota` was added to the `<numeric>` header with the first modern version of C++; C++11. Ever since then it hasn't changed much. The only modification is that since C++20 it is `constexpr`.

But what does it do, after all? The name doesn't help much - at least not me.

It iterates over a range that is denoted by two iterators (beginning and end) and also takes a value. It fills the first item with the passed in value and then for each iteration it increases it's value by (`++value`).

Here is an example:

```cpp
#include <iostream>
#include <numeric>
#include <vector>

int main(){
    std::vector myInt(10, 0);
    std::iota(myInt.begin(), myInt.end(), 42);
    for (auto i : myInt) {
        std::cout << i << ' ';
    }
    std::cout << '\n';
}
/*
42 43 44 45 46 47 48 49 50 51 
*/
```

As cryptic its name, as simple its behaviour is.

## `inner_product`

`std::inner_product` is a bit more complex function.

It has two overloads, and since C++20, both are `constexpr`.

In its simpler form, it takes 4 values. The first three are iterators and they denote two ranges. The first is identified by its beginning and its end and the second one is only by its beginning. It's up to the caller to ensure that it has as many elements as the second one.

The fourth parameter is a value is an initial value for the accumulation of the products.

Let's see a simple example:

```cpp
#include <algorithm>
#include <iostream>
#include <numeric>
#include <vector>

int main(){
    std::vector v1 {1, 2, 3};
    std::vector v2 {1, 2, 3};
    
    auto product = std::inner_product(v1.begin(), v1.end(), v2.begin(), 0);
    std::cout << product << '\n';
    
    std::reverse(v2.begin(), v2.end());
    auto product2 = std::inner_product(v1.begin(), v1.end(), v2.begin(), 0);
    std::cout << product2 << '\n';
}
/*
14
10
*/
```
`inner_product` takes the elements in the same positions of both ranges, takes their products and accumulate them.

Hence, when we call `inner_product` on two vectors with the same elements (1, 2, 3 in our example), it basically sums up the squares of elements => 1 \* 1 + 2 \* 2 + 3 \* 3 = 14.

When we reverse the second range, we calculate 1 \* 3 + 2 \* 2 + 3 \* 1 and we end up with 10 as a result.

There are other overloads where as the fifth and sixth parameters you can pass in two binary operations. The first one replaces the summing part and the second one replaces the multiplication.

This piece of code performs the very same thing as the previous example:

```cpp
#include <algorithm>
#include <iostream>
#include <numeric>
#include <vector>
#include <functional>

int main(){
    std::vector v1 {1, 2, 3};
    std::vector v2 {1, 2, 3};
    
    auto product = std::inner_product(v1.begin(), v1.end(), v2.begin(), 0, std::plus<>(), std::multiplies<>());
    std::cout << product << '\n';
    
    std::reverse(v2.begin(), v2.end());
    auto product2 = std::inner_product(v1.begin(), v1.end(), v2.begin(), 0, std::plus<>(), std::multiplies<>());
    std::cout << product2 << '\n';
}
/*
14
10
*/
```

## `partial_sum`

`std::partial_sum` is an interesting algorithm. What do you think it means without reading forward? Partially summing up a range doesn't make much sense as it's the caller who decides from when till then a sum (`std::accumulate`) should go.

`partial_sum` does something different. It starts summing up the elements from the left to the right and after each step, it writes the - running - result to an output range. As a first element, it doesn't output the sum of the first two elements, but simply the first element of the input range. As such, it ensures that the output range will have the same number of elements as the input. Otherwise, it would have `n-1` elements, where `n` is the size of the input range.

```cpp
#include <numeric>
#include <iostream>
#include <vector>

int main() {
    std::vector v{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    std::vector<int> partial_sums{};
    partial_sums.reserve(v.size());
    std::partial_sum(v.begin(), v.end(), std::back_inserter(partial_sums));
    for (auto ps: partial_sums) {
        std::cout << ps << " ";;
    }
    std::cout << std::endl;
}
/*
1 3 6 10 15 21 28 36 45 55
*/
```
In this example, we have a vector of numbers from 1 to 10 and in the output, first, we have 1, then 3 (1+2), then 6 (1+2+3), etc.

We also have the possibility to pass in a binary operation as the fourth parameter. We could replace our previous call, with `std::partial_sum(v.begin(), v.end(), std::back_inserter(partial_sums), std::plus<int>());` taking `std::plus<int>()` from the `<functional>` header and we'd get the same results, but of course, with the help of the custom binary operation we could change the behaviour.

## `adjacent_difference`

`std::adjacent_difference` is moving from item to item and saves the difference between the current and the previous item into the output range. In order that the output size matches the input size, the first item is copied to the output.

By default, `adjacent_difference` takes 3 iterators as inputs. The first two iterators denote the beginning and the end of the range to work on and the third iterator is the beginning of the output rage which must be able to accommodate as many elements as the original input range.

```cpp
#include <numeric>
#include <iostream>
#include <vector>

int main() {
    std::vector v{1, 3, 6, 10, 15, 21, 28, 36, 45, 55};
    std::vector<int> diffs{};
    diffs.reserve(v.size());
    std::adjacent_difference(v.begin(), v.end(), std::back_inserter(diffs));
    for (auto diff: diffs) {
        std::cout << diff << " ";;
    }
    std::cout << std::endl;
}
/*
1 2 3 4 5 6 7 8 9 10 
*/
```

In this example, we took the output of the previous `partial_sum` example, and we called `adjacent_difference` on them. With that, we got back the original input of the `partial_sum` example. 1 is simply copied, then 3-1=>2, 6-3=>3, and so on.

Once again, we have the possibility to customize the binary operation, which is `std::minus` by default:

```cpp
#include <functional>
#include <numeric>
#include <iostream>
#include <vector>

int main() {
    std::vector v{1, 3, 6, 10, 15, 21, 28, 36, 45, 55};
    std::vector<int> diffs{};
    diffs.reserve(v.size());
    std::adjacent_difference(v.begin(), v.end(), std::back_inserter(diffs), std::minus<>());
    for (auto diff: diffs) {
        std::cout << diff << " ";;
    }
    std::cout << std::endl;
}
```

## Conclusion

This time, we continued to explore the `<numeric>` header and learnt about 4 algorithms; `iota`, `inner_product`, `partial_sum` and `adjacent_difference`. There are still 4 algorithms in this header that we have not discussed yet, all of them ending with `*_scan`. We'll explore them next time.

Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!