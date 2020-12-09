---
layout: post
title: "The big STL Algorithms tutorial: modifying sequence operations - rotate functions"
date: 2020-12-9
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we will finish the episodes on modifying sequence operations by three functions involving randomicity.
<!--more-->

* `random_shuffle`
* `shuffle`
* `sample`

## `random_shuffle`

So that we don't forget let's start by the fact that `random_shuffle` is dead. It had a couple of different signatures and all of them were remove latest in C++17 and were replaced by `shuffle`.

`random_shuffle` takes its input range by defining its beginning and end as usual by two iterators and it also takes an optional random number generator (RNG).

When an RNG is not provided it usually uses `std::rand`, but that's implementation specific.

Here is a simple example, when we use the built-in generator.

http://www.cplusplus.com/reference/algorithm/random_shuffle/
```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  std::vector<int> numbers {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
  std::random_shuffle (numbers.begin(), numbers.end());

  std::cout << "numbers vector contains:";
  for (const auto n: numbers) {
    std::cout << ' ' << n;
  }
  std::cout << '\n';

  return 0;
}
```

And here is the other when you pass in your RNG.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <ctime>
#include <cstdlib>


int myrandom (int i) { return std::rand()%i;}

int main () {
  std::srand(unsigned(std::time(0))); // 1
  std::vector<int> numbers {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};

  std::random_shuffle (numbers.begin(), numbers.end(), [](auto number){return std::rand()%number;}); // 2

  std::cout << "numbers vector contains:";
  for (const auto n: numbers) {
    std::cout << ' ' << n;
  }
  std::cout << '\n';

  return 0;
}
```

If you look at the include statements, you can observe that we are using legacy headers (`<ctime>` and `<cstdlib>` start with C) and the third (optional) argument.

At 1), from `<ctime>` we use `std::time` in order to provide a seed for `<cstdlib>`'s `std::srand`. Then in 2) we simply have to pass any function as a third - optional - parameter to `std::random_shuffle`. Yes, any. You could simply pass a function always returning 42. Would it work well? Try and see it!


## `shuffle`
A good API, a good library is easy to use and hard to misuse. As we saw, `std::random_shuffle` might be easy to use, but it's just as easy to misuse. `std::shuffle` takes standard generators as those defined in `<random>` and it's not optional. 

```cpp
#include <iostream>
#include <algorithm>
#include <std::vector>    
#include <random>
#include <chrono>

int main () {
  std::vector<int> numbers {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};

  unsigned seed = std::chrono::system_clock::now().time_since_epoch().count();

  std::shuffle (numbers.begin(), numbers.end(), std::default_random_engine(seed));

  std::cout << "numbers vector contains:";
  for (const auto n: numbers) {
    std::cout << ' ' << n;
  }

  return 0;
}
```

You can observe that in this case instead of the C libraries, we `<random>` and `<chrono>` that were introduced in C++11. In case you want to understand better how to create a random number generator, you should start your journey at the [`<random>`](https://en.cppreference.com/w/cpp/numeric/random) header.
  
## `sample`

While the previous two algorithms randomly reordered the elements in their input ranges, `sample` - available since C++17 - leaves its input intact. The input doesn't change, but on the other hand, it will take a given (by you) number of elements randomly from its input and push it into its out container.

Once you understand this concept, and you've seen a few STL algorithms -  and we did, this is the 15th episode of [this series] - you can guess it's signature with a quite high level on confidence.

First, you pass in two iterators denoting the input range, then one iterator for the output. At this point, you might think a bit, but the next one is the number of elements you want to pick and finally, you have to pick a random number generator.

What questions can we have? We have already discussed a lot about [what happens if we don't pass the iterators propoerly](http://sandordargo.com/blog/2019/08/14/stl-alogorithms-tutorial-part-5-copy-operations). So regarding the parameters, we can have only one question. What happens if the number of elements we want to pick is more than the size of the input range?

One could think that in that case some elements will be picked multiple times, but it wouldn't really be a "sample" anymore. So if `n > input.size()` then `n` will be the size of the input by default.

One more thing is worth to note, in case you one of the standard iterators to populate the output range (like `std::back_inserter`), this algorithm is stable. Meaning that the relative order of the selected items is the same as in the input range. If 4 became 3 in the input, it will come before in the output too.

Here is a simple example:
```cpp
#include <iostream>
#include <algorithm>
#include <vector>    
#include <random>
#include <chrono>


int main () {
  std::vector<int> numbers {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
  std::vector<int> out;

  unsigned seed = std::chrono::system_clock::now().time_since_epoch().count();

  std::sample (numbers.begin(), numbers.end(), std::back_inserter(out), 3, std::default_random_engine(seed));

  std::cout << "out vector contains:";
  for (const auto n: out) {
    std::cout << ' ' << n;
  }

  return 0;
}
```

## Conclusion

Today, we learned about our last modifying sequence algorithms, 3 functions involving some randomness in how they reorder or select items. We are far from finishing the exploration of the `<algorithm>` header, next time we will learn about partitioning operations. Stay tuned!

## Connect deeper

If you found interesting this article, please [subscribe to my personal blog](http://eepurl.com/gvcv1j) and let's connect on [Twitter](https://twitter.com/SandorDargo)!