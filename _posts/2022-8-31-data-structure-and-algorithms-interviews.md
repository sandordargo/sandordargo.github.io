---
layout: post
title: "Should you really memorize algorithm complexities?"
date: 2022-8-31
category: dev
tags: [career, watercooler, interview, learning]
excerpt_separator: <!--more-->
---
A few years ago I read one of Yegor Bugayenko's articles about his feelings about applying to a BigTech company. He wrote something like he would never be accepted and could never pass such interviews because he is not prepared for such nonsense and would not waste his time doing so.

I felt the same. I thought I was already a good programmer and I wouldn't need those. In any case, who wouldn't want to hire me? (Hint: in fact, many, way too many...)

I was partly wrong.

I realized that when I started to go through some interview processes with different companies.

It turned out quite fast that what I want from a job are things that many others want too. It must be remote, it must be a senior/senior+ position and it must pay a hefty salary.

Companies receive lots of applications for such positions and they must be able to select somehow. The competition is strong. They must be able to assess your skills. One relatively fast way of doing that is asking some questions about data structures and algorithms in an interview.

Let's see how I approached the first interviews and how my views changed.

## Algorithms

By the time I went to my first interviews, I could figure out many solutions to easy and medium [LeetCode problems](https://leetcode.com/problem-list/top-interview-questions/) on my own without using hints. I could analyze the space and time complexity of the solutions, although not in a very sophisticated manner. More on that just in a minute.

My experience is that many problems can be solved in C++ as a combination of standard algorithms. There are more than 100 of them, combining them is very powerful. Although [C++ Reference lists](https://en.cppreference.com/w/cpp/algorithm/binary_search) the *complexity* of each algorithm, I didn't always know them. You can imagine how it impacted my complexity analysis performance.

![Binary search complexity]({{ site.baseurl }}/assets/img/binary_search-complexity.png)

So how did I change my approach?

I still think that being able to implement the different sorts on a whiteboard is superfluous. You'll almost never need them. In most positions, using what the standard library of your language provides is a way better choice than implementing whatever sorting algorithm. And if one day you have to implement it, it's not a big deal to look up the algorithm in a book.

No sane interviewer should ask for it.

On the other hand, it is extremely useful to know the complexity of the different algorithms and their requirements. For example, if you want to decide in C++ whether a value is in a container or not, you might use `binary_search` or `find`. While `std::binary_search` is faster (*O(logN)* vs *O(N)*), it requires that you have a sorted container.

As sorting a container has the complexity of *O(N\*logN)*, it's also obvious that it's not worth sorting a container just so that you can binary search it.

Knowing the standard functions, their requirements and their complexities lets us decide which path to take in different situations. You don't have to be able to implement them on the spot. [(Even though implementing a simplified version of `find` with *O(N)* is really not complex)](https://github.com/llvm-mirror/libcxx/blob/master/include/algorithm#L912-L922)

## Data structures

The case for data structures though is a bit different. When it comes to algorithms, they are quite specific and you don't have plenty of different functions to achieve the same thing. With data structures, it's different. In the C++ STL you have quite a few:

- array
- vector
- list, forward_list
- deque, stack, queue, priority_queue
- set, multiset
- map, multimap
- unordered_set, unordered_multiset
- unordered_map, unordered_multimap
- span

Some of them are just adapters, but still. They are all there to store data. This list has 20 elements, while there are more than a hundred algorithms. I'd advise that you learn them well.

Understand how they are implemented. No, you don't have to be able to implement them with all their special cases, but know that a `list` is a doubly linked list, and know that `vector`s are dynamic size arrays with contiguous element storage. Know that a `map` is implemented as a tree, while an `unordered_map` is a hashmap.

Also focus a bit on learning how complex it is to access, insert or delete an element from these data structures. When you know the underlying data structure, most of these questions become quite easy given that you learnt about the structures once. If not, it's high time to do so.

I find this important because when you implement an algorithm on your own, it becomes way easier to analyze its complexity. You'll understand easily that finding an item in a map is *O(logn)*, while doing the same in a hashmap is constant on average.

Once you understand these complexities it also becomes simpler to understand when and why to use each of these data structures.

To learn all these, you don't need any fancy tool or platform. Browsing [C++ Reference](https://en.cppreference.com/w/cpp/container) and taking some notes of the complexities are more than enough.

## Conclusion

In this article, I shared with you how unimportant I found knowing details about algorithms and data structures before I started interviewing companies after working for 9 years at the same corporation. I was pretty much wrong. I still think that knowing how to implement all the algorithms would be an overkill and I would not focus on that while preparing, but knowing the complexities of some algorithms of your standard library is essential. 

What I found even more important is that you understand deeply the data structures offered by the standard library of your chosen language. Understand what kind of data structures are used for their implementations and the complexities of their operations.

Invest a few hours doing so and you'll significantly improve your chances to make good decisions to choose a data structure and nailing the complexity analysis at an interview.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
