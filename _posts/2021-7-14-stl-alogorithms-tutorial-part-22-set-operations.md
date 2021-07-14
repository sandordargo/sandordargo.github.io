---
layout: post
title: "The big STL Algorithms tutorial: set operations"
date: 2021-7-14
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we are going to talk about set operations on sorted ranges:
<!--more-->

* `includes`
* `set_difference`
* `set_intersection`
* `set_symmetric_difference`
* `set_union`

Before we start, it is worth mentioning that ***set*** operations do not mean that these operations are applied on containers of type `std::set`.

The *set* prefix simply means that these are operations on subsets of collections.

Let's have a look.

## `includes`

Yes, this one doesn't have the *set* prefix. Never mind.

`std::includes` in its simplest form takes 4 parameters, 4 iterators. The first two defining one range, and the second two another range.

This algorithm returns a boolean and returns `true` in particular if the second range is a subsequence of the first one.

Let's see a simple example.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector nums {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
  std::vector subsequece {3, 4, 5};
  std::vector subset {5, 4, 3};
  std::vector otherNums {42, 51, 66};
  
  std::cout << std::boolalpha;
  std::cout << "std::includes(nums.begin(), nums.end(), subsequece.begin(), subsequece.end()): " << std::includes(nums.begin(), nums.end(), subsequece.begin(), subsequece.end()) << '\n';
  std::cout << "std::includes(nums.begin(), nums.end(), subset.begin(), subset.end()): " << std::includes(nums.begin(), nums.end(), subset.begin(), subset.end()) << '\n';
  std::cout << "std::includes(nums.begin(), nums.end(), otherNums.begin(), otherNums.end()): " << std::includes(nums.begin(), nums.end(), otherNums.begin(), otherNums.end()) << '\n';
}

/*
std::includes(nums.begin(), nums.end(), subsequece.begin(), subsequece.end()): true
std::includes(nums.begin(), nums.end(), subset.begin(), subset.end()): false
std::includes(nums.begin(), nums.end(), otherNums.begin(), otherNums.end()): false
*/
```

We can observe that in order to get a positive result from the algorithm, the second range must be a subsequence of the first one. Having the elements to be a subset of the first container is not enough.

What would happen if the first container would not be sorted?

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector nums {1, 2, 5, 4, 3, 6, 7, 8, 9, 10};
  std::vector subseq {5, 4, 3};
  
  std::cout << std::boolalpha;
  std::cout << "std::includes(nums.begin(), nums.end(), subseq.begin(), subseq.end()): " << std::includes(nums.begin(), nums.end(), subseq.begin(), subseq.end()) << '\n';
}
/*
std::includes(nums.begin(), nums.end(), subseq.begin(), subseq.end()): true
*/
```

We can see that our first range is not ordered, but `std::includes` was able to find a subsequence in it. Yet, you should not rely on this. If you don't pass sorted ranges to `std::includes`, the behaviour is undefined.

`std::includes` can take two extra parameters, I'd say the usual ones.

Before all others, it can take an execution policy and at the last position, it can a custom comparator in the form of a function pointer, function object or lambda expression to compare items of the two passed in containers.

## `set_difference`

This algorithm takes 2 ranges and will copy all the elements from the first range that is not in the second range to a destination range.

Just like each algorithm in this article, `set_difference` is only guaranteed to work with sorted ranges.

As we could already get used to it, the two input ranges are taken by a pair of iterators and the output range is only denoted by its beginning point. As usual, it's the caller's responsibility to make sure that the destination range can accommodate enough items. You can also pass an inserter iterator.

`std::set_difference` can also take the usual two extra parameters, like an execution policy before all the others or a comparator after all the parameters.

Let's have here an example:

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector nums {1, 2, 3, 4, 5, 5};
  std::vector otherNums {1, 2, 3, 6, 7};
  std::vector<int> difference;
  
  std::set_difference(nums.begin(), nums.end(), 
                      otherNums.begin(), otherNums.end(),
                      std::back_inserter(difference));
  for (auto n : difference) {
    std::cout << n << " "; 
  }
  std::cout << '\n';
}
/*
4 5 5
*/
```

It's worth noticing that if the same value appears multiple times in the first container but never in the second, then it will be copied multiple times into the output range.

In the above example, we had `5` twice in `nums` and not at all in `otherNums`, so it appears twice in `difference`. But if `5` appears once in `otherNums` too, it will still appear in the `difference`, but then only once. After all, that's the difference. If it appears twice in the first input and only once in the second, that is the difference.

## `set_intersection`

`set_intersection` takes the same parameters as `set_difference`.

Two pairs of iterators as input, an output iterator an optional execution policy and a comparator.

It will copy each element to the destination range that is both in the input and the output range.

If a value appears multiple times in both ranges, it will be copied multiple times. To be more exact, if it appears in the first range `m` times and `n` times in the second, it will be copied `std::min(m,n)` times.

`std::set_intersection` also keeps the items in their relative order, the order of the items in the input and in the output range is the same.

Here are some examples:

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector nums {1, 2, 3, 4, 5};
  std::vector sameNums {1, 2, 3, 4, 5};
  std::vector otherNums {1, 2, 7};
  std::vector<int> intersectionOfSame;
  std::vector<int> otherIntersection;
  
  std::set_intersection(nums.begin(), nums.end(), 
                      sameNums.begin(), sameNums.end(),
                      std::back_inserter(intersectionOfSame));
  for (auto n : intersectionOfSame) {
    std::cout << n << " "; 
  }
  std::cout << '\n';
  
  std::set_intersection(nums.begin(), nums.end(), 
                      otherNums.begin(), otherNums.end(),
                      std::back_inserter(otherIntersection));
  for (auto n : otherIntersection) {
    std::cout << n << " "; 
  }
  std::cout << '\n';
}
/*
1 2 3 4 5 
1 2 
*/
```

