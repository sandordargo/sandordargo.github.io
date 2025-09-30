---
layout: post
title: "How to print a string_view?"
date: 2025-X-X
category: dev
tags: [cpp, format, print, string_view]
excerpt_separator: <!--more-->
---
First of all, how to print? Originally, this post would been short because I only wanted to cover an *"ancient"* way of printing, but it would have been too short. So first we enumerate the different ways of printing to the console and then we see how `std::string_view` goes with them.

As of C++23/26, we have these different ways to print:
- using the good old `<iostream>` library and `std::cout`/`std::cerr`/`std::clog` triplet 
- using the `std::print` function from the `<print>` header, probably combined with `std::format`
- using `printf` or `std::printf` from `<cstdio>`, quite C-style
- using `puts` or `std::puts` again from `<cstdio>`, yet again, quite C-style

## Printing a `string_view` with `<iostream>`

That's simple. No matter what kind of string our `string_view` contain, printing is as simple as passing the variable to the stream operator.

```cpp
```