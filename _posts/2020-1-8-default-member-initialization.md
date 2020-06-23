---
layout: post
title: "Default Member Initializers in C++"
date: 2020-1-8
category: dev
tags: [cpp, tutorial, oop]
excerpt_separator: <!--more-->
---
This topic came up recently during a coding dojo in our department, while we were working on the [ugly trivia](https://kata-log.rocks/ugly-trivia-kata) kata. We wanted to extract a struct, containing the player data. Later we wanted to turn it into a real class with logic in it. Only later, as I prefer doing small steps at a time. Hence we started with a pure data container class, a.k.a. a struct in C++.
<!--more-->

## How class members are initialized?

But how should we properly initialize a class or a struct? How should we initialize the members? After all, even if someone just started out in C++, most probably already heard of the burdens of uninitialized members. But how to avoid them in the correct way?

So first question. How members got initialized?
* For objects (e.g. `std::string`) the default constructor is called. If there is no default constructor nor explicit initialization, there is a compile-time error.
* Primitive types (including pointers) will contain whatever (garbage) was at the given memory location previously
* References must be initialized, you simply cannot compile the code if not done.

Is it complicated? Or do you find it simple?

I don't think it is very complex, but before writing this article I had to look it up and verify just to make it sure.

So I'm still convinced that the best thing you can do is that you explicitly initialize all your members. Being implicit makes the reader think and unnecessary thinking is often a source of errors.

How would you perform that initialization?

## Constructor delegation

The good old way is just to initialize everything in the constructor's member initializer list, in the order of the declaration of the members.

```cpp
class T {
public:
T() : num(0), text("") {};

T(int iNum, std::string iText) : num(iNum), text(iText) {};

private:
  int num;
  std::string text;
};
```

If you look closer, there is a little bit of duplication going on here. Both constructors enumerate and set the two members one by one. It would be nice to call the second constructor with the default parameters, like this.

```cpp
class T {
public:
T() : T(0, "") {};

T(int iNum, std::string iText) : num(iNum), text(iText) {};

private:
  int num;
  std::string text;
};
```

The good news is this is possible for almost 10 years, since C++11 and it's called constructor delegation. Something that has been available in Java for even longer if I'm not mistaken.

## Default Member Initialization

Constructor delegation can be quite handy and simplify code, but for this very use-case, I have a better way that I want to show you.

```cpp
class T {
public:
T()=default;
T(int iNum, std::string iText) : num(iNum), text(iText) {};

private:
  int num{0};
  std::string text{};
};
```
So what is going on here. Let's go from the top to the bottom. 

Given our original example, we need the default constructor, the one not taking any parameters. But we don't want to implement it on our own, so we just leave it to the compiler by appending `=default` to its declaration.

What is even more interesting for is the declaration of the members. We don't just declare them, but we also initialize them right away. This default member initialization is also something that is available since C++ 11.

It has at least two advantages. If you consistently follow this practice you will not have to worry that you forgot to initialize something and you'll not have to scroll anywhere else to find the default value.

Please also note that we used the brace initialization instead of the assignment operator (`=`). There are - again - two reasons behind
* it's "new" so it's fancy... just kidding...
* the assignment operator allows narrowing (e.g. -1 can be assigned to an `unsigned int`), while the brance initialization would end up in a compiler error in such situations. 

Even though we already gave some default values with our shiny brace initializers, we can override those values in any constructors. In case, we initialize a member both in-place and in a constructor, the constructor wins.

You might ask if it means that the members will first be assigned to their default value and then reassigned with the values from the constructor.

[GodBolt compiler explorer](https://godbolt.org/) is our friend. Even without any explicit compiler optimization, we can find that there are no extra assignments. The compiler is smart enough to know which value to use and it avoids extra assignments.

If you are the person of guidelines, the [C++ Core Guidelines](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c45-dont-define-a-default-constructor-that-only-initializes-data-members-use-in-class-member-initializers-instead) is your friend in this case. [C.45](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c45-dont-define-a-default-constructor-that-only-initializes-data-members-use-in-class-member-initializers-instead):

> "Don't define a default constructor that only initializes data members; use in-class member initializers instead"

## Conclusion

It this article, we saw how C++ initializes class members, how constructor delegation works in order to introduce _Default Member Initialization_. This latter helps us not to implement the default constructor manually, but instead to assign default values to members right where they are declared. This makes code more readable and leaves space for less accidentally uninitialized variables.

Happy coding!
