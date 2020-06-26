---
layout: post
title: "The big STL Algorithms tutorial: transform"
date: 2019-11-6
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we will discover only one algorithm, the `transform`. I find very important, plus it doesn't have variants like the `copy` or `move` algorithms. On the other hand, it has two quite distinct constructors. Let's check them one by one.
<!--more-->

* Unary `transform`
* Binary `transform`

## Unary `transform`

Unary `transform` is - let's say - the basic transformation algorithm. It does exactly what I would have expected from such a function. It takes a range of inputs, applies a given operation on each element and puts the results into an output range.

It's return value - just like for the other overloaded version - is an iterator pointing to right after last output element.

As a unary operator, as usual, you can pass a function pointer, a functor or a lambda expression. For the sake of brevity, I'll stick to the lambdas in the coming examples.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () { 
    
auto values = std::vector<int>{1,2,3,4,5};
auto doubleValues = std::vector<int>{};
std::transform(values.begin(), values.end(), doubleValues.begin(), [](int number) {return 2*number;});

std::for_each(doubleValues.begin(), doubleValues.end(), [](int number){ std::cout << number << "\n";});
return 0;
}
```

What happens if you run this? You'll get a very nice core dump due to a segmentation fault!
What does this mean in practice?

If you remember, we hit this problem already in the episode about [`std::copy`](http://sandordargo.com/blog/2019/08/14/stl-alogorithms-tutorial-part-5-copy-operations). `doubleValues` has been initialized to zero members, and there is simply not enough space in it to insert new elements.

There are two ways to resolve this. One is to reserve enough space for the vector in the memory by zero initializing enough elements. This is totally acceptable if you know how many elements you'd need and when zero initialization is cheap.

```cpp
auto doubleValues = std::vector<int>(values.size());
```

Another option, is that instead of `doubleValues.begin()`, you pass an inserter iterator such as `std::back_inserter()`. That will take care of the job.

Here is a working example:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () { 
    
auto values = std::vector<int>{1,2,3,4,5};
auto doubleValues = std::vector<int>{};
std::transform(values.begin(), values.end(), std::back_inserter(doubleValues), [](int number) {return 2*number;});

std::for_each(doubleValues.begin(), doubleValues.end(), [](int number){ std::cout << number << "\n";});
return 0;
}
```

This will work whatever size the output will be.

To gain some resources [we can preallocate some memory in our vector](https://stackoverflow.com/questions/7397768/choice-between-vectorresize-and-vectorreserve), but most of the time it won't make any difference.

## Binary `transform`

So what is a binary transformation? It means that the last parameter of the constructor will be a lambda (or functor, function, etc. as usual) that takes two inputs instead of one.

But from where that second parameter comes from?

From another input iterator! 

But while the first input range is defined by two iterators (begin and end), the second one is defined by only it's start point as it should have at least the same number of elements as the second one. What happens if the second range contains fewer elements? Nasty things that we'll see in another article. As a rule, keep in mind that always the first range should be the shorter/smaller one.

Let's see an example respecting the rules:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () { 
    
auto values = std::vector<int>{1,2,3,4,5};
auto otherValues = std::vector<int>{10,20,30,40,50};
auto results = std::vector<int>{};
std::transform(values.begin(), values.end(), otherValues.begin(), std::back_inserter(results), [](int number, int otherNumber) { return number+otherNumber; });

std::for_each(results.begin(), results.end(), [](int number){ std::cout << number << "\n";});
return 0;
}
```

In this example, you could see that we define two input ranges and our lambda expression takes two elements, one from the first and one from the second range.

Can you combine elements of different types?

Of course, you can as long as you respect the types of the containers.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () { 
    
auto values = std::vector<int>{1,2,3,4,5};
auto otherValues = std::vector<float>{10.1f,20.2f,30.3f,40.4f,50.5f};
auto results = std::vector<std::string>{};
std::transform(values.begin(), values.end(), otherValues.begin(), std::back_inserter(results), [](int number, float otherNumber) {return std::to_string(number+otherNumber);});

std::for_each(results.begin(), results.end(), [](const std::string& number){ std::cout << number << "\n";});
return 0;
}
```

In this example, we combined `int` and `float` elements and returned `string` ones. It works, but if you run the code you also received a nice example of why it's difficult to work with floating-point numbers when you need precision.

## Conclusion

Today, we learnt about the `transform` algorithm. It takes elements of one or two ranges and puts the results of the transformation into another container.

Next time we’ll start learning about the replace algorithms. Stay tuned!