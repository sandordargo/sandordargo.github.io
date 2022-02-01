---
layout: post
title: "The big STL Algorithms tutorial: the memory header"
date: 2022-2-2
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
We are slowly reaching the end of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), and in this last but one part we are going to cover a record high 14 operations that are part of the `<memory>` header. I decided to take all of them because they are quite similar to each other. 
<!--more-->

- `uninitialized_copy`
- `uninitialized_copy_n`
- `uninitialized_fill`
- `uninitialized_fill_n`
- `uninitialized_move`
- `uninitialized_move_n  `
- `uninitialized_default_construct`
- `uninitialized_default_construct_n  `
- `uninitialized_value_construct`
- `uninitialized_value_construct_n`
- `destroy`
- `destroy_n  `
- `destroy_at`
- `construct_at`
  
## `uninitialized_copy` / `uninitialized_copy_n`

`std::uninitialized_copy` takes an input range and copies the elements to an uninitialized area that is denoted by an iterator pointing at the beginning of the output range.

Potentially, you can also set the execution policy.

The only difference `std::uninitialized_copy_n` has compared to `std::uninitialized_copy` is that it doesn't take the input range by two iterators defining the beginning and the end of the input range, but instead it takes the beginning of the range and the size.

```cpp
#include <algorithm>
#include <iostream>
#include <memory>
#include <string>
#include <tuple>
#include <vector>

int main()
{
    std::vector<std::string> v = {"This", "is", "an", "example"};
 
    auto sz = std::size(v);
 
    if(void *pbuf = std::aligned_alloc(alignof(std::string), sizeof(std::string) * sz))
    {
        try
        {
            auto first = static_cast<std::string*>(pbuf);
            auto last = std::uninitialized_copy(std::begin(v), std::end(v), first);
 
            for (auto it = first; it != last; ++it) {
                std::cout << *it << ' ';
            }
            std::cout << '\n';
 
            std::destroy(first, last);
        }
        catch(...) {}
        std::free(pbuf);
    }
    
    
    std::string* p;
    std::tie(p, sz) = std::get_temporary_buffer<std::string>(v.size());
    sz = std::min(sz, v.size());
 
    std::uninitialized_copy_n(v.begin(), sz, p);
 
    for (std::string* i = p; i != p+sz; ++i) {
        std::cout << *i << ' ';
        i->~basic_string<char>();
    }
    std::return_temporary_buffer(p);
}
```
## `uninitialized_move` / `uninitialized_move_n`

`std::uninitialized_move` and `std::uninitialized_move_n` - unsurprisingly - work very similarly compared to their copy versions, but instead of copying the items from the input ranges, they move the items.

The range to be moved is either defined by two iterators denoting its beginning and end (`uninitialized_move`) or by an iterator to its beginning and the number of positions to fill (`uninitialized_move_n`).

The output range is defined only by its beginning as usual, and as a caller, we have to make sure that it can accommodate all the necessary elements to avoid undefined behaviour.

Before all the other parameters, we can also define an execution policy.

```cpp
#include <cstdlib>
#include <iomanip>
#include <iostream>
#include <memory>
#include <string>
 
void print(auto rem, auto first, auto last) {
    for (std::cout << rem; first != last; ++first)
        std::cout << std::quoted(*first) << ' ';
    std::cout << '\n';
}
 
int main() {
    std::string in[] { "Home", "Work!" };
    print("initially, in: ", std::begin(in), std::end(in));
 
    if (
        constexpr auto sz = std::size(in);
        void* out = std::aligned_alloc(alignof(std::string), sizeof(std::string) * sz)
    ) {
        try {
            auto first {static_cast<std::string*>(out)};
            auto last {first + sz};
            
            std::uninitialized_move(std::begin(in), std::end(in), first);
            // comment the previous line and uncomment the next one
            // to see uninitialized_fill_n in action
            // std::uninitialized_move_n(std::begin(in), sz, first);
 
            print("after move, in: ", std::begin(in), std::end(in));
            print("after move, out: ", first, last);
 
            std::destroy(first, last);
        }
        catch (...) {
            std::cout << "Exception!\n";
        }
        std::free(out);
    }
}
```

