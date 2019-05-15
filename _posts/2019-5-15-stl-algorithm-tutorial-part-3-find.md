---
layout: post
title: "The big STL Algorithms tutorial: find et al."
date: 2019-5-15
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we will discover the different functions of the `<algorithm>` header that we can use to find an item in a container.
<!--more-->

Namely, we are going to examine the following functions:
* `find`
* `find_if`
* `find_if_not`
* `find_end`
* `find_first_of`
* `search`
* `search_n`
* `adjacent_find`

If you have a feeling that some functions are missing, you might think of `find_first_not_of` and similar functions. They are not part of the `<algorithm>` header but they are provided by the `<string>` header and as such, they operate on only strings. Thus, they are not part of this series.

## `find`
Our first function for today is `find` and it can be used to find an element a container by passing the container and the value to the `find` method.

It is as simple as that. It returns an iterator to the first element that matches the value we are looking for. In case of no elements matched, the iterator points at the end (after the last element) of the container.

```
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto myvector = std::vector<int>{1, 2, 3, 4, 5};

  auto it = std::find(myvector.begin(), myvector.end(), 3);
  if (it != myvector.end()) {
    std::cout << "Element found in myvector: " << *it << '\n';
  } else {
    std::cout << "Element not found in myvector\n";
  }

  return 0;
}
```

## `find_if`
The difference between `find` and `find_if` is that while find is looking for a value in the container, `find_if` takes a unary predicate and checks whether the predicate returns `true` or `false` to a given element. 

It will return an iterator pointing at the first element for which the predicate returns `true`. As usual, in case of no match, the iterator will point at the very end of the container.

