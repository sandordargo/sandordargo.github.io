---
layout: post
title: "Why should we always catch exceptions by reference?"
date: 2020-7-22
category: dev
tags: [cpp, errorhandling, exceptions, bestpractices]
excerpt_separator: <!--more-->
---
Do you use exceptions in your code? Do you always catch the most generic one or do you write multiple catch blocks? Do you rethrow them or just swallow the exceptions right after they occur? Do you have an error-handling strategy after all?
<!--more-->

These are daunting questions and it would be probably worth addressing them one by one in different posts, but for the time being, I write about just a small slice of these. 

It is almost always better to pass around objects by (`const`) reference, that's something we learned by heart. But what about exceptions? If you don't catch the most generic exception (`catch(...)`)and instead of swallowing it you even plan to rethrow it, it's critical to catch by (`const`) reference.

## What's the problem?

Consider the following piece of code. There is a new exception type declared (1). In function `a()` we throw it (2) and then right there we catch a quite generic `std::exception` by value (3). After logging it, we rethrow the exception (4). In `main()`, we catch our custom exception type by `const` reference (5):

```cpp
#include <iostream>
#include <string>
#include <exception>

class SpecialException : public std::exception { // 1
public:
    virtual const char* what() const throw() {
       return "SpecialException";
    }
};

void a() {
    try {
        throw SpecialException(); // 2
    } catch (std::exception e) { // 3
        // std::cout << "exception caught in a(): " << e.what() << std::endl;
        throw; // 4
    }
}

int main () {
    try {
        a();
    } catch (SpecialException& e) { //5
        // std::cout << "exception caught in main(): " << e.what() << std::endl;
    }
}
```

What will be the output? Think about it before you actually click on [this link](http://coliru.stacked-crooked.com/a/c1f8591d71f76097) and check it for yourself.

...
..
.

So the output is apart from a compiler warning advising you not to catch anything by value is:

```
exception caught in a(): std::exception
exception caught in main(): SpecialException
```

## Why do we log a narrower exception later?

How is that even possible? Let's ignore now that it's very strange that first, we logged a wide exception than a narrow one. These kinds of questions should be addressed by our error handling policy.

What is interesting here is that when we logged a standard exception by value, we lost some of the information. Even though a `SpecialException` was flying around, in order to squeeze it into an `std::exception` variable, the compiler had to get rid of some parts of that exception. In other words, it got _sliced_. Had we caught it by reference, we would have kept its original type.

So because of slicing, we lost some information. But we got that it back after rethrowing the exception. How could that happen?

When you rethrow an exception simply by calling `throw;`, it will rethrow the original exception. There is no move, no copy taking place, if you'd check the address of the exception from catch to catch it would be the same - that's something impossible if you caught by value as it already makes a copy. And here lies the point. Catching by value makes a copy of the exception. But you don't rethrow the copy. You rethrow the original exception that was copied.

As such, any modification to the exception caught by value will be lost, including the slicing.

So as we rethrow the original exception, not the one we use within the `catch` block, but the one that left the `try` block we still keep that narrower `SpecialException`.

### Can we alter an exception in a persistent way after all?

Let's assume that our `SpecialException` has an `append(std::string message)` member function. We want to add some information to the exception before we'd rethrow it and of course, we want to retain that information. Is this possible?

Yes, but you must catch by reference and you have catch the type that has that `append()` function:

```cpp
catch(SpecialException& e) {
    e.append("Some information");
    throw;
}
```

As you caught by reference, you don't create a copy but you got a handle to the original exception. If you modify that one, it will be reflected in the rethrown exceptions.

### Are there other ways to rethrow?

As you could observe, we used a simple `throw;` but you might have encountered situations where - given that you caught an exception with the name `e` - `throw e;`  was written.

The difference is that even if you caught `e` by reference if you `throw e;`, the rethrown exception will be copied from e. One potential issue with that is its cost - after all, we copy an object pretty much in vain. Then you might now rethrow the same type as was caught. To be more specific, if you caught `std::exception` by reference and you just simply use `throw;`, you will still rethrow the original `SpecialException`, while if you `throw e`, that `SpecialException` will be copied into `std::exception` so we lose information pretty much the same way as we lost information in the case of catching by value.

## Conclusion

Today we saw the main differences between catching errors by reference and value.

So why should you always catch by (`const`) reference instead of by value and use simply `throw;` instead of `throw e;` (where `e` is the caught exception)?

The most important reason is to be unequivocal. While small performance difference can be an argument, I think that is are negligible compared to being clear on intent and meaning. If you catch by reference there is no question of type and no question of what you operate on. 

Always catch your exceptions by reference.
