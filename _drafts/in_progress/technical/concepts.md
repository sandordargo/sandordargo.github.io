The idea of concepts is one of the major new features added to C++20. Concepts are an extension for templates. They can be used to perform compile-time validation of template arguments through boolean predicates.

Concepts can be used in 4 different ways as we'll see, we can use predefined concepts that we can find in the `std` namespace defined in the `<concepts>` header but we can also write our own ones.

In this article, we'll see 
- the motivation behind concepts
- how they related to templates
- the 4 ways to use them
- some of the predefined concepts
- how to write our own ones

## The motivation behind concepts

Let's say you want to write a function that adds up two numbers. You want to accept both integers and floating point numbers. What are you going to do?

You could accept `double`s, maybe even `long double`s and return a value of the same type. 

```cpp
#include <iostream>

long double add(int a, int b) {
    return a+b;
}

int main() {
  int a{42};
  int b{66};
  std::cout << add(a, b) << '\n';
}
```

The problem is that `add` with `int`s they will be casted to `long double`. You might want a smaller memory footprint, or maybe you'd like to tkae into account the maximum or minimum limits of a type.

Defining overloads for the different types is a way out, but definitely tedious.

```cpp
#include <iostream>

long double add(long double a, long double b) {
  return a+b;
}

int add(int a, int b) {
  return a+b;
}

int main() {
  int a{42};
  int b{66};
  std::cout << add(a, b) << '\n';
}
```
Imagine that you want to do this for all the different numeric types. Thanks, but no thanks.

Another option is to define a template!

