---
layout: post
title: "The big STL Algorithms tutorial: binary_search et al."
date: 2021-5-19
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we cover the binary search operations. I use plural because there isn't simply `std::binary_search` available for us, but other algorithms as well:
<!--more-->

* `binary_seach`
* `equal_range`
* `lower_bound`
* `upper_bound`

## `binary_seach`

`std::binary_seach` helps us - guess what - to search for an element in a container. As the first two parameters, you have to pass two iterators defining the input range.

Give that we haven't discussed algorithms for a while, here are a few reminders:

- the two iterators must point to the same container, otherwise, the behaviour is undefined
- the compiler has no way to validate this requirement, it's up to the caller

Binary search operations have an additional requirement for their input ranges, **they must be sorted**, otherwise, the behaviour is undefined.

The first time when I learned about this I felt a bit perplexed. Isn't that a bit too much? Shouldn't the algorithm take care of this? Maybe just sort it when needed.

If you think about it a little bit longer, it makes perfect sense. One of the main principles of (C and) C++ is that you should only pay for what you use. The name `binary_seach` is pretty straightforward. It will search for elements with a given mathematical algorithm. Why should it sort anything? Sorting doesn't come for free. If you need to sort your container, use [`std::sort` or something similar](https://www.sandordargo.com/blog/2021/02/03/stl-alogorithms-tutorial-part-19-sorting#sort), if you need to check first if the input range is sorted, [use `is_sorted` or `is_sorted_until`](https://www.sandordargo.com/blog/2021/02/03/stl-alogorithms-tutorial-part-19-sorting#is_sorted).

If `binary_seach` would do anything else, it would be a problem for those who use it as it is supposed to be used. Given that checking this semantic requirement has a cost, it's preferable to simply declare that the outcome is undefined if we pass in unsorted elements.

That's enough about the input range.

As a third parameter we have to pass in the value that we are searching for and there is an optional 4th element that is the comparator. As we got used to it, it can be a lambda, a function pointer or a function object (a functor). It's a binary function, it accepts two elements and returns a value that is convertible to a `bool`.

In any case, `binary::search` will return a boolean, `true` in case an element is found in the input range, `false` otherwise.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>


int main() {
  std::vector numbers {6, 8, 1, 5, 9, 4, 7, 2, 3};
  
  std::cout << std::boolalpha;
  
  std::cout << "Looking for 1 in the unsorted container\n";
  std::cout << std::binary_search(numbers.begin(), numbers.end(), 1) << '\n';
  
  std::sort(numbers.begin(), numbers.end());
  std::cout << "Looking for 1 once the container is sorted\n";
  std::cout << std::binary_search(numbers.begin(), numbers.end(), 1) << '\n';
  
  auto is_equal = [](int lhs, int rhs){return lhs == rhs;};
  
  std::cout << "Looking for 1 once the container is sorted with custom comparator\n";
  std::cout << std::binary_search(numbers.begin(), numbers.end(), 1, is_equal) << '\n';
}
```

But what if you need the element that you were looking for? We [learnt about the different `find*` algorithms earlier](https://www.sandordargo.com/blog/2019/05/15/stl-algorithm-tutorial-part-3-find) that you can use to return find one element, but if you look for a range of elements, you have other options than nesting `std::find` into a loop.

## `equal_range`

`equal_range` returns a pair of iterators giving a handle to all the matching items.

The first iterator points to the first element that is not less than the searched for value and the second element points to the first element that is greater than that value. We are going to check what are the different scenarios, but first, we have to briefly talk about the inputs.

The input parameters are the very same as for `binary_seach`:
- at the first two positions we define the input range
- then the value we are looking for
- and finally the optional comparator

Once again, the input range should be fully sorted.

So let's get back to the different scenarios.

### Value not found and it's bigger than any elements

When the value cannot be found and it is bigger than any elements, both `first` and `last` point right after the container.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
  std::vector numbers {1, 2, 2, 3, 4, 5, 5, 5, 7};
  std::cout << "Size of numbers: " << numbers.size() << '\n';
  const auto [first, last] = std::equal_range(numbers.begin(), numbers.end(), 8);
  std::cout << "First distance from numbers.begin(): " << std::distance(numbers.begin(), first) << '\n';
  std::cout << "Value of first: " << *first << '\n';
  std::cout << "First distance from numbers.last(): " << std::distance(numbers.begin(), last) << '\n';
  std::cout << "Value of last: " << *last << '\n';
}
/*
Size of numbers: 9
First distance from numbers.begin(): 9
Value of first: 0
First distance from numbers.last(): 9
Value of last: 0
*/
```

