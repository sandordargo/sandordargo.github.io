---
layout: post
title: "Loops are bad, algorithms are good! Aren't they?"
date: 2020-5-13
category: dev
tags: [cpp, stl, algorithms, bestpractices]
excerpt_separator: <!--more-->
---
This is a statement frequently repeated by people who either just more familiar with the `<algorithms>` header in C++ and/or are advocates of functional programming in C++. And of course, let's not forget about the people who just repeat what others say without understanding the reasons behind.
<!--more-->

We shouldn't act like people who are just part of a herd. Even if a message is valid, we shouldn't just broadcast it because someone knowledgeable said so. We should understand why they are right.

Today, let's discuss the reasons usually mentioned to prove why the good old loops are considered worse than using predefined structures of the standard library.

1. If you have to write something a thousand times, there is a fair chance that you'll make some mistakes once in a while. On the other hand, if you use functions that were written before and used a million times, you won't face any bugs.
2. Algorithms have a better performance
3. Algorithms are more expressive

Are these points valid?

## Loops are error-prone

Few are humble enough to admit this. "I'm not a moron, I can write a simple for loop that will break whenever an element is found."

Until you can't. 

This is mostly not about your experience. It's about being human. If you do, you err. No matter what. You can put in place procedures that will limit the quantity and the scope of your mistakes, like having code reviews and unit tests, but you cannot eradicate the possibility of [screwing it up](http://sandordargo.com/blog/2020/02/26/what-to-do-when-screwed-up). 

Interestingly, these objections usually come from people who also complain that coding dojo exercises are too easy for them. People who claim cannot learn from [refactoring the gilded rose](http://sandordargo.com/blog/2018/08/08/gilded-rose-revisited).

Using a predefined structure, an algorithm is a lot about being humble and accept the wisdom of thousands if not millions.

## Algorithms have a better performance

This is only partially true. If we speak about C++, functions in the `<algorithms>` header are not optimized for corner cases. They are optimized for a certain portability between different systems and container types. You can use them on any STL container without knowing their exact type. As such, we cannot assume that they can take advantage of the characteristics of the underlying datasets. Especially that they don't operate directly on the containers, but through the iterators that give access to data behind. I say that we cannot assume, because in fact, very few people understand what is going on under the hoods of the compiler and you might find or write an implementation of the standard library that is much bigger than the usual ones, but optimized for each container type.

At the same time, chances are good that your for loops are not optimized either. And it's alright. Of course, as you write your loops, you are in control. You can optimize them, you can get the last cycles out of them. You cannot do the same with the already written functions of a library, even if it's the standard library. 

But honestly, most probably you don't need those last drops of performance. If you do, you are in a small minority and probably the standard implementation of the STL is not for you. But there are others, like the [Eastl](https://github.com/electronicarts/EASTL) focusing on performance.
In nominal cases, algorithms will provide better performance. In addition, since C++17 you can set [execution policies](https://en.cppreference.com/w/cpp/algorithm/execution_policy_tag_t) for the algorithms of the standard library.

In short, just by passing an optional parameter to an algorithm, you can parallelize the execution of it.

It's that simple:

```cpp
std::vector<int> v{0,9,1,8,2,7,3,6,4,5};
std::sort(std::par_unseq, v.begin(), v.end());
```

If you have access to the necessary hardware and compiler supporting parallel execution, try this new feature to have a better visibility on the possible performance gain!

## Algorightms are more expressive than loops

I truly believe so.

You can use algorithms in a more expressive way than `for` or `while` loops.

But it doesn't come automatically, there is no automation for this. You need some practice to find the good one.

Let's take an example.

In python, it's very easy to check if an element is in a list.

```python
isIncluded = searchedOne in collection
```

How would you do this in C++?

```cpp
bool isIncluded = false;
for (const auto& item : collection) {
  if (searchedOne == item) {
    isIncluded = true;
    break;
  }
}
```
And this is not the worst possible form as I already took advantage of the range based for loop.

While it's a bit verbose, it is also easy to understand. We loop through a collection and as soon as we found the element we were looking for, we break out of the loop. As I wrote, it's a bit lengthy, but otherwise, it's OK.

Let's see what happens, if use `std::find` instead.

```cpp
auto foundPosition = std::find(collection.begin(), collection.end(), searchedOne);
bool isIncluded = (foundPosition != collection.end());
```

The first thing we can observe is that it's terse, only two lines compared to the 7 we had earlier. And in fact, we could make all this a one-liner.

```cpp
auto isIncluded = (std::find(collection.begin(), collection.end(), searchedOne) != collection.end());
```

But this is just to show it's possible, not to say it's more readable than the 2 line version. Actually I think that the line version is optimal here.

On the first line, we search for the position of an element. If it's not part of the container, it'll point behind the last element, so at `std::vector<>::end()` meaning that it's not part of the collection.

In the second line, we just make the comparison between the result of find and `end` to see if we found what we've been looking for.

Recently in a code review, in the unit tests, I ran into a similar `for` loop. Similar, yet a bit different.

The difference was that it also contained a condition. Here is the original for loop:

```cpp
for (const std::string& key : keys) {
  std::string aValue;
  if (not iCache.read(key, aValue) || expectedValue != aValue) {
    return false;
  }
}
return true;
```

Without too much thinking, I just asked if we could use an algorithm, like `std::find_if`. The discussion went on and we came up with this code.

```cpp
auto found = std::find_if(keys.begin(), keys.end(),
    [&expectedValue, &iCache](const std::string& key) {
  std::string aValue;
  return not iCache.read(key, aValue) || expectedValue != aValue;
});
return found != keys.end();
```

It's not really shorter than the original code, probably it's even longer a bit. And while the variable name `found` is clear enough and the meaning of `std::find_if` is also straightforward, there is something that is difficult to understand. Maybe it's not doing the same thing as the original code. The lambda is our scapegoat. It's a bit complex. How could we do it better?

We could save and name the lambda, but first, let's just try to write down in plain English what do we want. If there is any key that we cannot find in the cache and whose value doesn't meet our expectations, we should return `false`, otherwise, we are fine.

In other words, in order to return `true`, there should be no element that doesn't match our expectations.

There should be no mismatch.

None of the elements should be a mismatch.

Bingo!

There is an algorithm exactly for that.

```cpp
auto valueMismatch = [&expectedValue, &iCache](const std::string& key) {
  std::string aValue;
  return (not iCache.read(key, aValue)) || expectedValue != aValue;
};
return std::none_of(keys.begin(), keys.end(), valueMismatch);
```

With this version, my colleague was convinced that it's better to use an algorithm than the original `for` loop.

The bottom line is that there is no magic algorithm to use instead of a for loop. But there something like 105 of them. Johnathan Boccara talked about all of them in about an hour.

If you know them and keep thinking for a bit, it's pretty sure that you'll find once matching your use case and you can make your code more expressive.

## Conclusion

It's important to understand why something is better than the other option. It's not enough just to keep repeating others' opinions.

Today we saw, why algorithms are most of the time better than plain old for loops.

They are less error-prone than loops as they were already written and tested - a lot. Unless you are going for the last drops of performance, algorithms will provide be good enough for you and actually more performant than simple loops.

But the most important point is that they are more expressive. It's straightforward to pick the good among many, but with education and practice, you'll be able to easily find an algorithm that can replace a for loop in most cases.

Happy coding!