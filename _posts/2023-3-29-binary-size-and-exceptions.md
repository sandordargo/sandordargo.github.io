---
layout: post
title: "Binary size and exceptions"
date: 2023-3-29
category: dev
tags: [cpp, binarysizes, noexcept, exceptions]
excerpt_separator: <!--more-->
---
In this series of articles about binary sizes, [we already talked about the `default` keyword](https://www.sandordargo.com/blog/2023/02/01/special-functions-and-binary-sizes) and said that the `default` keyword will make a special function `noexcept`, whenever it can.

## What is `noexcept`?

[The `noexcept` specifier specifies whether a function could throw an exception.](https://en.cppreference.com/w/cpp/language/noexcept_spec). If a function has the `noexcept` or `noexcept(expression == true)` specifier, then it cannot throw an exception. If it still throws either explicitly or through another function it calls then the program will call `std::terminate` immediately.

## Do unthrown exceptions have a cost?

When we learn about exceptions, we are often taught about the table below.

![Operation costs in CPU cycles]({{ site.baseurl }}/assets/img/cpu-costs-ignatchenko.png "Operation costs in CPU cycles")

We learn about the significant cost of CPU operations of throwing and catching C++ exceptions. As you can see, the costs are quite high, even compared to reading from the main RAM, not to mention reading from the cache or calling even a `virtual` function.

Still, we can often pay this price for the abstraction of exceptions, because we don't have that speed pressure or because other parts of the program are much slower and the relative slowness of exceptions is negligible. Or we simply consider that when an exception is thrown then something has already gone wrong and the speed penalty is acceptable.

We are often told that exceptions are zero-cost when they are not thrown.

That's not completely true.

The CPU cost is not the only cost.

Your binary will grow if you use exceptions. Obviously, that has nothing to do with the number of actual exceptions thrown and caught.

Handling exceptions require quite some overhead and [the level of available details can be overwhelming](https://itanium-cxx-abi.github.io/cxx-abi/exceptions.pdf). If we want to simplify it as much as possible, we can say that the compiler needs to store in the binary what kind of exceptions can be thrown by the different parts of the code and how they should be handled. The exact implementation details are out of our scope.

If a piece of code cannot throw exceptions at all, then no such information has to be stored. At least that's the theory, now let's see how much this is true in practice.

## Let's turn exceptions completely off

Let's start by completely prohibiting exceptions in our code.

If we compile, with the flag `-fno-exceptions`, we can tell the compiler not to allow exceptions at all. This does not only mean that exception handling will not happen and the program will terminate in case of an exception, but it also means that you are not allowed to write any exception handling code.

At the same time, you are allowed to use external code that throws, but your program will terminate in case of an error is thrown. If you want to test it, call the `at(size_t)` method on a `vector`. It's among the few methods in the standard library that throws. `at(size_t)` does bounds checking and throws an instance of `std::out_of_range` exception in case you try to access something beyond the limits of a container.

### A class with default or empty special functions

Let's see what happens if turn exceptions off for some of the code, that we wrote during the last few weeks. First, I turned exceptions of a simple piece of code [where we tested the binary sizes of classes with `default`ed special functions](https://www.sandordargo.com/blog/2023/02/01/special-functions-and-binary-sizes).

|   Version            | Binary size  | 
|----------------------|--------------|
|  default special functions with exceptions -O0| 116,302          |
| default special functions without exceptions -O0     | 116,302         |  
|  default special functions with exceptions -O3| 116,270          |
| default special functions without exceptions -O3     | 116,270         |  
|  default special functions with exceptions -Os| 116,270          |
| default special functions without exceptions -Os     | 116,270         |  

As you can see, we didn't gain anything at all. When we only had a class with its defaulted special functions, removing the support for exceptions didn't change a thing. As I wrote earlier, the `default` implementation of a special function is not simply about generating empty (or the simplest) bodies for special functions. It also adds `noexcept` where it is possible. In our case, it's certainly possible.

With that in mind, let's run our experiment with a similarly simple piece of code. In this case, the member functions are not `default`ed, but they have an empty implementation and they are ***not*** `noexcept`.

|   Version            | Binary size  | 
|----------------------|--------------|
|  empty special functions with exceptions -O0| 33,983          |
| empty special functions without exceptions -O0     | 33,823         |  
|  empty special functions with exceptions -O3| 16,879          |
| empty special functions without exceptions -O3     | 16,879         |  
|  empty special functions with exceptions -Os| 16,879         |
| empty special functions without exceptions -Os     | 16,879      | 

In this case, `-fno-exceptions` helped a bit, but only without optimization. With optimization turned on, we gained nothing as the compiler is smart enough on its own without hints. But it cannot always deduce all this kind of information. 

In order to demonstrate that, we need a more complex example.

### The decorator pattern

Let's have a look at [the decorator pattern that we discussed recently](https://www.sandordargo.com/blog/2023/03/08/binary-sizes-and-decorator-pattern). Now we see a difference with all the different optimization levels that we tried.

|   Version            | Binary size  | 
|----------------------|--------------|
|  modern decorator pattern with exceptions -O0| 76,481          |
| modern decorator pattern without exceptions -O0     | 58,881         |  
|  modern decorator pattern with exceptions -O3| 35,729          |
| modern decorator pattern without exceptions -O3     | 35,457         |  
|  modern decorator pattern with exceptions -Os| 36,257         |
| modern decorator pattern without exceptions -Os     | 36,193      | 

We see a difference, but it's not significant, apart from the `-O0` optimization level. If we examine the classic implementation of the decorator pattern, the difference is even smaller. In fact, it completely disappears if we compile with `-O3`.

### The observer pattern

As we haven't seen any significant differences, let's continue and try [the observer pattern](https://www.sandordargo.com/blog/2023/03/15/binary-sizes-and-observer-pattern). Once again, we see *some* differences for the classic implementation. 

|   Version            | Binary size  | 
|----------------------|--------------|
|  classic observer pattern with exceptions -O0| 86,081          |
|  classic observer pattern without exceptions -O0     | 84,769         |  
|  classic observer pattern with exceptions -O3| 36,385          |
|  classic observer pattern without exceptions -O3     | 36,225        |  
|  classic observer pattern with exceptions -Os| 37,569         |
|  classic observer pattern without exceptions -Os     | 37,265      | 

In this case, the difference persists both with the modern and the classic implementation!

|   Version            | Binary size  | 
|----------------------|--------------|
|  modern observer pattern with exceptions -O0| 84,689          |
|  modern observer pattern without exceptions -O0     | 83,345         |  
|  modern observer pattern with exceptions -O3| 35,137          |
|  modern observer pattern without exceptions -O3     | 34,977        |  
|  modern observer pattern with exceptions -Os| 36,305         |
|  modern observer pattern without exceptions -Os     | 36,017      | 

The difference is not great, but it is there. Let's have a look at the assembly to see if we can spot something interesting.

As a reminder, if you compile using `clang` with the `-S` flag you can get the assembly code. As we've seen above, when you compile without `-fno-exceptions` you get a slightly bigger assembly. I'm not going to list all the changes, I'd suggest that you try it for yourself if you are interested. Here are some excerpts that you can only find in the version with exceptions.

```asm
Lfunc_begin0:
	.cfi_startproc
	.cfi_personality 155, ___gxx_personality_v0
	.cfi_lsda 16, Lexception0

// ...

Lfunc_begin0:
	.cfi_startproc
	.cfi_personality 155, ___gxx_personality_v0
	.cfi_lsda 16, Lexception0
; %bb.0:
	stp	x20, x19, [sp, #-32]!           ; 16-byte Folded Spill
	stp	x29, x30, [sp, #16]             ; 16-byte Folded Spill
	add	x29, sp, #16
	sub	sp, sp, #528
	.cfi_def_cfa w29, 16
	.cfi_offset w30, -8
	.cfi_offset w29, -16
	.cfi_offset w19, -24
	.cfi_offset w20, -32
Lloh0:
	adrp	x8, __Z15propertyChangedRK6PersonNS_11StateChangeE@PAGE
Lloh1:
	add	x9, x8, __Z15propertyChangedRK6PersonNS_11StateChangeE@PAGEOFF
Lloh2:
	adrp	x8, __ZZ4mainEN3$_08__invokeERK6PersonNS0_11StateChangeE@PAGE
 // few hundred lines
LBB2_33:
	ldur	x0, [x29, #-128]
	bl	__ZdlPv
	mov	w0, #0
	add	sp, sp, #528
	ldp	x29, x30, [sp, #16]             ; 16-byte Folded Reload
	ldp	x20, x19, [sp], #32             ; 16-byte Folded Reload
	ret


// another lengthy section
Lfunc_end0:
	.cfi_endproc
	.section	__TEXT,__gcc_except_tab
	.p2align	2
GCC_except_table2:
Lexception0:
	.byte	255                             ; @LPStart Encoding = omit
	.byte	255                             ; @TType Encoding = omit
	.byte	1                               ; Call site Encoding = uleb128
	.uleb128 Lcst_end0-Lcst_begin0
Lcst_begin0:
	.uleb128 Ltmp0-Lfunc_begin0             ; >> Call Site 1 <<
	.uleb128 Ltmp5-Ltmp0                    ;   Call between Ltmp0 and Ltmp5
	.uleb128 Ltmp21-Lfunc_begin0            ;     jumps to Ltmp21
	.byte	0                               ;   On action: cleanup
	.uleb128 Ltmp6-Lfunc_begin0             ; >> Call Site 2 <<
	.uleb128 Ltmp7-Ltmp6                    ;   Call between Ltmp6 and Ltmp7
	.uleb128 Ltmp8-Lfunc_begin0             ;     jumps to Ltmp8
	.byte	0                               ;   On action: cleanup
	.uleb128 Ltmp9-Lfunc_begin0             ; >> Call Site 3 <<
	.uleb128 Ltmp10-Ltmp9                   ;   Call between Ltmp9 and Ltmp10
	.uleb128 Ltmp21-Lfunc_begin0            ;     jumps to Ltmp21
	.byte	0                               ;   On action: cleanup
	.uleb128 Ltmp11-Lfunc_begin0            ; >> Call Site 4 <<
	.uleb128 Ltmp12-Ltmp11                  ;   Call between Ltmp11 and Ltmp12
	.uleb128 Ltmp13-Lfunc_begin0            ;     jumps to Ltmp13
	.byte	0                               ;   On action: cleanup
	.uleb128 Ltmp14-Lfunc_begin0            ; >> Call Site 5 <<
	.uleb128 Ltmp15-Ltmp14                  ;   Call between Ltmp14 and Ltmp15
	.uleb128 Ltmp21-Lfunc_begin0            ;     jumps to Ltmp21
	.byte	0                               ;   On action: cleanup
	.uleb128 Ltmp16-Lfunc_begin0            ; >> Call Site 6 <<
	.uleb128 Ltmp17-Ltmp16                  ;   Call between Ltmp16 and Ltmp17
	.uleb128 Ltmp18-Lfunc_begin0            ;     jumps to Ltmp18
	.byte	0                               ;   On action: cleanup
	.uleb128 Ltmp19-Lfunc_begin0            ; >> Call Site 7 <<
	.uleb128 Ltmp20-Ltmp19                  ;   Call between Ltmp19 and Ltmp20
	.uleb128 Ltmp21-Lfunc_begin0            ;     jumps to Ltmp21
	.byte	0                               ;   On action: cleanup
	.uleb128 Ltmp20-Lfunc_begin0            ; >> Call Site 8 <<
	.uleb128 Lfunc_end0-Ltmp20              ;   Call between Ltmp20 and Lfunc_end0
	.byte	0                               ;     has no landing pad
	.byte	0                               ;   On action: cleanup
Lcst_end0:
	.p2align	2
```

Even if we don't understand every bit of information, we can see *except* or *exception* appearing at several places in the code. We see here parts of the exception tables.

## Or at least make noexcept whatever we can

Now that we've seen how `-fno-exceptions` affect our binary, let's see what happens when you cannot turn exceptions off, but you still want to reduce the toll of exceptions, let's use the `noexcept` specifier.

Let's work with the modern observer. As a first step, I took all the classes and I added `noexcept` to all the user-provided constructors.

|   Version            | Binary size  | 
|----------------------|--------------|
|  modern observer pattern -O0| 84,689          |
|  modern observer pattern noexcept constructors -O0     | 83,345         |  
|  modern observer pattern -O3| 35,137          |
|  modern observer pattern noexcept constructors -O3     | 34,977        |  
|  modern observer pattern -Os| 36,305         |
|  modern observer pattern noexcept constructors -Os     | 36,017      | 

So as you can see nothing changed at all. Then I started to add everywhere. I knew it can be harmful in production code, but I wanted to see if I can shave off the difference between the versions built with `-fno-exceptions` and without it by using `noexcept` extensively.

|   Version            | Binary size  | 
|----------------------|--------------|
|  modern observer pattern -O0| 84,689          |
|  modern observer pattern noexcept everywhere -O0     | 84,785         |  
|  modern observer pattern -O3| 35,137          |
|  modern observer pattern noexcept everywhere -O3     | 35,425        |  
|  modern observer pattern -Os| 36,305         |
|  modern observer pattern noexcept everywhere -Os     | 36,577      | 

To my plain horror, the binary size became bigger than it was without any `noexcept`!

I kept quite some time trying to figure out what happened. For some time, I thought that I messed up the numbers or my script. But no, the numbers are right, the script does its job.

When I looked into the assembly code, I found that from `main.s`, all exception-related code disappeared, but `person.h` became much bigger. By much I mean that it grew from 23KB to 29KB.

When I looked it up, I found such exception tables:

```
Lexception0:
	.byte	255                             ; @LPStart Encoding = omit
	.byte	155                             ; @TType Encoding = indirect pcrel sdata4
	.uleb128 Lttbase0-Lttbaseref0
// ... it's much longer
``` 

After quite some googling, I found [an RFC on llvm.org](https://discourse.llvm.org/t/rfc-add-call-unwindabort-to-llvm-ir/62543). It turns out that indeed, the binary should decrease and it does on GCC. But there is a bug on llvm and the compiler emits code for stack unwinding and exception handling, whereas it could and should just terminate.

This doesn't mean that `noexcept` will always make your binary bigger on `clang`, it means that you might not get the benefits you expect and you should measure.

## Conclusion

Today we discussed exceptions and how they affect your binary. Even if exceptions are often considered zero-cost when they are not thrown, it's not true. There is no free lunch in life, and in this case, you have to pay for a bigger binary.

If you don't use exceptions at all and you want to get rid of the overhead, you should tell the compiler so. If you don't use exceptions at all, you can turn them off, if you just want to communicate that some functions cannot throw, you should use `noexcept`.

If you worry more about the binary size than what you communicate to the readers of your code, don't forget to measure, because compilers are not perfect and they might not do what you expect them to do.


## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!

