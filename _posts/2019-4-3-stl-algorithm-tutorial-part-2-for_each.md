---
layout: post
title: "The big STL Algorithms tutorial: for_each"
date: 2019-4-3
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](/blog/2019/01/30/stl-algos-intro), I'll explain only one function. The `for_each` algorithm.
<!--more-->

What does it do?

`for_each` takes a range and a function to apply on each element of the given range.

[As we have seen](any-all-none), a range (unless you are using the ranges library) means two iterators describing the beginning and the end of a range.

The function must be unary, meaning that it should take one parameter that has the type of that given range element. Or at least it should be convertible to it (e.g. an int can be converted to a boolean).

But how to pass a function? What is a function in this context? 

It can be either the function itself or a function pointer, a function object or a lambda function.

Let's have all of them in the next example:

```
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>

void printElement(const std::string& i_element) {
    std::cout << i_element << "\n";
}

class ElementPrinter {
public:
    
    void operator()(const std::string& i_element) const {
        std::cout << i_element << "\n";
    }
};

int main () {
    
  std::vector<std::string> strings {"The", "best", "revenge", "is", "not", "to", "be", "like", "your", "enemy"};
  
  std::for_each(strings.begin(), strings.end(), printElement);
  std::for_each(strings.begin(), strings.end(), ElementPrinter());
  std::for_each(strings.begin(), strings.end(), [](const std::string& i_element) {
        std::cout << i_element << "\n";
  });
    
  return 0;
}

```

The first for_each takes a function. 

The second one takes an instance of a functor.

In the third case, we use a [lambda expression](http://sandordargo.com/blog/2018/12/19/c++-lambda-expressions).

Which one should you use? It really depends on your use case. Sometimes you'll need a one-off logic and you don't want to store it anywhere and you go with a lambda. In some other cases, you might use any of the previous two. For more details refer to [my introduction to lambda functions](http://sandordargo.com/blog/2018/12/19/c++-lambda-expressions).

If you go with a functor, pay special attention to [the rule of five](https://en.cppreference.com/w/cpp/language/rule_of_three). `for_each` needs functors to be move and copy constructible. Use a lambda and no such issues - everything needed is generated.

You should also note that it doesn't matter what the applied function returns, it will be omitted.

You might remember that `for_each` is a non-modifying sequence operation. Does it mean, that we cannot modify what we have in a sequence?

Let's give it a try!

```
#include <iostream>
#include <vector>
#include <algorithm>

int main () {
    
  std::vector<int> numbers {1,2,3,4,5};
  
  std::for_each(numbers.begin(), numbers.end(), [](int& i) {
        i = i * i;
  });
  
  for(auto num : numbers) {
    std::cout << num << "\n";
  }
    
  return 0;
}
```

What is the output?

```
1
4
9
16
25
```

So we could modify the list! We just had to pass the element to the function by reference. Great! But again, what about that non-modifying part?

You cannot modify the number of the elements in a container with for_each, you cannot add or delete elements, but you can modify the value of the given elements. Anyway, it would be quite difficult to iterate over a sequence that is being modified in its length during the iteration, right?

## The Alternatives

We've seen what `for_each` is used for, we've seen how to use it, but why should we use it? What are its alternatives?

### For loop with index

The good old way of iterating over a container. Sooooo uncool, isn't it?

```
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>

int main () {
    
  std::vector<std::string> strings {"The", "answer", "is", "within", "you"};

  for (size_t i=0; i<strings.size();++i) {
      std::cout << strings[i] << "\n";

  }
  
  return 0;
}
```

Well, coolness is not the issue. But handling the index in a for loop is tedious and not elegant. But if you need the index of an element, it's the goto option. Unless you have boost at your hands and [want to use something fancy](https://www.fluentcpp.com/2018/10/26/how-to-access-the-index-of-the-current-element-in-a-modern-for-loop/).

### For loop with iterators

You can use iterators to walk through a list. You don't have to take care of the index anymore!

```
#include <iostream>
#include <vector>
#include <string>

int main () {
    

  std::vector<std::string> strings {"Be", "tolerant", "with", "others", "and", "strict", "with", "yourself"};
  for (std::vector<std::string>::iterator it = strings.begin(); it != strings.end(); ++it) {
      std::cout << *it << "\n";

  }
  
  return 0;
}

```

Initializing the iterator is simply awful, isn't it? They have a long type, that's the reason. Besides iterators act like pointers, hence you need to dereference it if you want to get the value.

Since C++11, we can easily get rid off that awful iterator declaration by using the `auto` keyword.

```
#include <iostream>
#include <vector>
#include <string>

int main () {
    

  std::vector<std::string> strings {"Be", "tolerant", "with", "others", "and", "strict", "with", "yourself"};
  for (auto it = strings.begin(); it != strings.end(); ++it) {
      std::cout << *it << "\n";
  }
  
  return 0;
}

```

You see, it's not inconvenient anymore. But we have better.

### Range-based for loop

We used the `auto` keyword to omit the iterator's type at declaration time. But we can use that `auto` for an even better purpose.

```
#include <iostream>
#include <vector>
#include <string>

int main () {
    

  std::vector<std::string> strings {"The", "best", "revenge", "is", "not", "to", "be", "like", "your", "enemy"};
  for (auto element: strings) {
      std::cout << element << "\n";
  }
  
  return 0;
}
```

### Range based `for` loops vs. `for_each`

The main question is when we don't need the indexes, what should we use? A range based for loop or the `for_each` algorithm?

To me, the range based for loop is the _go to_ solution. On the other hand, it can be used only with the whole container, while with `for_each` it's up to you to specify the range you want to iterate over.

If you want to abstract out the logic that the loop has to perform on each element than using a `for_each` might be more elegant.

```
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>

void printElement(const std::string& i_element) {
    std::cout << i_element << "\n";
}

int main () {
    
  std::vector<std::string> strings {"The", "best", "revenge", "is", "not", "to", "be", "like", "your", "enemy"};

  std::for_each(strings.begin(), strings.end(), printElement);

  for(const auto& element: strings) {
    printElement(element);
  }
   
  return 0;
}

```

Which one reads better? Probably the first one. But it would not be worthwhile to use the `for_each` with a lambda.

```
std::for_each(strings.begin(), strings.end(), [](const std::string& i_element) {
    std::cout << i_element << "\n";
}]);
```

This doesn't read well. So the choice is mainly a question of abstraction. [Here](https://www.fluentcpp.com/2018/03/30/is-stdfor_each-obsolete/) you can read a deeper analysis on this topic.

## Conclusion

Today, we've seen the `for_each` algorithm which was a cool enhancement in the pre-C++11 times when we didn't have range-based `for` loops around. In comparison with it, it's not a default solution for looping over containers, but we still have its fair usage. And don't forget the pearls of wisdom of Marcus Aurelius and Seneca hidden in this post.

Keep tuned, in the next episode we'll discuss how to find items in a container.
