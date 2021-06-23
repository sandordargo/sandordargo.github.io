---
layout: post
title: "The big STL Algorithms tutorial: merge and inplace_merge"
date: 2021-6-23
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we are going to talk about two operations merge on sorted ranges:
<!--more-->

* `merge`
* `inplace_merge`

# `merge`

`std::merge` takes two sorted input ranges, merges them and returns an iterator that points past the last copied element.

Not let's see the details.

The first 4 parameters are input iterators denoting the 2 input ranges. Pass the `begin()` and `end()` iterators of the first range, then the `begin()` and `end()` iterators of the second range.

Both of the ranges must be sorted, otherwise, the behaviour is undefined. `merge` will not take care of sorting its inputs.

As a fifth parameter, an iterator to the output range is passed. You have to make sure that the output range has enough space to accommodate all the elements from the two inputs. Either you have to reserve enough space for the vector in the memory by zero initializing enough elements:

```cpp
auto results = std::vector<int>(input1.size() + input2.size());
```
Or another option is that instead, you pass an inserter iterator such as `std::back_inserter()`. That will also take care of the job.

The sixth parameter is optional, you might pass as well a binary predicate, a comparator. 

There is another version of the constructors taking first an execution policy (since C++17).

Here is an example for using `merge` correctly:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
  std::vector nums {1, 3, 5, 8, 9};
  std::vector otherNums {2, 4, 6, 7, 10};
  std::vector<int> results;
  std::merge(nums.begin(), nums.end(), otherNums.begin(), otherNums.end(), std::back_inserter(results));
  for(auto n: results) {
    std::cout<< n << ' ';
  }
  std::cout << '\n';
}
/*
1 2 3 4 5 6 7 8 9 10 
*/
```

And here is another one, where one of the inputs is not sorted:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
  std::vector nums {1, 3, 5, 8, 9};
  std::vector unsortedNums {4, 2, 10, 7, 6};
  std::vector<int> results;
  std::merge(nums.begin(), nums.end(), unsortedNums.begin(), unsortedNums.end(), std::back_inserter(results));
  for(auto n: results) {
    std::cout<< n << ' ';
  }
  std::cout << '\n';
}
/*
1 3 4 2 5 8 9 10 7 6 
*/
```

The results of these examples might give us an idea of how the merge is implemented. First, the first elements of both inputs are compared (1 and 4) and the smaller is taken. Then the second of the first and the first of the second ranges are compared (3, 4), again the first is taken. Then the third and first elements are compared (5, 4), so the second is taken, and so on...

Indeed, we must make sure that the input ranges are well sorted.

# `inplace_merge`

`inplace_merge` takes two sorted ranges which are connected! As the documentation says, they must be consecutive and as a result, the whole range will be sorted.

The function doesn't return anything, it's a void function.

By default, `inplace_merge` takes 3 parameters.

As the first parameter, you should send the beginning of the first sorted range.

As the second parameter, you should pass the end of the first sorted range that is supposed to be also the beginning of the second sorted range.

Finally, in the end, you should pass in the end of the second sorted range.

Just as we saw for `merge`, `inplace_merge` can take an optional comparator as a fourth parameter and it also might an execution policy (since C++17) before all the other parameters.

This time, we don't have to pay attention to the size of the output range as it's implicitly the same as the input. Even the name of the function tells us that it's "inplace".

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
  std::vector nums {1, 3, 5, 8, 9, 2, 4, 6, 7, 10};
  std::inplace_merge(nums.begin(), nums.begin()+5, nums.end());
  for(auto n: nums) {
    std::cout<< n << ' ';
  }
  std::cout << '\n';
}
/*
1 2 3 4 5 6 7 8 9 10 
*/
```

We can observe that with `nums.begin()+5` the first sorted range containing elements `1, 3, 5, 8, 9` end and another sorted subrange `2, 4, 6, 7, 10` starts. As a result of `inplace_merge`, the full container is merged.

# Conclusion

This time, we learned about `merge` and `inplace_merge` algorithms. We saw how to merge two sorted containers into one and also how to merge two consecutive sorted ranges into one. 

Next time we'll discover set algorithms.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