## `uninitialized_fill` / `uninitialized_fill_n`

`std::uninitialized_fill` and `std::uninitialized_fill_n` fills an uninitialized memory area with a given value.

The range to be filled is either defined by two iterators denoting its beginning and end (`uninitialized_fill`) or by an iterator to its beginning and the number of positions to fill (`uninitialized_fill_n`).

In both cases, the value comes after, and execution policy can also be defined.

```cpp
#include <algorithm>
#include <iostream>
#include <memory>
#include <string>
#include <tuple>
 
int main()
{
    std::string* p;
    std::size_t sz;
    std::tie(p, sz) = std::get_temporary_buffer<std::string>(4);
    
    std::uninitialized_fill(p, p+sz, "Example");
    // comment the previous line and uncomment the next one
    // to see uninitialized_fill_n in action
    // std::uninitialized_fill_n(p, sz, "Example");
 
    for (std::string* i = p; i != p+sz; ++i) {
        std::cout << *i << '\n';
        i->~basic_string<char>();
    }
    std::return_temporary_buffer(p);
}
```

## `uninitialized_default_construct` / `uninitialized_default_construct_n`

`std::uninitialized_default_construct` and `std::uninitialized_default_construct_n` fills an uninitialized memory area with the default initialized instances of the contained type.

The range to be filled is either defined by two iterators denoting its beginning and end (`uninitialized_default_construct`) or by an iterator to its beginning and the number of positions to fill (`uninitialized_default_construct_n`).

In both cases, the value comes after, and execution policy can also be defined.

```cpp
#include <iostream>
#include <memory>
#include <string>
 
struct S { std::string m{ "Default value" }; };
 
int main()
{
    constexpr int n {3};
    alignas(alignof(S)) unsigned char mem[n * sizeof(S)];
 
    auto first {reinterpret_cast<S*>(mem)};
    auto last {first + n};

    std::uninitialized_default_construct(first, last);
    // comment the previous line and uncomment the next one
    // to see uninitialized_default_construct_n in action
    // std::uninitialized_default_construct_n(first, n);

    for (auto it {first}; it != last; ++it) {
        std::cout << it->m << '\n';
    }

    std::destroy(first, last);
}
```

We should also note that `std::uninitialized_default_construct` and `std::uninitialized_default_construct_n` do not zero-fill the memory area for trivial types!

```cpp
#include <iostream>
#include <memory>
#include <cstring>
 
int main()
{
    // Notice that for "trivial types" the uninitialized_default_construct
    // generally does not zero-fill the given uninitialized memory area.
    int v[] { 1, 2, 3, 4 };
    const int original[] { 1, 2, 3, 4 };
    std::uninitialized_default_construct(std::begin(v), std::end(v));
    // comment the previous line and uncomment the next one
    // to see uninitialized_default_construct_n in action
    // std::uninitialized_default_construct_n(std::begin(v), std::distance(std::begin(v), std::end(v)));
    for (const int i : v) { std::cout << i << ' '; }
    std::cout << '\n';
    // Maybe undefined behavior, pending CWG 1997.
    std::cout <<
        (std::memcmp(v, original, sizeof(v)) == 0 ? "Unmodified\n" : "Modified\n");
    // The result is unspecified.
}
```
 
## `uninitialized_value_construct` / `uninitialized_value_construct_n`

`uninitialized_value_construct` / `uninitialized_value_construct_n` has the same signatures as `uninitialized_default_construct` and `uninitialized_default_construct_n`.

Besides, they practically work the same way for object types, they both invoke the default constructor of the contained type. However while `uninitialized_default_construct` and `uninitialized_default_construct_n` didn't zero-fill trival types (POD types), `uninitialized_value_construct` / `uninitialized_value_construct_n` will do it.