A unary predicate can be a function object, a pointer to a function or a lambda function. [It depends on your use-case which one you should use.](http://sandordargo.com/blog/2018/12/19/c++-lambda-expressions)

```
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto myvector{1, 2, 3, 4, 5};

  auto it = find_if(myvector.begin(), myvector.end(), [](int number){return number % 2 == 0;});
  if (it != myvector.end()) {
    std::cout << "Even element found in myvector: " << *it << '\n';
  } else {
    std::cout << "No even element found in myvector\n";
  }

  return 0;
}
```

## `find_if_not`
Almost the same as `find_if`. But instead of the first match of the predicate in the given collection, it returns the first mismatch.

For demonstration purposes, let take our previous example and modify it only by adding a single _not_:


```
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  auto myvector{1, 2, 3, 4, 5};

  auto it = find_if_not(myvector.begin(), myvector.end(), [](int number){return number % 2 == 0});
  if (it != myvector.end()) {
    std::cout << "Even element found in myvector: " << *it << '\n';
  } else {
    std::cout << "No even element found in myvector\n";
  }

  return 0;
}
```

While the previous example with `find_if` returned all the even numbers, `find_if_not` with the same predicate would return all the odd numbers.

## `find_end`
You can use `find_end` to look for a subsequence in a container. As the `end` suffix implies, it will return something related to the last match. That something will be an iterator to the first element of the matching subsequence (which is the last matching subsequence). You can use it in two different ways. In the first example, the items are compared by values.

```
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  std::vector<int> numbers {1,2,3,4,5,1,2,3,4,5};

  std::vector<int> subsequence {1,2,3};

  
  auto it = std::find_end (numbers.begin(), numbers.end(), subsequence.begin(), subsequence.end());

  if (it!=numbers.end()) {
    std::cout << "needle1 last found at position " << (it-haystack.begin()) << '\n';
  }

  return 0;
}
```

The other possibily is to pass in a predicate as comparision function. Apart from using that one instead a _by value_ comparision, there is no difference:

```
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  std::vector<int> numbers {1,2,3,4,5,1,2,3,4,5};

  std::vector<int> subsequence {4,5,1};

  // using predicate comparison:
  auto it = std::find_end (numbers.begin(), numbers.end(), subsequence.begin(), subsequence.end(), [](int i, int j){return i == j;});

  if (it!=numbers.end())
    std::cout << "subsequence last found at position " << (it-numbers.begin()) << '\n';

  return 0;
}
```
As usual, the predicate can be any either a lambda, a function object or a function itself.

Personally what I found strange is that based on the name I would expect the same behaviour from `find_end` as from `find` apart from the direction of the search. From `find` I would expect the first match, from `find_end` the last one. Instead, `find` looks for one single value, but `find_end` tries to match a whole subsequence.

While you can use `find_end` make a subsequence of length one to look for the last matching element, you cannot use `find` to search for a subsequence.

## `find_first_of`
And now probably you expect that I'm going to present the function that looks for a subsequence from the beginning of a container. Sorry, but if you really expected that, I have to disappoint you.

`find_first_of` is similar to `find_end` in a sense that it either takes two pairs of iterators or two pairs of iterators and predicate. But what does it do with the inputs?

It will return an iterator to the first pair of iterators and to the first element that matches any of the elements of the second passed range or any of the elements of the second range for which the predicate evaluates to true.

Take the following example:

```
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  std::vector<int> numbers {1,2,3,4,5,1,2,3,4,5};

  std::vector<int> targets {4,5,2};

  // using predicate comparison:
  auto it = std::find_first_of (numbers.begin(), numbers.end(), targets.begin(), targets.end(), [](int i, int j){return i == j;});

  if (it!=numbers.end())
    std::cout << "first match found at position " << (it-numbers.begin()) << '\n';

  return 0;
}
```

The output will be

```
first match found at position 1
```

Let's check why. The first element of the `targets` is 4. Its first occurrence in `numbers` is at position 3 (starting from zero). The next element 5 can be found at position 4, the last element, 1 can be found at position 1. This means that it is 1 that can be found earliest in the `numbers` container.

## `search`

And here we go! Do you remember that `find_end` looks for the last match of a subsequence in a container? Here you have its counterpart that looks for the first one. For the sake of intuitivity (watch out, irony just passed by), it is called `search`!

Just like the previous two presented functions `find_end` and `find_first_of`, it can either take two ranges defined by two pairs of iterators or the same plus a predicate.

Here you have it in action.

```
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  std::vector<int> numbers {1,2,3,4,5,1,2,3,4,5};

  std::vector<int> subsequence {4,5,1};

  // using predicate comparison:
  auto it = std::search (numbers.begin(), numbers.end(), subsequence.begin(), subsequence.end(), [](int i, int j){return i == j;});

  if (it!=numbers.end())
    std::cout << "subsequence first found at position " << (it-numbers.begin()) << '\n';

  return 0;
}
```

## `search_n`

`search_n` can also compare by value or with the help of a predicate. It will look for `n` matching occurrences of the value or the value/predicate combination.

What it will return is an iterator pointing at the first matching element. If there is no match, as usual, the returned iterator will point right after the last element.

```
#include <iostream>
#include <algorithm>
#include <vector>

int main () {

  std::vector<int> myvector{10,20,30,30,20,10,10,20};
  
  auto it = std::search_n (myvector.begin(), myvector.end(), 2, 30);

  if (it!=myvector.end()) {
    std::cout << "two 30s found at position " << (it-myvector.begin()) << '\n';
  } else {
    std::cout << "match not found\n";
  }

  it = std::search_n (myvector.begin(), myvector.end(), 2, 10,  [](int i, int j){return i == j;});

  if (it!=myvector.end()) {
    std::cout << "two 10s found at position " << int(it-myvector.begin()) << '\n';
  } else {
    std::cout << "match not found\n";
  }

  return 0;
}
```

## `adjacent_find`

First I didn't intend to discuss `adjacent_find` in this episode, but later I felt it belongs more to here than to other topics. After all, it is also used to find elements.

Like we could get used to it, this another find method offers two overloaded signature, one that takes a predicate and one that doesn't. Besides that optional parameter, it only takes two iterators defining a range that it should iterate upon.

Unless you write the predicate as such, `adjacent_find` doesn't look for a particular value in a container. Rather, it looks for any two neighbouring elements that are matching, or any two elements next two each other satisfying a condition passed in with the predicate. An important note is that you have to do the test on both elements in the lambda as you're going to see in a minute.

_As usual_, it returns an iterator to the first matching element, in case of no match, to the end of the container.

We are going to see two examples on the same container. With the first call, we are going to return the first two adjacent matching elements and with the next call the first two neighbouring elements that are even.

```
#include <iostream>
#include <algorithm>
#include <vector>

int main () {

  std::vector<int> myvector{1, 0, 1, 1, 2, 3, 4, 6};
  
  auto it = std::adjacent_find (myvector.begin(), myvector.end());

  if (it!=myvector.end()) {
    std::cout << "two 1s found next to each other starting at position " << (it-myvector.begin()) << '\n';
  } else {
    std::cout << "no two equal elements found next to each other\n";
  }

  it = std::adjacent_find (myvector.begin(), myvector.end(), [](int i, int j){return (i % 2 == 0) && (j % 2 == 0);});

  if (it!=myvector.end()) {
    std::cout << "two adjacent even numbers found starting at position " << int(it-myvector.begin()) << '\n';
  } else {
    std::cout << "no two neighbouring equal numbers found\n";
  }

  return 0;
}
```

## Conclusion

In this article, we learnt about functions in the standard library that can be used to search for one or multiple elements in containers without ever modifying them.

We could also see some quirks of the STL. Like the unexpected differences between `find` and `find_end` and the non-matching name of the complementary `search` algorithms. But if you think more about it, it's also strange that `find_end`, `search` and `search_n` take a predicate as an optional parameter while `find` and `find_if` are different methods. I don't have te exact reason behind, but I think it's historical and the committee didn't want to change the existing API and neither wanted to overcomplicate the additional accepted new methods.

Regardless of all these oddities, the presented functions are more than useful and they should be part of each C++ developer's toolkit.

Keep tuned, in the next episode we'll discuss the rest of the non-modifying sequence operations..