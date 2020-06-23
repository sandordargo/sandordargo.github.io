---
layout: post
title: "Class initialization and nasty cores"
date: 2018-4-11
category: dev
tags: [cpp, code, debugging]
header: "Are you the type of person who doesn't pay attention to details? Start using C++. It will give the missing discipline if there is any chance. If you don't pay attention to details, C++ will teach you to do it, or you will leave this language pretty soon. It simply does not tolerate ignorance and laziness. It will hit back when you expect it the least with bloating memory leaks and dramatic core dumps."
---
I've started to work on an old and big application recently so that I can practice what I read in [Michael Feathers'](https://michaelfeathers.silvrback.com/) must-read book on [Working with legacy code](https://amzn.to/2Jf4EWC).

A week ago my most experienced colleague (experience != years of service) sent me link pointing to a file in our code repository with the brief message of "spot the core dump".

It turned out that the erroneous code was there for quite a significant amount of time and it was easily reproducible "just by two lines". To be more exact you could navigate in just two lines your object into a state where it would core on the necessary function call. It doesn't sound a difficult scenario, does it?

Here is a simplified version of the code:

```
class Member {
public:
  int getANumber() const {
    return _number;
  }

private:
  int _number;
};

class CoringClass {
public:
  CoringClass() {
    _member = 0;
  }
  
  CoringClass(const CoringClass& other) {
    if (other._member) {
      _member = new Member();
      *_member = (*(other._member));
    }
  }
  
  Member* accessMember() {
    return _member;
  }

private:
  Member* _member;
};

```

Can you already see the error? If yes, you have great eyes! If not, don't worry. It took some time for my colleague. For me, even more. In fact, that's why I'm writing this article. To help others as well as me to recognize such issues more easily.

Now I'm convinced that even if you wouldn't write such a code, it's more difficult to recognize it than not causing it.

Here are the three lines where the last one will actually produce undefined behaviour but in a more realistic class, it would core.

```
CoringClass notYetCoring;
CoringClass coring(notYetCoring);
int whatHappens = coring.accessMember()->getANumber();

```

The biggest problem with the above code is that `CoringClass` in certain conditions fails to initialize its member variable.

Let's have a quick reminder how C++ initializes its members:
[POD](https://en.wikipedia.org/wiki/Passive_data_structure) members of the class will be zero-initialized through the default constructor, even without an explicit initiation in the constructor. [But a raw pointer as a member of the class will not be zero initialised!](https://stackoverflow.com/questions/26142100/c-default-constructor-does-not-initialize-pointer-to-nullptr)

It means that `coring.acceddMmember()` can point anywhere in the memory. If you are lucky, when you try to use it, it will core directly. If you are less fortunate, it will return you some nonsense value and your application will keep running using that value.

Check what happens if you print `coring.accessMember()`. This is a possible output:

```
0x722da2fc9910
```

In order to fix the code, there are several options, but the copy constructor must be fixed. When you use the copy constructor, you must take care of the initialization of the new object. The default constructor is not called, so `_member` should be initialized in the copy constructor.

One way is that you explicitely initialize the `_member` to 0.
```
CoringClass(const CoringClass& other) : _member(0) {
  ...
}
```

If you print `coring.accessMember()` now, you'll get a predictable `0`. That's good. The behaviour is not undefined anymore, you can make checks against that `0` value. 

It is an option now to change the `accessMember()` function so that in case it points to `0`, it initializes itself.

```
Member* accessMember() {
   if (_member == nullptr) {
       _member = new Member();
   }
   return _member;
}
```

You can also choose to check the nullity of `_member` returned by `accessMember()` whenever you try to access it. Although it is more secure if you have a default `Member` rather than dealing with `nullptr`s.

The key lesson here is that you should always initialize all the members of a class in C++. 

You might count on the compiler because it actually does initialize some members by default, but it's safer to be always explicit - it leaves <strike>no</strike> fewer opportunities to fail... But either you are explicit or not, always make sure that all of the constructors initialize all the members.