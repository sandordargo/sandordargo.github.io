---
layout: post
title: "Special functions and binary sizes"
date: 2023-2-1
category: dev
tags: [cpp, binarysizes, executables, compilation]
excerpt_separator: <!--more-->
---
These months, I try to better understand how our code affects binary sizes. Last week, we had a look into storage durations and memory allocations. This week, let's have a look into special member functions.

We are going to discuss whether it matters or not if we make our special functions `default` or if we provide `empty` implementations. We are also going to see if we should have the implementations in the header or in the cpp files. Or maybe not having them all is the best?

## The simplest cases

For my first tests, I used this very simple piece of code as a basis:

```cpp
#include <array>

class C {
public:
	C() = default;
	~C() = default;

	C(const C&) = default;
	C(C&&) = default;
	C& operator=(const C&) = default;
	C& operator=(C&&) = default;

};

std::array<C, 10000> a{};

int main() {
	
}
```

I took this simple class and modified it in various ways. I compiled 6 different versions:
- special functions with defaulted at declaration time
- special functions with empty implementation in the class declaration
- no special functions at all, abiding to the rule of 0
- I tried all these both with `virtual` and with non-`virtual` destructors


The results are here:

|   Version            | Binary size  | 
|----------------------|--------------|
|  default non-virtual | 16K          |
| default virtual      | 116K         |  
|  empty non-virtual   | 17-34K       |
|  empty virtual       | 33-34K       |
|  missing non-virtual | 16K          |
|  missing virtual     | 116K         |

I found no huge differences in compile time and runtime, even though my tests here were not very thorough. I created classes with a virtual destructor, but I haven't used them in a polymorphic way. That's not for today.

I made here two interesting observations. If our class doesn't have a `virtual` destructor, then either defaulting the special functions or completely omitting them potentially leaves us with a smaller binary than if we provide empty implementations. With an empty implementation, it depends on the optimization level. I think it's because if we let the compiler generate these functions, it can optimize better. We also have to mention that a defaulted special function can be implicitly `noexcept`. But it won't be `noexcept` if you provide an empty implementation and you don't declare it explicitly `noexcept`

Another interesting observation was that for classes with a virtual destructor and an empty implementation always generated smaller code than the default implementations.

While it's clear that we shouldn't add a `virtual` destructor to a class if it was not meant to be used polymorphically, this latter observation bothers me a bit.

Then I realized that while the first tests gave me an interesting start, I should clearly be closer to real usage.

Classes are rarely declared at the same translation unit with `main()`, they are often used by different translation units and classes are very often separated into a different header and implementation file.

Besides, classes more often than not have member variables.

So I decided to run more experiments.

## The results of the next tests

I incrementally implemented the missing items to see how the numbers change.

### `main` and the non-virtual class in different files

First of all, I separated the `main()` and the class into different files. As a first step, I kept the implementation in the header file.

```cpp
// c.h
class C {
public:
	C() = default;
	~C() = default;

	C(const C&) = default;
	C(C&&) = default;
	C& operator=(const C&) = default;
	C& operator=(C&&) = default;

};

// main.cpp
#include "c.h"
#include <array>

std::array<C, 10000> a;

int main() {

}
```

Then I also ran some tests where I defaulted the implementations in the cpp file.
Then I did the same experiments with empty implementations. Something we can so often see in old code.

I ended up with these numbers:


|   Version                        | Binary size  |
|----------------------------------|--------------|
|  default non-virtual header/only | 16K          |
| default non-virtual separated    | 34K          |   
|  empty non-virtual header-only   | 15-34K       | 
|  empty non-virtual separated     | 34K          |


I skipped run-times and compile-times here, because the differences are so little, that I don't think it makes sense to include those.

So what seems interesting to see is that for these non-polymorphic classes, the header-only versions produced a smaller binary both when they are defaulted and also when the implementations are left empty. 

### `main` and the virtual class in different files

Let's see what the numbers say for a virtual class.

|   Version                        | Binary size  |
|----------------------------------|--------------|
|  default virtual header/only.    | 116K         |
| default virtual separated        | 34K          |
|  empty virtual header-only       | 33-34K       |  
|  empty virtual separated         | 17-34        |

Polymorphic and non-polymorphic classes seemingly behave the opposite way. While for non-`virtual` classes header-only implementations resulted in a smaller binary, for polymorphic classes moving the implementations to the cpp file was a clear win and the empty implementation was even smaller than the defaulted one.

Don't get me wrong, I'm not advocating for an empty implementation over a default one, the default has several advantages, I'm simply sharing what I found while I was implementing.

In the defaulted version's assembly, we can see the line `.quad	__ZTV1C+16` exactly 10,000 times. For the empty implementation, instead of that, we can observe once again a `.zero fill`. For some reason, the compiler was able to deduce something for the empty implementation that it could not for the `default`ed one.

## So how should we implement special functions?

First of all, we should not. Whenever possible, we should follow the rule of zero. By all means, that's the best.

But assuming that you have to implement your special functions what to do is not obvious. In our examples, we saw that empty implementations (`{}`) might produce smaller binaries than defaulted ones in case you have a `virtual` destructor. Still, I wouldn't go for empty implementations. The ones generated by the compiler might not be completely the same, and the compiler knows it better than us. A defaulted special function might be `noexcept`, while an empty implementation will only be `noexcept` if you explicitly state so.

Some time ago, [I posted a question on Twitter and I got some answers from Jason Turner](https://twitter.com/lefticus/status/1406965561326596101). His opinion was that I should certainly default in the header. "If you forward declare an explicitly defaulted constructor then you're going to tell the compiler "Ooooh, I have something important for you, you'll find it later" Then later you say "haha, just kidding, it's really just a default constructor!"

Later he also mentioned that this can be a technique if you explicitly want to out-of-line a defaulted constructor because of compile times. And I have to add that eventually for binary size too.

At the same time, it should be documented somewhere and widely known in the organization, because it's not intuitive and someone will come to move those definitions back to the header where it belongs. Jason is right. I've been there, I've done that myself. No, not the documentation part...

For classes with non-`virtual` destructors, we should definitely avoid out-of-line destructors because of three reasons:
- as said, it decreases readability
- it might increase the binary size
- a `default` outside of the class declaration is considered a user-provided constructor/destructor and therefore the class is not trivial anymore

For polymorphic classes, it's different though. First of all, they are not trivial. Second, the binary size would actually decrease by moving things out of line.

So it boils down to binary size and probably a bit of compile-time vs readability.

Then the answer is it depends. If you optimize for binary size, then I think you should move virtual destructors out of line. And if you do it for the destructor, I'd take the others too so that it looks less strange. But don't forget to measure to see if it really has an impact. The class has to be used with lots of instances.

## Conclusion

Today we were looking for understanding what to do with special member functions of a class if we want to walk away with the smallest binary. We did different experiments and we saw that the outcomes and suggestions are different based on whether the destructor is virtual or not. 

For classes with virtual destructors, it's worth considering moving the implementation (even if it's the default) out of line, out of the header file. On the other hand for classes with non-virtual destructors, there is no reason to provide special members out of line.

Next time we'll keep discovering how having a virtual destructor is affecting our binaries.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!