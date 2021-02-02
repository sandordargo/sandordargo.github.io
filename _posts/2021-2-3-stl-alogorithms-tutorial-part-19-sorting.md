---
layout: post
title: "The big STL Algorithms tutorial: sorting operations"
date: 2021-2-3
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we cover the sorting operations - except for ranges which will be covered in a different series.
<!--more-->

* `sort`
* `stable_sort`
* `partial_sort`
* `partial_sort_copy`
* `is_sorted`
* `is_sorted_until`
* `nth_element`

## `sort`

Is it a bit too much to say that `std::sort` is the flagship algorithm of the above sorting algorithms? Probably not, at least if we discuss the basics for this algorithm, we don't need to discuss all the details for each other.

By default, `std::sort` takes two parameters, two iterators that define a range that the user wants to sort.

There is a third optional parameter to define, the comparator that is used for the sorting. As usual, it can be a lambda, a function pointer or a function object (a functor). It's a binary function, it accepts two elements and returns a bool - or at least a value that is convertible into bool. This function shouldn't modify any of its components that seems quite reasonable. The function should return `true` if the first parameter should precede the second in the sorted range.

`std::sort` is a void algorithm, it doesn't return anything. Let's see an example with and without a comparator.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

enum class Transmission {Automatic, Manual};

struct Car {
  int horsePower;
  Transmission transmission;
};

int main() {
  std::vector<int> numbers {1,9,7,4,5,6,3,8,2};
  std::sort(numbers.begin(), numbers.end());
  std::for_each(numbers.begin(), numbers.end(), [](auto num){ std::cout << num << " ";});    
  std::cout << '\n';
  
  std::vector cars {
    Car{100, Transmission::Automatic},
    Car{80, Transmission::Manual},
    Car{250, Transmission::Manual},
    Car{100, Transmission::Manual},
    Car{120, Transmission::Automatic},
  };
  
  std::sort(cars.begin(), cars.end(), [](const Car& lhs, const Car& rhs){return lhs.horsePower < rhs.horsePower;});
  std::for_each(cars.begin(), cars.end(), [](auto car){ std::cout << "Car.hp " << car.horsePower << " " << ((car.transmission == Transmission::Manual) ? "manual" : "automatic") << '\n';});    
}
```
I think the above examples are pretty straightforward, what is worth to notice is how the comparator is written. As smaller performance cars should come before the stronger ones - at least in our examples - the comparator returns `true` if the first passed in car is weaker than the second. That's how we built an ascendingly sorted container.

## `stable_sort`

What is the difference between `stable_sort` and `sort`?

`stable_sort` gives us a guarantee that the order of equivalent elements will be preserved after the algorithm applied. `sort` doesn't give any such promise.

In other words, sticking with the example of cars, if in the input container a manual gearbox car precedes an automatic one and they both have the same performance, it'll come before it even after calling `stable_sort` on them.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

enum class Transmission {Automatic, Manual};

struct Car {
  int horsePower;
  Transmission transmission;
};

int main() {
  std::vector cars {
    Car{100, Transmission::Automatic},
    Car{80, Transmission::Manual},
    Car{250, Transmission::Manual},
    Car{100, Transmission::Manual},
    Car{120, Transmission::Automatic},
  };
  
  std::stable_sort(cars.begin(), cars.end(), [](const Car& lhs, const Car& rhs){return lhs.horsePower < rhs.horsePower;});
  std::for_each(cars.begin(), cars.end(), [](auto car){ std::cout << "Car.hp " << car.horsePower << " " << ((car.transmission == Transmission::Manual) ? "manual" : "automatic") << '\n';});    
}
```

## `partial_sort`

As the name suggests, this algorithm is not going to sort the whole container. But what does it sort exactly?

It takes three iterators as an input, plus an optional comparator that isn't different from the comparators we already saw. Let's focus on the three iterators.

The first one denotes the beginning of the input range, the third one the end of it.

The middle one gives the point up until you want the range to be sorted. It's worth to emphasize that this iterator denotes the position up until you want to sort the range, not the last sorted value.

