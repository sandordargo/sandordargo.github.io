---
layout: post
title: "Does this string declaration compile?"
date: 2019-11-27
category: dev
tags: [cpp, puzzle, tricky]
excerpt_separator: <!--more-->
---
Fellow C++ developers out there! 

I have a question for you! Will the following code compile? If not, why not? If it does, why?
<!--more-->

```cpp
#include <string>

int main() {
  std::string(foo);
}

```

Spend some time thinking about it before you paste it to [coliru](http://coliru.stacked-crooked.com/a/c26a89cce6f9e07e) or to [godbolt](https://godbolt.org/z/_obTTn) directly.

## The answer is...

...obviously 42. And if you treat the integer of 42 a boolean? It is considered `true`! So yes, this above code does compile.

To be more exact, it depends... It depends on wehther you treat warnings as errors or not. But let's not run forward so fast.

Why would it compile in any case? Foo is not a variable defined anywhere, not even in the global namespace.

I've seen this question in [a video from the CppCon 2017](https://youtu.be/3MB2iiCkGxg?t=1745) and about 90% of the attendees got it wrong.

I would have thought that this code will try to create a temporary string with the content of the variable foo. And of course, if foo is not defined, the code will not compile.

Check [this code](http://coliru.stacked-crooked.com/a/4afcd5b18a0fd9f6):

```cpp
#include <string>

int main() {
  auto bar = std::string(foo);
}
```
The compiler tells you that _'foo' was not declared in this scope_.

But let's go back to our example that only emits a warning. Go and check on [godbolt](https://godbolt.org/z/_obTTn) the assembly code generated for the above snippet. You can see that it actually creates a string.

What it exactly does is creating an empty string and assign it to a variable called `foo`.

The following two lines mean the same:

```
std::string(foo);
std::string foo;
```

I'm not fooling you.

## The cause

Have you ever heard about [the most vexing parse](https://en.wikipedia.org/wiki/Most_vexing_parse)?

If not and if you code in C++, I'm pretty sure you made a similar mistake at some point in your coding career:

```cpp
// ...
Widget w();
// ...

```

And while you wanted to define a local variable of type `Widget` calling its default constructor, instead what you got was a compiler error. It's C++. So pages of compiler errors.

In short, the most vexing parse says that if something can be interpreted as a declaration, it will be interpreted as a declaration.

The above line can be interpreted as a declaration of function `w` that takes no parameters and returns a Widget, so according to the section 8.2 of the [C++ language standard](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/n4713.pdf) this code not just can be but will be interpreted as a declaration.

For inexperienced eyes (like mine are), the standard seems quite cryptic, but you have to read (a couple of dozens time) section 8.6 and 8.2 to get to the point.

The bottom line is that you should avoid writing ambiguous code because you might end up with unwelcome surprises.

How to write unambiguous code? Use [brace initalization](http://www.modernescpp.com/index.php/initialization) if you are at least on C++11!

What is that? It's simple, instead of parentheses, use braces to call the constructor!

```
Widget w(); // declaring a function w()
Widget w{}; // calling Widget::Widget() with an empty list of parameters!
```

Using the braces this programs stops compiling, as expected. It's not ambiguous anymore! And by the way, ambiguous code emits warnings by the compiler if you treat your warnings as errors, even the original code would not compile.

```cpp
#include <string>

int main() {
  std::string{foo};
}
```

## And in real life?

Now think about a more complex case than declaring a string. Think about a mutex.

```cpp
#include <mutex>
 
static std::mutex m;
static int shared_resource;
 
void increment_by_42() {
  std::unique_lock<std::mutex>(m);
  shared_resource += 42;
}
```

What is happening here?

At the beginning of the article, you might have thought about that okay, we create a temporary unique_lock, locking mutex m. Well. No. I think you can tell on your own what's happening there. It might be sad, but true. According to the talk that inspired this article, this was a quite recurring bug at Facebook. They just created a lock on the type of a mutex and called that lock m. But nothing got locked.

But if you express your intentions by naming that lock, or if you brace initialization it'll work as expected.

```cpp
#include <mutex>
 
static std::mutex m;
static int shared_resource;
 
void increment_by_42() {
  std::unique_lock<std::mutex> aLock(m); // this works fine
  // std::unique_lock<std::mutex> {m}; // even this would work fine
  shared_resource += 42;
}
```

By the way, using `-Wshadow` compiler option would have also caught the problem by creating a warning. Treat all warnings as errors and be happy!

## Conclusion

C++ can be tricky and the standard is long but at least not easy to read. We've seen what most vexing parse is and how ambiguity can lead to unexpected behaviour. You have a couple of good weapons that will help you fight against these unwanted surprises.
 
* Brace initialization removes ambiguity
* Treat warnings as errors if you have the opportunity!
* Read, watch talks and educate yourself to know about nuances!

Happy coding!