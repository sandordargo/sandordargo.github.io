---
layout: post
title: "C++23: The `<expected>` header; expect the unexpected"
date: 2022-5-4
category: dev
tags: [cpp, cpp23, errors, expected]
excerpt_separator: <!--more-->
---
What do you do when you have to return multiple values from a function? Do you return an instance of some data structure? Do you use output variables? Maybe you throw an exception to get rid of the error codes?

It's not an obvious choice.

C++23 offers a standardized solution to deal with status codes and expected return values at the same time in a sophisticated manner with the help of the `<expected>` library.

## How we used to do it?

Handling nominal return values and status codes is a problem we have had several solutions for, though none of them is perfect.

### Using output parameters

Probably the most ancient and most used way of returning both a status code and a nominal result (i.e. an expected value) is via "out" variables. Let's say you have a function that returns an error code - which is often an integer - and the nominal return value via an out variable. It could be the other way around, but more often it's like this. 

Why and why not the other way around?

I see two reasons for that:

- the error code is easier to ignore if it resides in an out variable. When it's a direct return value, you'll check it with a higher chance
- usually the return values are bigger in terms of size. By passing the memory location for it by reference, you are guaranteed that it'll be allocated only once. No (N)RVO is needed.

Here is an example:

```cpp

int fetchCitiesFromDb(const std::string& query, std::vector<std::string>& results) {
    // ...
};

// ...

std::vector<std::string> v;
int errorCode = fetchCitiesFromDb(aQuery, v);
```

The drawback is that such APIs are not so easy to understand. By default, one would expect to find the output at the return value and would interpret parameters as inputs, but it's not the case with this solution. It's simply not straightforward.

### Using exceptions

Another option for error handling is the obvious one! Use exceptions. Well, if you can. They are not allowed in all the different environments, some teams ban their usage.

It's a controversial topic in the sense that people can be very opinionated about exceptions, many books and conference talks could be and were filled with exceptions.

Some argue that they are costly. As always, it depends. If you spend most of your time reading from a database or sending data over the network, then it's not so important.

On the other hand, what is always true is that it goes completely around the normal control flow and it's very difficult to reason about code using exceptions. Often you don't see, or it's very difficult to see starting from where you end up where. Many misuse exceptions, log errors several times and keep rethrowing exceptions.

I think the only way it could work well is if you have an exception-handling strategy that is clearly stated as part of the documentation. Even then there is a fair chance that it won't be respected. Especially in a big corporation where people always come and go.

### Using `std::variant` or `std::optional`

A modern answer to this burning question is to use one of the data structures introduced by C++17, `variant` or `optional`. If you pick `std::optional`, then the presence of a value would mean that everything went fine, as expected. If the value is missing, or in other words, if the return value is `std::nullopt`, you have to deal with the error case.

This approach has (at least) one shortcoming, there is no way to differentiate between different error cases.

A status code can have many values, you might throw exceptions of many different types and/or with different messages, but you either have a value present or not with `std::optional`.