Let's have a look at a simple example.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>


int main() {
  std::vector numbers {6, 8, 1, 5, 9, 4, 7, 2, 3};
  
  std::partial_sort(numbers.begin(), numbers.begin()+4, numbers.end());
  std::for_each(numbers.begin(), numbers.end(), [](auto number){ std::cout << number << ' ';});    
}
/*
1 2 3 4 9 8 7 6 5 
*/
```

In this example, we have a vector of numbers from 1 to 9 in random order. (Notice how you can omit the contained type with C++20!) We call `partial_sort` on the whole container where the _middle_ element is `numbers.begin()+4`.

`numbers.begin()+4` points at the position of `9` in the original vector, which is the fifth number (position 4 starting from 0). So our call to `partial_sort` means that we want to sort the elements up until the fifth element (excluded), so the first four elements.

The result that is `1 2 3 4 9 8 7 6 5` exactly shows that. In the first 4 places, we have the elements sorted, and after not. It seems like they follow a reversed sorting, but don't be deceived, that's just coincidence. The elements after position `middle` do not follow any particular order.

## `partial_sort_copy`

`partial_sort_copy` is more different from `partial_sort` then many would expect. Based on what we have seen so far in this series, you most probably think that it has the same signature apart from an extra parameter denoting the beginning of the output range.

But it's not the case.

Instead of three input iterators, it only takes two. One for the beginning and one for the end of the range we want to partially sort. Then it takes two output iterators one for the beginning and one for the end of the range we want to copy our sorted elements.

And of course, there is the usual optional comparator.

The length of this output range defines how many elements will be sorted. Let's have a look at the example:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>


int main() {
  std::vector numbers {6, 8, 1, 5, 9, 4, 7, 2, 3};
  std::vector<int> output(4);
  
  std::partial_sort_copy(numbers.begin(), numbers.end(), output.begin(), output.end());
  std::for_each(output.begin(), output.end(), [](auto number){ std::cout << number << ' ';});    
}
/*
1 2 3 4 
*/
```

There are a couple of things to notice.
- Only the sorted elements will be copied.
- `std::partial_sort_copy` checks the size of the output range, not its capacity. In other words, if you default initialize a vector and then you reserve a capacity, nothing will be copied over because the size of the output vector is still 0.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>


int main() {
  std::vector numbers {6, 8, 1, 5, 9, 4, 7, 2, 3};
  std::vector<int> output;
  output.reserve(4);
  
  std::partial_sort_copy(numbers.begin(), numbers.end(), output.begin(), output.end());
  std::cout << std::boolalpha << "is the output empty? " << output.empty() << '\n';
}
/*
is the output empty? true
*/
```

Personally, I find the signature of this algorithm not so great. It's not following the practices we got used to in the `<algorithms>` header. I think that defining the output range is impractical. It's safer than asking only for the beginning where the caller has to make sure that output is big enough to accommodate all the inserted elements. Yet, with this solution, you must initialize a vector to a certain size and that means either copying the same element n times at initialization or the default initialization of n elements. It might be cheap, but in certain cases, it might be expensive. Whereas when you can simply pass in a `std::back_inserter` as an output, it's not an issue.

## `is_sorted`

`is_sorted` is super simple. It takes the beginning and the end of a range an optional comparator and tell you whether the range is sorted or not by returning a `bool`

```cpp
#include <iostream>
#include <algorithm>
#include <vector>


