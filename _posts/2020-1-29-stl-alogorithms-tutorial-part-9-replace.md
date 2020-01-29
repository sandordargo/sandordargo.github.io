---
layout: post
title: "The big STL Algorithms tutorial: modifying sequence operations - replace*"
date: 2020-1-29
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we will discover the 4 algorithms starting with the word `replace`:
<!--more-->
* `replace`
* `replace_if`
* `replace_copy`
* `replace_copy_if`

Let's get started!

## `replace`

There is not much surprise in this algorithm, it does what its name suggests and that's a good thing. As [François-Guillaume RIBREAU](https://blog.fgribreau.com/) said at [DevOps D-Day](), an API should be boring, meaning that among others its signature should be straightforward.

`replace` takes a range defined by the iterators pointing at the first and last elements of it, plus it takes an old value that should be replaced by the value.

The only question you might get based on the name, whether it replaces the first occurrence of the to be replaced item, or all of them. It will replace all of them. Here is an example:

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers { 1, 2, 3, 4, 5, 4, 7, 4, 9, 10 };

  std::replace(numbers.begin(), numbers.end(), 4, 42); 

  std::cout << "numbers after replace: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';

  return 0;
}
```

But how to replace just the first (n) elements? That's a story for another day.

## `replace_copy`

`replace_copy` is quite similar to `replace`, the difference is that it leaves the input range untouched and writes results into another container. 

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers { 1, 2, 3, 4, 5, 4, 7, 4, 9, 10 };
  std::vector<int> otherNumbers (numbers.size());

  std::replace_copy(numbers.begin(), numbers.end(), otherNumbers.begin(), 4, 42); 

  std::cout << "numbers after replace_copy have not changed: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  
  std::cout << "otherNumbers after replace: ";
  for (const auto& number : otherNumbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';

  return 0;
}
```

Some important notes:

* The output range is defined not by two, but by one iterator pointing at the first element of the output range. Don't forget the output range has to be at least as big as the input one. If not, the behaviour is [undefined](http://sandordargo.com/blog/2019/12/18/stl-alogorithms-tutorial-part-8-transform-non-matching-sizes).
* Not only the replaced elements are written to the output range, but every element. If you want to copy only replaced items, you have to combine two algorithms. One possibility is this:

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers { 1, 2, 3, 4, 5, 4, 7, 4, 9, 10 };
  std::vector<int> otherNumbers;

  std::copy_if(numbers.begin(), numbers.end(), std::back_inserter(otherNumbers), [](int number){return number == 4;});
  std::replace(otherNumbers.begin(), otherNumbers.end(), 4, 42); 

  std::cout << "numbers after replace have not changed: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  
  std::cout << "otherNumbers after replace: ";
  for (const auto& number : otherNumbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';

  return 0;
}
```

## `replace_if`

Just like `replace`, `replace_if` also takes a range defined by the iterators pointing at the first and last elements of it, then right after the range and before the new value instead of an old value it takes a unary predicate.

This result of predicate decides whether a value should be replaced or not. As usual, it can be a pointer to a function, a functor or a [lambda exprssion]().

Here is an example:

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers { 1, 2, 3, 4, 5, 4, 7, 4, 9, 10 };

  std::replace_if(numbers.begin(), numbers.end(), [](auto number){return number == 4;}, 42); 

  std::cout << "numbers after replace: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';

  return 0;
}
```

## `replace_copy_if`

Finally, let's have a quick look at `replace_copy_if`. I'm sure you can guess what it does and how it accepts its parameters after just having read about `replace_copy` and `replace_if`. It works the same way as `replace_copy`, but instead of the fourth parameter defining the old value, it accepts a unary predicate, just like `replace_if`.

As a reminder, a unary predicate can be a pointer to a function, a functor or a [lambda exprssion](). I always use the latter in my examples as they are so short.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers { 1, 2, 3, 4, 5, 4, 7, 4, 9, 10 };
  std::vector<int> otherNumbers (numbers.size());

  std::replace_copy_if(numbers.begin(), numbers.end(), otherNumbers.begin(), [](auto number){return number == 4;}, 42); 

  std::cout << "numbers after replace_copy have not changed: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  
  std::cout << "otherNumbers after replace: ";
  for (const auto& number : otherNumbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';

  return 0;
}
```

## Conclusion

Today, we had a peek into the algorithms replacing elements of a container. We saw that there 4 different versions depending on whether we want an in-place or a copy replace and whether we want to identify to-be-replaced elements by value or by a more elaborate condition.

We also saw that the `replace*` algorithms can only replace all the items matching the condition, in order to replace a given number of items, other algorithms have to be used - a topic for another blog post.

Next time we’ll learn about the fill and generate algorithms. Stay tuned!