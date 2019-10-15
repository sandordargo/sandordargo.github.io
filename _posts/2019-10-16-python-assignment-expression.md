---
layout: post
title: "The brand new assignment expression of Python 3.8"
date: 2019-10-16
category: dev
tags: [python, tutorial]
excerpt_separator: <!--more-->
---
Python 3.8 is coming and not surprisingly it comes with [a bag of new features](https://docs.python.org/3.9/whatsnew/3.8.html). In this post, I'd like to present only one that I've been really waiting for: assignment expressions!
<!--more-->
## The problem

Whenever we see a new solution, we have to understand the problem or if there is a problem at all in the first place.

Let's take this piece of code

```py
def f(s):
  result = s
  # ... do some stuff
  return result

def g():
  return True # for the sake of the example

t=input()
if f(t) and g():
   p = f(t)
   # do something with p
```

Here we can identify at least two problems:

* We have code duplication as we wrote `f(s)` twice, while it's part of the same logical branch
* What is evident, but it is even worse, we don't just write, but we call f(s) twice. Even if we assume that with the same input we'll always have the same outputs, so the function is deterministic and might even be pure, we might face some issues. What if it triggers expensive calculations? Well, then we do it twice, which might have bad consequences.

There is a solution, we can call f(s) once before the `if` block, and save the result in a variable!

```py
def f(s):
  result = s
  # ... do some stuff
  return result

def g():
  return True # for the sake of the example

t=input()
p = f(t)
if p and g():
   # do something with p
```

Is that better? Well, it depends.

On the one hand, you type a bit less and if the calculations in `f(s)` are costly, you eliminated that expensive function call, that's great!

On the other hand, now you have a variable that is accessible outside the `if` block where you orientally wanted to use it. This might be unsafe. Imagine that you create your variable as a reference to something that takes a long-chained command to retrieve.

But before you use it, you want to make a validity check.

If you create the variable outside the if block, so before making the validity check, you have to remember that if you want to use that variable later on in that function (which would probably be a bad practice anyway), you must do the validity check again.

Python 3.8 and [PEP 572](https://www.python.org/dev/peps/pep-0572/) provides us with the ultimate solution you probably always wanted for such issues.

You can create a variable in the `if` expression whose scope is only the whole if block. Isn't that awesome?! You can do things like this, just to continue with the previous example:


```py
def f(s):
  result = s
  # ...
  return result

def g():
  return True # for the sake of the example

t=input()
if p:=f(t) and g():
   # do something with p
   print(p)
```

By the "whole `if` block", I meant that `else` is also included. To generalize, we can say that the scope of the variable assigned in the assignment expression is just the current scope. If it's an if, then an if, if it's the whole function, it's the whole function.

What I also like a lot is that we now can simplify list comprehensions as well. Look at this example:

```py
inputs = [1, 2, 3, 56, 78, 42, 36, 54, 35, 99]
numbers_and_int_square_roots = {k: int(math.sqrt(k)) for k in inputs if int(math.sqrt(k)) == math.sqrt(k)}
print(*numbers_and_int_square_roots.items())
```
So, we take a list of numbers and we want to keep the ones that are squares of an integers and we also want to keep their squared and non-squared values. We have to calculate the square roots twice!

Only if I could save the square root in that generator expression! Lo and behold, now I can:

```
inputs = [1, 2, 3, 56, 78, 42, 36, 54, 35, 99]
numbers_and_int_square_roots = {k: v for k in inputs if (v:=int(math.sqrt(k))) == math.sqrt(k)}
print(*numbers_and_int_square_roots.items())
```
That is just super cool to me! Less typing, less calculations, faster runtime!

What do you think?

## Conclusion

[Python 3.8](https://docs.python.org/3.9/whatsnew/3.8.html) introduces [assignment expressions](https://www.python.org/dev/peps/pep-0572/) which lets us create new variables in places where we always wanted but never could in a usable way. Probably the best way to use this new feature is to create new variables in an `if` and use it in the scope of that block.

For more information, you can read the specs [here](https://www.python.org/dev/peps/pep-0572/).

As an online interpreter, for the moment you can use [this](https://tio.run/#python38pr).

And if you want to install python3.8 locally, you can refer to [this article](https://dev.to/mortoray/how-to-install-python-3-8-on-ubuntu-1bp4)

Happy coding!