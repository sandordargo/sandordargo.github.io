## The worst things to do with your binary size

In the next few (?) articles, let's a couple of worst practices that you can do to your binary. The goal of this exercise is twofold.

The less important goal is to five you some pattern that you can look for in your code if you want go for some low hanging fruits.

The other goal is to actually understand what type of declarations, allocations end up where, what will slightly and what will seriously increase the binary size.

First, let's have a look at a very simple example and we'll iterate from there.

```cpp
// file: big.cpp
const int N = 10'000;

struct Node {
    int a = 1, b = 1;
};

Node segtree[40*N];

int main() {}

// file: small.cpp
const int N = 10'000;

struct Node {
    int a = 1, b = 1;
};

int main() {
    Node segtree[40*N];
}
```

I compile and measure it's their size in a very simple way:

```sh
clang++ -std=c++17 -strdlib=libc++ myFile.cpp -o myFile && wc -c myFile
```

So the size of the first one is `3,253,232` bytes, around 3 MBs for such a small program, while the second is only `33,618` bytes large, about 32 kBytes. There is 100x difference.

Why the first one is bog compared to the other? And can we make up other similar scenarios when that would be so big?

If we keep the declaration in local, but we make it static or constexpr the size will rocket once again. Please note that const is not enough, it has to be constexpr.

So it's pretty obvious that when the compiler can initialize something at compile time that will make the executable larger. In more formal, anything that has a static storage duration such as static and global variables will be initialized during compile-time, they do not have to be const.

