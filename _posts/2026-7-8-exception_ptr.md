---
layout: post
title: "Propagating exceptions from destructors with std::exception_ptr"
date: 2026-7-8
category: dev
tags: [cpp, exceptions, destructors, noexcept]
excerpt_separator: <!--more-->
---
A few weeks ago, I wrote about [what happens when a destructor actually throws](https://www.sandordargo.com/blog/2026/04/01/when-a-destructor-throws) and why it is a dangerous idea. One of the readers commented that he was once in a situation where he had to propagate an exception from a destructor. But as a destructor cannot safely throw and it also cannot return any value, he needed a better solution.

And that solution was `std::exception_ptr`.

Let's look into what this type is and how it can be used.

<!--more-->

## What is `std::exception_ptr`?

`std::exception_ptr` is a nullable pointer-like type that was added in C++11. We can assign any captured exception to it with the help of `std::current_exception`.

> To be precise, we can also create an instance of an exception pointer with `std::make_exception_ptr`. But it's really just shorthand for throwing an exception and capturing it in a `catch` block before returning it.

If you wonder what `std::current_exception` is — it's a utility also introduced in C++11 that returns an instance of `std::exception_ptr`. When called during exception handling, it captures the current exception object and wraps it in an `exception_ptr`.

If for any reason - such as low memory - it fails and throws, `current_exception` would return that new exception instead. But don't worry, you cannot get into an endless loop — it's cut off with the help of `std::bad_exception`.

Let's look at a small example:

```cpp
// https://godbolt.org/z/5q1rMKTee

#include <exception>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers{1, 2, 3};
    std::exception_ptr eptr;

    try {
        std::cout << numbers.at(66) << '\n';  // Throws std::out_of_range!
    } catch (...) {
        eptr = std::current_exception();  // capture
    }
    return 0;
}
```

We can see how an `exception_ptr` can survive a catch block without the exception actually being handled. The exception is stored — not swallowed — so it can be dealt with later.

Now we can pass that pointer to a function dedicated to handling it.

Let's update the previous example by adding and calling `handle_exception_pointer`.

```cpp
// https://godbolt.org/z/K4hM5b1fP

#include <exception>
#include <iostream>
#include <vector>

void handle_exception_pointer(std::exception_ptr eptr) {
    try {
        if (eptr) {
            std::rethrow_exception(eptr);
        }
    } catch (const std::exception& e) {
        std::cout << "Caught exception: '" << e.what() << "'\n";
    }
}

int main() {
    std::vector<int> numbers{1, 2, 3};
    std::exception_ptr eptr;

    try {
        std::cout << numbers.at(66) << '\n';  // Throws std::out_of_range!
    } catch (...) {
        eptr = std::current_exception();  // capture
    }

    handle_exception_pointer(eptr);

    return 0;
}
```

A couple of remarks.

We take `std::exception_ptr` by value — after all, it's a pointer-like type and cheap to copy.

At the same time, we cannot dereference it or call `what()` on it directly — neither with pointer nor value semantics. So if we actually want to extract the information from it, we have to use `std::rethrow_exception`, which rethrows the exception previously captured into the pointer. Once it's rethrown inside the new `try` block, we can catch and handle it normally.

This separation of capture and handling is precisely what makes `std::exception_ptr` so useful.

## How this can help with destructors

Let's see now an example of how it can help propagate exceptions that originate in a destructor.

As we established, a destructor cannot safely throw an exception, and it can never have a return value. But sometimes a destructor performs cleanup operations that might fail, and silently ignoring those failures is not acceptable either.

The trick is to pass an empty `exception_ptr` — by reference — to the type whose destructor might encounter an error. The destructor then catches any exception internally (keeping it `noexcept`), and stores it in the `exception_ptr` for the caller to inspect after the object is destroyed.

Here is an example:

```cpp
// https://godbolt.org/z/r17xfGW9q

#include <exception>
#include <iostream>
#include <stdexcept>

struct Raii {
    std::exception_ptr& err;

    Raii(std::exception_ptr& e) : err(e) {}

    ~Raii() noexcept {
        try {
            // let's simulate a failure
            throw std::runtime_error("destruction failed");
        } catch (...) {
            if (!err) {
                err = std::current_exception();
            }
        }
    }
};

void handle_exception_pointer(std::exception_ptr eptr) {
    try {
        if (eptr) {
            std::rethrow_exception(eptr);
        }
    } catch (const std::exception& e) {
        std::cout << "Caught exception: '" << e.what() << "'\n";
    }
}

int main() {
    std::exception_ptr err;

    {
        Raii r{err};
    }  // destructor stores exception into err

    handle_exception_pointer(err);
}
```

The destructor stays `noexcept` — it never lets an exception escape. Instead, it stores the error in the shared `exception_ptr`. After the scope ends and the object is destroyed, the caller can check whether anything went wrong and act accordingly.

Note the `if (!err)` guard inside the destructor. This is important in cases where the object is destroyed while another exception is already in flight. We don't want to silently overwrite a pre-existing exception. The caller can only handle one at a time, and the first one is likely the root cause.

## Conclusion

`std::exception_ptr` is one of those C++11 additions that doesn't get nearly as much attention as it deserves. It gives us a clean way to capture, store, transfer, and rethrow exceptions — decoupling the point of failure from the point of handling.

Its most practical use case is exactly the one described above: propagating errors out of destructors without violating `noexcept`. Instead of either silently swallowing errors or triggering `std::terminate`, we can store the exception and let the caller decide how to handle it.

The pattern is simple, safe, and expressive. If you ever find yourself in a situation where a destructor needs to communicate failure, reach for `std::exception_ptr`.

{% include connect-deeper.html %}