### Value not found and it's smaller than any elements

When the value cannot be found and it is smaller than any elements, both `first` and `last` point at the first element.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
  std::vector numbers {1, 2, 2, 3, 4, 5, 5, 5, 7};
  std::cout << "Size of numbers: " << numbers.size() << '\n';
  const auto [first, last] = std::equal_range(numbers.begin(), numbers.end(), 0);
  std::cout << "First distance from numbers.begin(): " << std::distance(numbers.begin(), first) << '\n';
  std::cout << "Value of first: " << *first << '\n';
  std::cout << "First distance from numbers.last(): " << std::distance(numbers.begin(), last) << '\n';
  std::cout << "Value of last: " << *last << '\n';
}
/*
Size of numbers: 9
First distance from numbers.begin(): 0
Value of first: 1
First distance from numbers.last(): 0
Value of last: 1
```

### Value not found, there are smaller and bigger items in the container

When the value cannot be found, but there are smaller and bigger elements as well in the container, both `first` and `last` point at the first element greater than the searched value.

It makes sense as that's the first element that is not less than the looked for value and also the first that is greater than it.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
  std::vector numbers {1, 2, 2, 3, 4, 5, 5, 5, 7};
  std::cout << "Size of numbers: " << numbers.size() << '\n';
  const auto [first, last] = std::equal_range(numbers.begin(), numbers.end(), 6);
  std::cout << "First distance from numbers.begin(): " << std::distance(numbers.begin(), first) << '\n';
  std::cout << "Value of first: " << *first << '\n';
  std::cout << "First distance from numbers.last(): " << std::distance(numbers.begin(), last) << '\n';
  std::cout << "Value of last: " << *last << '\n';
}
/*
Size of numbers: 9
First distance from numbers.begin(): 8
Value of first: 7
First distance from numbers.last(): 8
Value of last: 7
```
### Value found

This is the nominal case and it behaves as it's expected. `first` is the last one smaller than the looked for value and `last` is the first one that is greater.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
  std::vector numbers {1, 2, 2, 3, 4, 5, 5, 5, 7};
  std::cout << "Size of numbers: " << numbers.size() << '\n';
  const auto [first, last] = std::equal_range(numbers.begin(), numbers.end(), 5);
  std::cout << "First distance from numbers.begin(): " << std::distance(numbers.begin(), first) << '\n';
  std::cout << "Value of first: " << *first << '\n';
  std::cout << "First distance from numbers.last(): " << std::distance(numbers.begin(), last) << '\n';
  std::cout << "Value of last: " << *last << '\n';
}
/*
Size of numbers: 9
First distance from numbers.begin(): 5
Value of first: 5
First distance from numbers.last(): 8
Value of last: 7
```
Having seen all these examples, we can observe that in case `equal_range` couldn't find the value it looked for, then both iterators will point at the same place, otherwise not. This means that we don't need `binary_search` to validate first the existence of the element when we look for the range, we can simply check if the two iterators are pointing to the same place or not

## `lower_bound` and `upper_bound`

While `equal_range` returns a pair of iterators `lower_bound` and `upper_bound` returns only one:

- `lower_bound` returns an iterator pointing to the first item not less than the searched value
- `upper_bound` returns an iterator pointing to the first element greater than the searched value

The parameters, these functions take are really the same as we saw before:
- at the first two positions we define the input range
- then the value we are looking for
- and finally the optional comparator

Now let's see a couple of examples.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
  std::vector numbers {1, 2, 2, 3, 4, 5, 5, 5, 7};
  std::for_each(numbers.begin(), numbers.end(), [](int num) {std::cout << num << " ";});
  std::cout << '\n';
  std::cout << "Size of numbers: " << numbers.size() << '\n';
  std::cout << '\n';
  {
      std::cout << "========\n";
      const auto value = 5;
      std::cout << "Looking for " << value << ", that is inside the container\n";
      auto lower = std::lower_bound(numbers.begin(), numbers.end(), value);
      auto upper = std::upper_bound(numbers.begin(), numbers.end(), value);
      std::cout << "lower's distance from numbers.begin(): " << std::distance(numbers.begin(), lower) << '\n';
      std::cout << "Value of lower: " << *lower << '\n';
      std::cout << "upper's distance from numbers.begin(): " << std::distance(numbers.begin(), upper) << '\n';
      std::cout << "Value of upper: " << *upper << '\n';
  }
  {
      std::cout << "========\n";
      const auto value = 0;
      std::cout << "Looking for " << value << ", that is smaller than the smallest item of the container\n";
      const auto lower = std::lower_bound(numbers.begin(), numbers.end(), value);
      const auto upper = std::upper_bound(numbers.begin(), numbers.end(), value);
      std::cout << "lower's distance from numbers.begin(): " << std::distance(numbers.begin(), lower) << '\n';
      std::cout << "Value of lower: " << *lower << '\n';
      std::cout << "upper's distance from numbers.begin(): " << std::distance(numbers.begin(), upper) << '\n';
      std::cout << "Value of upper: " << *upper << '\n';
  }
  {
      std::cout << "========\n";
      const auto value = 9;
      std::cout << "Looking for " << value << ", that is larger than the largest item of the container\n";
      const auto lower = std::lower_bound(numbers.begin(), numbers.end(), value);
      const auto upper = std::upper_bound(numbers.begin(), numbers.end(), value);
      std::cout << "lower's distance from numbers.begin(): " << std::distance(numbers.begin(), lower) << '\n';
      std::cout << "Value of lower: " << *lower << '\n';
      std::cout << "upper's distance from numbers.begin(): " << std::distance(numbers.begin(), upper) << '\n';
      std::cout << "Value of upper: " << *upper << '\n';
  }
  {
      std::cout << "========\n";
      const auto value = 6;
      std::cout << "Looking for " << value << ", that is not in the container that contains both smaller and larger values than " << value << '\n';
      const auto lower = std::lower_bound(numbers.begin(), numbers.end(), value);
      const auto upper = std::upper_bound(numbers.begin(), numbers.end(), value);
      std::cout << "lower's distance from numbers.begin(): " << std::distance(numbers.begin(), lower) << '\n';
      std::cout << "Value of lower: " << *lower << '\n';
      std::cout << "upper's distance from numbers.begin(): " << std::distance(numbers.begin(), upper) << '\n';
      std::cout << "Value of upper: " << *upper << '\n';
  }
}
/*
1 2 2 3 4 5 5 5 7 
Size of numbers: 9

========
Looking for 5, that is inside the container
lower's distance from numbers.begin(): 5
Value of lower: 5
upper's distance from numbers.begin(): 8
Value of upper: 7
========
Looking for 0, that is smaller than the smallest item of the container
lower's distance from numbers.begin(): 0
Value of lower: 1
upper's distance from numbers.begin(): 0
Value of upper: 1
========
Looking for 9, that is larger than the largest item of the container
lower's distance from numbers.begin(): 9
Value of lower: 0
upper's distance from numbers.begin(): 9
Value of upper: 0
========
Looking for 6, that is not in the container that contains both smaller and larger values than 6
lower's distance from numbers.begin(): 8
Value of lower: 7
upper's distance from numbers.begin(): 8
Value of upper: 7
*/
```
I don't share any comments on the results, essentially they are the same as for `equal_range`, you can find a deeper explanation at that section.

## Conclusion

This time, we learned about binary search algorithms. We saw how to check if an item can be found in the container and also how to get its or even their position (in case there are multiple items with the same value).

Next time we'll discover merging algorithms.

## Connect deeper

If you found interesting this article, please [subscribe to my personal blog](http://eepurl.com/gvcv1j) and let's connect on [Twitter](https://twitter.com/SandorDargo)!
