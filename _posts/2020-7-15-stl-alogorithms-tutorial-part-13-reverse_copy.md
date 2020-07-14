---
layout: post
title: "The big STL Algorithms tutorial: modifying sequence operations - turn things around"
date: 2020-7-15
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we will continue with two algorithms that help us to reverse the order of elements in a range:
<!--more-->

* `reverse`
* `reverse_copy`

Let's get started!

## `reverse`

It's as simple as you can imagine - by an STL algorithm. It takes a range defined by a pair of iterators and reverses the range, in place. It doesn't even have a return value. The only thing you have to pay attention to is to pass in the right iterators.me

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  std::vector<int> numbers {1, 2, 3, 4, 5, 6, 7, 8, 9};

  std::reverse(numbers.begin(),numbers.end());

  std::cout << "numbers of original vector are reversed: ";
  for (auto number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << std::endl;

  return 0;
}
```

## `reverse_copy`

This algorithm is not much more complicated than `reserve`. The only difference is that, while `reverse` made in-place changes to a container, `reverse_copy` leaves the input intact and fills up an output container.


As you could have got used to it, the output is passed in by an iterator (here as a third parameter). You either make sure that the container under the iterator is big enough to accommodate all the elements that are going to be inserted or you pass in an iterator that can insert new elements, like in the below example.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  std::vector<int> numbers {1, 2, 3, 4, 5, 6, 7, 8, 9};
  std::vector<int> reversedNumbers{};
  reversedNumbers.reserve(numbers.size());

  std::reverse_copy(numbers.begin(),numbers.end(),std::back_inserter(reversedNumbers));

  std::cout << "order of numbers in original vector hasn't changes: ";
  for (auto number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << std::endl;

  std::cout << "numbers in reversedNumbers are reversed: ";
  for (auto number : reversedNumbers) {
    std::cout << ' ' << number;
  }
  std::cout << std::endl;

  return 0;
}
```

You can consider it as a certain part of our immutable programming toolbox, Why just a certain? As STL algorithms operate through iterators, not directly on containers, it cannot return you a new container, but it will modify the one that the passed in iterator points to.

Therefore it's not strongly part of an immutable toolbox, but one can make the point that it just takes a potentially empty container that was created on the line before, and then it should not be modified.

Sadly, your copy cannot be `const` given that `reverse_copy` already had to modify it. If you really what to show that it's a const vector, you have to wrap the transformation into its own function, somehow like this:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

template<typename T>
const std::vector<T> reverse_const_copy(const std::vector<T>& input) {
    std::vector<T> reversedInput{};
    reversedInput.reserve(input.size());
    std::reverse_copy(input.begin(),input.end(),std::back_inserter(reversedInput));
    return reversedInput;
}

int main () {
  std::vector<int> numbers {1, 2, 3, 4, 5, 6, 7, 8, 9};
  const std::vector<int> reversedNumbers = reverse_const_copy(numbers);
  
  // reversedNumbers.push_back(0); // this does not compile as reversedNumbers is const

  std::cout << "order of numbers in original vector hasn't changes: ";
  for (auto number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << std::endl;

  std::cout << "numbers in reversedNumbers are reversed: ";
  for (auto number : reversedNumbers) {
    std::cout << ' ' << number;
  }
  std::cout << std::endl;

  return 0;
}
```

## Conclusion

Today, we learned about 2 algorithms that help us to reverse the elements of a container, I think their usage didn't hide any big surprise. Besides we also saw how we can make const reversed copies, even if it needs some additional tinkering.

Next time we’ll learn about Sean Parent's favourite algorithms `rotate` and `rotate_copy`. Stay tuned!



