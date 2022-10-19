---
layout: post
title: "The evolution of statements with initializers in C++"
date: 2022-5-4
category: dev
tags: [cpp, ...]
excerpt_separator: <!--more-->
---
In the coming two articles, we'll see how C++ evolved in terms of writing different statements that include initializers. Simple? Boring? I don't think so, it just shows how far we got in C++ and in programming in general in terms of readability and maintainability of code.

In the first - shorter - part, we are going to cover some basics and conditionals and in the second, we'll discuss loops.

But first of all, what's a statement?

[According to Wikipedia](https://en.wikipedia.org/wiki/Statement_(computer_science)), "a statement is a syntactic unit [...] that expresses some action to be carried out". Statements can be either simple such as assignments, assertions or function calls, etc. or compound such as loops and if statements.

Initialization cannot only happen in simple statements, like in an assignment, but also in compound ones. That's probably not relevant news, but it's also worth seeing that more and more compound statements offer options to initialize variables.

Let's see their evolution, but let's have a glimpse of variable initialization first.

## Declaring a variable and initializing it at the same time

The first time I started to learn programming was in elementary school. I was around 9-10 years old. The language we used was Pascal. I don't remember much of it, but I do remember that we had to declare all the variables on the top of a function. In many cases, you don't know what value a variable should hold, with what value a variable should be initialized... so you have to modify those variables later on.

While sometimes it was possible to initialize the variable, often you could not.

Therefore when we take it for granted that we can declare and initialize a variable at the same time, let's step a bit back and remember that was not always the case. In the past, we didn't have the possibility of declaring and initializing a variable anywhere in a function (or in a class). I won't bore you with a so dull example.

The good thing about being able to declare and initialize a variable just anywhere is that we can keep scopes smaller, lifetimes shorter - yes, in programming it's a good thing.

## Conditionals with initializers

Keeping the lifetimes the shortest that is still meaningful is not always evident. C++17 offers a new way both for `if-else` and `switch` statements.

### If statement with initializer

If you need a variable that is available only during the scope of a conditional statement, you didn't have too many choices. If you didn't want to use it in the condition within, you could obviously create it within the block of the `if`. But if you needed the same value in the `else` branch too, you needed to declare it there too. That's obviously code duplication and our program might have to perform some expensive computations for the initialization. Besides, if the computation has side effects, you might run into extra problems.

```cpp
if (condition) {
    MyType aVarWithShortScope;
    //...
} else {
    MyType aVarWithShortScope;
    //...
}
```

So the best thing you can do is to declare the variable right before the `if-else` block. This will also work if you want to use the value of the variable in the condition.

```cpp
MyType aVarWithShortScope; // now only once
if (condition && aVarWithShortScope.isValid()) {
    
    //...
} else {
    //...
}
```

With C++17, we can declare new variables between the `if` keyword and the subsequent code block, between the usual parenthesis, and right before the conditional expression. The new variable does not have to be used within the conditional expression and it is available in all branches of the `if-else`.

```cpp
#include <iostream>

int getValue() {
    return 21; // put here random
}

int main () {
    if (int val = getValue(); val<10) {
        std::cout << "val smaller than 10: " << val << '\n';
    } else if (val < 5) {
        std::cout << "val smaller than 5: " << val << '\n';
    } else {
        std::cout << "val is bigger than 10: " << val << '\n';
    } 
}
```

You are free to declare new variables in any of the `else if` branches, but they will be accessible only in the following branches, not in the previous ones! 

```cpp
#include <iostream>

int getValue() {
    return 21;
}

int main () {
    if (int val = getValue(); val<10) {
        std::cout << "val smaller than 10: " << val << " " << val2 << '\n';
    } else if (int val2 = getValue(); val < 5) {
        std::cout << "val smaller than 5: " << val << " " << val2 << '\n';
    } else {
        std::cout << "val is bigger than 10: " << val << " " << val2 <<  '\n';
    } 
}
/*
<source>:9:63: error: use of undeclared identifier 'val2'; did you mean 'val'?
        std::cout << "val smaller than 10: " << val << " " << val2 << '\n';
                                                              ^~~~
                                                              val
*/
```

This might be completely a no-brainer, but I still found it worth mentioning as the compiler can rearrange the different branches for optimization purposes. Rearrangements do not affect the scope of the variables introduced in `else if` statements.

### `switch` statement with initializer

It's not only `if`s that can benefit from initializer statements, but also `switch` statements. C++17 gave us the possibility to include variable declaration and initializations right before the condition, just like for `if`s. It's a neat way to limit the scope of a variable that we use for the selection. When the role of a variable is only to provide the value for the selection, it's useful to "officially" limit its scope accordingly.

So instead of having:

```cpp
Foo foo;
switch (foo.getValue()) {
    case 1: //...
            break;
    case 2: //...
            break;
    case 3: //...
            break;
    default:
            //...
            break;
}
```

Now you can write:

```cpp
switch (Foo foo; foo.getValue()) {
    case 1: //...
            break;
    case 2: //...
            break;
    case 3: //...
            break;
    default:
            //...
            break;
}
```

By this time you might get the idea that we could have limited the scope differently by enclosing the declaration and the statement in a code block:

```cpp
{
    Foo foo;
    switch (foo.getValue()) {
        case 1: //...
                break;
        case 2: //...
                break;
        case 3: //...
                break;
        default:
                //...
                break;
    }
}
```

Still, it's obvious that the new way requires less discipline, it's shorter and it reads better.

## Conclusion

In this article, we saw how C++ evolved in terms of providing conditional statements with initializers. While one might think that these are just syntactic sugar and indeed, we could always achieve the same effects one way or another. But in reality, they help us write safer code that is better scoped, and make our code more expressive. The expressiveness, therefore the readability and maintainability of our code can never be overestimated.

Do you use `if`/`switch` statements with initializers? What do you do when you need them but you are stuck on an earlier version?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!