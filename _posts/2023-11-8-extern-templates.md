---
layout: post
title: "Extern templates to reduce binary size"
date: 2023-11-8
category: dev
tags: [cpp, binarysize, extern, template]
excerpt_separator: <!--more-->
---
In my quest towards a smaller binary size, someone asked me if I considered extern templates. I did not. I didn't even know about them. But I was more than happy for the suggestion and now here I am to share what learned about extern templates.

## What are they?

You can use the `extern` keyword with template specializations and it means that no local object code will be generated for the template specialization in the local object code. With the `extern` keyword you show that the code will be generated elsewhere which the linker should find.

```cpp
template <typename T>
class Wrapper {
public:
	// ...
private:
	T wrapped;
};

// No object code will be generated for Wrapper<int> in this translation unit
extern template class Wrapper<int>;
```

This feature was introduced in C++11 and it's an optimization tool. By default, if you specialize a template in a translation unit, object code will be generated for the specialization. If you specialize the template in 10 different translation units, then object code will be generated in all those translation units, but the linker will only use one. The rest is unused, in other words, they were generated in vain.

Therefore, by using `extern template` you **might** decrease both compilation time and binary size.

## Where and how to use extern templates?

Let's say we have 5 translation units, one defines the template and the other four are using it.

```cpp
// wrapper.h
#pragma once

template <typename T>
class Wrapper {
public:
	explicit Wrapper(T t) : wrapped(t) {}

	T get() const { return wrapped; }

	void set(T t) { wrapped = t; }


private:
	T wrapped;
};


// moduleA, moduleB, moduleC and moduleD are all following the same logic
// moduleA.h
#pragma once

namespace moduleA {

void foo();

};

// moduleA.cpp
#include "moduleA.h"
#include "wrapper.h"

#include <iostream>

namespace moduleA {
	void foo() {
		Wrapper w{42};
		w.set(51);
		std::cout << w.get() << '\n';
	}
}

// main.cpp
#include "moduleA.h"
#include "moduleB.h"
#include "moduleC.h"
#include "moduleD.h"

int main() {
	moduleA::foo();
	moduleB::foo();
	moduleC::foo();
	moduleD::foo();
}
```

Do we have to put the `extern template class Wrapper<int>` in all four module headers (you can read only one of them in the above listing) or only one is enough?

When I was first thinking about this problem, I could argue (with myself) for putting the `extern` declaration in each translation unit. After all, if you want to use a template specification in a given translation unit it would be meaningful to tell the compiler that hey, you'll get it from a different place and it's also easy to understand for the readers of the code. At the same time, it's also true that this approach repeats code. You'd have the same `extern` declaration in every translation unit where you want to use a certain template specialization. If you forget to put it into one translation unit, the template specialization will be generated for that one.

Nevertheless, this approach works.

But it's not too much work and easy to miss a few places. Instead, if you want to benefit from extern templates and keep it simple and readable, put `extern template class Wrapper<int>` right after the template declaration - in our case into `wrapper.h` - and create an implementation file - `wrapper.cpp` - in which you include the header plus you put an explicit template specialization such as `template class Wrapper<int>`.

By doing so, by including `wrapper.h` in any translation unit you also include the `extern template` declaration and the `wrapper.cpp` which is a standalone translation unit will provide the specialization and as such the generated code.

Do we gain anything?

In this case, not really. On the contrary. Originally, we had 5 translation units: `moduleA.cpp`, `moduleB.cpp`, `moduleC.cpp`, `moduleD.cpp` and `main.cpp`. With the introduction of the sixth one `wrapper.cpp`, we didn't help ourselves anyhow. It's true that the intermediary object files are smaller when we use extern templates, even if we add up all of them, including `wrapper.s`, yet overall, the compilation time slightly got bigger and even the executable size.

On the other hand, if `Wrapper` resides in its own translation unit anyway and we have to compile `wrapper.cpp` anyways, then using `extern template` declaration might reduce the compilation, but we still don't gain a byte from our final binary size.

