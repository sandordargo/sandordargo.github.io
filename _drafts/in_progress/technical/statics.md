**What does a static member variable in C++ mean?**

Denoted by the static keyword, a static variable is allocated storage, in the static storage area, only once during the program lifetime. Given that there is only one single copy of the variable for all objects, it's also called as a class member.

When we declare a static member variable inside a class, we’re telling the compiler about the existence of a static member variable, but not actually defining it (much like a forward declaration). Because static member variables are not part of the individual class objects (they are treated similarly to global variables, and get initialized when the program starts), you must explicitly define the static member outside of the class, in the global scope.

class A {
	static MyType s_var;
};

MyType A::s_var = value;

Though there are a couple of exceptions. First, when the static member is a const integral type (which includes char and bool) or a const enum, the static member can be initialized inside the class definition:

class A {
	static int s_var{42};
};

Static constexpr members can be initialized inside the class definition.

class A {
	static constexpr std::array<int, 3> s_array{ 1, 2, 3 } // this would work for any class supporting consexpr initialization
};

If you are calling a static data member within a member function, member function should be declared as static.

It's been already mentioned but it's worth to emphasize that static member variables are created when the program starts and destoryes when the program ends and as such static members exist even if no objects of the class have been instantiated.


**What does a static member function in C++ mean?**

Static member functions can be used to work with static member variables in the class or to perform operations that do not require an instance of the class, yet conceptually it strongly relates to the class. Some key points about static member functions.

-Static member functions don't have `this` pointer
-Any static member function can't be virtual
-Static member functions cannot access non-static members
-The const, const volatile, and volatile declaration aren't available for static member functions

In practice, being a static member means that you can call a function without having the object instantiated. If you have a `static void bar()` on `Foo` class, you can call it like this `Foo::bar()` without having `Foo` ever instantiated. (You can also call it on an isntance by the way: `aFoo.bar()`). As `this` pointer always holds the memory address of the current object and to call a static member you don't need an object at all, it cannot have a `this` pointer.

A virtual member is something that doesn't relate directly to any class, only to an instance.  A "virtual function" is (by definition) a function which is dynamically linked, i.e. it's chosen at runtime depending on the dynamic type of a given object. Hence, it there is no object, there cannot be a virtal call.

Accessing a non-static member requires that the object has been constructed but for static calls, we don't pass any instantiation of the class. It's not even guaranteed that any instance has been constructed.


Once again, the `const` and the `const volatile` keywords modify whether and how an object can be modified or not. As there is no object...

**What is the static initialization order fiasco?**

The static initialization order fiasco is a subtle aspect of C++ that many don't know about, don't consider or misunderstand. It's hard to detect as the error often occurs before `main()` would be invoked.

Static or global variables in one translation unit are always initialized according to their definition order. On the other hand, there is no strict order for which translation unit is initialized first. 

In case, you have a trasnlation unit A with a static variable sA, which depends on static variable sB from translation unit B in order to get initialized, you have 50% chance to fail. This is the **static initialization order fiasco**.

Let's see an example for the fiasco. 

```cpp
//Logger.cpp
#include <string>

std::string theLogger = "aNiceLogger";


//KeyBoard.cpp
#include <iostream>
#include <string>

extern std::string theLogger;

std::string theKeyboard = "The Keyboard with logger: " + theLogger;

int main() {
  std::cout << "theKeyboard: " << theKeyboard << "\n";
}

```

In this example, we have a Keyboard (driver) and a Logger. Let's consider that the Keyboard driver needs a logger, so in case of an issue, it has somewhere to log them. Due to some reasons, we decided there can be one keyboard and one logger, so we made them global. Obviously not a great design decision, but it's not unique and it's also for the sake of the example.

If the keyboard is initialized sooner than the logger, there is an issue, as you can see it in the below example:
```
g++ -c Logger.cpp
g++ -c Keyboard.cpp
g++ Logger.o Keyboard.o -o LoggerThenKeyboard
g++ Keyboard.o Logger.o -o KeyboardThenLogger

sdargo@host ~/personal/dev/examples/static_fiasco $ ./KeyboardThenLogger 
theKeyboard: The Keyboard with logger: 

sdargo@host ~/personal/dev/examples/static_fiasco $ ./LoggerThenKeyboard 
theKeyboard: The Keyboard with logger: aNiceLogger

```
This is the static initialization order fiasco in action.

Beware that dependencies on static variables in different translation units are a code smell and in fact, should be a good reason for refactoring. 

References:
[C++ FAQ](http://www.cs.technion.ac.il/users/yechiel/c++-faq/static-init-order.html)
[ModernesC++](https://www.modernescpp.com/index.php/c-20-static-initialization-order-fiasco)

**How to solve the static initialization order fiasco?**

As a reminder, static or global variables in one translation unit are always initialized according to their definition order. On the other hand, there is no strict order for which translation unit is initialized first. 

In case, you have a trasnlation unit A with a static variable sA, which depends on static variable sB from translation unit B in order to get initialized, you have 50% chance to fail. This is the **static initialization order fiasco**. 

Dependencies on static variables in different translation units are a code smell and in fact, should be a good reason for refactoring. Hence the most straightforward way to solve this problem is to remove such dependencies.

I recite here our example from yesterday.

```cpp
//Logger.cpp
#include <string>

std::string theLogger = "aNiceLogger";


//KeyBoard.cpp
#include <iostream>
#include <string>

extern std::string theLogger;

std::string theKeyboard = "The Keyboard with logger: " + theLogger;

int main() {
  std::cout << "theKeyboard: " << theKeyboard << "\n";
}

```

There is a 50-50% chance of failure. If the compilation unit Logger.cpp gets initialized first, we are good. If the keyboard, not so.

The probably simplest solution is that we replace `theLogger` variable in Logger.cpp with a function like this:

```cpp
std::string theLogger() {
  static std::string aLogger="aNiceLogger";
  return aLogger;
}
```

Then in the Keyboard.cpp we just have to make sure that we use extern on the function and we call the function later on instead of referecing the variable. This works because the local `static std::string aLogger` variable will be initialized the first time that `theLogger()` function is called. Hence, it's guaranteed that when `theKeyboard` is constructed, `theLogger` will be initialized.

You might face other issues if `theLogger` would be used during program exit by another static variable after `theLogger` got constructed. Again, dependencies on static variables in different translation units are a code smell...

Starting from C++20, the static initialization order fiasco can be solved with the use of `constinit`. In this case, the static variable will be initialized at compile time, before any linking. You can check that solution [here](https://www.modernescpp.com/index.php/c-20-static-initialization-order-fiasco).

References:
[C++ FAQ](http://www.cs.technion.ac.il/users/yechiel/c++-faq/static-init-order.html)
[ModernesC++](https://www.modernescpp.com/index.php/c-20-static-initialization-order-fiasco)
