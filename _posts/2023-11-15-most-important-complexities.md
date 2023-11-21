---
layout: post
title: "C++: The most important complexities"
date: 2023-11-15
category: dev
tags: [cpp, algorithms, complexities, jobhunt]
excerpt_separator: <!--more-->
---
It's been about a year since I started to work for Spotify. That time [I wrote a few articles about my job search experience](https://www.sandordargo.com/blog/2022/09/28/5-tips-to-find-your-next-job). Among others, I shared [how much I underestimated the importance of complexity analysis and the big-O notations](https://www.sandordargo.com/blog/2022/08/31/data-structure-and-algorithms-interviews) when it comes to senior roles.

My assumption was that knowing the space and time complexities of the different algorithms and operations on different data structures isn't something important. Dealing with such issues is rare in most jobs. Still, asking about complexities might be an easy way to rate someone's knowledge when it comes to candidates with no or short experience. On the other hand, when you're looking for experienced people, there should be many other more meaningful ways to understand someone else's skills. I still think that this is true, but I also realized that many big (and small) companies just go with the standard questions involving complexity analysis.

Therefore, finally, I collected some complexities and some tips to learn them.

## Communication over end results

The end result of your complexity analysis is important. Yet, it's secondary. It doesn't differ from other interview questions in this sense. The most important is clear communication. You have to share your thread of thoughts partly because it proves how good a communicator you are and partly because sharing your thoughts is insurance against your mistakes. If you have a mistake in your analysis, you'll still show that otherwise, your thinking was right. Whereas if you simply share a result and it's bad, you failed to share anything positive at all.

This is even more important if you know that you have holes in your knowledge - just like me. When you start analyzing a problem, tell what you know for sure and share the assumptions you have and start your analysis from there.

Most probably you'll understand how loops are composed, how they (don't) add up if they are sequential and how they influence the complexity of an algorithm if they are nested. At the same time, most probably there will be some container manipulations - either directly or through some standard functions - within the loops and you might lack some knowledge regarding the complexity of those calls. Sharing your assumptions will at least balance a bit of that lack of knowledge. In the worst case, you'll have a useful discussion with your interviewer.

When I was interviewing for a position at a code analysis company, I clearly failed the interview due to my lack of knowledge in this field, but at least I learned a few tips about how to approach complexity analysis in future interviews, which clearly helped me get an offer from Spotify.

## How to learn

