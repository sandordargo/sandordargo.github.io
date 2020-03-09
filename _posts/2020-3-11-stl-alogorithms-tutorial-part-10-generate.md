---
layout: post
title: "The big STL Algorithms tutorial: modifying sequence operations - fill and generate"
date: 2020-3-11
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we will discover the 4 modifying sequence algorithms that fill in or generate data:
<!--more-->

* `fill`
* `fill_n`
* `generate`
* `generate_n`

Let's get started!

## `fill`

This is a fairly simple algorithm that takes two iterators defining a range and value that it will assign to each and every element in the range.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers(8); // a vector of 8 elements zero initialized
  std::cout << "numbers after the initialization of the vector: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  
  std::fill(numbers.begin(), numbers.begin()+4, 42);
  std::fill(numbers.begin()+4, numbers.end(), 51); 

  std::cout << "numbers after filling up the vector: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }

  return 0;
}
```

When can you use it? If you want to initialize a vector with the same items, do use it, you can pass the value in the vector's constructor like this:

```cpp
std::vector<int> numbers(8, 42); // a vector of 8 elements initialized to 42
```

Otherwise, in case you need to create a vector with sequences of the same item as we did in the first example of the section, it comes quite handy.

## `fill_n`

`fill_n` is pretty similar to `fill`, the only difference is that while `fill` takes two iterators defining a range, `fill_n` takes one iterator pointing at the beginning of the range and instead of the second iterator, it takes a number indicating how many elements have to be filled.

Here is the example used for `fill` with the necessary changes:

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers(8); // a vector of 8 elements initialized to 42
  std::cout << "numbers after the initialization of the vector: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  
  std::fill_n(numbers.begin(), 4, 42);
  std::fill_n(numbers.begin()+4, 4, 51); 

  std::cout << "numbers after filling up the vector: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }

  return 0;
}
```

What you really have to pay attention to is that pass in a valid number as a second parameter. If you don't, it's undefined behaviour. That means you cannot really know what would happen, but it's better not to play with production code.

There might be no visible consequences. For example, when I changed the second fill command to update 5 items (the 9th is already out of the vector), I still get the expected output. But when I pass 8, so half of those are out of the vector's bounds, I got a core dump when the vector's memory is deallocated.

Just pay attention to pass in the good values.

## `generate`

How `generate` works, is similar to `fill`. It also takes two iterators defining a range that has to be updated. The difference is that while `fill` takes a value as a third parameter, `generate` takes a - drumbeat, please - generator, that's right!

But what is a generator?

It is any function that is called without any arguments and that returns a value convertible to those pointed by the iterators.

As it's the simplest example, it can be just a function always returning the same value. It's not very useful, especially not compared to `fill`, but let's use it just to show how this algorithm works. As usual, the generator doesn't have to be a function, it can be a function object or a lambda just as well.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> numbers(8); // a vector of 8 elements initialized to 0
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  
  auto staticGenerator = [](){ return 42; };
  
  std::generate(numbers.begin(), numbers.end(), staticGenerator);

  std::cout << "numbers after filling up the vector: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }

  return 0;
}
```

It's as simple as that.

To get random numbers, you have to use a random generator. How random generation works is outside the scope of this article.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
#include <random>

int main() {
  std::vector<int> numbers(8); // a vector of 8 elements initialized to 0
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  
  // Random generator beginning
  std::random_device rd;
  std::mt19937 mt(rd());
  std::uniform_real_distribution<double> distribution(1.0, 10.0);
  
  auto randomGenerator = [&distribution, &mt](){ return distribution(mt); };
  // Random generator end
  
  std::generate(numbers.begin(), numbers.end(), randomGenerator);

  std::cout << "numbers after filling up the vector: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }

  return 0;
}
```

## `generate_n`

If you read the last three sections with care, this one will not give you any surprise at all.

It works like `fill_n` in terms of passing in the values to be updated - a start iterator and a number of items -  and like `generate` in terms of generating the values to be assigned - a function not taking any parameter but returning a value that can be converted into to the target type.

Which one to use, `generate` or `generate_n`? It should depend on your use-case to see which provides better readability. If you focus on a range, then use `generate`, but if the number of items to be filled/generated is more important, use the `_n` version.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
#include <random>

int main() {
  std::vector<int> numbers(8); // a vector of 8 elements initialized to 0
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }
  std::cout << '\n';
  
  // Random generator beginning
  std::random_device rd;
  std::mt19937 mt(rd());
  std::uniform_real_distribution<double> distribution(1.0, 10.0);
  
  auto randomGenerator = [&distribution, &mt](){ return distribution(mt); };
  // Random generator end
  
  std::generate_n(numbers.begin(), 8, randomGenerator);

  std::cout << "numbers after filling up the vector: ";
  for (const auto& number : numbers) {
    std::cout << ' ' << number;
  }

  return 0;
}
```

## Conclusion

Today, we learned about 4 algorithms filling up values in a container. `fill` and `fill_n` put static values in a container, while `generate` and `generate_n` dynamically creates the values populating the target.

Their usage should depend on your use-case, whether you need a fixed number of generated values or a containerful of items.

Next time we’ll learn about the `remove` algorithms. Stay tuned!