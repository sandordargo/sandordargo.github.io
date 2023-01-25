---
layout: post
title: "Object initialization and binary sizes"
date: 2023-1-18
category: dev
tags: [cpp, binarysizes, executables, compilation]
excerpt_separator: <!--more-->
---
Let's have this piece of code that generates a big binary.

```cpp
#include <array>

struct Node {
    int a = 1, b = 1;
};

std::array <Node, 10'000> a;

int main() {}
```

If you compile this piece of code, you'll get quite a big binary. Of course, the exact size depends on your platform, compiler, etc. On my computer, its size is around 99 KB.

```bash
clang++ -std=c++20 -O3 -stdlib=libc++ array-global-10k.cpp -o array-global-10k
printf "%'d\n" $(wc -c < "$1")
# 99,437
```

That makes sense as the size of `Node` is 8 bytes and the `a` array contains *10,000* of them, that's already 80K.

In any case, that binary is big considering what the program does. Let's take it apart and see what goes on.

## The choice of container

First of all, the binary became so big, because we have declared a big container with [static storage duration](https://www.sandordargo.com/blog/2022/05/18/scope-linkage-name). 

We can easily decrease the size of the binary if we use a different type of container or if we declare it with a different storage duration. 

### Container type

Anything that requires dynamic memory allocation will **not** make your binary huge as dynamic memory allocations cannot happen compile-time. Okay, now a `vector` can be `constexpr`, but only if it's deallocated by the end of a `consteval` function. It cannot survive, you cannot have it as a member.

So in practice, c-style arrays and `std::array`s can significantly increase your binary sizes, but others with dynamic memory allocation, such as `std::vector` or `std::list` or different kinds of associative containers cannot.

Let's see a few examples of binary sizes and compile times.

The above code with `std::array` generated a binary of 99K and it took 2.8 seconds to compile it 100 times.
The same with a `std::vector` generated a binary of 33-40K depending on the optimization settings (it made no difference for `std::array`) and the compile time was roughly the same. The runtime for this very simple code doubled. Running it a 100 times took 0.75 seconds instead of 0.39. It makes sense as the initialization is now completely part of the runtime.

Using a c-style array resulted in the very same binary size and had essentially the same runtime, but the compile time is much faster than with the other two options. My assumption is that it's basically because we don't have to include and link any headers. So what took 2.8 seconds with `std::array` and with `std::vector`, it only took 0.75 with a c-style array.

It's interesting, but I wouldn't replace `std::array` with c-style arrays.

|   Container   | Binary size  | Compile-time  | Run-time  |
|---------------|--------------|---------------|-----------|
|  std::array   | 99K          |   2.8         | 0.39      |
| c-style array | 99K          |   0.75        | 0.39      |
|  std::vector  | 33-40K       |   2.8         | 0.75      |

### Storage duration

If the variable is declared as global or `static`, it will have a static storage duration and it'll be part of the binary. Depending on whether it's `const` or not, initialized or not, it might go into different parts of the TEXT or DATA segment.

If you declare the `std::array` as a local variable, it'll be created on the go. Even if it's `const`. `constexpr` locals might be initialized at compile-time and therefore be part of the binary.

Let's see some numbers.

|   Container         | Binary size  | Compile-time  | Run-time  |
|---------------------|--------------|---------------|-----------|
|  global             | 99K          |   2.2         | 0.25      |
|  local              | 33K          |   2.2         | 0.87      |
|  local static       | 16-99K       |   2.2         | 0.89      |
|  local static const | 16-82K       |   2.2         | 0.84      |
|  local constexpr    | 16-99K       |   2.8         | 0.93      |

## The initialization

I found this a particularly interesting point. We saw that depending on the storage duration and the container type we can have executables with completely different sizes and characteristics. But there is one more thing you can do with this simple piece of code in order shrink the size of the executable.

You can change how it's initialized. Both members are initialized to non-default values. On the other hand, if you choose to default initialize them, the size will shrink.

To see what is behind, we should rather check the assembly code that we can get with the `-S` flag, e.g., with the command `clang++ -std=c++20 -stdlib=libc++ -S array-global-arbitrary.cpp`. When we initialize members with an arbitrary value (such as `1` in the above example), we will see this pattern in our assembly code in a great length:

```
    .section    __DATA,__data
    .globl  _a                              ; @a
    .p2align    2
_a:
    .long   1                               ; 0x1
    .long   1                               ; 0x1
    .long   1                               ; 0x1
    // ...
    .long   1                               ; 0x1
```

However, when we initialize our member `int`s with a default value, we'll only see one line:

```asm
.zerofill __DATA,__common,_segtree,3`200`000,2
```

This explains why the binary bloats in the function of the size of the array that we have to initialize. But when we zero-initialize the values, it doesn't matter anymore.

What was also interesting to see is that if I add another member, such as a `std::string`, this doesn't matter anymore, the size stays small(er) no matter how I initialize variables. I think the reason behind this is that `std::string`s involves dynamic memory allocations, so we cannot completely initialize our array at compile-time anymore. Looking at the assembly code we observe that lots of code is now in the TEXT segment.

On the other hand, if we add a third member that is of another type that doesn't require dynamic allocations, such as a `float` or a new class, such as `struct NoDynamic {int c=0; int d=0;};`, we can still observe the same difference. We end up with a very small assembly and binary with a nice `.zerofill` command.

Essentially, this is the same observation we made for containers. Wherever there is dynamic memory allocation, there is no initialization at compile-time, so our binary remains relatively small.

## Conclusion

Today we saw different factors that influence how much space an object takes up in our binary.

It depends on how we store it, them. If we allocate on the heap (either directly or through a container such as a vector), it doesn't change our binary size a lot. But if we allocate through c-style or a `std::array` involving no dynamic allocations, the size will grow as the number of objects allocated grows.

But it's not as simple as that, it also depends on the storage duration of the variable. Only variables with static storage duration will be initialized at compile time.

Last but not least, it also depends on the class itself, what kind of members it has. If initializing an instance of a type requires dynamic memory allocation, it cannot be done at compile time.

But as we also saw, as the size changes, compilation- and run-times might also change but so far we haven't seen strong correlations.

Next time, we'll still deal with very basic features of C++, we'll have a look at special functions of a class.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
