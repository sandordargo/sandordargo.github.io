Fellow C++ developers out there! 

I have a question for you! Will the following code compile? If not, why not? If it does, why?

```
#include <string>

int main() {
  std::string(foo);
}

```

This about it a little, before you paste it to [coliru](http://coliru.stacked-crooked.com/a/c26a89cce6f9e07e) or to [godbolt](https://godbolt.org/z/_obTTn) directly.

## The answer...

...is obviously 42. And if you treat the integer of 42 a boolean. it is considered true. So yes, this above code does compile.

To be more exact, it depends. It depends on whether you treat warnings as errors or not. But let's not run forward so fast.

But why would it compile in any case? Foo is not a variable defined anywhere, not even in the local namespace.

I've seen this question in a video from the CppCon 2017 and about 90% of the attendees got it wrong.

I would have thought that this code will try to create a string with the content of the foo variable, but we don't save it into any other variable. And of course if `foo` is not defined, the code will not compile.

Check [this code](http://coliru.stacked-crooked.com/a/4afcd5b18a0fd9f6):

```
#include <string>

int main() {
  auto bar = std::string(foo);
}

```

The compiler tells you that _'foo' was not declared in this scope_.

But let's go back to our example that only emits a warning. If you check on [godbolt](https://godbolt.org/z/_obTTn) what is the assembly code is generated for the above snippet. You can see that it actually creates a string.

What it exactly does is creating an empty string under a variable called `foo`.

I'm not fooling you.

## The cause

Have you ever heard about [the most vexing parse](https://en.wikipedia.org/wiki/Most_vexing_parse)?

If not and if you code in C++, I'm pretty sure you made a similar mistake at some point in your carreer:

```
// ...
Widget w();
// ...

```

And while you wanted to define a local variable of type `Widget` calling its default constructor, instead what you got was a compiler error.

In short, the most vexing parse says that if something can be interpreted as a declaration, it will be interpreted as a declaration.

The above line can be interpreted as a declaration of function `w` that takes no paramters and returns a Widget, so according the section 8.2 of the [C++ language standard]() this code not just can be but will be interpreted as a declaration.

For inexperieced eyes (like mine are), the standard seems quite cryptic, but you have to read (a couple dozens time) section 8.6 and 8.2 to get to the point.

The bottom line is that you should avoid writing ambiguous code, because you might end up with unwelcome suprises.

How to write unambiguous code? Use brace initalization if you are on at least C++11!

What is that? It's simple, instead of perentheses, use braces to call the constructor!

```
Widget w(); // declaring a function w()
Widget w{}; // calling Widget::Widget() with an empty list of parameters!
```

Using the braces this programs stops compiling, as expected. It's not ambiguous anymore!

```
#include <string>

int main() {
  std::string{foo};
}

```

And now think about a more complex case, then declaring a string. Think about a mutex.

```
#include <mutex>
 
static std::mutex m;
static int shared_resource;
 
void increment_by_42() {
  std::unique_lock<std::mutex>(m);
  shared_resource += 42;
}

```

What is happening here?

At the beginning of the article, you might have thought about that okay, we create a temporary unique_lock, locking mutex m. Well. No. I think you can tell on your own what's happening there. It might be sad, but true. According to the talk that inspired this article this was quite recurring bug at Facebook.

But if you express your inentions by naming that lock or by using the brace initalization, it'll work as expected.

```
#include <mutex>
 
static std::mutex m;
static int shared_resource;
 
void increment_by_42() {
  std::unique_lock<std::mutex>lock(m);
  shared_resource += 42;
}

```

# Takeaways




https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL53-CPP.+Do+not+write+syntactically+ambiguous+declarations

6.2 and 8.2
http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/n4713.pdf
https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL53-CPP.+Do+not+write+syntactically+ambiguous+declarations

# Check why
If it's ambiguos and it can be a  declaration it's a declation.


# Share the example of mutexes

# Takeaways