## `set_symmetric_difference`

Regarding the possible parameters, we don't have a difficult job today. `set_symmetric_difference` still operates on the very same list of parameters as our previous two algorithms.

Two pairs of iterators as input, an output iterator an optional execution policy and a comparator.

What does computing a symmetric difference mean?

It means that in the output range you'll find all the elements that are found in either of the two input ranges, but not in both.

In a way, you can consider it that it's the combination of two `std::set_difference`, with the input ranges swapped between the two calls.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector nums {1, 2, 5, 6, 8};
  std::vector otherNums {3, 4, 7};
  std::vector<int> difference;
  std::vector<int> symmetricDifference;
  
  std::set_symmetric_difference(nums.begin(), nums.end(), 
                      otherNums.begin(), otherNums.end(),
                      std::back_inserter(symmetricDifference));
  for (auto n : symmetricDifference) {
    std::cout << n << " "; 
  }
  std::cout << '\n';
  
  std::set_difference(nums.begin(), nums.end(), 
                      otherNums.begin(), otherNums.end(),
                      std::back_inserter(difference));
  std::set_difference(otherNums.begin(), otherNums.end(),
                      nums.begin(), nums.end(), 
                      std::back_inserter(difference));
  for (auto n : difference) {
    std::cout << n << " "; 
  }
  std::cout << '\n';
}
/*
1 2 3 4 5 6 7 8 
1 2 5 6 8 3 4 7 
*/
```

The difference between calling `set_symmetric_difference` and calling `set_difference` - as you can see above - is that `set_symmetric_difference` will output a sorted range while calling `set_difference` twice will leave us with a container that has two sorted parts (the result of each call), but not sorted overall.

And anyway, the implementation of `set_symmetric_difference` is optimal for its purpose, unlike calling `set_difference` twice.

## `set_union`

If you followed through the previous sections, you won't encounter many surprises while learning about `set_union`. This algorithm takes two ranges and will build another out of the elements that are present in either one or the other container.

If an element can be found in both, then first all the elements will be taken from the first range and then if there were more elements with the same value in the second one, the excess will be copied from there.

Regarding the parameters, `set_union` behaves like the previous ones. It takes two pairs of iterators as input, an output iterator an optional execution policy and a comparator.

Let's see an example:

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector nums {1, 1, 2, 2, 5, 6, 8};
  std::vector otherNums {2, 5, 5, 7};
  std::vector<int> unionOfNums;
  
  std::set_union(nums.begin(), nums.end(), 
                      otherNums.begin(), otherNums.end(),
                      std::back_inserter(unionOfNums));
  for (auto n : unionOfNums) {
    std::cout << n << " "; 
  }
  std::cout << '\n';
}
/*
1 1 2 2 5 5 6 7 8 
*/
```

We can observe that those items that only appear in one of the inputs appear exactly the same times in the output. We have two values that appear in both inputs.

`2`, appears twice in the first input and once in the second. So it's taken twice from the first, and there is no excess in the second, so we are done.

`5` appears once in the first, so it's taken once from there and then there is one more item in the second input (2-1==1), so one more is taken there.

You might ask, why don't we say that it's just taken twice from the second range. Because that's what the specs say and there is a good reason behind it. The fact that two values are considered equal after comparison doesn't mean that they are identical. We'll have a look at this the next time based on Walter Brown's talk about the Italian C++ Conference 2021.

## Conclusion

This time, we learned about set operations on sorted ranges, which work on any containers not only on sets. The term set is used in its mathematical sense, it's not referring to the type of containers. Apart from that, they are quite logical, they don't have lot's of surprises, but we have to keep in mind especially for unions and intersections that items that are equal are not necessarily identical and it does matter which equal element we take.

Next time we'll discover heap operations. Stay tuned.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
