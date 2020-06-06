---
layout: post
title: "The big STL Algorithms tutorial: modifying sequence operations - remove calls"
date: 2020-6-3
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we will discover the 4 modifying sequence algorithms that will help you removing elements from containers:
<!--more-->

* `remove`
* `remove_if`
* `remove_copy`
* `remove_copy_if`

Let's get started!

## `remove`
Remove is a fairly simple algorithm. You pass in a container, or better to say a range defined by two iterators (its beginning and its end) as a third parameter a value that you want to remove. If there are multiple elements in the range matching the passed in value, all of them will be removed. The element next to the one removed takes its place and the range will be shortened by one element.

Let's be more precise here. The elements that are removed, are not __really__ removed, they are not deleted. They are shifted to the end of the original range, and the iterator pointing at the end of the container is updated. What does this mean?

Many things.
- The size of the container doesn't change.
- Elements are still there at the end of the container
- Destructors are not called by running `std::remove`
- In fact, what elements are there at the end is undefined beaviour. They might be the elements you removed or the original elements at those positions. Up to the implementation.

At the time of writing, [coliru](http://coliru.stacked-crooked.com/a/8df97662d3faa232), compiled with gdb and with version C++ 17, keep in positions the original values, while they are also copied to the left.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers { 1, 2, 3, 4, 5, 4, 7, 4, 9, 10 };

  std::cout << "number of elements in vector: " << numbers.size() << "\n";
  std::cout << "numbers before remove: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  std::cout << '\n';
  
  auto beginning_of_removed_items = std::remove(numbers.begin(), numbers.end(), 4); 
  std::cout << "number of elements in vector after remove/before erase: " << numbers.size() << "\n";
  std::cout << "numbers after after remove/before erase: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  std::cout << '\n';
  numbers.erase(beginning_of_removed_items, numbers.end());
  
  std::cout << "number of elements in vector after erase: " << numbers.size() << "\n";
  std::cout << "numbers after erase: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';

  return 0;
}
```

Hence you don't usually use `std::remove` on its own, but by combined with `<your container type>::erase` that actually removes items in the passed in range.

As `std::remove` returns an iterator to the first element that has been moved to the end by passing that one and the original `end()` iterator to `erase` will do the work for you.

By the way, if you think about it, `std::remove` can be a quite slow operation. Removing an element than having another to take its place - depending on the underlying data structure - can be very slow. If it's a linked list, this can mean just updating a link (or two if its a doubly-linked list) - apart from scanning the items for comparison -, but if we talk about a vector, in other words, a dynamic array where elements are stored in a contiguous memory area, removing an element will invoke copy operations. Probably a lot. Each on the right of the element being removed will be copied. Then if there is another element to be removed, the same will happen, elements on the right, shifted by one to the left.

Hence, you have to choose wisely the data structure you want to use, depending on the use case...

I digressed a bit, but I think it was important.

Please note that what I mentioned in this section are true for the other `remove` algorithms, except that elements are compared to the values passed in 

## `remove_if`

Just like `std::remove`, `std::remove_if` takes the passed in range the usual way, but as a third parameter it accepts a unary predicate. It can be a function, a function object, or a [lambda function](http://sandordargo.com/blog/2018/12/19/c++-lambda-expressions) that takes an element of the container and compares it against something defined in the function and returns a boolean. If it returns true, that element will be removed - as remove was defined in the previous section -, if not, the element survives.
Just like for `remove`, as a return value you get back an iterator pointing to the beginning of the removed values. Prefer using `remove` combined with `erase`.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers { 1, 2, 3, 4, 5, 4, 7, 4, 9, 10 };

  std::cout << "original numbers: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  std::cout << '\n';
  
  numbers.erase(std::remove_if(numbers.begin(), numbers.end(), [](auto number) {return number % 2 == 0;}), numbers.end());
  
  std::cout << "numbers after removing/erasing the even ones: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';

  return 0;
}
```

## `remove_copy`

`remove_copy` doesn't change the input range. It will copy whatever doesn't match the passed in value, into another container. I'd dare to say that `remove_copy` is not the best possible name for this algorithm, I'd rather call it `copy_unless` or `copy_if_not`.

It accepts the input range with the usual two iterators pointing to the beginning and the end of the range. As a third parameter, it takes another iterator, pointing to the beginning of the range, you want to copy the non-matching elements to. The last parameter is the value that will not be copied to the new container.

Here is an example.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers { 1, 2, 3, 4, 5, 4, 7, 4, 9, 10 };
  std::vector<int> copiedNumbers;

  std::remove_copy(numbers.begin(), numbers.end(), std::back_inserter(copiedNumbers), 4);
  
  std::cout << "copied numbers: ";
  for (const auto& number : copiedNumbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';

  return 0;
}
```

As we learned for the [`std::copy` algorithms](https://dev.to/sandordargo/the-big-stl-algorithms-tutorial-modifying-sequence-operations-copy-et-al-1751), the output container either has to be big enough to accommodate the values copied into it, or you have to use an inserter, such as [back inserter](https://en.cppreference.com/w/cpp/iterator/back_inserter).


## `remove_copy_if`

`remove_copy_if` is the combination of `remove_copy` and `remove_if`. It takes an input range defined by the usual two parameters, then just like `remove_copy`, it takes the third one to define the beginning of the output range - where elements will be copied to - and as `remove_if`, it takes a predicate as the last parameter that helps to decide whether an element should be removed, in other words not copied, or kept, a.k.a. copied.

I'm sure that by now you know that the predicate can be a [lambda expression](http://sandordargo.com/blog/2018/12/19/c++-lambda-expressions), a functor or a function pointer. 
```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers { 1, 2, 3, 4, 5, 4, 7, 4, 9, 10 };
  std::vector<int> copiedNumbers;

  std::remove_copy_if(numbers.begin(), numbers.end(), std::back_inserter(copiedNumbers), [](auto number) {return number % 2 == 0;});
  
  std::cout << "copied numbers: ";
  for (const auto& number : copiedNumbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';

  return 0;
}
```


## Conclusion

Today, we learned about 4 algorithms removing values from a container. `remove` and `remove_if` will perform in-place modifications, while `remove_copy` and `remove_copy_if` will not touch the input, but instead will create a new output range without the values that we wanted to remove.

Next time we’ll learn about the `reverse` algorithms. Stay tuned!