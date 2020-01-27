---
layout: post
title: "Railway oriented programming with C++"
date: 2019-X-X
category: dev
tags: [clean code, functional programming, craftsmanship, showmethecode]
excerpt_separator: <!--more-->
---
In 2017, I already wrote about [Railway Oriented Programming](http://sandordargo.com/blog/2017/09/27/railway_oriented_programming). You can read the details in that article, but let me give you a recap on it.

In our code we mostly focus on the happy path in the beginning. Then comes the necessarily error handling and we mess up our nice flow with `if`s and `try-catch` blocks everywhere.

Railway Oriented Programming offers a way to keep the logic clean and push down the error handling to the building blocks of our logic.

And while railways? Because this way we have two pair of rails, a happy and an unhappy and at each building block we can either keep our track or we can change to the other one.

## The scope

What I tried this time was not recreating `map`, `flatmap`, `orElse` and the rest. Instead I just wanted a universal return type, that I can use for both returning normal results of functions and well detailed errors.

I ended up with a relatively simple structure with the three underlying implementation. Let me walk you through all of them.

## The example I'm going to use

I've taken the example taken from [ProAndroidDev's article](https://proandroiddev.com/railway-oriented-programming-in-kotlin-f1bceed399e5#dfe8) on Railway Oriented Programming. A very simplified e-mail sending flow is going to serve us in this article as example.

Here is pseudocode of the _happy path_:

```cpp
Input input{};
Parser parser{};
auto parsed = parser.parse(input.read());
Email email{parsed};
bool res = send(email);
```
And now where is a possible _unhappy path_:

```cpp
Input input{};
Parser parser{};
try {
  if (input.isvalid()) {
    auto parsed = parser.parse(input.read());
  }
}
catch(...) {
  // some error handling
}

bool result = false;
if (parsed.isvalid()) {
  Email email{parsed};
  try {
    result = send(email);
  }
  catch() {
    // some more error handling
  }
}
return result
```

----------------
# Restate our goal

So our goal is to keep the main logic, our flow readable.
What is interesting is that read the input, parse, create an email and then send it. It's more interesting than how its obfuscated with error handling.


#With pointers, c++11 compatible
The first solution doesn't need the newest versions of C++, it works well with C++11. But in fact, if we put spaces in between > signs in nested template instantiations and if we get rid off the smart pointers, it can work with even older versions of C++.

The idea is that will accompany us through the other versions to create a tamplate class Maybe<ReturnType>. It either holds an instance of return type of an error. It should never hold both and it's achieved by not having such a constructor and by keeping the class immutable.

The reason why I decided to use (smart) pointers instead of simple variables on the stack is to be able to pass derived classes.

Then our entities through our logic should accept as parameters and return as return values Maybe<> values. As such, we don't have to error handling in our main function, we can push this down to each function on the way.

# With optional

What we can observe in the first iteration is that the smart pointers either have a value or not. We have these two options. Indeed, these are optional.

So why don't we use optional?

Optional is only available since C++17. I said why not giving it a try!

The API is the same and the implementation is not really different. After all, is it really better?

At least we don't store nullptrs, but std::nullopts.

If I didn't want to support polymorphism, then we could simplify a lot using simply optionals. But I didn't want to do that.

So we have two optionals and each of them might contain a smart pointer or not. But logically only one of them can contain a non-null value.

Is there anything else we could use?

# With variant

Since C++17, yes we can. We don't only have std::optional, but also std::variant in our hands. It's really similar to boost::variant that we already had available, but now it's part of the langugage standard which is another level.

So what is std::variant after all?

You can define a type that can contain an instance of any of the defined elements, but only one at a time. It might contain an integer or a string for example if you declare it like this:

```cpp
#include <string>
#include <variant>
...
std::variant<std::string, int> intOrString;
```

As in our case we always store the value of either an exception or the templated type, this is ideal for us.

The API rest the same, but the implementation is a bit simpler and why before it was only logic and lack of corresponding constructor that blocked us from having two values at the same type, now it is guranteed.

# What I like

The happy path

# What I don't like

That I have to pass the template parameter for 






so happy

```
#include <iostream>
#include <string>

auto read() -> std::string {
    return "an email";
}

auto parse(const std::string& input) -> std::string {
    return "a valid email";
}

auto send(const std::string& email) -> void {
    std::cout << "email sent: " << email << std::endl;
}

int main () {
  auto input = read();
  auto email = parse(input);
  send(email);
  return 0;
}
```


```
https://proandroiddev.com/railway-oriented-programming-in-kotlin-f1bceed399e5#dfe8
#include <iostream>
#include <vector>
#include <string>

class Email {
    Email(const std::string& aRecepient, const std::string& aSubject, const std::string& aBody) :
        _subject(aSubject...

}

auto read() -> std::vector<std::string> {
    return std::vector<std::string>{"an recepient", "a subject", "a body"};
}

auto parse(const std::vector<std::string>& input) -> Email {
    return "a valid email";
}

auto send(const std::string& email) -> void {
    std::cout << "email sent: " << email << std::endl;
}

int main () {
  auto input = read();
  auto email = parse(input);
  send(email);
  return 0;
}````