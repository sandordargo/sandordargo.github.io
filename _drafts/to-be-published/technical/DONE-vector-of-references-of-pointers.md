---
layout: post
title: "Storing references of pointers in containers in C++"
date: 2021-X-X
category: dev
tags: [cpp, tutorial, lifetimemanagement, pointers]
excerpt_separator: <!--more-->
---
This article is about the problem of storing vectors in a container and a bug I faced recently.
<!--more-->
Many would quickly find a conclusion that you should not store raw pointers, but you should work with smart pointers. I think they are right. When you have issues with dangling pointers, with lifetime and ownership, that's an indication that you should have chosen a smarter way to manage your pointers.

Many would argue that you also have architecture issues, if you face such problems. Again, they are right.

Meanwhile, when you are working on a huge and old codebase, you don't necessarily have the freedom of updating dozens of components to meet such expectations.

Let's assume that we have a container of pointers. We add elements to it not at construction time, just to emulate a realistic scenario where pointers are added later: 

```cpp
#include <vector>
#include <iostream>


int main() { 
  std::vector<int*> numbers;
  
  int* a = new int{42};
  numbers.push_back(a);
  
  int* b = new int{51};
  numbers.push_back(b);
  
  int* c = new int{66};
  numbers.push_back(c);
  
  for (auto* n : numbers) {
    std::cout << *n << '\n';
  }
}
```

What can go wrong?

Many things and we'll see some simplistic examples.

Let's say we want to delete one of the pointers, and we do that.

```cpp
#include <vector>
#include <iostream>


int main() { 
  std::vector<int*> numbers;
  
  int* a = new int{42};
  numbers.push_back(a);
  
  int* b = new int{51};
  numbers.push_back(b);
  
  int* c = new int{66};
  numbers.push_back(c);
  
  delete numbers[1];
  
  for (auto* n : numbers) {
    std::cout << *n << '\n';
  }
}
/*
42
585960360
66
*/
```
We still have three outputs and `585960360` is not exactly what we wanted.

You might add a guard statement in the for loop to skip an iteration, in case you get a `nullptr`, but it won't help.

```cpp
for (auto* n : numbers) {
  if (n == nullptr) { continue; }
  std::cout << *n << '\n';
}
```

After deletion, we didn't set the pointer to `nullptr`.

```cpp
#include <vector>
#include <iostream>


int main() { 
  std::vector<int*> numbers;
  
  int* a = new int{42};
  numbers.push_back(a);
  
  int* b = new int{51};
  numbers.push_back(b);
  
  int* c = new int{66};
  numbers.push_back(c);
  
  delete numbers[1];
  numbers[1] = nullptr;
  
  for (auto* n : numbers) {
    if (n == nullptr) { continue; }
    std::cout << *n << '\n';
  }
  std::cout << '\n';
  std::cout << numbers.size() << '\n';
}
/*
42
66

3
*/
```

Now it's better, we indeed skipped the second number, but from our last line we can still see that even though we deleted a pointer, the size of the vector haven't changed.

We deleted a number, but not the element of the vector.

To complete the removal, if that's what we wanted, we have to erase the pointer from the vector:

```cpp
  delete numbers[1];
  numbers[1] = nullptr;
  numbers.erase(numbers.begin()+1);
```
Note, that `erase` doesn't accept an index, it takes an iterator. If we run the full example, we can see that know size of our vector is down to 2.