int main() {
  std::vector sortedNumbers {1, 2, 3, 4, 5, 6, 7, 8, 9};
  std::vector unsortedNumbers {6, 8, 1, 5, 9, 4, 7, 2, 3};
  std::vector descendingNumbers {9, 8, 7, 6, 5, 4, 3, 2, 1};
  std::cout << std::boolalpha << "is the sortedNumbers sorted? " << std::is_sorted(sortedNumbers.begin(), sortedNumbers.end()) << '\n';
  std::cout << std::boolalpha << "is the unsortedNumbers sorted? " << std::is_sorted(unsortedNumbers.begin(), unsortedNumbers.end()) << '\n';
  std::cout << std::boolalpha << "is the descendingNumbers sorted? " << std::is_sorted(descendingNumbers.begin(), descendingNumbers.end()) << '\n';
  std::cout << std::boolalpha << "is the descendingNumbers sorted? " << std::is_sorted(descendingNumbers.begin(), descendingNumbers.end(), [](auto lfs, auto rhs){ return lfs > rhs; }) << '\n';
  std::cout << std::boolalpha << "is the descendingNumbers sorted? " << std::is_sorted(descendingNumbers.begin(), descendingNumbers.end(), std::greater<>{}) << '\n';
}
/* 
is the sortedNumbers sorted? true
is the unsortedNumbers sorted? false
is the descendingNumbers sorted? false
is the descendingNumbers sorted? true
is the descendingNumbers sorted? true
*/
```

It's worth to remind ourselves that being sorted is calculated based on using `operator<`. Order matters, even if you think that `descendingNumbers` are nicely sorted, `std::is_sorted` doesn't think so by default. If you want to compare based on another comparator you have to pass it, just like you can see in the last two lines.

## `is_sorted_until`

`is_sorted_until` takes a range defined by its beginning and its end and an optional comparator. It returns an iterator that points to the last sorted element starting the first item.

Meaning that if you call `is_sorted` with the beginning of the inspected range and with the return value `is_sorted_until`, it will return `true`. On the other hand, if you call it with the return value + 1, the result will be `false`.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>


int main() {
  std::vector numbers {1, 2, 3, 4, 9, 5, 6, 7, 8, 9};
  auto lastSortedNumber = std::is_sorted_until(numbers.begin(), numbers.end());
  std::cout << "Last sorted number in numbers: " << *lastSortedNumber << '\n';
  std::cout << std::boolalpha;
  std::cout << "std::is_sorted(numbers.begin(), lastSortedNumber): " << std::is_sorted(numbers.begin(), lastSortedNumber) << '\n';
  std::cout << "std::is_sorted(numbers.begin(), lastSortedNumber+1): " << std::is_sorted(numbers.begin(), lastSortedNumber+1) << '\n';
}
/*
Last sorted number in numbers: 5
std::is_sorted(numbers.begin(), lastSortedNumber): true
std::is_sorted(numbers.begin(), lastSortedNumber+1): false
*/
```

## `nth_element`

`nth_element` is a function that told me nothing by its name when I looked at it. Do you get it just like that?

Ok, I tell you. Let's ignore for a moment the arguments it takes.

`nth_element` will rearrange the container in a way that at the nth position you will find the element that would be there if the container was sorted.

Before there will be smaller or equal elements not following any particular order and larger ones after.

The parameters are quite similar to `partial_sort`. The first parameter denotes the beginning, the third the end and in the middle, you have the nth element. As usual, you can pass in a custom comparator.

Let's have a look at an example.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>


int main() {
  std::vector numbers {6, 8, 1, 4, 9, 5, 7, 2, 3};
  std::nth_element(numbers.begin(), numbers.begin()+4, numbers.end());
  std::for_each(numbers.begin(), numbers.end(), [](auto number){ std::cout << number << ' ';});
  std::cout << '\n';
  std::cout << "The fifth largest element is: " << numbers[4] << '\n';
}

/*
3 2 1 4 5 6 7 8 9 
The fifth largest element is: 5

*/
```
In the above example, by passing in `numbers.begin()+4` as the middle parameter we determined what is the 5th largest element in `numbers`.

## Conclusion

Today, we learned about sorting algorithms. Some are pretty straightforward (such as `sort`, `partial_sort` or `is_sorted`), while `nth_element` made us - at least me - think and `partial_sort_copy` gave us some surprises and inconsistencies. I hope you enjoyed today's discoveries, next time we'll move from sorting algorithms to binary searches.

## Connect deeper

If you found interesting this article, please [subscribe to my personal blog](http://eepurl.com/gvcv1j) and let's connect on [Twitter](https://twitter.com/SandorDargo)!