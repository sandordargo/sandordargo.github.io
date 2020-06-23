---
layout: post
title: "Try-catch everything without macros"
date: 2020-6-24
category: dev
tags: [cpp, template, bestpractices, cleancode]
excerpt_separator: <!--more-->
---
We all have our vices. One of mine is that I tend to jump in code reviews quickly, without considering how much time will be taken if I find something I don't like.

Recently I opened PR that seriously increased my WTF/minute level. Something struck me so hard that I felt I had to block the merge right away and take a cup of water before saying something thoughtlessly.

A new macro. In 2020. 
<!--more-->

To me, that's an automatic no-no. It's not a definitive no as there might be some justifiable cases, but in the vast majority, they have no raison d'etre. So better to block before enough less pedantic fellows would approve and merge.

## So what was the problem?

We have been introducing a new data logging framework to allow us to have more detailed insights on the requests we process. In turned out that some data that we wanted to add to our logs were not always available. While we tried to access them in their absence, exceptions were thrown. After taking into account several possibilities, the team decided to wrap the calls with try-catch blocks.

But how do that?

## The naive approach

An obvious option is to wrap each call separately.

The could code look like this:

```cpp
void fill1(params...) {
  try {
    auto someData = call1(params...);
    log(someFixedKey, someData);
  } catch (const ExceptionType& ex) {
    //...
  } catch (...) {
    //...
  }
}

//...
void fill2(params...) {
  try {
    auto someData = call2(params...);
    log(someFixedKey, someData);
  } catch (const ExceptionType& ex) {
    //...
  } catch (...) {
    //...
  }
}
```

And repeat this n times.

It's cumbersome to write it, difficult to maintain, and as such error-prone. In case we need a modification in try-catch blocks, there is a fair chance to make a mistake.

You might argue that multiple calls should be wrapped together, but if one call fails we would like to go with the next one. Wrapping all together is not a viable option as it would end the logging on the first failure.

## Precompiling the macros

The solution implemented in the pull request was using the precompiler, so a macro, shortening significantly the implementation:

```cpp
# DEF...

void fill1(params...) {
  BEGIN_TRY
  auto someData = call1(params...);
  log(someFixedKey, someData);
  END_TRY
}

//...
void fill2(params...) {
  BEGIN_TRY
  auto someData = call2(params...);
  log(someFixedKey, someData);
  END_TRY
}

```

This is a shorter way to achieve the same functionality, and you might argue that it's more maintainable. After all, in case you want to add a new catch block, or if you just want to modify an existing one, you have to modify it in one place, where you declare the macro.

So it's shorter and you have one single point to update in case of modification. Then what's the matter? Don't we have a permanent solution?

It's very easy to make a mistake while writing a macro simply because it's difficult to write one. It follows a different and less readable syntax, one we are not used to. Thus it will be a hotbed of bugs. For the author, it's more difficult to write and for the code reviewer, it's also more difficult to read.

In addition, it'll be more difficult to hunt down bugs as the debugging of macros is more difficult. Why? After all, a macro is not a function. It's just text replaced by its definition right before the compilation starts (by the precompiler).

This fact also complicates life if you use static code analyzers. Sometimes macros just create a bunch of false positives and there is no great way to get rid of them - except for getting rid off the macros.

But even the compiler can have false positives. When we were removing all our compiler warnings from our codebase, the compiler considered variables only used in a macro an unused variable.

