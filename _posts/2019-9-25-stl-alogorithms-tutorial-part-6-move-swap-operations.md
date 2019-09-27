---
layout: post
title: "The big STL Algorithms tutorial: modifying sequence operations - move and swap"
date: 2019-9-25
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we will discover some of modifying sequence operations who involve either move or swap:
<!--more-->
* `move`
* `move_backward`
* `swap`
* `swap_ranges`
* `iter_swap`

## `move`

`move` is pretty similar to [`copy`](http://sandordargo.com/blog/2019/08/14/stl-alogorithms-tutorial-part-5-copy-operations), they both take two iterators defining an input range and one to mark the beginning of the output range.

While `copy` leaves the input intact, `move` will _transfer_ objects from one range to another. It uses the move semantics introduced in C++11 eleven, meaning that the algorithm itself is available since C++11.

What happens to the source objects is normally defined in its move assignment operator. But be aware that if for example the move assignment operator is not implemented, calling `std::move` on the object will not fail. You won't even get a compiler warning. Instead, the available assignment operator will be called.

The usage of `std::move` is a possibility, not something you can take for granted. Just to repeat, this means that if the compiler doesn't find an implementation for the move constructor/move assignment operator, then it will simply use the copy constructor/assignment operator.

With your types, you can control it, but in a big old codebase, you might not see or forget to check if move semantics are supported or not, you think you can use them and in fact, you don't. This might cost you some performance overhead you don't want to use.

Here is a sample example of how to use it.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
#include <string>
class A {
public:  
A(const std::string& a) : m_a(a) {
  // std::cout << "this is not a move but simple ctr\n";    
}  

A(const A& a) : A(a.m_a) {
  // std::cout << "this is not a move but copy ctr\n";
}   

A& operator=(const A& other) {    
  this->m_a = other.m_a;
  std::cout << "this is not a move but an assignment operator\n";
  return *this;
}   

A& operator=(A&& other) {    
  this->m_a = std::move(other.m_a);
  std::cout << "this is now move assignment\n";
  return *this;
}

std::string toString() const {
  return m_a;
}

private:
  std::string m_a;
};

int main() {  

  auto myVect = std::vector<A>{A("1"), A("2"), A("3"), A("4"), A("5")}; 
  auto outputVect = std::vector<A>{5, std::string("0")};
  outputVect.reserve(myVect.size());
  std::cout << "The content of myVect: ";
  for (const auto& a : myVect) {
    std::cout << a.toString() << " ";
  }  
  
  std::cout << "\n";
  std::cout << "The content of outputVect: ";
  for (const auto& a : outputVect) {
     std::cout << a.toString() << " ";
  }
  std::cout << "\n";

  std::cout << "LET'S MOVE\n";
  std::move(myVect.begin(), myVect.end(), outputVect.begin());
  std::cout << "MOVES are done\n";

  std::cout << "The content of myVect: ";
  for (const auto& a : myVect) {    
    std::cout << a.toString() << " ";
   }  
  std::cout << "\n";
  std::cout << "The content of outputVect: ";
  for (const auto& a : outputVect) {
    std:: cout << a.toString() << " ";
  }  
  std::cout << "\n";
  return 0;
}
```
As we discussed for [`copy`](http://sandordargo.com/blog/2019/08/14/stl-alogorithms-tutorial-part-5-copy-operations), the output range either has to provide enough space for the object you want to move into it, or you can also use an inserter operator. as its name suggests it will help you to add new elements to the output vector. You can use it like this:

```cpp
std::move(myVect.begin(), myVect.end(), std::back_inserter(outputVect));
```

In this case, you can simply use the default constructor when you create your output vector and/or the reservation of a big enough space for it.

A particular problem you might think off is that our output container is empty in the beginning and it grows and grows. In how many steps? We can’t really know in advance that’s an implementation detail of the compiler you are using. But if your input container is big enough, you can assume that the output operator will grow in multiple steps. Resizing your vector might be expensive, it needs memory allocation, finding continuous free areas, whatever.

If you want to help with that, you might use `std::vector::reserve`, which will reserve a big enough memory area for the vector so that it can grow without new allocations. And if the reserved size is not enough, there won’t be a segmentation fault or any other issue, just a new allocation.

What we could observe is that `std::move`, just like `std::copy`, doesn’t insert new elements on its own, but it overwrites existing elements in the output container. It can only insert if an inserter iterator is used.

## `move_backward`
`move_backward` is similar to `copy_backward`. This algorithm moves elements from the input range but starting from the back going towards the beginning.

Does it produce a reversed order compared to the input? No, it doesn't. It keeps order. So why does this `move_backward` exists? What is its use? The answer and the example are pretty much the same that the one for `copy_backward`.

Let's think about the following case.

We have an input range of `{1, 2, 3, 4, 5, 6, 7}` and we want to move the part `{1, 2, 3}` over `{2, 3, 4}`. To make it more visual:

```cpp
{1, 2, 3, 4, 5, 6, 7} => { , 1, 2, 3, 5, 6, 7}
```
So we try to use `std::move` and the output container is the same as the input.

You might try this code:
```cpp
#include <iostream>
#include <algorithm>
#include <vector>
int main () { 
 auto inputNumbers = std::vector<std::string>{"1", "2","3","4","5","6","7"};
 std::move(std::begin(inputNumbers), std::begin(inputNumbers)+3, std::begin(inputNumbers)+1);
 for (auto number : inputNumbers) {  
  std::cout << number << "\n";
 } 
 return 0;
}
```

The output might be different compared to what you expected - it depends on your expectation and compiler:

```cpp



1
5
6
7
```

So what happened?

First, the first number (`inputNumbers.begin()`) is moved over the second one (inputNumbers.begin()+1). So 2 is overwritten by 1 and the original 1 is cleared now. Then the second number (`inputNumbers.begin()+1`) is getting moved to the third (`inputNumbers.begin()+2`) position. But by this time, the second number is 1, so that's what will be moved to the third. And so on.

_(It is possible that you're using a compiler that is smart enough to overcome this issue)_

`std::move_backward` will help you to not to have this issue. First, it will move the last element of your input range and then it will one by one towards the first element, keeping the relative order in the output. Use `move_backward` when you move to the right and the input range is overlapping with the output one. Just keep in mind that when you use `std::move` as an output you add the first output position (from the beginning of the container) and with `std::move` you have to pass the last one.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
int main () { 
  auto inputNumbers = std::vector<std::string>{"1", "2","3","4","5","6","7"};
  std::move_backward(std::begin(inputNumbers), std::begin(inputNumbers)+3, std::begin(inputNumbers)+4);
  for (auto number : inputNumbers) {  
    std::cout << number << "\n";
  } 
  return 0;
}
```

## `swap`

`std::swap` doesn't hold many surprises for us. Is swaps the content of the two passed in variables. They can be of built-in types, containers, user-defined objects.

Before C++11, it used the copy constructor to create a temporary object and the copy assignment operator to do perform the assignments.

Starting from C++11 it takes advantage of move semantics when it's available.

Here is a very simple example of its usage:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
int main () { int x=42, y=51;
 std::cout << "Before swap x: " << x << ", y: " << y << "\n";
 std::swap(x,y);
 std::cout << "Before swap x: " << x << ", y: " << y << "\n";
 return 0;
}
```

## `swap_ranges`

`swap_ranges` takes three iterators as parameters. The first two defines one of the ranges to be swapped and the other range to be swapped is only characterized by its beginning. It makes sense as the two ranges should have the same length.

I wrote should, not must. 

If there is nothing to swap with, there is no error, no warning. We'll lose the what we swap out from our first range and instead, we'll get a default constructed object.

Which means that you `swap_ranges` can be dangerous if not used properly.

Here is an example you can play with:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
int main () { 
 std::vector<int> foo(5,10);
 std::vector<int> bar(5,33);
 // change the first parameter to get vector of differnt size
 std::cout << "BEFORE SWAP:\n";
 std::cout << "foo contains:";
 for (std::vector<int>::iterator it=foo.begin(); it!=foo.end(); ++it) {
  std::cout << ' ' << *it;
 }
 std::cout << '\n';

 std::cout << "bar contains:";
 for (std::vector<int>::iterator it=bar.begin(); it!=bar.end(); ++it)  {
  std::cout << ' ' << *it;
 }
 std::cout << '\n';

 std::swap_ranges(foo.begin(), foo.end(), bar.begin());

 std::cout << "AFTER SWAP:\n";
 std::cout << "foo contains:";
 for (std::vector<int>::iterator it=foo.begin(); it!=foo.end(); ++it)  {
  std::cout << ' ' << *it;
 }
 std::cout << '\n';

 std::cout << "bar contains:";
 for (std::vector<int>::iterator it=bar.begin(); it!=bar.end(); ++it)  {
  std::cout << ' ' << *it;
 }
 std::cout << '\n';

 return 0;
}
```

## `iter_swap`

`iter_swap` is very similar to swap, but while `swap` changes the contents of two elements, `iter_swap` changes the content of two iterators.

You can use the previous example to experiment, we just have to change one line to remove the superfluous argument and of course to change `swap_ranges` to `iter_swap`.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () { 
 
 std::vector<int> foo(5,10);
 std::vector<int> bar(5,33);
 // change the first parameter to get vector of differnt size
 std::cout << "BEFORE SWAP:\n";
 std::cout << "foo contains:";
 for (std::vector<int>::iterator it=foo.begin(); it!=foo.end(); ++it) {
   std::cout << ' ' << *it;
 }
 std::cout << '\n';

 std::cout << "bar contains:";
 for (std::vector<int>::iterator it=bar.begin(); it!=bar.end(); ++it)  {
  std::cout << ' ' << *it;
 }
 std::cout << '\n';


 std::iter_swap(foo.begin(), bar.begin());

 std::cout << "AFTER SWAP:\n";
 std::cout << "foo contains:";
 for (std::vector<int>::iterator it=foo.begin(); it!=foo.end(); ++it) {
   std::cout << ' ' << *it;
 }
 std::cout << '\n';

 std::cout << "bar contains:";
 for (std::vector<int>::iterator it=bar.begin(); it!=bar.end(); ++it)  {
  std::cout << ' ' << *it;
 }
 std::cout << '\n';

 return 0;
}
```

## Conclusion

Today, we had a peek into the algorithms that either perform move or swap operations on single elements or on containers. (Well, technically on iterators).

Next time we’ll start learning about the transform algorithm. Stay tuned!