> *In the case of static storage duration, the storage is allocated when the program begins and deallocated when the program ends. Variables with static storage duration have only one instance. Which objects have static storage duration? All that were declared with the static keyword! Besides, all of the objects that were declared at a namespace level or declared with the extern keyword.* - [C++ basics: scopes, linkage, names](https://www.sandordargo.com/blog/2022/05/18/scope-linkage-name)

Let's have a look at the below table with sizes.

TABLE HERE

We can see an interesting tradeoff happening here.

With an automatic storage duration (non-static local variable), we have a small binary size and we have a runtime of X. When we have a local const or a global array, the size grows big. The whole array is in the executable basically.  A Node has a size of two integers, 8 bytes and we allocate 400,000 of it, that's 3,200,000 bytes. The full size of the executable is about 3250000. 

The runtime is 1,000,000 times faster. It's bascially negligable, which makes sense. For global variables the initaializion happens before the main executaion, for local static before at the first call of the function. 

The memory for all static variables is allocated at program load. But local static variables are created and initialized the first time they are used, not at program start up. There's some good reading about that, and statics in general, here. In general I think some of these issues depend on the implementation, especially if you want to know where in memory this stuff will be located.
https://stackoverflow.com/questions/55510/when-do-function-level-static-variables-get-allocated-initialized

We clearly trade space and compile time for execution.

Should you do it all the time when possible?

I cannot tell. It's a compromise, it depends on your usecase, your constraints!

There is something more interesting! A local constexpr array.

It almost has the same size as the others with a static storage duration. On the other hand, the runtime is only 2x as fast. THat's clearly not the same tradeoff.

Let's try to undersrtand what happens there.
https://godbolt.org/z/a581Kha7n
https://godbolt.org/z/P3qz5dKr1

So as we can see in the assembly of the static version, nothing happens. Probably some little parts are optimized away as a local static should be initialized the first time it's called?! We'll get back to that later. But in any case, whatever happens it will happen only once and we see that nothing else goes on.

On the other hand, instructions are emitted for the constexpr case to initialize the array.

While it makes sense to see a difference, this answer hardly satisfied me.

By calling `clang` with the option `-S` we can skip stop right after the compilation step and get the assembly code. By having a look into the generated assembly codes we can see what is in the different sections we discussed in the last episode.

If you remember well, data segment is the part of the binary where you find the variables that are initialized during the compilation. In the text segment, you'll find the code to be executed. While the static version has everything in the data segment, constexpr has everything in the text segment, it consists of code that will have to be executed hence the diff in speed.

Now if you do this for a simple local with automatic storage duration, you don't have all this. You only get a couple of instructions in main to initialize the local variable. On the other hand, your code is a bit slower, but much faster to compile.

So do we have to choose between fast runtime and small binaries?

It depends.

If you have another look at the above code, you can see that we initialized the members of node properly. But we initialized them to `1` which is not the default value for an integer. If instead we initialize these members to `0` we get the best from both worlds. Our compilation time decreases, the binary size becomes small and the runtime is still optiomal.

If we have a look at the instructions, we can see this in the data segment:

```
.zerofill __DATA,__common,_segtree,3200000,2
```

Using static storage duration is not always a good option in object oriented or in fact in any kind of programming. So let's see what happened to our `constexpr` version. The size is decreased there too, it's still in the text segment. We are still getting some benefits without paying for the size.

Now what's interesting and something I couldn't decipher is about what happens if I add another member, like a string. No more size issues, even if...

for the static the assembled code has actually nothing. everything is done
for the constexpr the memory allocation was alrady done, here is the initialization part

https://levelup.gitconnected.com/constexpr-all-the-things-but-gently-f567a8b93603

text vs data segment
https://mcuoneclipse.com/2013/04/14/text-data-and-bss-code-and-data-size-explained/

when make it default 0 size is optimized

```
.zerofill __DATA,__common,_segtree,3200000,2
```

use default values, whenever you can
what happens though when we add the string?


<!--- ------->





<!--
What is needed for this

first have some set up commands that I can use to compile, execute (to validate) and output the size
clang++ -std=c++17 -strdlib=libc++ old.cpp -o old && ./old && wc -c old


I need examples for both
then I explain
use these examples to explain what goes where

https://codeforces.com/blog/entry/79909
try it with c-style array global 3,253,242, 4.47   | 5.49 0.401
try it with c-style array local 33,625 1.4         | 1.86 0.65 
try it with c-style local static 3,253,232 4.58    | 5.5 0.65
try it with c-style local static const 3,236,726 4.55 | 5.5 0.49
try it with c-style local const 33,631 1.41
try it with c-style local constexpr 3,253,455 4.55  | 5.5 0.45 | 

try it with array global 3,253,241
try it with array local 33,608
try it with array local static 3,253,231
try it with array local static const 3,236,725

try it with vector global 40,922
try it with vector local 40,729
try it with vector local static 41,104
try it with vector local static const 41,110

try it with vector init global 40,863
try it with vector init local 40,670
try it with vector init local static 41,061
try it with vector init local static const 41,067

measure compilation time!

How should I measure the execution time?
I can add the usual start-end stuff to main, but it doesn't cover things that are initialized outside of main. That is not considered, but it's still part of execution time?!

But only once and hwile in our example we have nothing to do more than once, a normal application runs for a long time...

We can check exec time in 2 ways then!
Once with inside time, and once with outside time. We might even understand how much time is spent outside.


loop inside, exec code about a hundred times
have a function that does it
what about outside?
compile in two ways?


try it with global and local and static
try it with const, non const

https://devblogs.microsoft.com/performance-diagnostics/sizebench-a-new-tool-for-analyzing-windows-binary-size/
try virtual vs non-virtaul
what if there is one virtual vs many

what if we have some hardcoded local (static? const?) maps here and there

what about template foldability
-->

https://quuxplusone.github.io/blog/2022/12/24/builtin-std-forward/



try auto fun vs std::function

```
// Copyright (c) Spotify AB

#pragma once

#include <utility>
#include <boost/config.hpp>

namespace spotify::async::functional {

// Wraps an `F` callable in a `noexcept` frame in order to terminate at the destruction or
// invocation of the wrapper _if_ the wrapped callable throws at destruction or invocation, in a
// guaranteed to be non-inlined frame containing source information about `F`.
//
//  - For the invocation case, we have observed that even though we enforce `noexcept` on the
//    user provided callables, those frames may sometimes be inlined such that for example
//    `GenericTimerManager` looks like the culprit, rather than the user provided callable.
//  - Destruction of user provided callables were a complete blindspot and all exceptions would
//    unwind the stack and give for example `~GenericTimerManager` the blame.
template <typename F>
class CallableWrapper {
 public:
  CallableWrapper(F &&f) : _f(std::forward<F>(f)) {}

  CallableWrapper(const CallableWrapper &cw) = delete;
  CallableWrapper(CallableWrapper &&cw) noexcept : _f(std::move(cw._f)) {}

  BOOST_NOINLINE ~CallableWrapper() noexcept {}

  CallableWrapper &operator=(const CallableWrapper &cw) = delete;
  CallableWrapper &operator=(CallableWrapper &&cw) noexcept {
    _f = std::move(cw._f);
    return *this;
  }

  BOOST_NOINLINE void operator()() noexcept { _f(); }

 private:
  F _f;
};

}  // namespace spotify::async::functional

```



split decl/def of special functions
```
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/special-defaults $ bem empty_non_virtual_dtor                                                                                                                                              17:11
  0.30s user 0.17s system 66% cpu 0.713 total
16,851
  0.04s user 0.13s system 30% cpu 0.530 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/special-defaults $ bem default_non_virtual_dtor                                                                                                                                            17:12
  0.30s user 0.14s system 67% cpu 0.655 total
16,853
  0.03s user 0.11s system 37% cpu 0.384 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/special-defaults $ bem empty_virtual_dtor                                                                                                                                                  17:12
  0.30s user 0.16s system 65% cpu 0.695 total
16,847
  0.03s user 0.10s system 32% cpu 0.410 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/special-defaults $ bem default_virtual_dtor                                                                                                                                                17:12
  0.30s user 0.15s system 68% cpu 0.656 total
16,849
  0.03s user 0.10s system 28% cpu 0.466 total


---------------


sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/special-defaults $ bem empty_non_virtual_dtor                                                                                                                                              17:15
  1.63s user 0.30s system 86% cpu 2.234 total
33,747
  0.05s user 0.14s system 34% cpu 0.536 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/special-defaults $ bem default_virtual_dtor                                                                                                                                                17:15
  1.74s user 0.31s system 88% cpu 2.308 total
116,305
  0.03s user 0.09s system 42% cpu 0.284 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/special-defaults $ bem empty_virtual_dtor                                                                                                                                                  17:15
  1.71s user 0.32s system 85% cpu 2.373 total
116,575
  0.04s user 0.10s system 36% cpu 0.382 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/special-defaults $ bem default_non_virtual_dtor                                                                                                                                            17:16
  1.63s user 0.30s system 86% cpu 2.234 total
16,885
  0.04s user 0.12s system 32% cpu 0.483 total




----------



sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/special-defaults $ bem default-spcial5-nonv                                                                                                                                                17:21
  1.66s user 0.31s system 89% cpu 2.206 total
16,881
  0.03s user 0.10s system 39% cpu 0.330 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/special-defaults $ bem default-spcial5-v                                                                                                                                                   17:21
  1.74s user 0.32s system 87% cpu 2.363 total
116,302
  0.03s user 0.10s system 45% cpu 0.283 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/special-defaults $ bem empty-spcial5-nonv                                                                                                                                                  17:22
  1.66s user 0.30s system 90% cpu 2.177 total
33,983
  0.04s user 0.09s system 47% cpu 0.275 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/special-defaults $ bem empty-spcial5-v                                                                                                                                                     17:22
  1.65s user 0.32s system 88% cpu 2.227 total
34,236
  0.07s user 0.16s system 37% cpu 0.621 total


so seemingly a default outside is showing a smaller size
try it once again with an array

doesn't matter if there is a member it seems

what if it's an interface? use it in a vector? how to check mem consumption...

today
header and main separated included
declare with impl outside
declare with impl outisde separated


declare something with a member!
something with vector
check mem consumption


run bloaty
exceptions
type erasure
rtti
noexcept
cRTP
```


https://vector-of-bool.github.io/2020/10/04/lib-configuration.html



https://blogit.spotify.net/2019/03/25/slaying-the-core-binary-bloat-dragon/
Spotify findings
- removing emtpry or default makes things smaller
- moving things to the header make things bigger
- moving initialization to in-place seems bad
- moving init to in-place and remove empty special functions is good!


What did I found?
- having things in the impl is bigger! but when something is initialized with a non-default, than having the init in the impl is smaller


https://stackoverflow.com/questions/62580224/should-functions-declared-with-default-only-go-in-the-header-file



https://quuxplusone.github.io/blog/2023/01/13/embed-and-initializer-lists/













But what if actually call virtual methods?

Makes no difference. THere is no allocation.
- local meth
- static meth

- From dyn allocated (dipsatch)
- From non dyn

When I started to store pointers zero init worked...




Readable info on virtuals in Many, plus a table like shit that is missing from the single.

- Throw back to first experiments where we saw that virtual significantly increased the size only for the virtual dtor

- check the assembly code

- create a small class with a few non-virtual functions

- start making the methods virtual one by one

###

As a next step I added some member variables to each and also some accessor methods.

|   Version                        | Binary size  | Compile-time  | Run-time  |
|----------------------------------|--------------|---------------|-----------|
|  default non-virtual header/only | 16K          |   2.6         | 0.8      |
| default non-virtual separated    | 34K       |   2.6       | 0.7      |
|  default virtual header/only | 281K          |   2.6         | 0.3-0.7      |
|  default virtual only dtor header/only | 281K          |   2.6         | 0.3-0.7      |
| default virtual separated    | 34K       |   2.2       | 0.4      |


I added three integer members and `const` accessors for each of them. At this point I decided to discard the implementations where special functions were emtpy as the new accessors have the same implementaions in every case. 

The sizes didn't change more than a few bytes for the non-virtual cases. And it didn't change either when destructor was (default) implemented in the .cpp file. On the other hand, by adding 3 members and their virtual accessors (only for the test) implemented in the header increased the size of the binary from 116K to 281K.

I was wondering if the size increase was because of the several new virtual functions, so I devirtualized the accessors. The size decreased only by a mere 96 bytes, while the increase is around ~165,000 bytes. So it's only about growing size of the class.

This means that having virtual destructors defined in the header have very serious consequences on the binary size. Even removing the functions didn't change the size at all.

281,414-281,430 vs 281,510vs 281,542

///
- create a simple OO structure with virtuals and see how it performs, try the car/wheels example
https://www.sandordargo.com/blog/2019/03/13/the-curiously-recurring-templatep-pattern-CRTP

- a reminder on when we can use crtp and when we cannot
///

- check two version
https://github.com/igl42/cpp_software_design/blob/main/G36_Compile_Time_Decorator.cpp
https://github.com/igl42/cpp_software_design/blob/main/G36_Runtime_Decorator.cpp


https://github.com/igl42/cpp_software_design/blob/main/G25_Modern_Observer.cpp
https://github.com/igl42/cpp_software_design/blob/main/G25_Classic_Observer.cpp


```cpp
#include <cassert>
#include <iostream>
#include <string>

class Vehicle {
public:
    virtual ~Vehicle();
    
    virtual int getNumberOfWheels() const = 0;
};

Vehicle::~Vehicle() = default;

class Car : public Vehicle {
public:
    int getNumberOfWheels() const override;
};

int Car::getNumberOfWheels() const {
    return 4;
}

class Scooter : public Vehicle {
public:
    int getNumberOfWheels() const override;
};

int Scooter::getNumberOfWheels() const {
    return 2;
}

class Bus : public Vehicle {
public:
    Bus(int numberOfWheels);
    int getNumberOfWheels() const override;
private:
    int m_numberOfWheels;
};

Bus::Bus(int numberOfWheels) : m_numberOfWheels(numberOfWheels) {
    
}

int Bus::getNumberOfWheels() const {
  return m_numberOfWheels;
}

int main() {
    return 0;
}
```











```cpp
#include <cassert>
#include <iostream>
#include <string>

template <typename T>
class Vehicle
{
public:
    double getNumberOfWheels() const
    {
        return static_cast<T const&>(*this).getNumberOfWheels();
    }
};

class Car : public Vehicle<Car>
{
public:
    double getNumberOfWheels() const {return 4;}
};

class Scooter : public Vehicle<Scooter>
{
public:
    double getNumberOfWheels() const {return 2;}
};


class Bus : public Vehicle<Bus>
{
public:
    explicit Bus(int value) : value_(value) {}
    double getNumberOfWheels() const {return value_;}
private:
    int value_;
};

int main() {
    return 0;
}
```








add noexcept!

























 and observer design patterns in terms of binary sizes. We saw that the modern version of the decorator pattern provides us a smaller binary, but you might want to measure it how it scales dependeing on the decorated objects.

On the other hand, the classic version of the observer pattern was smaller and scaled sometimes better than the modern implementation. On the ohter hand, if you don't need a `std::function`, but you can use a function pointer to take your observers, you can win big and end up with a nicely scaling implementation that is faster and smaller than the classic version.

Next week, we'll look into...






Is it about the included <functional>
  let's try it
  NOT THE PROBLEM

  what if we don't use a functional, but we tamplate it.


extra observer
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/observer/classic $ bem5 main person address_observer name_observer random_observer observer                                                  16:50
  9.37s user 1.23s system 95% cpu 11.085 total
86,081
  10.05s user 1.28s system 96% cpu 11.800 total
36,385
  9.95s user 1.27s system 95% cpu 11.772 total
37,569
  9.42s user 1.24s system 95% cpu 11.203 total
86,081
  0.05s user 0.16s system 22% cpu 0.919 total

with lambda
  1.35s user 0.19s system 83% cpu 1.844 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/observer/modern $ bem3 main observer person                                                                                                  16:52
  6.53s user 0.85s system 95% cpu 7.760 total
159,937
  7.25s user 0.83s system 95% cpu 8.464 total
40,961
  7.23s user 0.87s system 95% cpu 8.485 total
42,465
  6.54s user 0.85s system 94% cpu 7.813 total
159,937
  0.07s user 0.19s system 33% cpu 0.784 total
with function
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/observer/modern $ bem3 main observer person                                                                                                  16:55
  6.38s user 0.83s system 92% cpu 7.798 total
142,305
  7.05s user 0.83s system 94% cpu 8.334 total
39,569
  6.78s user 0.81s system 95% cpu 7.954 total
41,041
  6.25s user 0.79s system 94% cpu 7.468 total
142,305
  0.08s user 0.22s system 30% cpu 0.986 total

changing the lambda to function!
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/observer/modern $ bem3 main observer person                                                                                                  16:56
  6.03s user 0.80s system 94% cpu 7.249 total
108,161
  6.78s user 0.81s system 95% cpu 7.972 total
38,161
  6.57s user 0.81s system 95% cpu 7.768 total
39,601
  6.05s user 0.80s system 94% cpu 7.258 total
108,161
  0.07s user 0.20s system 32% cpu 0.819 total


original modern with only functions
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/observer/modern $ bem3 main observer person                                                                                                   6:38
  6.06s user 0.82s system 92% cpu 7.407 total
108,065
  6.76s user 0.81s system 96% cpu 7.881 total
38,065
  6.59s user 0.82s system 96% cpu 7.684 total
39,489
  6.08s user 0.82s system 95% cpu 7.260 total
108,065
  0.05s user 0.13s system 29% cpu 0.626 total
the growth is smaller 


lambdas are taking a toll

we can drive down the size by replacing lamdbas with functions
not nice, i don't say you should do it, but it's worth considering if that's what you're into



one more thing that you migth want to replace is the std::function
instead use a typedef for a function pointer
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/observer/modern $ bem3 main observer person                                                                                                   6:47
  5.92s user 0.82s system 95% cpu 7.088 total
84,481
  6.37s user 0.82s system 96% cpu 7.484 total
35,073
  6.32s user 0.83s system 95% cpu 7.507 total
36,241
  5.85s user 0.81s system 95% cpu 6.949 total
84,481
  0.06s user 0.18s system 32% cpu 0.759 total


Now if we move back the lambda instead of the function, we get a sligthly bigger, binary, but still smaller than the classic solution. Plus the growth is smaller.


sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/observer/modern $ bem3 main observer person                                                                                                   6:49
  5.91s user 0.82s system 94% cpu 7.132 total
84,689
  6.46s user 0.83s system 96% cpu 7.576 total
35,137
  6.34s user 0.82s system 94% cpu 7.614 total
36,305
  5.88s user 0.81s system 95% cpu 7.039 total
84,689
  0.07s user 0.21s system 35% cpu 0.790 total

https://github.com/igl42/cpp_software_design/blob/main/G25_Modern_Observer.cpp
https://github.com/igl42/cpp_software_design/blob/main/G25_Classic_Observer.cpp



What's the problem with stud function?









[In one of the previous articles on binary sizes](https://www.sandordargo.com/blog/2023/02/08/binary-sizes-and-virtual), we discussed how making a class polymorphic by using the `virtual` keyword affects the binary size. Turning a function into `virtual` has a substantial effect on the binary size, but adding more and more `virtual` methods to a class that already has at least one `virtual` function does not change that much.

To have an elaborate example I went to the publicly available code examples of [C++ Software Design](https://www.amazon.com/Software-Design-Principles-Patterns-High-Quality/dp/1098113160?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=e9b6f64671aac55ff52ecfd91e137d6e&camp=1789&creative=9325) by [Klaus Iglberger](https://coaches.xing.com/profile/Klaus_Iglberger/trainings). As I explained [here](https://www.sandordargo.com/blog/2022/12/31/8-best-books-in-2022), it's one of the best books I read in 2022. If you are interested in software design and in C++, in my opnion, it's a must-read.

In the book, you can find different implementations of various design patterns. All the discussed design patterns are first presented through their classic implementation, usually based on polymorphism and then modern alternatives are also explained.

Sometimes, the modern implementations offer the same functionality, sometimes they are restricted for compile-time needs, but that's often enough for the needs.

In this and the next article, I want to go through two implementations of two design patterns and focus on their effects on binary sizes. Today, it's the decorator pattern on the plate.

<!--
[In the book](https://www.amazon.com/Software-Design-Principles-Patterns-High-Quality/dp/1098113160?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=e9b6f64671aac55ff52ecfd91e137d6e&camp=1789&creative=9325), there is also a third version implemented with type-erasure. It offers some of the best of two worlds: a value-semantics based solution, with run-time flexibility. If you want to learn more about it, I highly recommend reading the book. Nevertheless, [the code is available here]().

While the internals are more complex (as type erasure is a bit complex design pattern), its usage is nicer as it doesn't require reference semantics. In terms of performance, it's very close to the classical run-time version. It has both tempaltes and a polymorphic structure internally. It's worth keeping an eye on it, if you use many different items and detect a binary bloat before it becomes problematic.

|   Version                        | Binary size at -O0 | Binary size at -O3 | Binary size at -Os | 
|----------------------------------|--------------|
|  classical decorator  | 16K          |  
| modern decorator  | 34K       |   


-->
But what about the third one
The interesting fact about these solutions is that with their limited scope, the



sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/decorator/runtime-virtual $                                                                                                                  17:13
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/decorator/runtime-virtual $ bem main                                                                                                         17:13
  2.55s user 0.40s system 85% cpu 3.437 total
54,689
  2.77s user 0.38s system 93% cpu 3.353 total
36,049
  2.81s user 0.38s system 95% cpu 3.354 total
36,625
  2.53s user 0.37s system 93% cpu 3.087 total
54,689
  0.05s user 0.14s system 26% cpu 0.703 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/decorator/runtime-virtual $ cd ../runtime                                                                                                    17:14
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/decorator/runtime $ bem main                                                                                                                 17:14
  2.54s user 0.38s system 92% cpu 3.159 total
54,753
  2.72s user 0.37s system 94% cpu 3.285 total
34,753
  2.68s user 0.37s system 95% cpu 3.198 total
34,753
  2.51s user 0.37s system 93% cpu 3.066 total
54,753
  0.05s user 0.14s system 35% cpu 0.540 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/decorator/runtime $ cd ../compiletime                                                                                                        17:14
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtual-or-not/decorator/compiletime $ bem main                                                                                                             17:14
  2.38s user 0.39s system 91% cpu 3.031 total
38,177
  2.48s user 0.38s system 90% cpu 3.171 total
33,649
  2.46s user 0.38s system 93% cpu 3.054 total
34,337
  2.38s user 0.38s system 91% cpu 3.033 total
38,177
  0.05s user 0.13s system 31% cpu 0.561 total






- check two version
https://github.com/igl42/cpp_software_design/blob/main/G36_Compile_Time_Decorator.cpp
https://github.com/igl42/cpp_software_design/blob/main/G36_Runtime_Decorator.cpp




refer to Klaus' book

compile time
runtime