> And even though since C++20 we have [std::erase](https://dev.to/pgradot/lets-try-c20-erase-elements-in-a-container-with-stderase-2d08) at our disposal to erase an element right away based on its value, it won't free the memory, it won't delete the pointer, so by using it we face a memory leak.

The takeaway?

If you want to delete an item of a vector, don't forget to set it to `nullptr` after the destruction so that you can detect in other parts of the code that it got deleted. If you also want to remove it from the container, don't forget to erase it.

Now let's go to another sort of problematic. Let's have a look at a part of our example.

```cpp
std::vector<int*> numbers;
  
int* a = new int{42};
numbers.push_back(a);
```

So we store raw pointers. Who owns those raw pointers? Well, nobody knows. Maybe the same entity that owns the `numbers` vector, maybe the same who created the pointers. In the above example it's the same function, it's not necessarily the case.

What if a pointer is deleted not through a vector but by the original owner?

To skip a couple of rounds, let's assume that we don't forget about setting the pointer to `nullptr` and that we have the `nullptr` guard in our for loop.

```cpp
#include <vector>
#include <iostream>

int main() { 
  std::vector<int*> numbers;
  
  int* a = new int{42};
  numbers.push_back(a);
  
  int* b = new int{51};
  numbers.push_back(b);
  
  int* c = new int{66};
  numbers.push_back(c);


  delete b;
  b = nullptr;

  for (auto* n : numbers) {
    if (n == nullptr) { continue; }
    std::cout << *n << '\n';
  }
  std::cout << '\n';
  std::cout << numbers.size() << '\n';
}
```

What do you think the results are?

It's something like this:

```
42
148114344
66
```

Which means that if you delete the original pointer, in the vector we don't know about it.

It makes perfect sense.

```cpp
#include <iostream>

int main() { 
  int* n = new int{66};
  int* n2 = n;
  
  std::cout << std::boolalpha;
  std::cout << "n is nullptr? " << (n == nullptr) << '\n';
  std::cout << "n2 is nullptr? " << (n2 == nullptr) << '\n';

  delete n;
  n = nullptr;
  
  std::cout << "n is nullptr? " << (n == nullptr) << '\n';
  std::cout << "n2 is nullptr? " << (n2 == nullptr) << '\n';

}
/*
n is nullptr? false
n2 is nullptr? false
n is nullptr? true
n2 is nullptr? false
*/
```

In this simplified example, `n2` is a copy of `n`. When we deleted `n`, we well destructed the entity that both `n` and `n2` pointed to. But it's only `n` that points to nowhere after, it's only `n` that was set to point to a `nullptr`. `n2` still points to the original memory address and it doesn't know that the object there has been already destructed.

If we go back to the previous example, as the `vector` contains only copies of the original pointers, during the interation there is no way to know that the original pointer was deleted.

What could be the way out of this madness?

Obviously the best would be to avoid using the `new` keyword and work with smart pointers. Either with `std::unique_ptr` or `std::shared_ptr` we would not use `delete` anymore and we wouldn't have this problem.

Another option if for some reasons we cannot go with smart pointers could be to store references to the original pointers.

As such, when the original pointers are deleted and they are set to `nullptr`, in the vector we'd know exactly about it.

The only problem is that in C++ one cannot store references to pointers.

Try to compile this line:

```cpp
std::vector<int*&> v;
```
You'll get way too long error messages scattered with phrases such as `error: forming pointer to reference type 'int*&'`.

We wouldn't talk about C++ here if there wasn't a way to circumwent it.

Have you heard about `std::reference_wrapper`? It was introduced with C++11 and it is a class template that wraps a reference in a copyable and assignable object. It is frequently used as a help store references inside standard containers which cannot normally hold references. You can find it in the `<functional>` header

If you decide to store *wrapped* pointers, you will not have a problem anymore not knowing about the deletion of a pointed object. It's true the other way around as well. You can delete (and erase) an item from the vector and we'll know about it at the original call place too.

```cpp
#include <functional>
#include <vector>
#include <iostream>

int main() { 
  std::vector<std::reference_wrapper<int*>> numbers;
  
  int* a = new int{42};
  numbers.push_back(a);
  
  int* b = new int{51};
  numbers.push_back(b);
  
  int* c = new int{66};
  numbers.push_back(c);


  delete b;
  b = nullptr;

  for (auto n : numbers) {
    if (n == nullptr) { 
        std::cout << "nullptr found \n";
        continue; 
    }
    std::cout << *n.get() << '\n';
  }
  std::cout << '\n';
  std::cout << numbers.size() << '\n';
  
  delete numbers[2].get();
  numbers[2].get() = nullptr;
  std::cout << "c is " << (c == nullptr ? "nullptr" : std::to_string(*c)) << '\n'; 
}
```

It's worth noticing that if you have to access the pointer itself, you have to call `.get()` on the wrapper object.

We also have to remark that setting the deleted pointer to `nullptr` is cruical. If we forget about that, there is no way that we can check afterwards if it was destructed or not. You might have learnt that setting pointers to `nullptr` after delete just masks double delete bugs and leaves them unhandled. In this case, it's not masking a double delete bug, but still it helps to mask some lifetime management problems.

You might argue that this solution has a different semantics than storing the pointers and it's also different from storing the smart pointers.

And you're right about that.

Yet, given that you can insert items to a container of wrapper pointers the very same way compared to the container of the pointers, it's something for consideration.

It's a new tool in your toolbox, when you want to fix a legacy codebase where ownership and resource management is unclear and you have to limit the number of places where you modify the code.

What about `boost::ptr_vector` you might ask.

That's a story for another day.

## Conclusion

Today we saw some of the problematics caused by bad pointer lifetime management. When there is no clear owner you'll always run into troubles and it's even worse when you make copies of the pointers, for example by adding them to a container.

The best would be not to use dynamic memory allocations and then the second best option is to use smart pointers.

It might happen that you cannot commit to make such changes. Then it's a potential best effor solution to store references to the pointers. As such, even where we access the pointers from the container, we'll be aware if the pointer was destructed - given that it was set to `nullptr` after.

Don't get me wrong, I'm far from advocating for this solution. But it might help in some desperate situations.

In the coming weeks, we'll see how `boost::ptr_vector` might help us. And also what other kind of issues you have to deal with when you have vector of pointers as class members.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