_You can find more details on why you should avoid macros in [this article from Arne Mertz](https://arne-mertz.de/2019/03/macro-evil/)_

## Using the power of templates

When I saw that we want to wrap each of those small functions, I immediately thought about decorators from Python (or Java for that matter).
Wouldn't it be perfect to write something like this?

```cpp
@try
void fill1(params...) {
  auto someData = call1(params...);
  log(someFixedKey, someData);
}
```

And then define that wrapper somewhere like this?

```cpp
auto try(F(params)) -> std::decltype(F(params)) {
  try {
    return F(params)

  } catch (const ExceptionType& ex) {
    //...
  } catch (...) {
    //...
  }
}
```
Obviously this is not valid syntax, but how could we achieve a similar effect? What are the problems we have to solve?

The main problem is that - as far as I know - you cannot just pass a function call with all its parameters to another function. At least not with the usual syntax of a function call: `a(b, c)`.

Instead, you can pass a function pointer and a list of arguments, that's easily doable.

So, in theory, we could have an interface that we can use somehow like this:

```cpp
safeFill(&fill1, param1, param2 /*etc*/);
```

As a first step, I tried to do something that works with one only parameter of a fixed type.

```cpp
#include <iostream>
#include <string>

class Logger {
public:
  void logA(std::string s) {
    std::cout << "A: " << s << std::endl;
  }
  
  void logB(std::string s) {
    std::cout << "B: " << s << std::endl;
  }
      
};

template <typename Function>
auto safeLog(Function f, Logger* l, std::string s) -> decltype((l->*f)(s)) {
  try {
    std::cout << "Logging s safely..." << std::endl;
    return (l->*f)(s);
  }
  catch(...) {
    std::cout << "s is not logged, we have an exception" << std::endl;
    throw;
  }
}

int main () {
  Logger l;
  std::string s("bla");
  safeLog(&Logger::logA, &l, s);
  safeLog(&Logger::logB, &l, s);
}
```

So where do we stand compared to what we wanted?

Now we can wrap any call with a given type of parameter with a try-catch block.

What are the things that I don't like:
- The return type (`decltype((l->*f)(s))`)
- The parameter is not flexible (nor in type or in numbers)
- We have to pass both a function pointer and a pointer to the instance containing that function.

### Getting rid of that fancy return type

While calling `decltype()` will only return the resulting type of the passed expression, it's something that would be nice to avoid. After all, it repeats our `return` statement.

Nothing is easier than that, you can simply omit it and have this instead:
```cpp
template <typename Function>
auto safeLog(Function f, Logger* l, std::string s) {
  // the body goes unchanged
}
```
But you can only do this if you use C++14 since it introduced the return type deduction for functions where all the returns return the same type. For C++11 you have to bear with `decltype`.

### Making our parameter list flexible

You want to be able to deal with any number/type of parameters? Easy-peasy, just squueze a little variadic template type into `safeFill`:

```cpp
template <typename Function, typename ... Args>
auto safeLog(Function f, Logger* l, Args&& ... args) {
  try {
    std::cout << "Logging s safely..." << std::endl;
    return (l->*f)(std::forward<Args>(args)...);
  }
  catch(...) {
    std::cout << "s is not logged, we have an exception" << std::endl;
    throw;
  }
}
```

Using variadic template types (`typename ... Args`) let us taking as many parameters as we want and of different types. Taking them by universal reference (`&&`) and perfect forwarding them (`std::forward<>()`) is not mandatory, but using both of them has positive impacts on performance due to fewer object copies. _(Going into details about perfect forwarding is out of scope today.)_

### Dealing with the need for a function pointer and a pointer to the object

The last point we wanted to address is that the call of the function is rather ugly:
```cpp
safeLog(&Logger::logA, &l, s);
```

It would be great to be able to call the function simply by `safeLog(&l::logA, s)`. It would be, but it's not possible. For the being, it's not possible to pass a pointer to a member function of a class instance.

If we reorganize our code and push `safeLog()` to be a member of `class Logger` and accept that it will only work with the current object then we can get rid off the second parameter:

```cpp
#include <iostream>
#include <string>

class Logger {
public:
  void logA(std::string s) {
    std::cout << "A: " << s << std::endl;
  }
  
  void logB(std::string s, int n) {
    std::cout << "B: " << s << " " << n << std::endl;
  }

  template <typename Function, typename ... Args>
  auto safeLog(Function f, Args&& ... args) {
    try {
      std::cout << "Logging s safely..." << std::endl;
      return (this->*f)(std::forward<Args>(args)...);
    }
    catch(...) {
      std::cout << "s is not logged, we have an exception" << std::endl;
      throw;
    }
  }
      
};

int main () {
  Logger l;
  std::string s("bla");
  l.safeLog(&Logger::logA, s);
  l.safeLog(&Logger::logB, s, 42);
}
```

### A more real-world example

So far we have seen how to use macros and templates in order to wrap function calls with try-catch blocks. Then we simplified the template as much as we could by pushing it to a class, using variadic templates, and by using C++14 we could remove even the return type and benefit from return type deduction.

Still it feels strange to use `safeLog` from the outside with some hardcoded variables. Here is a more complete example also with a safely swallowed exception:

```cpp
#include <iostream>
#include <string>
#include <exception>

class DataAccessor {
public:

  std::string getA() const {
    // normally in these functions there would be more comlpex computation
    // or calls to the DB, etc
    return a;
  }
  
  int getB() const {
    return b;
  }
  
  float getC() const {
    throw std::exception{};
  }

private:
  std::string a{"this is a string"};
  int b{42};
};

class Logger {
 private:
  // this has to come before we use it
  // with a header file this is not an issue
  template <typename Function, typename ... Args>
  auto safeLog(Function f, Args&& ... args) {
    try {
      std::cout << "Logging safely..." << std::endl;
      return (this->*f)(std::forward<Args>(args)...);
    }
    catch(...) {
      std::cout << "s is not logged, we have an exception" << std::endl;
        
    }
  }

 public:
  void logData(const DataAccessor& data) {
    safeLog(&Logger::logA, data);
    safeLog(&Logger::logB, data);
    safeLog(&Logger::logC, data);
  }
  // void logOtherKindOfData(...);
 private:
  void logA(const DataAccessor& data) {
    std::cout << "A: " << data.getA() << std::endl;
  }
  
  void logB(const DataAccessor& data) {
    std::cout << "B: " << data.getB() << std::endl;
  }
  
  void logC(const DataAccessor& data) {
    std::cout << "C: " << data.getC() << std::endl;
  }
  // ...
};

int main () {
    DataAccessor d;
    Logger l;
    l.logData(d);
}
```

This is still a simplified example, but it's closer to a real-life one with an object that is responsible to fetch some data (possibly from a database).

A reference to our data accessor class is passed to the logger which takes care of calling the right getters to read the data from somewhere else. This `DataAccessor` in the example is simplified as much as possible.

On the other hand, it's realistic that the code of `Logger::logA`, `Logger::logB`, and the rest is not just dumped into a huge method. As such moving from the conventional logging to a safe log is very easy. By replacing `logA(data)` with `safeLog(&Logger::logA, data)` we get a version that is not prone to exceptions thrown in the `DataAccessor`.

## Conslusion

Today we saw how to wrap function calls with try-catch blocks in C++ with macros and with templates. Macros are error-prone and hard to debug as the precompiler changes the code that you actually wrote before the compilation started.

The other solution, using templates gives us a bit more boilerplate (still on a manageable level I think) and the calling syntax is a little bit different, but I think it's worth the advantages of not having a macro, but debuggable code and in overall, better readability.

What do you think?

Happy coding!