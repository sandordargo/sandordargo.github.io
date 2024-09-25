---
layout: post
title: "Will std::move_only_function reduce your binary size?"
date: 2024-1-3
category: dev
tags: [cpp, cpp23, binarysizes, move_only_function]
excerpt_separator: <!--more-->
---
As I shared it a few weeks ago, C++23 is introducing `std::move_only_function`. It resembles to `std::function`, but you can also use it with callables that are not copyable, but only movable.

As a reminder, `std::function` is a class template that wraps callable objects. You can use it to store, copy, and invoke any callable object, including functions, function pointers, member functions, lambdas, and even function objects.

It acts as a type-erased container for functions, allowing them to be passed around and stored in a more flexible manner. It's very versatile and you can use it in situations when a function pointer or a simple template is not enough. At the same time, [it's quite bloaty](https://www.sandordargo.com/blog/2023/04/05/binary-size-and-templates) and often considered harmful if your goal is to limit the binary size.

The main functional limitation of `std::function` is that it cannot take care of objects that are non-copyable. In case a lambda, that can be when there is a `std::unique_ptr` or any other non-copyable type is captured. In case of function objects, that will mean non-copyable member variables or explicitely deleted copy operations.

```cpp
#include <functional>
#include <memory>

class NonCopyable1 {
public:
    NonCopyable1(std::unique_ptr<int> ptr): m_ptr(std::move(ptr)) {}
    
    bool operator()(int i) const {
        return i != *m_ptr;
    }
private:
    std::unique_ptr<int> m_ptr;
};


class NonCopyable2 {
public:
    NonCopyable2() = default;
    ~NonCopyable2() = default;

    NonCopyable2(const NonCopyable2&) = delete;
    NonCopyable2& operator=(const NonCopyable2&) = delete;

    NonCopyable2(NonCopyable2&&) = default;
    NonCopyable2& operator=(NonCopyable2&&) = default;

    bool operator()(int i) const {
        return i != 42;
    }
private:
};


int main() {
    std::function<bool(int)> nonCopyable1 = NonCopyable1(std::make_unique<int>(42)); 
    std::function<bool(int)> nonCopyable2 = NonCopyable2(); 
    std::function<bool(int)> nonCopyable3 = [ptr = std::make_unique<int>(42)](int i){return i != *ptr;}; 
}
```

In this above listing's `main`, all the three variable declarates lead to a compilation error as none of the callables on the right side is copy-constructible.

We can make this callable with `std::move_only_function`.

https://www.youtube.com/watch?v=OJtGOJI0JEw

## Jason's example

```cpp
#include <functional>
#include <iostream>
#include <memory>
#include <vector>

std::vector<std::function<int(int)>> callback_registry;

void register_callback(std::function<int (int)> callback) {
    callback_registry.push_back(callback);
}

void invoke_callbacks(int value) {
    // Retrieve callbacks from the registry and invoke them
    for (const auto& callback : callback_registry) {
        int result = callback(value);
        std::cout << "Callback result: " << result << std::endl;
    }
}

int main() {
    std::function<int (int)> cb {
        [i = std::make_shared<int>(42)] (const int val) {
            return val + *i;
        }
    };

    register_callback(cb);
    invoke_callbacks(51);
}
```
  4.18s user 0.66s system 78% cpu 6.142 total
-O0 100,880
  4.61s user 0.59s system 101% cpu 5.121 total
40,784



```cpp
#include <functional>
#include <iostream>
#include <memory>
#include <vector>

std::vector<std::move_only_function<int(int)>> callback_registry;

void register_callback(std::move_only_function<int (int)> callback) {
    callback_registry.push_back(std::move(callback));
}

void invoke_callbacks(int value) {
    // Retrieve callbacks from the registry and invoke them
    for (auto&& callback : callback_registry) {
        int result = callback(value);
        std::cout << "Callback result: " << result << std::endl;
    }
}

int main() {
    register_callback([i = std::make_unique<int>(42)] (const int val) {
            return val + *i;
        });
    invoke_callbacks(51);
}
```


## Example where no unique is needed

We saw that ... Now let's see if it's worth using `std::move_only_function` if we could also copy it.


## Conclusion


## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!