[Here is a nice little comparison between default and value initialization.](https://stackoverflow.com/a/6032889/3238101)

Here is a merged example:

```cpp
#include <iostream>
#include <memory>
#include <string>

struct S { std::string m{ "Default value" }; }; 

int main()
{
    constexpr int n {3};
    alignas(alignof(S)) unsigned char mem[n * sizeof(S)];
 
    auto first {reinterpret_cast<S*>(mem)};
    auto last {first + n};

    std::uninitialized_value_construct(first, last);
    // comment the previous line and uncomment the next one
    // to see uninitialized_default_construct_n in action
    // std::uninitialized_value_construct_n(first, n);

    for (auto it {first}; it != last; ++it) {
        std::cout << it->m << '\n';
    }

    std::destroy(first, last);
 
    // Notice that for "trivial types" the uninitialized_value_construct
    // zero-fills the given uninitialized memory area.
    int v[] { 1, 2, 3, 4 };
    for (const int i : v) { std::cout << i << ' '; }
    std::cout << '\n';
    std::uninitialized_value_construct(std::begin(v), std::end(v));
    // comment the previous line and uncomment the next one
    // to see uninitialized_default_construct_n in action
    // std::uninitialized_value_construct_n(std::begin(v), std::distance(std::begin(v), std::end(v)));
    for (const int i : v) { std::cout << i << ' '; }
    std::cout << '\n';
}
```

## `destroy` / `destroy_n` / `destroy_at`

If you have read the code snippets thoroughly in this article, you could already see `std::destroy` at work and I'm sure you can guess how `std::destroy_n` works compared to it.

`std::destroy` and `std::destroy_n` take a range of objects and invoke the destructor of those. `std::destroy` takes a pair of iterators, while `std::destroy_n` takes the beginning of a range and the number of objects to be destroyed. It's also possible to set the execution policy.

Both can be implemented as a loop iterating over the range and in the body they call `std::destroy_at` which takes only one parameter, a pointer.

```cpp
#include <memory>
#include <new>
#include <iostream>
 
struct Tracer {
    int value;
    ~Tracer() { std::cout << value << " destructed\n"; }
};
 
int main()
{
    alignas(Tracer) unsigned char buffer[sizeof(Tracer) * 8];
 
    for (int i = 0; i < 8; ++i) {
        new(buffer + sizeof(Tracer) * i) Tracer{i}; //manually construct objects
    }
 
    auto ptr = std::launder(reinterpret_cast<Tracer*>(buffer));
 
    std::destroy(ptr, ptr+8);
    // you can alternatively try this 
    // std::destroy_n(ptr, 8);
    // or this
    // for (int i = 0; i < 8; ++i)
    //     std::destroy_at(ptr + i);
}
```

## `construct_at`

`std::construct_at` takes a memory address of an object of type T and a variable number of parameters and it constructs a T object with all the passed arguments.

```cpp
#include <iostream>
#include <memory>
 
struct S {
    int x;
    float y;
    double z;
 
    S(int x, float y, double z) : x{x}, y{y}, z{z} { std::cout << "S::S();\n"; }
 
    ~S() { std::cout << "S::~S();\n"; }

    friend std::ostream& operator<<(std::ostream& os, const S& o) {
        os << "S { x=" << o.x << "; y=" << o.y << "; z=" << o.z << "; };\n";
        return os;
    }
};
 
int main()
{
    alignas(S) unsigned char storage[sizeof(S)];
 
    S* ptr = std::construct_at(reinterpret_cast<S*>(storage), 42, 2.71828f, 3.1415);
    std::cout << *ptr;
 
    std::destroy_at(ptr);
}
```
## Conclusion

This time, we learned about algorithms for dynamic memory management from the `<memory>` header. To be completely honest with you, in the almost 9 years that I spent with C++, I've never used them and there was no situation when I would have had to use them.

Still, it's good to know about them and even better to avoid dynamic memory management as much as you can and let the compiler do it for us.

In the very last part of this series, we are going to conclude what we learned about STL algorithms, the key points to keep in mind.

Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
