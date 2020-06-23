---
layout: post
title: "The big STL Algorithms tutorial: all_of, any_of, none_of"
date: 2019-2-20
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this first part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), I'll start with the first chunk of the non-modifying sequence operations.
<!--more-->

Namely, in this post, you are going to read about `all_of`, `any_of` and `none_of` functions.

Their names are quite intuitive and as you might suspect it, they all return booleans and they operate on STL containers.

Unless you use ranges (that should be part of another post), you don't pass them directly a container, but rather two iterators on the same container. Those iterators define the range the function will work on.

After the two iterators, you pass in a predicate. That predicate can be a function pointer or a function object (including [lambdas](http://sandordargo.com/blog/2018/12/19/c++-lambda-expressions)) returning a boolean or at least something that is convertible to boolean.

It means that the next code does NOT even compile:

```
#include <iostream>
#include <vector>
#include <algorithm>

int main()
{

  auto nums = {1,2,3,4,5,3};
  if (std::any_of(std::begin(nums), std::end(nums), 3) {
      std::cout << "there is a 3 in the list" << std::endl;
  } else {
      std::cout << "there is NOT ANY 3 in the list" << std::endl;
  }
    
}

```

Instead, let's see two implementation that works. The first one will use a function object:

```
#include <iostream>
#include <vector>
#include <algorithm>


class IsEqualTo {
public:
    IsEqualTo(int num) : m_num(num) {}
    
    bool operator()(int i) {
        return i == m_num;
    }
private:
    int m_num;
};

int main()
{

auto nums = {1,2,3,4,5,3};
if (std::any_of(std::begin(nums), std::end(nums), IsEqualTo(3))) {
      std::cout << "there is a 3 in the list" << std::endl;
  } else {
      std::cout << "there is NOT ANY 3 in the list" << std::endl;
}
    
}
```

It's a bit long, but thanks to the well-named functor (function object) it is easily readable.

Now let's have a look at bersion with a lambda expression:

```
#include <iostream>
#include <vector>
#include <algorithm>

int main()
{

  auto nums = {1,2,3,4,5,3};
  if (std::any_of(std::begin(nums), std::end(nums), [](int i){return i == 3;})) {
      std::cout << "there is a 3 in the list" << std::endl;
  } else {
      std::cout << "there is NOT ANY 3 in the list" << std::endl;
  }
    
}

```

This version is a lot shorter, a lot more dense and instead of the whole definition of our class `IsEqualTo` you have only this lambda expression: `[](int i){return i == 3;})`.

Which one is better to use? It depends on the context. You can read some details about this question and on how to write lambdas in C++ in general in [this article](http://sandordargo.com/blog/2018/12/19/c++-lambda-expressions).

Now, let's talk a little bit about what the 3 mentioned functions do, but maybe it's already clear to you.

### `std::all_of`

`std::all_of` will return true if the predicate is evaluated to true or can be converted to true for __all__ the items, false otherwise.

That _"can be converted"_ part means that the predicate doesn't have to return a boolean. It can return a number for example. But really anything that can be treated as a boolean.


### `std::any_of`

`std::any_of` will return true if the predicate is evaluated to true or can be converted to true for __any__ of the items, false otherwise. Meaning that if the predicate is true only for one element out of a hundred, `std::any_of` will return true.

### `std::none_of`

`std::none_of` will return true if the predicate is evaluated to true or can be converted to true for __none__ of the items, false otherwise. Turning it around, `std::none_of` return true if the predicate is false for __all__ the items! If there is at least one returning true, the function itself will return false.

## Conclusion

That's it for the first part. The three presented functions - `all_of`, `any_of` and `none_of` - can replace ugly loops with an if and a break inside your code, making it much more expressive and readable. Use them without moderation wherever you can and stay tuned for the next episodes!