That's why for cases that would require different error codes, `std::variant` is often picked. A `std::variant` is often referred to as a type-safe union. In reality, it's a class template taking different types and it will always hold exactly one of those. That's why it's mandatory for the first type to be default constructible (or use the [`std::monostate`](https://en.cppreference.com/w/cpp/utility/variant/monostate) as a placeholder). When you instantiate a variant without specifying its value, it'll be the default constructed version of the first type.

But let's get back to error handling. How does a `variant` help?

You can say that you want your `variant` to hold either the nominal return type or a status code.

That's a neat and elegant solution.

The problem here is the additional syntax you need to access either one or the other value. It's not made for error handling, so the names are not so descriptive, and you might find the overhead a bit too much.

On the other hand, the syntax for `std::optional` is quite straightforward and easy to read.

Wouldn't it be great to combine the capabilities of a `variant` with the API of an `optional`?

## What does `std::expected` offer?

`std::expected` offers exactly that! The freedom of storing either the expected return value or an error code and you get a very neat `optional`-like API to access the different values.

To get a bit more into the details, `std::expected` is a template class taking two parameters: `std::expected<T, E>`. The first one, `T`, represent the nominal return type (the expected type), and `E` is the type to return in case of an error. Meaning that it cannot only be an error code, it can be any type that you think would serve well to represent an error.

Just like `std::optional`, it offers the following API:

- `has_value()`
- `operator bool()`
- `operator*()`
- `operator->()`
- `value()`
- `value_or()`

In addition, it offers another API function:
- `error()`

Now let's see what each does.

`has_value()` and `operator bool()` are the same. They both return `true` if the instance of `std::expected<T, E>` contains a value of type `T`, `false` otherwise.

`operator*()` and `operator->()` are to access the nominal return type either via value or pointer semantics. But beware, should you use any of these operators while the `std::expected` object doesn't contain the expected value, the behaviour is undefined.

On the other hand, if you try to get a reference to the expected value via the `value()` function and there is nothing to return, the behaviour is well-defined, and you'll get a `std::bad_expected_access` exception thrown. If you want to have a default value, in case there is none in the object, use `value_or()` and you can specify the default as a parameter.

Last but not least, the `error()` member function returns the unexpected, the error value.

## How to use it? Is it already supported?

The `<expected>` header is already supported both by *gcc* and *msvc*. You can always check the level of compiler support on [C++ Reference](https://en.cppreference.com/w/cpp/compiler_support).

Now let's see `std::expected` in action!

Let's continue working with `fetchCitiesFromDb` which uses our favourite new wrapper, `std::expected`, as a return type.

```cpp
enum class ErrorCode { InvalidConnectionString, InvalidQuery };

std::expected<std::vector<std::string>, ErrorCode> fetchCitiesFromDb() {
    // ... ?
}
```

In order to return the expected value, we don't have to wrap it with anything. We can simply return it as we'd do without the `expected` wrapper.

```cpp
enum class ErrorCode { InvalidConnectionString, InvalidQuery };

std::expected<std::vector<std::string>, ErrorCode> fetchCitiesFromDb() {
    return std::vector<std::string>{"Budapest", "Nice", "Stockholm"};
}
```

Now let's introduce an input parameter which we'll use in a guard close to provoke an error. In case that parameter is empty let's return an error code by using the `std::unexpected` constructor.

```cpp
enum class ErrorCode { InvalidConnectionString, InvalidQuery };

std::expected<std::vector<std::string>, ErrorCode> fetchCitiesFromDb(std::string connectionString) {
    if (connectionString.empty()) {
        return std::unexpected<ErrorCode> { ErrorCode::InvalidConnectionString };
    }
    return std::vector<std::string>{"Budapest", "Nice", "Stockholm"};
}
```

Thanks to [CTAD](https://en.cppreference.com/w/cpp/language/class_template_argument_deduction), we could also return simply `std::unexpected{ErrorCode::InvalidConnectionString}`.

And now let's introduce another parameter and another guard close so that we can see another way to return an unexpected value.

```cpp
#include <iostream>
#include <vector>
#include <expected>
#include <string>

enum class ErrorCode { InvalidConnectionString, InvalidQuery };

std::expected<std::vector<std::string>, ErrorCode> fetchCitiesFromDb(std::string connectionString, std::string query) {
    if (connectionString.empty()) {
        return std::unexpected { ErrorCode::InvalidConnectionString }; 
        // return std::expected<std::vector<std::string>, ErrorCode> (std::unexpected(ErrorCode::InvalidConnectionString) ); would also work if we wanted to be more verbose
    }
    if (query.empty()) {
        return std::expected<std::vector<std::string>, ErrorCode> (std::unexpect_t{}, ErrorCode::InvalidQuery );
    }
    return std::vector<std::string>{"Budapest", "Nice", "Stockholm"};
}

int main () {
  fetchCitiesFromDb("", "");
}
```

In the above example, we saw how to return either an expected or an unexpected value. Now let's see how we can use them.

```cpp
#include <iostream>
#include <vector>
#include <expected>
#include <string>
#include <iomanip>

enum class ErrorCode { InvalidConnectionString, InvalidQuery };

std::ostream& operator<<(std::ostream& os, ErrorCode ec) {
    switch (ec) {
        case ErrorCode::InvalidConnectionString: 
            os << "ErrorCode::InvalidConnectionString";
            break;
        case ErrorCode::InvalidQuery: 
            os << "ErrorCode::InvalidQuery";
            break;
    }
    return os;
}

std::ostream& operator<<(std::ostream& os, const std::vector<std::string>& v) {
    os << '[';
    for (const auto& s : v) {
        if (&s != &v[0]) {
            os << ", ";
        }
        os << std::quoted(s);
    }
    os << ']';
    return os;
}

std::expected<std::vector<std::string>, ErrorCode> fetchCitiesFromDb(std::string connectionString, std::string query) {
    if (connectionString.empty()) {
        return std::unexpected { ErrorCode::InvalidConnectionString };
    }
    if (query.empty()) {
        return std::expected<std::vector<std::string>, ErrorCode> (std::unexpect_t{}, ErrorCode::InvalidQuery );
    }
    
    return std::vector<std::string>{"Budapest", "Nice", "Stockholm"};
}

int main () {
    auto r = fetchCitiesFromDb("", "");
    if (r) { // (1)
        std::cout << "fetchCitiesFromDb(\"\", \"\") returned " << r.value().size() << " cities.\n";
    } else {
        std::cout << "fetchCitiesFromDb(\"\", \"\") returned an error! " << r.error() << '\n';
    }

    auto r2 = fetchCitiesFromDb("demo", "");
    std::cout << "fetchCitiesFromDb(\"demo\", \"\") gave us this vector: " << r2.value_or(std::vector<std::string>{}) << '\n';  // (2)
    //std::cout << "fetchCitiesFromDb(\"demo\", \"\"); returned" << r2.value_or(r.error()) << '\n'; 

    auto r3 = fetchCitiesFromDb("demo", "effects");
    if (r3.has_value()) { // (3)
        std::cout << "fetchCitiesFromDb(\"demo\", \"effects\") returned " << *r3 << " which has a size of: " << r3->size() << '\n'; // (4)
    } else {
        std::cout << "fetchCitiesFromDb(\"demo\", \"effects\") returned an error! " << r3.error() << '\n';
    }
}
```

First, we can see at *(1)* that we used `operator bool()` and if the returned value is the expected, then it evaluates to `true`, otherwise to `false`. If `operator bool() == false`, then `error()` should return a valid unexpected value.

At line *(2)* we see that if we don't want to query the error, but we want to use a default value if there is no expected value available, then we can simply call `value_or(<default value>)`. It's worth noting that the default value, the one that we pass with `value_or()`, must be of the same type as `value()` would return. In other words, it must be an instance of the expected type.

At lines *(3)* and *(4)* we see if `has_value()` returns `true` then we can safely use the `operator->()` in order to access a member of the expected value.

## How it fits the alternatives? When to use exceptions?

After having seen all that, it's to ask ourselves how this goes with the alternatives. What way of error handling should we choose?

In my opinion, this new utility type, `std::expected` is clearly a superior way to all non-exception ways of error-handling.

I never liked having a return type and out parameters. Such APIs are not simple enough. If there should be multiple expected return values, create a struct, a class whichever fits better your needs. If one represents the success of the operation, you might have wanted to go with `optional`, but if you have to deal with multiple error codes you might have wanted to go with a variant.

Now all this - when error codes are involved - should be replaced with the `<expected>` header.

When it comes to exceptions, the situation is a bit more tricky.

First of all, there are environments, teams even companies where the usage of exceptions is banned. That's an easy question then, `<expected>` will serve you well. Otherwise, exceptions are not easy to use well. They go completely across the normal flow of control, often logging is either neglected or it's done multiple times. In my opinion, how exceptions are handled should be documented in a project and they should follow a clear strategy. That strategy should strike out the documentation, so both newcomers on the project and code reviewers can easily see it.

Using exceptions is a decision and `std::expected` does not provide the same functionality.

On the other hand, it's also worth remembering the idea behind exceptions. When should they be used? In exceptional cases! An error that you expect, is not exceptional. It's unexpected but probable. Accessing a vector out of bounds, running out of memory and therefore having an allocation issue, and accessing a value of an optional when it doesn't contain any are exceptional cases that should not happen. I think using exceptions is allowed and meaningful in such situations.

On the other hand, in situations which are likely to happen in unexceptional circumstances (like a user is passing a value out of the expected range), when you hesitate between using exceptions are error codes in any way, I think going with the `<expected>` header is the right call.

## Conclusion

Writing functions that are supposed to return both a normal return value and a status code at the same has been painful for decades. You either had to create your custom data structure or besides return values, you had to use our parameters too. Later C++17 introduced both `std::optional` and ` std::variant` offering some alternatives, but they were not designed to deal with error handling.

C++23 will bring as `std::exceptional` that is a standardized data structure with a `std::optional`-like API aiming to hold an expected value or an error code. It's a readable, easy-to-use solution that will increase the readability of our APIs!

Sounds very promising!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!