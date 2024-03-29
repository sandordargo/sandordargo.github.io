---
layout: post
title: "The big STL Algorithms tutorial: the rest of non-modifying sequence operations"
date: 2019-7-24
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we will discover all the non-modifying sequence operations that we haven't seen yet.
<!--more-->

Namely, we are going to have a deeper look at the following functions:
* `count`
* `count_if`
* `equal`
* `mismatch`
* `is_permutation`

## `count`
The name speaks for itself, right? `count` takes a range of iterators and as a third parameter, it takes a value that it'd look for in the passed in range. As simple as that

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto myvector = std::vector<int>{1, 2, 3, 1, 4, 5};
  auto count = std::count(myvector.begin(), myvector.end(), 1);
  std::cout << "Number of occurences of '1' in myvector: " << count;
  
  return 0;
}
``` 

Unsurprisingly the answer is 2.

## `count_if`
`count_if` differs from `count` the same way as `find_if` differs from `find`. Just like `count` (or `find`) it takes a range of iterators, but instead of a value as a third parameter, it takes a unary predicate and returns how many times the predicate evaluates to `true` by passing to it each element of the list.

A unary predicate can be a function object, a pointer to a function or a lambda function. [It depends on your use-case which one you should use.](http://sandordargo.com/blog/2018/12/19/c++-lambda-expressions)

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto myvector = std::vector<int>{1, 2, 3, 1, 4, 5};
  auto count = std::count_if(myvector.begin(), myvector.end(), [](int number){return number % 2 == 0;});
  std::cout << "Number of even numbers in myvector: " << count;
  
  return 0;
}
```

In this case, the answer will be again two. But while in or example for `count` number `1` was counted twice, here `2` and `4` were counted as even numbers.

## `equal`

`equal` function returns a boolean with its value depending on whether all the elements of two ranges are equal or not. That's the simple explanation, but life can be a bit different.

With the constructor that takes only the parameters, it is definitely the case. The first two iterators define a range and the third parameter defines the start of another range. Let's take a simple case, where we have two vectors with the same contents:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto myvector = std::vector<int>{1, 2, 3, 1, 4, 5};
  auto otherVector(myvector);
  if (std::equal(myvector.begin(), myvector.end(), otherVector.begin())) {
      std::cout << "The two vectors are equal\n ";
  } else {
      std::cout << "The two vectors are NOT equal\n ";
  }
  return 0;
}
```

Easy-peasy, our two vectors are equal. In our next example we insert an element at the beginning of the second vector and we try to ignore it from the comparison by not passing the `begin()` iterator, but an iterator of `begin()+1`. What should happen?

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto myvector = std::vector<int>{1, 2, 3, 1, 4, 5};
  auto otherVector(myvector);
  otherVector.insert(otherVector.begin(),42);
  if (std::equal(myvector.begin(), myvector.end(), otherVector.begin()+1)) {
      std::cout << "The two vectors are equal\n ";
  } else {
      std::cout << "The two vectors are NOT equal\n ";
  }
  return 0;
}
```

We start a comparison after the mismatching element, so if in the second vector we have the same number of elements afterwards as in the original vector and these elements match, `equal()` will also say so.

So what is going to happen if insert the same element at the end of the vector and we start the comparison from the beginning?

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto myvector = std::vector<int>{1, 2, 3, 1, 4, 5};
  auto otherVector(myvector);
  otherVector.push_back(42);
  if (std::equal(myvector.begin(), myvector.end(), otherVector.begin())) {
      std::cout << "The two vectors are equal\n ";
  } else {
      std::cout << "The two vectors are NOT equal\n ";
  }
  return 0;
}

```

Equal will return `true`! What? So what does equal do again? It checks if the first passed range is part of the second one starting from that specified point defined by the third parameter. It doesn't check if two containers are equal or not. For that, you can simply compare two containers.

There is a second constructor through which it's possible to pass a binary predicate as a comparator of the two ranges. Otherwise it works the same way.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto myvector = std::vector<int>{1, 2, 3, 1, 4, 5};
  auto otherVector(myvector);
  otherVector.push_back(42);
  if (std::equal(myvector.begin(), myvector.end(), otherVector.begin(), [](int i, int j){return i==j;})) {
      std::cout << "The two vectors are equal\n ";
  } else {
      std::cout << "The two vectors are NOT equal\n ";
  }
  return 0;
}

```

## `mismatch`

`mismatch` is quite similar to `equal`. It also exposes two constructors and you can choose among them based on the way you'd like to compare the two ranges that you pass in the same way as you did it for 'equal'.

The difference is that while `equal` returns an integer, mismatch returns a pair of iterators. An iterator to the first range and to the second one pointing at the positions of the first mismatch.

In case of failure, so in case of no mismatch, the iterator of the first range is points right after to its last element and the second iterator points to the second range at the same relative position as the first one. So in case the two ranges are equal, both points after the last element. When the first range is part of the second range, but the second range is longer, the second iterator points to the first element that is not in the first range.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto myvector = std::vector<int>{1, 2, 3, 4, 5};
  auto otherVector = std::vector<int>{1, 2, 3, 42, 5};
  auto result = std::mismatch(myvector.begin(), myvector.end(), otherVector.begin(), [](int i, int j){return i==j;});
  std::cout << "Mismatching elements are " << *result.first << " and " << *result.second << "\n";
  return 0;
}
```

## `is_permutation`

`is_permutation` is also similar to `equal`. It offers two constructors, both taking two ranges, the first one defined by its beginning and end while the other is only defined by its start point. And as we've seen with `equal` and `mismatch`, `is_permutation` also accepts an optional binary predicate that is used to compare the elements of the first and second ranges.

Like `equal`, `is_permutation` also returns a boolean which will be `true` in case all the elements match. But for `is_permutation` the order doesn't matter. It'll return `true` if the two queried ranges consist of the same elements regardless of their positions.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto myvector = std::vector<int>{1, 2, 3, 1, 4, 5};
  auto otherVector(myvector);
  std::random_shuffle(otherVector.begin(), otherVector.end());
  if (std::is_permutation(myvector.begin(), myvector.end(), otherVector.begin())) {
      std::cout << "The two vectors are permutations of each other\n ";
  } else {
      std::cout << "The two vectors are NOT permutations of each other\n ";
  }
  return 0;
}
```

More about random_shuffle in another post later, but given it's name, you can safely assume that it will shuffle the elements of a vector.

## Conclusion

In this article, we finished discussing the non-modifying sequence operations of the `<algorithm>` header. We saw how `count`, `count_if`, `equal`, `mismatch` and `is_permutation` work. 

Next time we'll start learning about the modifying sequence operations. Stay tuned!