But the impacts also depend on the size of the templates, how many times it's used, how many specializations you can externalize. I'd encourage you to make your own measurements specific to your use case.

## Use `extern template` across shared libraries

Now let's explore what happens when we don't use `extern template` simply across translation units, but when we go beyond library boundaries.

For the next example, I used the very same code as before, but instead of having 6 translation units (`wrapper`, `moduleA` to `moduleD` and `main`) to generate our executable, I went with translating each file into its own library and linked them together to get an executable. I wanted to see how much space we might gain.

On macOS, I used these commands to build the shared libraries and the final executable:

```sh
clang++ -std=c++20 -stdlib=libc++ -dynamiclib -include wrapper.h wrapper.cpp -o libwrapper.dylib
clang++ -std=c++20 -stdlib=libc++ -dynamiclib -L . -lwrapper -include moduleA.h moduleA.cpp -o libmoduleA.dylib
clang++ -std=c++20 -stdlib=libc++ -dynamiclib -L . -lwrapper -include moduleB.h moduleB.cpp -o libmoduleB.dylib
clang++ -std=c++20 -stdlib=libc++ -dynamiclib -L . -lwrapper -include moduleC.h moduleC.cpp -o libmoduleC.dylib
clang++ -std=c++20 -stdlib=libc++ -dynamiclib -L . -lwrapper -include moduleD.h moduleD.cpp -o libmoduleD.dylib
clang++ -std=c++20 -stdlib=libc++ -L . -lmoduleA -lmoduleB -lmoduleC -lmoduleD main.cpp -o main  
```

*As the example code is simple, I don't use a strong optimization level because that would probably eliminate all the template usages.*

The size of `libwrapper.dylib` is not influenced by using `extern template` declarations or their lack of. On the other hand, each `libmodule<X>.dylib`'s size was affected. As we talk about shared libraries, which must be kept along with the executable, this gain matters.

When **not** using the `extern template`, they generated code for `Wrapper<int>` and their size grew by 64 bytes each. That's not a lot, I know. But what if you have many extern templates and bigger templates? Or what if you have bigger templates?

Just for curiosity, I added `Wrapper<double>` and used it, and with that, the difference grew from 64 bytes to 112 bytes, so by 48 bytes. At this point, we can question how much this is all worth. We see the potential to gain some space, but considering that `libwrapper.dylib` grew from 16k to 35k at the moment we introduced the explicit template specialization in it, we could do the math how many times we would have to use `Wrapper<int>` and `Wrapper<double>` so that it pays off.

But that's not the end of the story.

Then I modified `Wrapper`, so that it holds also a string as a member. The size difference in libwrapper is essentially the same with or without using extern and providing the definitions as it was without the `string` member. On the other hand, in each library depending on libwrapper, now the difference is almost 700 bytes. And we are still dealing with a small template. 

Now we can see the potential how `extern template` can help us gain space with templates. In general, we can also say that the more machine code the compiler has to generate, the more time it will be, so we don't only gain space but also time.

As a side note, I want to mention, that using `extern template`s prevents the compiler from inlining and you'll have to pay for function calls. If you still want to benefit from inlining, think about *interprocedural optimization*, which you might know as *link time optimization*. [I wrote about IPO/LTO here.](https://www.sandordargo.com/blog/2023/07/19/binary-sizes-and-compiler-flags#link-time-optimization) You can also learn about this problem in [Jason Turner's C++ Weekly](https://www.youtube.com/watch?v=pyiKhRmvMF4&t=235s).

## Conclusion

In this article, we learned about another tool we might consider using in order to decrease the binary footprint of our program and that is `extern template`. By default, each translation unit that uses a certain template specialization compiles that and stores it in the intermediary compiled file. But with `extern template`, we can tell the compiler that it shouldn't worry about compiling a certain specialization, because the definition will be provided later at link time. Depending on the complexity of the template and the number of its usages, you might gain both space and compilation time. Don't forget to measure before you blindly apply.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!