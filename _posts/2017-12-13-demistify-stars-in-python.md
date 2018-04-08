---
layout: post
title: "Demistify stars in Python"
date: 2017-12-13
category: dev
tags: [python]
header: "There are a lot of ways to use stars (*) in Python. Let's check all these possibilities."
---
Okay, not all. I'll skip the one when we use it in a string literal.

I promised that I walk you through how it can be used, but let's check first how it cannot be used. It cannot be used as part of a variable name. And this perfectly makes sense because in that case, it's a multiplication operator.

# Multiplication operator

So just in most of the other languages, `*` is an infix operator that is used to mutiply operands.

```
>>> 5 * 6
30
```

You can also multiply a number with a string:

```
>>> 5 * "ab"
'ababababab'
```

But you cannot multiply a string with another string, or a float with a string.

If you multiply a number with a boolean, it will convert the boolean to an int (1 if bool else 0). This is true even with a string multiplication.

# Import everything from

Use can import every submodule from a module using the star in a such way:

# N to the power of

If you use double star operators, you can raise a number to the power of a given number:

```
>>> 2 ** 3
8
```

# Import submodules

Star can be useful when you want to import a whole module, but you don't want to access the submodules by their fully qualified names, only by their submodule names.

If you import the whole `os` module, you can access submodules with their full names as such:

```
>>> import os
>>> os.listdir(".")
``` 

Whereas if you import from `os` all the submodules using the `*`, you can omit `os.`:


```
>>> from os import *
>>> listdir(".")
```

# Unpacking Argument Lists

Using `*` or `**` you can pass multiple parameters to a function passing only one variable. What? Okay, let check.

## Passing a list of values

Let's suppose you have a simple function, waiting for three parameters. 

```
>>> def winery_info(winery, region, subregion):
...   print("{} is located in the subregion of {} that is part of the region of {}".format(winery, subregion, region))
```

The tradition way of calling this function would be:
```
>>> winery_info("Vesztergombi Winery", "Pannon", "Szekszard")
Vesztergombi Winery is located in the subregion of Szekszard that is part of the region of Pannon

```

It might happen that you collect this data from a spreadsheet, whatnot, and you have them as a list. You don't have to specifiaclly say that the first element will the winery, the second the region and the third one will be the subregion. You can just pass the whole list preceding by a `*`.

```
>>> winery_data = ["Vesztergombi Winery", "Pannon", "Szekszard"]
>>> winery_info(*winery_data)
```
Vesztergombi Winery is located in the subregion of Szekszard that is part of the region of Pannon

In other words you can transform a list of values into values of positional arguments.

## Passing key value pairs

The other conventional way of calling the previously defined `winery_info()` function would be to pass the paramater values not by position, but by name:

```
>>> winery_info(region="Pannon", subregion="Szekszard", winery="Vesztergombi Winery")
```

Vesztergombi Winery is located in the subregion of Szekszard that is part of the region of Pannon

Similarly to the previous situation, it might happen that you get those values in a data structure. Maybe as a `dictionary`. You can simply pass the dictionary to the function. Just put the two stars (`**`) before the variable name:

```
>>> winery_data = {"region":"Pannon", "subregion":"Szekszard", "winery":"Vesztergombi Winery"}
>>> winery_info(**winery_data)
Vesztergombi Winery is located in the subregion of Szekszard that is part of the region of Pannon
```

In other words `**` transforms key-value pairs into named arguments.

# Functions with arbitrary number of arguments

## Functions with arbitrary number of positional arguments
One can use `*` and `**` not just when passing arguments to a function call, but when defining a function. If in the function header you write *arguments, your function will wait for any number of values:

```
>>> def adder(*nums):
...   print(sum(num for num in nums))
... 
>>> adder(1, 2, 3, 4)
10

```
_The example is dumb and please don't use this code in production environment_

What happens if you have multiple parameters. Let's take an example:

```
>>> def adder(a, *nums, c):
...   print(sum(num for num in nums))
... 
```

What do you think would happen if you passed in three numbers: `adder(1, 2, 3)`?

`1` will be assigned to `a`, `[2, 3]` to `b` and you will receive error message from the interpreter:

```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: adder() missing 1 required keyword-only argument: 'c'
```

So if you use a starred attribute in a function description, all the following parameters will become keyword arguments.

## Functions with arbitrary number of keyword arguments

If you write a function where an arguments is preceeded by two stars, you can pass in as many keyword arguments, as you want and then you can access both the keys and values in the function:

```
>>> def print_attributes(**attributes):
...   for key, value in attributes.items():
...     print("{}={}".format(key, value))
... 
>>> print_attributes(name="Luke", age=32, occupation="Jedi")
name=Luke
occupation=Jedi
age=32
>>> 

```

# Unpack variables at assignment time

We have seen that if we use a starred attribute in a function declaration, then all the following arguments will become keyword arguments. If you expected that the first value would go to the first attribute, the last to the last one and all the rest to middle starred one, you must be deceived.

But you can expect and will have exactly the just described behaviour if you use a starred variable in an assignment command:

```
>>> head, *pack, tail = "Chris", "Nairo", "Romain", "Alberto", "Jim" 
>>> head
'Chris'
>>> pack
['Nairo', 'Romain', 'Alberto']
>>> tail
'Jim'
```

If you have a starred variable in a multiple assignment command, one value will be assigned to each non-starred variable and the rest to the starred one.

Obviously, you cannot have multiple starred variables in one assignment.

That's it for today, I hope you learnt something about how to use stars in Python, I definitely have. If you know other uses of the star operator, leave a comment.
