---
layout: post
title: "The big STL Algorithms tutorial: permutation operations"
date: 2021-11-10
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
Last time I promised to continue with the `<numeric>` header, but I realized that I forgot about a draft I already had. So in this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we are going to talk about permutations:
<!--more-->

* `is_permutation`
* `next_permutation`
* `prev_permutation`

You might have noticed that at episode 4 - more than 2 years ago! - we already talked about `is_permutation`, but I really cannot mention 
`next_permutation` or `prev_permutation` without it, so I prefer to include it once more.

## `is_permutation`

`std::is_permutation` was introduced by C++11. It takes two ranges and checks whether the elements of one range can be rearranged in a way that the two ranges are matching with each other.

This algorithm either takes 3 or 4 iterators to define the two ranges.

With the 3 iterator version, you pass in the beginning and the end of the first range and the beginning of the second range. You must make sure that the second container has at least as many elements as the first one, the algorithm will not explicitly check it. If you fail to comply with this expectation, the behaviour is undefined.

With the 4 iterator version, which is available since C++14, you fully define both of the ranges, by passing their beginning and end.

While `std::is_permutation` is not parallelizable, so you cannot pass in an execution policy, you can pass in a custom binary predicate that will be used instead of `operator==` to compare two elements.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  std::vector myvector {1, 2, 3, 1, 4, 5};
  auto myvectorCopy(myvector);
  std::vector otherNumbers {1, 2, 3};
  
  std::random_shuffle(myvectorCopy.begin(), myvectorCopy.end());

  std::cout << std::boolalpha;  
  std::cout << "myvectorVectorCopy is a permutation of myvector: " 
            << std::is_permutation(myvectorCopy.begin(), myvectorCopy.end(), 
                                    myvector.begin()) << '\n';
  std::cout << "otherNumbers is a permutation of myvector: " 
            << std::is_permutation(otherNumbers.begin(), otherNumbers.end(), 
                                   myvector.begin(), myvector.end()) << '\n';
}
```

*We already learnt about [std::random_shuffle here](https://www.sandordargo.com/blog/2020/12/09/stl-alogorithms-tutorial-part-15-shuffle)*

## `next_permutation`

A range of elements has a finite number of permutations. But which one is next? `std::next_permutation` (and also `std::prev_permutation`) considers that these permutations should be [lexicographically oredered](https://www.sandordargo.com/blog/2021/09/22/stl-alogorithms-tutorial-part-25-comparison-operations#lexicographical_compare).

`next_permutation` will change the received range to its next permutation if it's possible and return `true`. If the input range is already the last permutation considering the lexicographical order of permutations, then the return value will be `false` and the range will be set back to the lexicographically first permutation.

The lexicographically first permutation is the same as the sorted container.

This algorithm takes two iterators denoting the first and last position of the range and you can also pass in a custom comparator to replace the default `operator<`.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

void printVector(std::vector<int> v) {
  for(auto n : v) {
    std::cout << n << " ";
  }
  std::cout << '\n';
}

int main () {
  std::vector numbers {1, 2, 3, 4, 5};
  std::vector<int> reverseNumbers;
  std::reverse_copy(numbers.begin(), numbers.end(), std::back_inserter(reverseNumbers));
  
  std::cout << std::boolalpha;
  printVector(numbers);
  std::cout << next_permutation(numbers.begin(), numbers.end()) << '\n';
  printVector(numbers);
  
  std::cout << '\n';
  
  printVector(reverseNumbers);
  std::cout << std::next_permutation(reverseNumbers.begin(), reverseNumbers.end()) << '\n';
  
  printVector(reverseNumbers);
  std::cout << std::is_sorted(reverseNumbers.begin(), reverseNumbers.end()) << '\n';
}
```

## `prev_permutation`

`std::prev_permutation` is very similar to `std::next_permutation`. The only difference is that it transforms the passed in range not to the next, but to the previous permutation.

And when there is no previous permutation lexicographic-wise, then the return value is `false` and the container will be transformed to its last lexicographically permutation.

The lexicographically last permutation is the same as the sorted then reversed container.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

void printVector(std::vector<int> v) {
  for(auto n : v) {
    std::cout << n << " ";
  }
  std::cout << '\n';
}

int main () {
  std::vector numbers {1, 2, 3, 4, 5};
  std::vector<int> reverseNumbers;
  std::reverse_copy(numbers.begin(), numbers.end(), std::back_inserter(reverseNumbers));
  
  std::cout << std::boolalpha;
  printVector(reverseNumbers);
  std::cout << prev_permutation(reverseNumbers.begin(), reverseNumbers.end()) << '\n';
  printVector(reverseNumbers);
  
  std::cout << '\n';
  
  printVector(numbers);
  std::cout << std::prev_permutation(numbers.begin(), numbers.end()) << '\n';
  
  printVector(numbers);
  std::cout << std::is_sorted(numbers.begin(), numbers.end(), std::greater<int>()) << '\n';
}
```

It's worth noting the small trick of how to check if a container is reversely sorted! The default comparison operator for `std::is_sorted` is `std::less<T>` and it has to be replaced by `std::greater<T>`.

## Conclusion

This time, we learned about permutation algorithms. With them, we can check either if a range is a permutation of another and we can also transform a range into its next or previous permutations.

We finished discussing all the algorithms defined in the `<algorithm>` header, and we already started with the `<numeric>` header, so we will continue exploring it the next time.

Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
