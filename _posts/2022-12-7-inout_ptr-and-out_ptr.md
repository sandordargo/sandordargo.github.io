---
layout: post
title: "C++23: std::out_ptr and std::inout_ptr"
date: 2022-12-7
category: books
tags: [cpp23, cpp, smartpointers, pointers]
excerpt_separator: <!--more-->
---
This week, let's continue exploring the new world of C++23. We are going to discuss two new standard library functions and their outputs (`std::out_ptr`, `std::inout_ptr`), two new standard library types (`std::out_ptr_t`, `std::inout_ptr_t`). 

## The motivation behind the new pointers

What's the reason behind introducing new pointer abstractions to C++? To be fair, if you're only dealing with modern C++, most probably you'll not need either `std::out_ptr` or `std::in_out_ptr`. But if you have to interact with C APIs, then you'll probably find them useful. Or maybe you even already knew them. Many companies, and projects have had their own implementations of these abstractions.

I remember from my university studies these function signatures where one parameter is "decorated" with multiple asterisks, such as `int** p_handle`. A pointer of a pointer? What sense does it make? Well, you barely need it in C++, but there are multiple answers to that question, One is that you want to change the address of the pointer it is pointing to.

To get a pointer of a pointer means that you have to get the address of a pointer:

```cpp
int* p = new int {42};
call_c_api(&p)
```

It just feels strange, doesn't it? It's not extremely readable and it only expresses intent on a very low level: it's reading the address of a pointer. But why? So some kind of a wrapper that does this for us - and probably even a bit more -, would probably represent intent better.

`out_ptr` and `in_out_ptr` does exactly that. They represent an intent and do even that "bit more" for us. They bring a bit more smartness by taking care of cleaning up after themselves.

It's time to get to the details.

## What do they offer?

The two new pointer types are part of the `<memory>` header. As mentioned they provide a more readable API to interact with C APIs. Both `std::out_ptr` and `std::inout_ptr` return pointers with a similar name (suffixed with `_t`). They are meant to be temporary objects that are destroyed at the end of the full expression where they were created. In practice, they are parts of function calls, where dangling references are avoided. They take a pointer and provide a pointer to that pointer. In the end, only the wrapper is destroyed, not the underlying pointer.

```cpp
#include <iostream>
#include <memory>

void old_c_api(int** p) {
  *p = new int{42};
}


int main() {
    auto pi = std::make_unique<int>(51);

    old_c_api(std::out_ptr(pi));

    std::cout << *pi << '\n';
}
```

The difference between the two pointer types is that while both types' destructor calls `reset()` on the stored smart pointer, `inout_ptr_t`'s constructor also calls `release()`. This means that if the API that you call already deletes a pointer, with `in_out_ptr`, you'll not run into double deletes as `release()` sets it to `nullptr` and deleting a `nullptr` is safe. 

Users can override these types to support their own smart pointers.

A few months ago [I wrote about smart pointers handle deleters](https://www.sandordargo.com/blog/2022/06/08/smart-pointers-and-deleters). In summary, deleters of `unique_ptr`s are part of their type, but deleters of `shared_ptr`s are not part of the type. `shared_ptr` erases the type of the deleter. This means that the authors of [P1132R8](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1132r8.html) had to consider whether a `shared_ptr` can be reset with another shared pointer with a different deleter. When an `out_ptr` is created the programmer must pass in the deleter too, so that at reset time, it's reset the same way.


```cpp
#include <iostream>
#include <memory>

void old_c_api(int** p) {
  *p = new int{42};
}


int main() {
    auto pi = std::make_shared<int>(51);

    // error C2338: static_assert failed: 'out_ptr_t with shared_ptr requires a deleter (N4892 [out.ptr.t]/3)'
    // old_c_api(std::out_ptr(pi));

    old_c_api(std::out_ptr(pi, std::default_delete<int>()));

    std::cout << *pi << '\n';
}
```

## Conclusion

Thanks to [P1132R8](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1132r8.html), C++23 introduces some nice new smart pointer types that help work together with existing C APIs that take pointers of pointers. These types do not come out of the blue, many companies had their own implementations. Now they become part of the standard and they make communication between C++ and C APIs more readable and also safer, especially with `std::inout_ptr` that makes sure there will be no double frees.

Still, I hope you won't have to use these a lot :)

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!