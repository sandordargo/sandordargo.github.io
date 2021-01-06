---
layout: post
title: "Three ways to use the = delete specifier in C++"
date: 2021-1-6
category: dev
tags: [cpp, tutorial, delete, moderncpp]
excerpt_separator: <!--more-->
---
In this post, we will discover the three different ways you could use the `delete` specifier in C++. We are going to see how 
- you can disallow an object from being copied
- you can limit what kind of implicit conversions you allow for a function call
- you can limit what kind of template instantiations you allow
 <!--more-->

## How to disallow copying/moving for a class?

The first question to answer is why would you need such a feature? You might not want a class to be copied or moved, so you want to keep related special functions unreachable for the caller.

In order to achieve this, there is a legacy and a modern option.

The legacy option is to declare them as private or protected and the modern one (since C++11) is that you explicitly delete them.

```cpp
class NonCopyable {
public:
  NonCopyable() {/*...*/}
  // ...
private:
  NonCopyable(const NonCopyable&); //not defined
  NonCopyable& operator=(const NonCopyable&); //not defined
};
```

Before C++11 there was no other option than declaring the unneeded special functions private and not implementing them. As such one could disallow copying objects (there was no move semantics available back in time). The lack of implementation/definition helps against accidental usages in member functions, friends, or when you ignore the access specifiers. It doesn't cause a compile-time failure, you'll face a problem at linking time.

Since C++11 you can simply mark them deleted by declaring them as `= delete;`

```cpp
class NonCopyable {
public:
  NonCopyable() {/*...*/}
  NonCopyable(const NonCopyable&) = delete;
  NonCopyable& operator=(const NonCopyable&) = delete;
  // ...
private:
  // ...
};
```
The C++11 way is a better approach because 
- it's more explicit than having the functions in the private section which might only be a mistake by the developer
- in case you try to make a copy, you'll already get an error at compilation time

It's worth to note that deleted functions should be declared as public, not private. In case, you make them private some compilers might only complain about that you call a private function, not that a deleted one.

## How to disallow implicit conversions for function calls?

You have a function taking integer numbers. Whole numbers. Let's say it takes as a parameter how many people can sit in a car. It might be 2, there are some strange three-seaters, for some luxury cars it's 4 and for the vast majority, it's 5. It's not 4.9. It's not 5.1 or not even 5 and a half. It's 5. We don't traffic body parts.

How can you enforce that you only receive whole numbers as a parameter?

Obviously, you'll take an integer parameter. It might be `int`, even `unsigned` or simply a `short`. There are a lot of options. You probably even document that the `numberOfSeats` parameter should be an integral number.

Great!

So what happens if the client call still passes a float?

```cpp
#include <iostream>

void foo(int numberOfSeats) {
    std::cout << "Number of seats: " << numberOfSeats << std::endl;
    // ...
}

int main() {
    foo(5.6f);
}
/*
Number of seats: 5
*/
```

The floating-point parameter is accepted and narrowed down into an integer. You cannot even say that it's rounded, it's implicitly converted, narrowed down into an integer.

You might say that this is fine and in certain situation it probably is. But in others, this behaviour is simply not acceptable.

What can you do in such cases to avoid this problem?

You might handle it on the caller side, but 
- if `foo` is often used, it'd tedious to do the checks at each call and code reviews are not reliable enough,
- if `foo` is part of an API used by the external world, it's out of your control.

As we have seen in the previous section, since C++11, we can use the `delete` specifier in order to restrict certain types from being copied or moved. But `= delete` can be used for more. It can be applied to any functions, member or standalone.

If you don't want to allow implicit conversions from floating-point numbers, you can simply delete foo's overloaded version with a float:

```cpp
#include <iostream>

void foo(int numberOfSeats) {
    std::cout << "Number of seats: " << numberOfSeats << std::endl;
    // ...
}

void foo(double) = delete;

int main() {
    // foo(5);
    foo(5.6f);
}

/*
main.cpp: In function 'int main()':
main.cpp:12:13: error: use of deleted function 'void foo(double)'
   12 |     foo(5.6f);
      |             ^
main.cpp:8:6: note: declared here
    8 | void foo(double) = delete;
      |      ^~~
*/
```

Et voila! - as the French would say. That's it. By deleting some overloads of a function, you can forbid implicit conversions from certain types. Now, you are in complete control of the type of parameters your users can pass through your API.

## How to disallow certain instantiations of a template

This kind approach also works with templates, you can disallow the instantiations of your templated function with certain types:

```cpp
template <typename T>
void bar(T param) { /*..*/ }
```

If you call this function, let's say with an integer, it will compile just fine:

```cpp
bar<int>(42);
```

However, you can delete the instantiation with `int`, and then you receive a similar error message compared to the previous one:

```cpp
#include <iostream>

template <typename T>
void bar(T param) { /*..*/ }

template <>
void bar<int>(int) = delete;

int main() {
    bar<int>(5);
}
/*
main.cpp: In function ‘int main()’:
main.cpp:10:15: error: use of deleted function ‘void bar(T) [with T = int]’
   10 |     bar<int>(5);
      |               ^
main.cpp:7:6: note: declared here
    7 | void bar<int>(int) = delete;
      |      ^~~~~~~~
*/
``` 
Just keep in mind, that `T` and `const T` are different types and if you delete one, you should consider deleting the other too. This is only valid for the templates, not when you delete function overloads.

## Conclusion

Today we saw 3 ways how to use the `delete` specifier that is available for us since C++11. We can make classes non-copyable and/or non-movable with its help, but we can also disallow implicit conversions for function parameters and we can even disallow template instantiations for any type. It's a great tool to create a tight, strict [API that is difficult to misuse](https://www.youtube.com/watch?v=nLSm3Haxz0I).
