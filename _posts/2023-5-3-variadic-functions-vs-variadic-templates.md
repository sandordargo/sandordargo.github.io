---
layout: post
title: "Variadic functions vs variadic templates"
date: 2023-5-3
category: dev
tags: [cpp, templates, variadic, tmp]
excerpt_separator: <!--more-->
---
A few months ago, I wrote a review on [Template Metaprogramming with C++ by Marius Bancila
](https://www.sandordargo.com/blog/2022/10/28/template-metaprogramming-with-cpp-by-marius-bancila) where I mentioned not only that it's a great book, but also that there are some topics which I'll cover more in detail. Some time ago we discussed constructor templates and today I want to discuss variadic functions and variadic function templates.

## What are variadic functions?

Even if you don't explicitly know about variadic functions, you have most probably used the `printf()` family in C and/or C++. A variadic function is a function that can take an arbitrary number of arguments of any type. It must be the last (group of) parameters in a function signature.

```
void printAll(std::string items, ...);
```

Then in order to process the arguments, you have to use a couple of macros defined in `<cstdarg>`.

First, we need `va_list` to hold the information needed by the other macros. Then using `va_start()` you get access to the first argument, then with `va_arg()` you get access to each coming one and with `va_end()` you finish the traversal.

```cpp
#include <cstdarg>
#include <iostream>

void printAll(size_t count, ...) {
    va_list args;
    va_start(args, count);
    for(size_t i = 0; i < count; ++i) {
      std::cout << va_arg(args, int);
      std::cout << " ";
    }
    std::cout << '\n';
    va_end(args);
}

int main() {
  printAll(4, 3, 2, 1);
  printAll(3, 8.2, 2, 1.1);
  printAll(5, 23, 32, 8, 11, 9);
}
```

The thing is, it's extremely error-prone. Only one out of the above 3 calls is correct.

Let's see the outputs.

```
3 2 1 -665149312 
2 -665177168 24116274 
23 32 8 11 9 
```

In the first call, we first pass 4 indicating that we are passing 4 arguments and that's what we do, we pass overall four and four values are printed. While the first three are fine, the fourth one is a negative number. Well, the function reads some value from uncharted memory territories. We should have passed in `3` as a count to show that we want to print 3 arguments. Sadly, the function didn't complain that we try to print more than was passed in.

In the second case, we send in the right amount of parameters, but while the function tries to read out `int` values, two of the inputs are floating point numbers. And while you might have expected that those numbers are truncated to integers, instead some odd values are printed. The reason is that an integer and a floating point number are represented differently in memory so when you try to read an `int` as a `float` or a `float` as an `int`, you'll end up with something completely different. Though I don't understand why the value `2` appears in the first place, instead of the second.

The third call is fine. We pass in the right amount of parameters and of the right type. But these short examples already showcased how easy it is to shoot ourselves in the leg with variadic functions. It's one thing that they rely on macros, which is clearly not the way in 2022, but they are also entirely unsafe and they rely on that you count the number of passed-in arguments.

Now let's see if variadic templates are safer.

## What are variadic templates?

What is going to be similar is the use of the three dots (or ellipses): `...`. But where should those dots appear?

They can both appear before and after the type parameter! Depending on where they appear, they have different meanings. So far, so bad!

Let's have a look at the implementation of our `printAll` function.

```cpp
#include <cstdarg>

#include <iostream>

template <typename T>
void printAll(T item) {
  std::cout << item << ' ';
}

template <typename T, typename... Args>
void printAll(T item, Args... args) {
  printAll(item);
  printAll(args...);
  std::cout << '\n';
}

int main() {
  printAll(3, 2, 1);
  printAll(8.2, 2, 1.1, "duck");
  printAll(23, 32, 8, 11, 9);
}
```

You notice immediately that we have 2 overloads. One for the normal case and one for the variadic one. The first one prints one argument, while the second one recursively calls the first as it expands its parameter pack.

Let's talk about two things here. First, the position of the ellipses and second a bug in our implementation and how to fix it.

In the template parameter list, there are three dots between the `typename` keyword and the name of the template parameter pack (`Args`). That's by convention that it's attached to the `typename` but they can be attached to the parameter (`...Args`) or they can be standalone as longs as the dots are consecutive (`typename ... Args`), the compiler does not care.

It's similar to the function parameter list. You can put the ellipses wherever you want between the parameter type and the parameter name. Again, by convention, we attach them to the parameter name meaning that there will be many of them coming.

In the implementation, the three dots must follow the parameters, it does not matter whether there is a space in between or not. But if the `...` are missing, in other words, if you don't unpack the parameter pack, the compilation fails. As long as there is more than one parameter in the pack, the same variadic overload is called recursively and once there is only one left, the other overload is pulled that will stop the recursion.

And now let's talk about the problem, which does not come up for example if you want to sum up a list of numbers, but I still decided to keep this example. We might learn something interesting.

With this version, We are printing *n-2* newlines after calling `printAll` where n is the number of parameters to be printed. One way to overcome this issue would be to have a parameter for indicating whether a new line should be printed at the end of the call, and it would be only `true` for the client `printAll()` call. To make the client pass in a boolean (or any other parameter) all the time would not be nice. It makes the API more difficult to use and also error-prone. Ideally, that could be done with a default argument, but both a default argument and a variadic template argument need to have the last place. Even though [there are some techniques to overcome this](https://stackoverflow.com/questions/14805192/c-variadic-template-function-parameter-with-default-value) they are too complex to keep our code clean.

We need a helper method.

```cpp
#include <iostream>

template <typename T>
void printAllImpl(T item) {
  std::cout << item << ' ';
}

template <typename T, typename ...Args>
void printAllImpl(T item, Args ... args) {
  printAllImpl(item);
  printAllImpl(args...);
}

template <typename... Args>
void printAll(Args&&... args) {
  printAllImpl(std::forward<Args>(args) ...);
  std::cout << '\n';
}

int main() {

  printAll(3, 2, 1);
  printAll(8.2, 2, 1.1, "duck");
  printAll(23, 32, 8, 11, 9);
}
/*
3 2 1 
8.2 2 1.1 duck 
23 32 8 11 9 
*/
```
With the separation of `printAllImpl` from the non-recurisve `printAll` we achieved that there will only be one newline character printed and we used perfect forwarding so that we reduce the unnecessary copies.

This solution is totally superior to the variadic function both in terms of readability and in terms of type safety.

## Conclusion

In this article, we discussed the differences between variadic functions and variadic templates. We saw how to use variadic functions, how they rely on macros and that we pass in exactly the types a variadic function expects and also the right amount of them. Variadic templates are easier to read and easier to use. They provide the type safety that is missing from the old variadic functions.

Do you still use variadic functions? If so what are your arguments?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
