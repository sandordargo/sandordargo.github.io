---
layout: post
title: "The evolution of statements with initializers in C++"
date: 2022-5-4
category: dev
tags: [cpp, ...]
excerpt_separator: <!--more-->
---
In these two articles, we see how C++ evolved in terms of writing different statements that include initializers. Simple? Boring? I don't think so, it just shows how far we got in C++ and in programming in general in terms of readability and maintainability of code.

[In this previous article](), we saw see how C++ evolved in terms of writing different statements that include initializers. Today, we discuss loops.

If you haven't read the first article, or if you don't remember, let me remind you wnat a statement is.

[According to Wikipedia](https://en.wikipedia.org/wiki/Statement_(computer_science)), "a statement is a syntactic unit [...] that expresses some action to be carried out". Statements can be either simple such as assignments, assertions or function calls, etc. or compound such as loops and if statements.

Now let's move on and see how loops evolved over the years and they offer - or not - variable initialization. Let's start by having a look at how we can write loops without init statements. Let's start with its earliest format.

## Loops with `goto` statements

Yes! The first way we are going to use here includes the infamous `goto` statement! Obviously, there are no init statements related to the `goto`. I hope you've never seen this in production code unless you're on the job market for decades.

In the old days of programming, the way to code loops was through labels and `goto` statements.

```cpp
#include <iostream>
#include <vector>

int main () {
    std::vector<int> v{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    size_t i = 0;
    
    LOOP:
        std::cout << v[i] << '\n';
        ++i;
    if (i < 10) {
        goto LOOP;
    }
    return 0;
}

```

Well, we can bring back the old days in C++ and we can still use `goto` statements and labels. There are a couple of issues though. The scope of `i` is not limited at all and even worse, we can *go to* `LOOP` from other places in the function, even when it wouldn't make sense. In other words, it's not safe and also difficult to reason about the correctness of our program. 

## Still no initialization: the `while` loop

The `while` keyword made loops a bit safer, as it's possible to jump into the body of the loop from random places, but as you cannot initialize a variable in its control block, limiting the scope of the loop index is still not possible, unless we embrace the loop and the index initialization between braces.

```cpp
#include <iostream>
#include <vector>

int main () {
    std::vector<int> v{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    size_t i = 0;
    
    while (i < 10) {
        std::cout << v[i] << '\n';
        ++i;
    }
    
    return 0;
}
```

Safety is not an issue as it was with the `goto`, but the scope of the index is not limited. In addition, we must take care of updating the loop variable as it's not something that can be done in the control block.

Despite these issues, a while loop still can be a good solution when you want to update the loop helper variable conditionally. For example, when you use the while loop with an iterator.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
 
int main() {
    std::vector<int> v = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
 
    auto it = v.begin();
    while (it != v.end())
    {
        if (*it % 2 == 1) {
            // As erase() invalidates the iterator, we use its returned value
            it = v.erase(it);
        } else {
            // Otherwise, we simply increment it
            ++it;
        }
    }
 
    for (const auto e: v) {
        std::cout << e << ' ';
    }
 
    return 0;
}
```

But to be fair, in most cases [you just want to use a standard algorithm](https://www.sandordargo.com/blog/2020/05/13/loops-vs-algorithms) and get rid off the problem of loops completely...

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
 
int main() {
    std::vector<int> v = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
 
    v.erase(std::remove_if(v.begin(), v.end(), [](auto e){return e % 2 == 1;}), v.end());
 
    for (const auto e: v) {
        std::cout << e << ' ';
    }

    return 0;
}

```

## The good old `for` loop

For loops are the remedy for all the issues we mentioned regarding loops. In its control block, we can create the loop variable, therefore its scope is limited and it also gets updated automatically after each iteration.

```cpp
#include <iostream>
#include <vector>

int main () {
    std::vector<int> v{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    for(size_t i = 0; i < 10; ++i) {
        std::cout << v[i] << '\n';
    }
    
    return 0;
}
```

What can go wrong? In the above case, we have a fixed end condition. If it wouldn't be fixed to `10`, but instead we called `v.size()`, it would mean a new function call before each iteration. And don't expect the compiler to optimize it away. I mean, there is no guarantee that the size of `v` wouldn't actually change and don't write bad code leaving it to the compiler to optimize it. So often you'll see that the end condition is fixed outside of the loop.

```cpp
// ...
const int vectorSize = v.size();
for(size_t i = 0; i < vectorSize; ++i) {
// ...
```

In the above examples, we had full control blocks. But it's not even necessary to use all the different parts of the control block, you can leave out the variable initialization or the increments. In fact, you can mimic a `while` loop with an initializer by using a `for` loop without the update part.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
 
int main() {
    std::vector<int> v = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
 
    for(auto it = v.begin(); it != v.end();)
    {
        if (*it % 2 == 1) {
            // As erase() invalidates the iterator, we use its returned value
            it = v.erase(it);
        } else {
            // Otherwise, we simply increment it
            ++it;
        }
    }
 
    for (const auto e: v) {
        std::cout << e << ' ';
    }
 
    return 0;
}
```

I don't think that this is a good idea. If you really need to decouple the increments from the control block and make it conditional, the purpose of your `for` loop is defeated and it would be better to use a different construct. Nevertheless, it's possible and maybe it's better than the `while` loop

You can also even leave out the exit condition and you'll get an infinite loop. In that case, you have to take care of the termination in a different way. Most probably you won't end up with a very idiomatic solution.

```cpp
#include <iostream>
#include <vector>

int main () {
    std::vector<int> v{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    for(size_t i = 0;  ;++i) {
        if (i >= 10) { break; }
        std::cout << v[i] << '\n';
    }
    
    return 0;
}
```

Now, just for the sake of fun and completeness, let's mention that you can even leave out all parts of the control block if you really wish so.

```cpp
#include <iostream>
#include <vector>

int main () {
    std::vector<int> v{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    size_t i = 0;
    for(;;) {
        if (i >= 10) { break; }
        std::cout << v[i] << '\n';
        ++i;
    }
    
    return 0;
}
```

These latter are kind of edge cases. Normally a simple `for` loop will suffice and you should strive not to use them. `for` loops give you safe updates and the limited scope of the loop index. So why would we need more?

## Range based `for` loop with initializer 

In most of the above examples, we interacted with the container through a loop index. But in reality, you'd often use an iterator. Before C++11, you couldn't use `auto`, you had to scrupulously type the full type of the iterator. The below one is simpler, but with some other - possibly template - types and with a `std::map` it could grow really long.

```cpp
#include <iostream>
#include <vector>

int main () {
    std::vector<int> v{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    
    for(std::vector<int>::const_iterator it = v.begin(); it != v.end(); ++it) {
        std::cout << *it << '\n';
    }
    
    return 0;
}
```

We already mentioned that storing the end, in other words, the sentinel value in a (`const`) variable makes sense, but to keep the control block shorter, many externalized also the iterator used for looping. With that, they ended up not limiting anymore the scope of the loop variable, thus they made the code less safe.

```cpp
#include <iostream>
#include <vector>

int main () {
    std::vector<int> v{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    std::vector<int>::const_iterator it;
    std::vector<int>::const_iterator end = v.end();
    for(it = v.begin(); it != end; ++it) {
        std::cout << *it << '\n';
    }
    
    return 0;
}
```

C++11 introduced range-based `for` loops to avoid this problem and provide a nicer way of iterating through containers where the coder doesn't have to dereference the iterator every time through `operator*`.

```cpp
#include <iostream>
#include <vector>

int main () {
    std::vector<int> v{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    for(const auto number: v) {
        std::cout << number << '\n';
    }
    
    return 0;
}
```

It's worth noting that under the hood the range-based `for` loop is translated into the iterator version with external iterator/sentinel definitions. This seems all fine and dandy and you might think that you won't ever again need anything else than range-based `for` loops. It's true in the vast majority of the cases (when you cannot replace it with a standard algorithm), you'll be better of with a range-based version. Yet you should do it with care, because [iterating over a reference to a temporary value is undefined behaviour](https://www.sandordargo.com/blog/2022/04/20/range-base-p2012).

Apart from that what do you do if you also need an index? It might happen that you really want comfortable access to each item but also their positions.

Check out [this article on FluentC++](https://www.fluentcpp.com/2018/10/26/how-to-access-the-index-of-the-current-element-in-a-modern-for-loop/), there are a couple of options such as using `boost::adaptors::indexed(0)`, calculating the distance from the beginning if you use iterators, or simply creating a loop index variable outside the loop, just like we did for `while` loops. It comes with the same implications for lifetime.

With C++20, we can have init statements even within a range based `for` loop's control block:

```cpp
#include <iostream>
#include <vector>

int main () {
    std::vector<int> v{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    for(size_t i = 0; const auto number: v) {
        std::cout << i << ": " << number << '\n';
        ++i;
    }
    
    return 0;
}
```

The only inconvenience is that we still have to increment that ourselves, but at least its scope is limited. It would be nice to have something as `enumerate()` in Python and within the standard library.

In any case, we can use init statements in range based `for` loops for other reasons too. Elaborating further [the example of Pierre Gradot](https://dev.to/pgradot/let-s-try-c-20-range-based-for-statements-with-initializer-3m6a), we can get rid of the undefined behaviour I referred to earlier by initializing a variable with the temporary:

```cpp
#include <iostream>
#include <string>
#include <vector>

std::vector<std::string> createStrings() {
    return {"This", "is", "a", "vector", "of", "strings"};
}

int main()
{
  for (auto w: createStrings()) {
      std::cout << w << " "; // this works fine
  }
  std::cout << std::endl;
  for (auto c: createStrings()[0]) {
      std::cout << c << " "; // this is UB
  }
  std::cout << std::endl;
  for (auto strings = createStrings(); auto c: strings[0]) {
      std::cout << c << " "; // this now works fine again
  }
  std::cout << std::endl;
}
```

## Conclusion

In the last two articles, we saw how C++ evolved in terms of providing conditional statements and loops with initializers. While one might think that these are just syntactic sugar and indeed, we could always achieve the same effects one way or another. But in reality, they help us write safer code that is better scoped, that exposes less or no undefined behaviour and in general make our code more expressive. The expressiveness, therefore the readability and maintainability of our code can never be overestimated.

I hope that C++ will keep evolving in this direction and also that we developers keep up with the changes and don't just stick to old practices.

Do you use `if`/`switch`/range based `for` loops with initializers? What do you do when you need them but you are stuck on an earlier version?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!