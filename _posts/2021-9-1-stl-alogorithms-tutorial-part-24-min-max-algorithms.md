---
layout: post
title: "The big STL Algorithms tutorial: Minimum/maximum operations"
date: 2021-9-1
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we are going to talk about minimum and maximum operations:
<!--more-->

* `max`
* `max_element`
* `min`
* `min_element`
* `minmax`
* `minmax_element`
* `clamp`

## `max` / `min`

`std::max` and `std::min` have a couple of different forms, all will essentially return the greatest or smallest elements:
- You might pass in two elements taken by `const` reference, and you'll get back a `const&` of the largest/smallest element
- You might pass in an initializer list and you'll get back a copy of the largest/smallest element
- Either way, you can pass in an optional comparator. In its absence, `operator<` will be used.

If all the passed in elements are equal, the leftmost one will be returned - both for `std::max` and `std::min`

```cpp
#include <algorithm>
#include <iostream>

int main() {
  int a = 42;
  int b = 51;
  int c = 66;
  int d = c;
  std::vector v{42, 51, 66};
  std::cout << std::max(a, b) << '\n';
  std::cout << std::min(a, b) << '\n';
  std::cout << std::max(c, c) << '\n';
  std::cout << std::min(c, c) << '\n';
  // std::cout << std::max(v) << '\n'; // ERROR: std::vector is not derived from std::initializer_list
  // std::cout << std::min(v) << '\n'; // ERROR: std::vector is not derived from std::initializer_list
  std::cout << std::max({a, b, c, d}) << '\n';
  std::cout << std::min({a, b, c, d}) << '\n';
}
/*
51
42
66
66
66
42
*/
```

It's worth noting that a `vector`, or other standard containers are not derivations of an initializer list, therefore you cannot pass them to `std::max`/`std::min`. For that, you have to use `max_element`/`min_element`.

## `max_element` / `min_element`

While `std::max` and `std::min` either take two values or an initializer list, `std::max_element` and `std::min_element` operates on a range. They resemble more to the standard algorithms we've seen in this series, notably:
- They take two iterators denoting the beginning and the end of a range
- They take an optional comparator, and when it's not specified `operator<` is used
- As an optional 0th parameter, you can pass in an execution policy

The return value will always be an iterator to the largest or smallest element. Interestingly, both `max_element` and `min_element` returns the leftmost element in case of equal elements are passed in.

```cpp
#include <algorithm>
#include <iostream>

int main() {
  std::vector v{42, 51, 66};
  std::cout << *std::max_element(v.begin(), v.end()) << '\n'; 
  std::cout << *std::min_element(v.begin(), v.end()) << '\n'; 
}
/*
66
42
*/
```

## `minmax`

What if you need both the smallest and the largest element of a container? You don't need to call `min` and `max` separately, you can simply call `std::minmax` and it will return a `std::pair` of the smallest and the largest value.

It's interesting to mention that in the case of equality both `std::min` and `std::max` return the leftmost element, `std::minmax` will return you two different elements all the time (except if you call it an initializer list of one element).

The algorithm has different forms following `std::min` and `std::max`:
- You might pass in two elements taken by `const` reference, and you'll get back a `const&` of the largest/smallest element
- You might pass in an initializer list and you'll get back a copy of the largest/smallest element
- Either way, you can pass in an optional comparator. In its absence `operator<` will be used.

```cpp
#include <algorithm>
#include <iostream>

int main() {
  int a = 42;
  int b = 51;
  int c = 66;
  auto minmax_ab = std::minmax(a,b);
  std::cout << minmax_ab.first << " " << minmax_ab.second << '\n';
  auto minmax_cc = std::minmax(c,c);
  std::cout << minmax_cc.first << " " << minmax_cc.second << '\n';
}
/*
42 51
66 66
*/
```

## `minmax_element`

Based on the previous section you probably already deduced what `std::minmax_element` does and how it works.

It works on containers and returns a pair of iterators to the smallest and largest elements of that container. In case, all the elements are equal, the smallest will be the leftmost one and the largest is the rightmost.

- It takes two iterators denoting the beginning and the end of a range
- It takes an optional comparator, and when it's not specified `operator<` is used
- As an optional 0th parameter, you can pass in an execution policy


```cpp
#include <algorithm>
#include <iostream>

int main() {
  std::vector v{42, 51, 66};
  auto minmax_v = std::minmax_element(v.begin(), v.end());
  std::cout << *minmax_v.first << " " << *minmax_v.second << '\n';
}
/*
42 66
*/
```

## `clamp`

`std::clamp` is a relatively new addition to the `<algorithm>` header, it is available since C++17. It takes 3 `const&` parameters by default and an optional comparator. It returns a `const&`, one of the three inputs.

The three inputs are usually referenced as `v` (value), `lo` (lowest value) and `hi` (highest value) in this order.

First, let's see the pseudo-code:

```
if v < lo:
  return lo
if hi < v:
  return hi
return v
```

It's not complicated, but probably it's not very functional to you. Well, it was not for me. So in practice, what does `clamp` do? It might help, if you know [the meaning of the verb clamp](https://www.merriam-webster.com/dictionary/clamp), but to me either reading the definition is not so helpful.

In practice, with `clamp`, you make sure that the value that you get back will be between the boundaries defined by `lo` and `hi`. The returned value will be never smaller than `lo` and never greater than `hi`.

If `hi<lo`, the behaviour is undefined. 

```cpp
#include <algorithm>
#include <iostream>

int main() {
  std::cout << "std::clamp(42, 51, 66): " << std::clamp(42, 51, 66) << '\n';
  std::cout << "std::clamp(51, 42, 66): " << std::clamp(51, 42, 66) << '\n';
  std::cout << "std::clamp(66,42,51): " << std::clamp(66,42,51) << '\n';
  std::cout << "UB: std::clamp(66,51,42): " << std::clamp(66,51,42) << '\n'; // Undefined Behaviour hi < lo
}
/*
std::clamp(42, 51, 66): 51
std::clamp(51, 42, 66): 51
std::clamp(66,42,51): 51
UB: std::clamp(66,51,42): 42
*/
```

# Conclusion

This time, we learned about min/max algorithms. We saw how to get the minimum or maximum elements from multiple variables or from containers. We also saw `clamp` that was added in C++17 which makes sure that we'll always have a value between the boundaries we define.

In the next episode of this series, we'll discuss comparison operators, but before there is something more to discuss.

Is it okay that `min` and `max` return the same element in case the inputs are equal? Is it okay that in that case, both return the leftmost element - or the rightmost depending on your compiler?

Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
