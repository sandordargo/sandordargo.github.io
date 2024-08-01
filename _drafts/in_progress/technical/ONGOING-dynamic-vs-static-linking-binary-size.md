---
layout: post
title: "Binary size: should we use static or dynamic linking?"
date: 2024-1-3
category: dev
tags: [cpp, binarysizes, staticlinking, dynamiclinking]
excerpt_separator: <!--more-->
---
If at the end of a conference talk, I cannot answer a question and there is nobody else to my rescuse, I offer to reply later in the form of a blog post.

At C++ on Sea, someone asked me what are the implications of dynamic linking when it comes to binary size. I hope I remember the question well! To phrase it in a different way. Assuming the same code, what if you deliver an executable where libraries are dyanimcically linked and what if they are statically linked? How much bigger the dynamic version will be overall? Or maybe the static version will be larger?

Let's take a small example. We'll reuse one that we created for [`constexpr` functions for smaller binary size](https://www.sandordargo.com/blog/2023/09/13/constexpr-and-binary-sizes).

```cpp
// moduleA.h
#pragma once

int foo(int x);

// moduleA.cpp
#include "moduleA.h"

#include "utils.h"

int foo(int x) {
	return Fun(x) + 42;
} 

// moduleB.h
#pragma once

int bar(int x);

// moduleB.cpp
#include "moduleB.h"

#include "utils.h"

int bar(int x) {
	return Fun(x) + 51;
} 

// moduleC.h
#pragma once

int foobar(int x);

// moduleC.cpp
#include "moduleC.h"

#include "utils.h"

int foobar(int x) {
	return Fun(x) + 69;
} 

// moduleD.h
#pragma once

int barfoo(int x);

// moduleD.cpp
#include "moduleD.h"

#include "utils.h"

int barfoo(int x) {
	return Fun(x) + 99;
}

// utils.h
#pragma once

constexpr auto Fun(int v);

// utils.cpp

#include "utils.h"

constexpr auto Fun(int v);
{
  return 42 / v;
}

// main.cpp
#include <iostream>

#include "moduleA.h"
#include "moduleB.h"
#include "moduleC.h"
#include "moduleD.h"

int main() {
	std::cout << foo(2) << '\n';
	std::cout << bar(2) << '\n';
	std::cout << foobar(2) << '\n';
	std::cout << barfoo(2) << '\n';
}
```

In our original example, we compiled this into 5 different shared libraries and linked them together. Here are the commands I used on MacOS:

```sh
clang++ -std=c++20 -stdlib=libc++ -dynamiclib -include utils.h utils.cpp -o libutils.dylib
clang++ -std=c++20 -stdlib=libc++ -dynamiclib -include moduleA.h moduleA.cpp -o libmoduleA.dylib
clang++ -std=c++20 -stdlib=libc++ -dynamiclib -include moduleB.h moduleB.cpp -o libmoduleB.dylib
clang++ -std=c++20 -stdlib=libc++ -dynamiclib -include moduleC.h moduleC.cpp -o libmoduleC.dylib
clang++ -std=c++20 -stdlib=libc++ -dynamiclib -include moduleD.h moduleD.cpp -o libmoduleD.dylib

clang++ -std=c++20 -stdlib=libc++ -L . -lutils -lmoduleA -lmoduleB -lmoduleC -lmoduleD main.cpp -o main
```
In order to calculate the full size of this example, we need need to sum up the sizes of the shared objects and the main executable.

|   Filename       | Binary size in bytes | 
|-----------------|--------------|
|  libutils.dylib      | 16,800         |
|  libmoduleA.dylib         | 33,392      |  
|  libmoduleB.dylib         | 33,392      |  
|  libmoduleC.dylib         | 33,392      |  
|  libmoduleD.dylib         | 33,392      |  
|  main         | 39,416      |  
|  overall         | 189,784      |  

To understand how much we lose with dynamic linking if we lose anything, we must try to link these libraries statically too.

Let me first put here the commands I ran and then let's review them together:

```sh
clang++ -std=c++20 -c -o libutils.o utils.cpp -include utils.h -fPIC
ar r libutils.a libutils.o
clang++ -c -std=c++20 -stdlib=libc++ -include moduleA.h moduleA.cpp -o libmoduleA.o -fPIC
ar r libmoduleA.a libmoduleA.o
clang++ -c -std=c++20 -stdlib=libc++ -include moduleB.h moduleB.cpp -o libmoduleB.o -fPIC
ar r libmoduleB.a libmoduleB.o
clang++ -c -std=c++20 -stdlib=libc++ -include moduleC.h moduleC.cpp -o libmoduleC.o -fPIC
ar r libmoduleC.a libmoduleC.o
clang++ -c -std=c++20 -stdlib=libc++ -include moduleD.h moduleD.cpp -o libmoduleD.o -fPIC 
ar r libmoduleD.a libmoduleD.o

clang++ -std=c++20 -stdlib=libc++ -L . -lutils -lmoduleA -lmoduleB -lmoduleC -lmoduleD main.cpp -o main-static
```

First, instead of compiling everything into a separate dynamic library, I compiled every translation unit (every `.cpp` file) into an object (`.o`) file. Then I used the `ar` command to create a different static library out of each object file.

As a last step, I compiled `main.cpp` and specified each library with the `-l` option.

It's worth noting that both when I created static and dynamic libraries, the file's name started with `lib` which I had to omit when I passed the library names.

Now the size of the individual libraries doesn't matter anymore in a sense that we don't have to sum them up. Everything that is needed, will be part of our main file. But it's still worth having a look at them just to see their sheer size.

|   Filename       | Binary size in bytes | 
|-----------------|--------------|
|  libutils.a      | 720         |
|  libmoduleA.a         | 864      |  
|  libmoduleB.a         | 864      |  
|  libmoduleC.a         | 872      |  
|  libmoduleD.a         | 872      |  

They are two orders of magniture smaller. 

|   Filename       | Binary size in bytes | 
|-----------------|--------------|
|  main      | 39,448         |

As we can see, the size of the executable grew a tiny bit. But let's not forget that with static linkage, we don't have to keep the `.a`/`.so`/`.dylib` files around, it works on it's own! We can easily test this, by deleting them and running the executables. The dynamically linked one will crash, while the static one will work fine.

So in fact, we cut the size from 190 KB to 40 KB.

We have some alternative ways to compile.

If we looked at the example attentively, we might have noticed that `libutils` in not used by the main executable, but by all the other libraries.

If we want, we can bundle the utils with each other library. With `ar` we cannot include a static library in another, but we can bundle the object files. Instead of uncompressing `libutils.a` and using its output, let's directly use `libutils.o`.
```sh
clang++ -std=c++20 -c -o libutils.o utils.cpp -include utils.h -fPIC
clang++ -c -std=c++20 -stdlib=libc++ -include moduleA.h moduleA.cpp -o libmoduleA.o -fPIC
ar r libmoduleA.a libmoduleA.o libutils.o
clang++ -c -std=c++20 -stdlib=libc++ -include moduleB.h moduleB.cpp -o libmoduleB.o -fPIC
ar r libmoduleB.a libmoduleB.o libutils.o
clang++ -c -std=c++20 -stdlib=libc++ -include moduleC.h moduleC.cpp -o libmoduleC.o -fPIC
ar r libmoduleC.a libmoduleC.o libutils.o
clang++ -c -std=c++20 -stdlib=libc++ -include moduleD.h moduleD.cpp -o libmoduleD.o -fPIC 
ar r libmoduleD.a libmoduleD.o libutils.o

clang++ -std=c++20 -stdlib=libc++ -L . -lmoduleA -lmoduleB -lmoduleC -lmoduleD main.cpp -o main-static
```

We can observe that while the size of the static libraries increased as they also include the object file created out of `utils.cpp`, the size of main didn't change at all. 

|   Filename       | Binary size in bytes | 
|-----------------|--------------|
|  libmoduleA.a         | 1,480      |  
|  libmoduleB.a         | 1,480      |  
|  libmoduleC.a         | 1,496      |  
|  libmoduleD.a         | 1,496      |  
|  main      | 39,448         |

In a certain way, this is safer. Each library contains what it needs. It doesn't depend on the final step to have its dependencies around. Besides, it doesn't increase the size of the executable. Of course, you'll need more space to store the static libraries and overall, packaging the libraries might take more time, but probably these won't be your main concerns.

As we are on a quest of decreasing binary sizes, let's also see what if we compile everything together:

```sh
clang++ -std=c++20 -stdlib=libc++ -include moduleA.h moduleA.cpp -include moduleB.h moduleB.cpp -include moduleC.h moduleC.cpp -include moduleD.h moduleD.cpp main.cpp -o main-static
```
The size of the executable didn't change, it's still `39,448` bytes.

It's worth noting that we didn't gain anything in terms of executable size.

At the end of the day, what is better for binary size? Dynamic or static linking?

As so often, the answer is: it depends.

This was a small example and the size of the dynamically linked executable (without considering the shared libraries) was only a little bit smaller than the static one. Other times, the size difference will be more significant. Then the question is whether you run different executables on the same machine which could reuse the same shared libraries. If so, you might end up with a smaller overall size than with statically linked executables. On the other hand, if you have only one executable to run, it's almost 100% sure that linking statically is what you'll benefit from the most. (*In this article we only care about binary size, there are of course other aspects as well.*)

## Conclusion

With this article, I tried to answer one of the questions I was asked at C++OnSea. How dynamic linking influences binary size? The short answer is *heavily*. The longer answer is that dynamic linking has a big cost. It's only worth paying if you share a library between several executables on the same device. If on one device you run only one executable and size is a concern for you for whatever reason, I'd go with static linking. 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!