If you want to learn complexity analysis, you might just revisit your CS studies where most probably it was covered in detail. An even better option with less fluff is to take a book such as [Cracking the Coding Interview](https://www.amazon.com/Cracking-Coding-Interview-Programming-Questions/dp/0984782850?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=57a2bee06dddc5cc8f05aa7e075379f7&camp=1789&creative=9325) and start reading it thoroughly and solving its exercises on a regular basis.

Ideally, you'd spend some time on it every day or at least once or twice a week. Even if you don't plan to interview in the near future.

Or maybe especially if you don't plan to interview in the near future!

I don't know about you, but I'm not a university student anymore who can pull in one-nighters just to pass an exam and forget about what I learned hoping that I won't need that crap for the rest of my life.

You never know what tomorrow will bring. Maybe you get laid off the next day, or you just realise that it's time to leave. Then it'll be a bit late to start preparing. It's better if you have already started your preparation!

But as I said many times, most people are lazy and disorganized. So what to do if you need to improve your complexity analysis skills as soon as possible before a C++ coding interview?

- identify the most important time/space complexities you should know
- go on the [C++ Reference](https://en.cppreference.com/w/) pages of the identified functions and methods and start memorising the complexities
- spend some time on this activity every day, consistency will do miracles
- even better if you revisit your CS basics when you learn about a complexity so that you understand why inserting into a `std::map` has the complexity of *O(log n)*. This will both make it easier for you to succeed in general complexity analysis tasks and it'll also make you realize that it's not so difficult to learn it, you just need some time.

In the next sections, I'll help you collect the most fundamental complexities.

## Notions

Before we get into discussing complexities, let's remind ourselves about the different notions.

`O(1)` stands for constant complexity, meaning that the complexity doesn't depend on the input size, it'll always be about the same. A variation of constant complexity is *amortized* constant complexity. It means that the runtime will be much slower once in a while, but very rarely, so overall, on average, that slowdown is amortized so we can still talk about a constant complexity.

Linear complexity `O(n)` means that as the input size grows, the necessary amount of operations grows the same way.

On the other hand, when we talk about logarithmic complexity `O(log n)`, the number of operations that need to be executed grows much slower than the input size.

When it comes to C++ standard containers and algorithms, these are the three notions that we use. Still, it's good to remind ourselves that there are also quadratic and exponential complexities. When we talk about quadratic complexity `O(n^2)`, the number of required operations is about the squared size of the input. That's quite bad. But not as bad as the exponential complexity `O(2^n)` where by each additional input element the necessary time or space doubles. *(While there are other complexities, these are the most widely used ones.)* 

## Containers and operations

First, let's see what are the most important containers you'll likely deal with in a coding interview, what are the underlying data structures and what are the related complexities. My goal is not to give you a deep analysis, just to provide you with the most necessary information, then you can do your own research.

### `std::array`

`std::array` is a fixed-size array, storing objects in contiguous memory locations.

- **accessing the first element**: with `front()` which has a complexity of `O(1)`
- **accessing the last element**: with `back()` which has a complexity of `O(1)`
- **accessing a random element**: with `at()` or with `operator[]` both have a complexity of `O(1)`

### `std::list`

`std::list` is a container that supports fast insertion and removal, but doesn't support fast random access. It is usually implemented as a doubly-linked list. `std::forward_list` is similar, but implemented with a singly-linked list, so it's more space efficient, but it supports iteration only in one direction

- **accessing the first element**: with `front()` which has a complexity of `O(1)`
- **accessing the last element**: with `back()` which has a complexity of `O(1)` (not supported by `std::forward_list`)
- **accessing a random element**: not supported

- **inserting at the front**: with `push_front()` which has a complexity of `O(1)`
- **inserting at the back**: with `push_back()` which has a complexity of `O(1)` (not supported by `std::forward_list`)
- **inserting at a random location**: with `insert()` which has a complexity of `O(1)` for one element, and complexity of `O(n)` for multiple elements, where `n` is the number of elements to be inserted (`insert_after` for `std::forward_list`)

- **removing an item from the front**: with `pop_front()` which has a complexity of `O(1)`
- **removing an item from the back**: with `pop_back()` which has a complexity of `O(1)` (not supported by `std::forward_list`)
- **removing an item from a random location**: with `erase()` which has a complexity of `O(1)` for one element, and a complexity of `O(n)` for multiple elements, where `n` is the number of elements to be erased (`erase_after` for `std::forward_list`)

### `std::vector`

`std::vector` is a dynamically sized sequence container, where the elements are stored contiguously. Random access is cheap, as well as operations at the end, unless reallocation is required. 

- **accessing the first element**: with `front()` which has a complexity of `O(1)`
- **accessing the last element**: with `back()` which has a complexity of `O(1)`
- **accessing a random element**: with `at()` or with `operator[]` both have a complexity of `O(1)`

- **inserting at the front**: with `insert()` which has a complexity of `O(n+m)` where `n` is the number of elements to insert and `m` is the size of the container 
- **inserting at the back**: with `push_back()` which has a complexity of amortized `O(1)` 
- **inserting at a random location**: with `insert()` which has a complexity of `O(n+m)` where `n` is the number of elements to insert and `m` is the distance between the elements to insert and the end of the container

- **removing an item from the front**: with `erase()` which has a complexity of `O(n+m)` where `n` is the number of elements erased (calls to the destructor) and `m` is the number of assignments to make - the size of the elements left in the vector
- **removing an item from the back**: with `pop_back()` which has a complexity of `O(1)`
- **removing an item from a random location**: with `erase()` which has a complexity of `O(n+m)` where `n` is the number of elements erased (calls to the destructor) and `m` is the number of assignments to make - at least the number of elements after the last erased item, worst the size of the whole container left 

### `std::map`

`std::map` is a sorted associative container providing search, removal and insertion at a logarithmic complexity. They are usually implemented as red-black trees.

- **accessing an element**: with `at()` or with `operator[]` both have a complexity of `O(log n)` where `n` is the size of the container

- **inserting an element at a random location**: with `insert()` or with `operator[]` both have a complexity of `O(log n)` where `n` is the size of the container. With `insert()` you can insert multiple elements, and then the complexity becomes `O(m * log n)`, where `m` is the number of elements to insert. `insert()` can also take a position as a hint where to insert. If the insertion happens there then the complexity is amortized `O(1)` otherwise `O(log n)`

- **removing an item**: with `erase()` which has a complexity of amortized `O(1)` if the erasure happens with an iterator, otherwise it's `O(log(n) + m)` where `n` is the size of the container and `m` is the number of elements to remove
- **finding an element**: with `find()` which has a complexity of `O(log n)` where `n` is the size of the container

### `std::unordered_map`

`std::unordered_map` is an unsorted associative container optimized for search, removal and insertion which come at a constant time complexity. `std::unordered_map` is a hash map internally.

- **accessing an element**: with `at()` or with `operator[]` both have a complexity of `O(1)` on average and `O(n)` at worst where `n` is the size of the container

- **inserting an element at a random location**: with `insert()` or with `operator[]` both have a complexity of `O(1)` on average and `O(n)` at worst, where `n` is the size of the map. If `m` elements are inserted then the average case is `O(m)` and the worst case is `O(m * n + n)`
- **removing an item**: with `erase()` which has a complexity of amortized `O(1)` if the erasure happens with an iterator, otherwise, on average it's `O(m)` where `m` is the number of elements to remove, worst case it's `O(n)` where `n` is the size of the container 
- **finding an element**: with `find()` it's `O(1)` on average and in the worst case it's `O(n)` where `n` is the size of the container

## Algorithms

If you use raw loops and you understand the containers, you don't have to deal with these. A surprising "advantage" of using raw loops - [please, prefer algorithms](https://www.sandordargo.com/blog/2020/05/13/loops-vs-algorithms)!

Otherwise, most probably you understand what standard algorithms do. Think about them and you'll be able to come up with their complexities in most cases. Let's have a look at some algorithms:

- `all_of`/ `any_of` / `none_of` have at most `O(n)` complexity where `n` is the size of the range the algorithm is applied on 
- `count_if` has a complexity of `O(n)` where `n` is the size of the range the algorithm is applied on 
- `find` / `find_if` have a complexity of `O(n)`. They need at most `n` applications of `operator==` or a predicate where `n` is the length of the range passed in
- `replace` / `replace_if` have a complexity of `O(n)`. They need `n` applications of `operator==` or of a predicate where `n` is the length of the range passed in and at most `n` assignments
- `copy` / `copy_if` have a complexity of `O(n)`. `copy` does  `n` assignments where `n` is the length of the passed-in range, for `copy_if` we also have to think about the application of the predicate, while the number of assignments might be smaller. 
- `transform` also has a complexity of `O(n)`. It performs exactly `n` applications of the operation, where `n` is the length of the passed-in range. 
- `generate` has a complexity of `O(n)` as it invokes `n` times the generator function and also performs the same amount of assignments.
- `remove_if` has a complexity of `O(n)` as it performs `n` applications of `operator==` or of a predicate where `n` is the length of the range passed in.
- `swap` has a complexity of `O(1)` if applied on single values and `O(n)` if applied on arrays where `n` is the size of the arrays to be swapped
- `reverse` performs exactly half as many swaps as the size of the range to be reversed, therefore the complexity is `O(n)`
- `rotate` also has a complexity of `O(n)`.

Quite boring, right? But boredom brings simplicity to your calculations.

## Conclusion

In this article, we talked about the complexity analysis of operations on containers and of algorithms which are so often make important part of a software developer job interview. We discussed some hints on how to approach such questions if you neglected complexity analysis during most of your preparation for interviews. Finally, we quickly went through the most important complexities of C++ containers and standard algorithms so that you can have the most basic characteristics that you'd need at a job interview. Good luck!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!