```cpp
#include <iostream>

template <typename T>
T add(T a, T b) {
    return a+b;
}

int main() {
  int a{42};
  int b{66};
  std::cout << add(a, b) << '\n';
  long double x{42.42L};
  long double y{66.6L};
  std::cout << add(x, y) << '\n';
  
}
```
If you have a look at [CPP Insights](https://cppinsights.io/lnk?code=I2luY2x1ZGUgPGlvc3RyZWFtPgoKdGVtcGxhdGUgPHR5cGVuYW1lIFQ+ClQgYWRkKFQgYSwgVCBiKSB7CiAgICByZXR1cm4gYStiOwp9CgppbnQgbWFpbigpIHsKICBpbnQgYXs0Mn07CiAgaW50IGJ7NjZ9OwogIHN0ZDo6Y291dCA8PCBhZGQoYSwgYikgPDwgJ1xuJzsKICBsb25nIGRvdWJsZSB4ezQyLjQyTH07CiAgbG9uZyBkb3VibGUgeXs2Ni42TH07CiAgc3RkOjpjb3V0IDw8IGFkZCh4LCB5KSA8PCAnXG4nOwogIAp9Cg==&insightsOptions=cpp17&std=cpp17&rev=1.0) you will see that code was generated both for an `int` and for a `long double` overload. There is no static cast taking place at any point.

Are we good yet?

Unfortuantely, no.

What happens if you try to call `add(true, false)`? You'll get a `1` as `true` is promoted to an integer, summed up with `false` promoted to an integer and then they will be turned back into a boolean.

What if you add up two string? They will be concatanted. But is that really what you want? Maybe you don't want that to be a valid operation and you prefer a compilation failure.

So you might have forbid that [template specialization](https://dev.to/pgradot/forbid-a-particular-specialization-of-a-template-4348). And for how many types do you want to forbid?

What if you could simply say that you only want add up integral or floating point types. And here come `concepts` into picture.

With concepts you can easily express such requirements on template parameters.

Let's see how. You have a few different ways.

## The 4 ways to use concepts

To be more specific, you have four different ways to do it.
For all the prsented ways, let's assime that we have a concept Number. Let's just use a dummy implementation, so if you want to play with it, you can, but it's incomplete in a functional sense. More about that later.

```cpp
#include <concepts>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;
```

### Using the `requires` clause

```cpp
template <typename T>
requires Number<T>
auto add(T a, T b) {
  return a+b;
}
```
Note how we use the concept, how we define in the `requires` clause that any `T` template parameter must satisfy the requirements of the concept `Number`.

In order to determine the return type we simply use `auto` type deduction, but we could be as well using `T` there too.

Unfortunately, we can only add up two numbers of the same type. We cannot add a `float` with an `int`

If we tried so, we'd get a bit long, but quite understandable error message:

```
main.cpp: In function 'int main()':
main.cpp:15:27: error: no matching function for call to 'add(int, float)'
   15 |   std::cout << add(5,42.1f) << '\n';
      |                           ^
main.cpp:10:6: note: candidate: 'template<class T>  requires  Number<T> auto add(T, T)'
   10 | auto add(T a, T b)  {
      |      ^~~
main.cpp:10:6: note:   template argument deduction/substitution failed:
main.cpp:15:27: note:   deduced conflicting types for parameter 'T' ('int' and 'float')
   15 |   std::cout << add(5,42.1f) << '\n';
      |                           ^

```

### Trailing `requires` clause

```cpp
template <typename T>
auto add(T a, T b) requires Number<T> {
  return a+b;
}
```
Here we have the same result we just wrote it with different semantics. It means we still cannot add up to numbers of the different type. In fact, we'd need to modify the template definition as such:

```cpp
template <typename T, typename U>
auto add(T a, U b) requires Number<T> && Number<U> {
  return a+b;
}
```
Not bad, we still have to satisfy the concept of `Number`, but beyond a certain number of parameters it becomes impractical.

### Constrained template parameter

```cpp
template <Number T>
auto add(T a, T b) {
  return a+b;
}
```

As you can see, we don't need any `requires` clause, we can simply define a requirement on our `T` tempalte parameter right where we declare it. We'll achieve the very same result.

At the same time, it's worth to note that this method has a limitation. When you use any of the requires clause, you can define an experssion such as `requires std::integral<T> || std::floating_point<T>`. When you use the template parameter, you cannot have such expressions `template <std::integral || std::floating_point T>` __is not valid__.

So with this way, you can only use single concepts, but it's a briefer form as the previous ones.

### Abbreviated function template

Oh, you looked for brevity? Here you go!

```cpp
auto add(Number auto a, Number auto b) {
  return a+b;
}
```
There is no need for any template parameter definition, nor requires clause. You can directly use the concept where the function arguments are enumerated.

There is one thing to notice and more to mention.

After the concept `Number` we put `auto`. As such we can see that Number is a constraint on the type, not a type itself. Imagine if you'd simply see `auto add(Number a, Number b)`. How would you know that `Number` is not a type but a concept?

The other thing I wanted to mention is that when you follow the abbriaved function template way, you can mix the types of the parameters. You can add up an `int` with a `float`.

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;

auto add(Number auto a, Number auto b) {
  return a+b;
}

int main() {
  std::cout << add(1, 2.5) << '\n';
}
/*
3.5
*/
```

### Putting them all together

We have just seen 4 ways to use concepts, let's have a look at them together.

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;

template <typename T>
requires Number<T>
auto addRequiresClause(T a, T b) {
  return a+b;
}

template <typename T>
auto addTrailinRequiresClause(T a, T b) requires Number<T> {
  return a+b;
}

template <Number T>
auto addConstrainedTemplate(T a, T b) {
  return a+b;
}

auto addAbbreviatedFunctionTemplate(Number auto a, Number auto b) {
  return a+b;
}

int main() {
    std::cout << "addRequiresClause(1, 2): " << addRequiresClause(1, 2) << '\n';
    // std::cout << "addRequiresClause(1, 2.5): " << addRequiresClause(1, 2.5) << '\n'; // error: no matching function for call to 'addRequiresClause(int, double)'
    std::cout << "addTrailinRequiresClause(1, 2): " << addTrailinRequiresClause(1, 2) << '\n';
    // std::cout << "addTrailinRequiresClause(1, 2): " << addTrailinRequiresClause(1, 2.5) << '\n'; // error: no matching function for call to 'addTrailinRequiresClause(int, double)'
    std::cout << "addConstrainedTemplate(1, 2): " << addConstrainedTemplate(1, 2) << '\n';
    // std::cout << "addConstrainedTemplate(1, 2): " << addConstrainedTemplate(1, 2.5) << '\n'; // error: no matching function for call to 'addConstrainedTemplate(int, double)'
    std::cout << "addAbbreviatedFunctionTemplate(1, 2): " << addAbbreviatedFunctionTemplate(1, 2) << '\n';
    std::cout << "addAbbreviatedFunctionTemplate(1, 2): " << addAbbreviatedFunctionTemplate(1, 2.5) << '\n';
}
```

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Number = (std::integral<T> || std::floating_point<T>) && (not std::same_as<T, bool>);

template <typename T> 
requires Number<T>
T f(T t) {
    return 2 * t + 3;
}

template <typename T> 
auto f2(T t) requires Number<T> {
    return 2 * t + 3;
}

template <Number T>
auto f3(T t) {
    return 2 * t + 3;
}

auto f4(Number auto t) {
    return 2 * t + 3;
}

int main () {
    std::cout << f(2) << std::endl;
    std::cout << f2(2.2) << std::endl;
    std::cout << f3(3) << std::endl;
    std::cout << f4(-2.4) << std::endl;
    // std::cout << f2(true) << std::endl; // auto f2(T) requires  Number<T> [with T = bool]' with unsatisfied constraints
}
```

Which form should we use? As always, the answer is it depends..

If you have a complex requirement, to be able to use an expression you need either the *requires clause* or the *trailing requires clause*. What do I mean by a complex requirement? Anything that has more than one concept in it! Like `std::integral<T> || std::floating_point<T>`. That is something you cannot express neither with a *constrained template parameter* nor with an *abbreviated template function*.

At the same time, it's worth to note that you could define your own concept for such an expression, just like we did when we defined the concept `Number`. On the other hand, if your concept uses multiple parameters (something we'll see soon), you still cannot use *constrained template parameters* or *abbreviated template function*.

If I have comlex requirements and I don't want to define and name a concept, I'd go with either of the first two options.

In case I have a simple requirement, I'd go with the *abbrevieated function template (AFT)*. Though we must remember that *AFTs* let you call your function with multiple different types at the same time, like how we called `add` with an `int` and with a `float`. If that is a problem and you despise the verboseness of the `requires` clause, choose a *constrained template parameter*.

Let's also remember that we talk about templates. For whatever combination, a new specialization will be generated by the compiler at compile time. It's worth to remember this in case you avoided templates already because of contraints on binary size or compile time.

### Predefined concepts

C++20 hasn't only introduced the ability to write concepts, but it also comes with more than 50 concepts in the Standard library shared across three different headers.

#### Concepts in the `<concepts>` header

In the `<concepts>` header you will find the most generic ones for core language concepts, compraision concepts and object concepts.

We are not going to explore all of them here for obvious reasons, you can find [the full list here](https://en.cppreference.com/w/cpp/concepts) and let me just pick three ones so we can get the idea.

- `std::convertible_to` helps you to express that you only accept types that are convertible to a another type that you specify. The converion can be both explicit or implicit. For example you can say that you only accept types that can be converted into a `bool`. At the first place, you will have `From` and at the second `To`.

```cpp
#include <concepts>
#include <iostream>
#include <string>

template <typename T>
void fun(T bar) requires std::convertible_to<T, bool> {
  std::cout << std::boolalpha << static_cast<bool>(bar) << '\n';
}

int main() {
 fun(5); // OK an int can be converted into a pointer
//  fun(std::string("Not OK")); // oid fun(T) requires  convertible_to<T, bool> [with T = std::__cxx11::basic_string<char>]' with unsatisfied constraints
}
```

- `std::totally_ordered` helps to accept types that specify all the 6 comparision operators (`==`,`!=`,`<`,`>`,`<=`,`>=`):

```cpp
#include <concepts>
#include <iostream>
#include <typeinfo> 

struct NonComparable {
  int a;
};

struct Comparable {
  auto operator<=>(const Comparable& rhs) const = default; 
  int a;
};


template <typename T>
void fun(T t) requires std::totally_ordered<T> {
  std::cout << typeid(t).name() << " can be ordered\n";
}

int main() {
  NonComparable nc{666};
//   fun(nc); // Not OK: error: use of function 'void fun(T) requires  totally_ordered<T> [with T = NonComparable]' with unsatisfied constraints
  Comparable c{42};
  fun(c);
}
```

In the above example, you can also observe how to easily use the `<=>` (a.k.a. spaceship) operator to generate all the comparison operators.

- `std::copyable` helps you to ensure that only such types are accepted whose instances can be copied. `std::copyable` object must be copy constructible, assignable and movable.

```cpp
#include <concepts>
#include <iostream>
#include <typeinfo> 

class NonMovable {
public:
  NonMovable() = default;
  ~NonMovable() = default;

  NonMovable(const NonMovable&) = default;
  NonMovable& operator=(const NonMovable&) = default;
  
  NonMovable(NonMovable&&) = delete;
  NonMovable& operator=(NonMovable&&) = delete;
};

class NonCopyable {
public:
  NonCopyable() = default;
  ~NonCopyable() = default;

  NonCopyable(const NonCopyable&) = default;
  NonCopyable& operator=(const NonCopyable&) = default;
  
  NonCopyable(NonCopyable&&) = delete;
  NonCopyable& operator=(NonCopyable&&) = delete;
};

class Copyable {
public:
  Copyable() = default;
  ~Copyable() = default;

  Copyable(const Copyable&) = default;
  Copyable& operator=(const Copyable&) = default;

  Copyable(Copyable&&) = default;
  Copyable& operator=(Copyable&&) = default;
};

template <typename T>
void fun(T t) requires std::copyable<T> {
  std::cout << typeid(t).name() << " is copyable\n";
}

int main() {
  NonMovable nm;
//   fun(nm); // error: use of function 'void fun(T) requires  copyable<T> [with T = NonMovable]' with unsatisfied constraints
  NonCopyable nc;
//   fun(nc); // error: use of function 'void fun(T) requires  copyable<T> [with T = NonCopyable]' with unsatisfied constraints
  Copyable c;
  fun(c);
}
```

#### Concepts in the `<iterator>` header

In the [`<iterator>`](https://en.cppreference.com/w/cpp/iterator#Algorithm_concepts_and_utilities) header you'll mostly find concepts that will come handy when you deal with [algorithms](https://www.sandordargo.com/blog/2019/01/30/stl-algos-intro). It makes sense if you think about it, as the members of the `<algorithms>` header operate on the containers through iterators.

There are concepts related to callables, e.g. you can specify that you accepts any unary predicates. I know that the example is not very practical, it's only for demonstational purposes.

```cpp
#include <iostream>
#include <iterator>
#include <vector>

template <typename F, typename I>
void foo(F fun, I iterator) requires std::indirect_unary_predicate<F, I> {
    std::cout << std::boolalpha << fun(*iterator) << '\n';
}

int main()
{
  auto biggerThan42 = [](int i){return i > 42;};
  std::vector numbers{15, 43, 66};
  for(auto it = numbers.begin(); it != numbers.end(); ++it) {
      foo(biggerThan42, it);
  }
}
```

In the [`<iterator>`](https://en.cppreference.com/w/cpp/iterator#Algorithm_concepts_and_utilities) header you'll not only find concepts related to callables but more generic ones as well. Such as whether two types are inderictly comparable. That sounds interesting, let's take a simple example:

```cpp
#include <iostream>
#include <iterator>
#include <string>
#include <vector>

template <typename Il, typename Ir, typename F>
void foo(Il leftIterator, Ir rightIterator, F function) requires std::indirectly_comparable<Il, Ir, F> {
    std::cout << std::boolalpha << function(*leftIterator, *rightIterator) << '\n';
}

int main()
{
  using namespace std::string_literals;
  
  auto binaryLambda = [](int i, int j){ return 42; };
  auto binaryLambda2 = [](int i, std::string j){return 666;};
  
  std::vector ints{15, 42, 66};
  std::vector floats{15.1, 42.3, 66.6};
  foo(ints.begin(), floats.begin(), binaryLambda);
//   foo(ints.begin(), floats.begin(), binaryLambda2); // error: use of function 'void foo(Il, Ir, F) requires  indirectly_comparable<Il, Ir, F, std::identity, std::identity> 
}
```

In this case, I've been left a bit puzzled by the [documentation](https://en.cppreference.com/w/cpp/iterator/indirectly_comparable):
- As a third template parameter it has `class R` which normally would refer to ranges. 
- But then according to its definition, it calls `std::indirect_binary_predicate` with `R` forwarded in the first position. 
- In [`std::indirect_binary_predicate`](https://en.cppreference.com/w/cpp/iterator/indirect_binary_predicate) at the first position, you accept a `class F` and F stands for a callable (often a function).

Why doesn't `R` called `F`? Why binary predicates are not mentioned in the textual descriptoin?

Probably only because this is still the beginning of the concepts journey. I'm actually going to submit a change request on this item.

#### Concepts in the `<ranges>` header

In the [`<iterator>`](https://en.cppreference.com/w/cpp/ranges#Range_concepts) header you'll find concepts describing requirements on different types of ranges.

Or simply that a parameter is a `range`. But you can assert for any kind of ranges, like `input_range`, `output_range`, `forward_range`, etc.

```cpp

#include <iostream>
#include <ranges>
#include <vector>
#include <typeinfo> 


template <typename R>
void foo(R range) requires std::ranges::input_range<R> {
  std::cout << typeid(range).name() << " is an input range\n";
}

int main()
{
  std::vector numbers{15, 43, 66};
  auto r = std::ranges::begin(numbers);
  foo(numbers);
}
```
### How to define our own concepts?

Let's define the simplest concepts first, just to see the syntax.

```cpp
template<typename T> 
concept Any = true;
```

First, we list the template parameters, here we have only one, `T`, but we could have multiple ones. Then after the keyword `concept` we declare the name of the concept and then after the `=` we define the concept.

In this example, we simply say `true`, meaning that for any type `T` the concept will be evaluated to `true`, any type is accepted. Should we wrote `false`, nothing would be accepted.

Now that we saw the simplest concept, let's check out of what components we can build a more detailed concept.

#### Use already defined concepts

Arguably the easiest way to define new concepts is by combining existing ones.

For example, in the next example we are going to create a concept `Number` by combining integers and floating point numbers.

```cpp
#include <concepts>

template<typename T> 
concept Number = std::integral<T> || std::floating_point<T>;
``` 

As you can see in the above example, we could easily combine with the `||` operator two concepts. But we could have used any other logical operator.

Probably it's self-evident, but we can use user-defined concepts as well.

```cpp
#include <concepts>

template<typename T> 
concept Integer = std::integral<T>;

template<typename T> 
concept Float = std::floating_point<T>;

template<typename T> 
concept Number = Integer<T> || Float<T>;
```

In this example, we just relabelled `std::integral` and `std::floating_point` to show that user-defined concepts can also be reused.

As we saw earlier, there are plenty of concepts defined in the different headers of the standard library so there is an endless way to combine them.

But how to define truly unique concepts?

#### Write your own requirements

##### Requirements on operations

You can simply express that you require two parameters, two types to support a certain operation or operator.

If you require that two parameters are addable you can create such concept:

```cpp
#include <iostream>
#include <concepts>

template <typename T>
concept Addable = requires (T a, T b) {
  a + b; 
};

auto add(Addable auto x, Addable auto y) {
  return x + y;
}

struct A {
  int b;
};

int main () {
  std::cout << add(4, 5) << '\n';
  std::cout << add(true, true) << '\n';
  // std::cout << add(A{4}, A{5}) << '\n'; // error: use of function 'auto add(auto:11, auto:12) [with auto:11 = A; auto:12 = A]' with unsatisfied constraints
}
/*
9
2 
*/
```
We can observe that when `add()` is called with parameters not supporting `operator+` the compilation fails with a rather descriptive error message (not the whole error message appears in the above example).

Writing requirement was easy. After the `requires` 
we basically wrote a wrote a function with first the parameters we want to set requirements against and that what kind of code they have to support.

##### Requirements on the interface (simple requirements)

Let's think about operations for a little bit more time. What does it mean after all to require the support of a `+` operation?

It means that you require types that have a function `T T::operator+(const T& other) const` function. Or it can even be `T T::operator+(const U& other) const`, as maybe you can add an instance of another type, but that's not the point. My point is that we made a requirement of having a function.

So we should be able to define a requirement on any function call, right?

Right, let's see how to do it.

```cpp
#include <iostream>
#include <string>
#include <concepts>

template <typename T> // 2
concept Number = requires (T t) {
    t.square();
};

class IntWithoutSquare {
public:
  IntWithoutSquare(int num) : m_num(num) {}
private:
  int m_num;
};

class IntWithSquare {
public:
  IntWithSquare(int num) : m_num(num) {}
  int square() {
    return m_num * m_num;
  }
private:
  int m_num;
};


void printSquare(Number auto number) { // 1
  std::cout << number.square() << '\n';
}


int main() {
  printSquare(IntWithoutSquare{4}); // error: use of function 'void printSquare(auto:11) [with auto:11 = IntWithoutSquare]' with unsatisfied constraints, 
                                    // the required expression 't.square()' is invalid
  printSquare(IntWithSquare{5});
}
```

In this example, we have a function `printSquare` (1) that requires a parameter satisfying the concept `Number` (2). In that concept we can see that it's really easy to define what interface we expect. In the requires clause we simply have to write down how we expect that the interface could be invoked.

If you expect that any passed in type have a function called `square`, you simply have to write `(T t) {t.square();}`

If you have requirements on the existance of multiple function calls, you just have to list all of them separated by a semicolon.

```cpp
template <typename T>
concept Number = requires (T t) {
    t.square();
    t.sqrt();
};
```

What about parameters? Let's define a `power` function that takes an `int` parameter for the exponent:

```cpp
template <typename T>
concept Number = requires (T t, int i) {
    t.power(i);
};

// ...

void printSquare(Number auto number) {
  std::cout << number.power(3) << '\n';
}
```

As such, we fix that the parameter will be something that is (convertible to) an `int`.

But what if we wanted to accept just any integral number as an exponent. Where is a will, there is a way and it's not different in this context either.

First, concept template should take two parameters. One for the base type `Number` and one for the exponent.

```cpp
template <typename T, typename U>
concept Number = std::integral<U> && requires (T t, U i) { 
    t.power(i);
};
```

Here we only make sure that a parameter of type `U` can be passed to `T::power`. 

The next step is to update our `printSquare` function. The concept `Number` has changed, now it takes two types, we have to take that into account:

```cpp
template<typename Exponent>
void printPower(Number<Exponent> auto number, Exponent exp) {
  std::cout << number.power(exp) << '\n';
}
```

And `printPower` can be called like this:

```cpp
printPower<int>(IntWithSquare{5}, 3);
printPower<long>(IntWithSquare{5}, 4L);
```

At the same time, the call `printPower<float>(IntWithSquare{5}, 3.0);` will fail because the type `float` is not integral.

Do we like what have now?

We have to pass type information to `printPower`, plus we can't even call it `IntWithSquare`.

Let's attack them one by one. First we should remove the need of explicit type information and concepts can help us. We have to put a requriement on `std::integral`.

```cpp
template<typename Exponent> 
requires std::integral<Exponent>
void printPower(Number<Exponent> auto number, Exponent exp) {
  std::cout << number.power(exp) << '\n';
}
```

Now we can call `printPower` with any `integral` type without calling passing the type explicitely: `printPower(IntWithSquare{5}, 3);`.

The other problem is bit more difficult. We want to be able to call `T::power()` with a custom type, like `IntWithSquare` and for that we need two things:

- `IntWithSquare` should be considered an `integral` type
- `IntWithSquare` should be convertible to something accepted by `pow` from `cmath`.

Let's go one by one.

By explicitely specifying the `type_trait` `std::is_integral` for `IntWithSquare`, we can make `IntWithSquare` an integral type. Of course, if we plan to do so in real life, it's better to make sure that our type has all the characteristics of an integral type, but that's beyond our scope here.

```cpp
template<>
struct std::is_integral<IntWithSquare> : public std::integral_constant<bool, true> {};
```

Now we have to make sure that `IntWithSquare` is convertible into a type that is accepted by [`pow`](http://www.cplusplus.com/reference/cmath/pow/). It accepts floating point types, but when it comes to `IntWithSquare`, in my opinion it's more meaningful to convert it to an `int` and let the compiler perform an implicit conversion to `float`. After all, `IntWithSquare` might be used in other contexts as well. 

For that we have to define `operator int`


```cpp
class IntWithSquare {
public:
  IntWithSquare(int num) : m_num(num) {}
  int power(IntWithSquare exp) {
    return pow(m_num, exp);
  }
  operator int() const {return m_num;}
private:
  int m_num;
}
```

If we check our example now, we'll see that both `printPower(IntWithSquare{5}, IntWithSquare{4});` and `printPower(IntWithSquare{5}, 4L);` will compile, but `printPower(IntWithSquare{5}, 3.0);` fails because `3.0` is not integral.

Right, we forgot that in standard library `pow` operates on floating-point numbers but we only accept integrals. Let's update our requirement on exponent and on Number:

```cpp
template <typename T, typename U>
concept Number = (std::integral<U> || std::floating_point<U>) && requires (T t, U i) { 
    t.power(i);
};

template<typename Exponent> 
requires (std::integral<Exponent> || std::floating_point<Exponent>)
void printPower(Number<Exponent> auto number, Exponent exp) {
  std::cout << number.power(exp) << '\n';
}
```

Now we can call `printPower` with any kind of type that satisfies the number concept and both with integral and floating-point numbers as an exponent.

Let's have a look at the full example now:

```cpp
#include <cmath>
#include <iostream>
#include <string>
#include <concepts>
#include <type_traits>

template <typename T, typename U>
concept Number = (std::integral<U> || std::floating_point<U>) && requires (T t, U i) { 
    t.power(i);
};

class IntWithSquare {
public:
  IntWithSquare(int num) : m_num(num) {}
  int power(IntWithSquare exp) {
    return pow(m_num, exp);
  }
  int get() const {return m_num;}
  operator int() const {return m_num;}
private:
  int m_num;
};

template<>
struct std::is_integral<IntWithSquare> : public std::integral_constant<bool, true> {};

template<typename Exponent> 
requires (std::integral<Exponent> || std::floating_point<Exponent>)
void printPower(Number<Exponent> auto number, Exponent exp) {
  std::cout << number.power(exp) << '\n';
}


int main() {
  printPower(IntWithSquare{5}, IntWithSquare{4});
  printPower(IntWithSquare{5}, 4L);
  printPower(IntWithSquare{5}, 3.0);
}
```
### Requirements on return types (also called compound rquirements)

We've seen how to write a requirement stating that a type needs to have a certain API, needs to have a certain function.

But so far, it's been quite vague, meaning that the return type doesn't matter.

`IntWithSquare` satisfy the `Number` concept both with `int square()` and `void square()`.

If you want to specify the return type, you must use something that is often referred to as a compound requirement.

Here is an example:

```cpp
template <typename T>
concept Number = requires (T t) {
    {t.square()} -> std::convertible_to<int>;
}; 
```

Notice the following:
- The expression on what you want to set a return type requirement must be surrounded by braces (`{}`), then after the arrow (`->`), you can define your constraint
- A constraint cannot simply be a type. Had you written `int`, you would have received an error message indicating that error: return-type-requirement is not a type-constraint. The original concepts TS allowed this, so if you experimented with that, you might be surprised by this error. This possibility was removed by [P1452R2](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1452r2.html).

There were a number of reasons this was removed, one of the motivations being what it would interfere with a future direction of wanting to adopt a generalized form of `auto`, like `vector<auto>` or `vector<Concept>.`

So instead of simply naming a type you have to choose among the following two options:

```cpp
{t.square()} -> std::same_as<int>;
{t.square()} -> std::convertible_to<int>;
```

I think that the difference is obvious. In case of `same_as`, the return value must be the same as specified as the template argument, while for `convertible_to` conversions are allowed.

In order to demonstrate all these, please have a look at the following example:

```cpp
#include <iostream>
#include <string>
#include <concepts>

template <typename T>
concept NumberSame = requires (T t) {
    {t.square()} -> std::same_as<int>;
};

template <typename T>
concept NumberConvertible = requires (T t) {
    {t.square()} -> std::convertible_to<int>;
};

class IntWithIntSquare {
public:
  IntWithIntSquare(int num) : m_num(num) {}
  int square() const {
    return m_num * m_num;
  }
private:
  int m_num;
};

class IntWithLongSquare {
public:
  IntWithLongSquare(int num) : m_num(num) {}
  long square() const {
    return m_num * m_num;
  }
private:
  int m_num;
};

class IntWithVoidSquare {
public:
  IntWithVoidSquare(int num) : m_num(num) {}
  void square() const {
    std::cout << m_num * m_num << '\n';
  }
private:
  int m_num;
};


void printSquareSame(NumberSame auto number) {
  std::cout << number.square() << '\n';
}

void printSquareConvertible(NumberConvertible auto number) {
  std::cout << number.square() << '\n';
}


int main() {
  printSquareSame(IntWithIntSquare{1}); // int same as int
//   printSquareSame(IntWithLongSquare{2}); // long not same as int
//   printSquareSame(IntWithVoidSquare{3}); // void not same as int
  printSquareConvertible(IntWithIntSquare{4}); // int convertible to int
  printSquareConvertible(IntWithLongSquare{5}); // int convertible to int
//   printSquareConvertible(IntWithVoidSquare{6}); // void not convertible to int
}
```

### Type requirements

With type requirements we can express that a certain type is valid in a specific context. IT can be usef to verify that
- a certain neted type exists
- a class template specializiation names a type
- an alias template specialization names a type

You have to use `typename` along with the type name:

```cpp
//maybe something Cars
template<typename T>
concept TypeRequirement = requires {
typename T::value_type;
typename Other<T>;
};
```
The concept TypeRequirement requires that the type T has a nested member value_type, and that the class template Other can be instantiated with T.

Let's see how it works:


```cpp
#include <iostream>
#include <vector>
template <typename>
struct Other;
template <>
struct Other<std::vector<int>> {};
template<typename T>
concept TypeRequirement = requires {
typename T::value_type;
typename Other<T>;
};
int main() {
TypeRequirement auto myVec= std::vector<int>{1, 2, 3};
}
// put example when Other cannot be used
```

The expression TypeRequirement auto myVec= std::vector<int>{1, 2, 3} (line 18) is valid.
A std::vector20 has an inner member value_type (line 14) and the class template Other can be
instantiated with std::vector<int> (line 13).



Mixing these together - compound
nested -< anonymous
https://en.cppreference.com/w/cpp/language/constraints



#include <cmath>
#include <iostream>
#include` <string>
#include <concepts>

template <typename T, typename U=T>
concept Number = std::integral<U> && requires (T t, U i) { 
    t.power(i);
};

class IntWithoutSquare {
public:
  IntWithoutSquare(int num) : m_num(num) {}
private:
  int m_num;
};

class IntWithSquare {
public:
  IntWithSquare(int num) : m_num(num) {}
  int power(int exp) {
    return pow(m_num, exp);
  }
private:
  int m_num;
};


template<typename U=int>
void printSquare(Number<U> auto number) {
  std::cout << number.power(3) << '\n';
}


int main() {
  //printSquare(IntWithoutSquare{4}); // error: use of function 'void printSquare(auto:11) [with auto:11 = IntWithoutSquare]' with unsatisfied constraints, 
                                    // the required expression 't.square()' is invalid
  printSquare(IntWithSquare{5});
}


But for the moment we didn't say anything about what kind of output it should produce. Hence adding up two `bool`s work well. They were casted to integers and handled as such. Probably it was not our intention.

Had we defined `add()` in a different way, the result would have been even casted back to a boolean:

```cpp
// ...

template <typename T>
T add2(T x, T y) requires Addable<T> {
  return x + y;
}
// ...
std::cout << add2(true, true) << '\n';
/*
1
*/

```

return type!

#include <iostream>
#include <concepts>

template <typename T>
concept Addable = requires (T a, T b) {
  {a + b}; //-> std::same_as<T>; 
};

template <typename T>
T add2(T x, T y) requires Addable<T> {
  return x + y;
}

auto add(Addable auto x, Addable auto y) {
  return x + y;
}

struct A {
  int b;
};

int main () {
  std::cout << add(4, 5) << '\n';
  std::cout << add2(true, true) << '\n'; // error: use of function 'auto add(auto:11, auto:12) [with auto:11 = bool; auto:12 = bool]' with unsatisfied constraints
//   std::cout << add(A{4}, A{5}) << '\n'; // error: use of function 'auto add(auto:11, auto:12) [with auto:11 = A; auto:12 = A]' with unsatisfied constraints
}










- requirements on the "relationship of types"
- requirements on what kind functions they defined
- requires in concept
- no type just num

- what are the building blocks?
  - old ones?
  - interfaces?
- how to create a simple?
- can we use multiple params?
- let's define a more complex ones
Even if they involve multiple parameters!





I'll write a more detailed post about the 4 ways of concepts.

_Please note that the concept `Number` is incomplete, it accepts `char` for example _



motivation for concepts

4 ways to use concepts

ceoncepts vs templates

predefined concepts

define some concepts

what other concepts

WTF to do with template <typename T, typename Ts ...>