---
layout: post
title: "Mocking virtual functions with gMock"
date: 2022-3-2  
category: dev
tags: [cpp, virtual, mocking, gmock]
excerpt_separator: <!--more-->
---
In this mini-series we are going to discover mocking with *gMock*, the probably most widely used C++ mocking framework.

I think that practical discussions should start with theoretical ones. In order to understand something from a practical point of view, we should understand the theoretical background.
<!--more-->

This is important because we will not simply try to mimic examples, but we will try to do things that make sense even from a birds-eye view.

## What are mocks and how do we get them wrong?

It seems evident that we want to speak about mocks when we want to learn about *gMock*. First, we should understand what mocks are and what are the competing concepts.

**Mocks** are objects that 
- are needed in a system under test and
- that are implementing the same interface as the original objects. 

Mocks can be used to observe and verify behaviour when we cannot verify something on the class under test and it has side effects; such as invoking methods on our mocks.

In other words, mocks are objects with pre-defined expectations on what kind of calls they should receive.

As we are going to see, mocks in *gMock* do fulfil this idea, but they do more. They also act as **stubs**. Stubs can be configured to respond to calls from the system under tests with the values or exceptions predefined.

Stubs are come in handy when you have to tests objects depending on external calls (such as calls to networks, databases, etc.). Stubs might not only be able to send these canned answers but they can also have a memory so that they "remember" what they sent. Such stubs might be referenced as spies. You might even define that the first 3 answers should be different from what is coming later.

We also have to make the distinctions of the **fakes** that have a working but very lightweight implementation. They might return hardcoded data unconditionally; always valid or always invalid data.

## What is *gMock*?

Let's leave behind the theory now and talk about the *gMock* framework. *gMock* is one of the most widely used frameworks in C++. *gMock* comes in handy, when we cannot simply fake all the parameters and calls. It is useful when we need some mocks to be able to write better tests or to be able to write tests at all.

Though *gMock* has its own set of assertions, it's often used only for mocking and for the assertions *gTest* is used. I even saw *gMock* being combined with non-Google unit testing frameworks.

*gMock* promises a declarative, easy to learn and easy to use syntax for defining mocks, though in my experience people don't necessarily share this opinion.

*gMock* used to live in his own on Github project, but a couple of years ago it was merged into the [*gTest* framework](https://github.com/google/googletest/tree/main/googlemock). There were also a couple of syntactical changes in v1.10. Unless I say so, in this series, you can assume that I'm using the syntax of the newer versions.

As the [*gMock* for dummies](https://github.com/google/googletest/blob/main/docs/gmock_for_dummies.md) mentions, there is a 3-step process to follow when you want to introduce a mock to your tests:

- describe the interface to be mocked
- create the mocks including all the expectations and behaviours
- exercise the code that uses the mock objects

Let's go through the three steps. My goal in these articles is not to cover each and every possibility, but to explain the main ones and provide you with the sources to find the details.

## Describe the interface to be mocked

In order to describe an interface, we have to use macros. While in general, it's good to avoid macros in your code, here you don't have any others options.

Taste the expression *"mocking an interface"*. While in C++ there is no strong equivalent to Java's `interface` keyword and object-type, the closest thing is an abstract class with pure virtual functions.

```cpp
class Car {
public:
  virtual ~Car() = default;
  virtual void startEngine() = 0;
  virtual int getTrunkSize() const = 0;
  virtual void addFuel(double quantity) = 0;
};
```

The second closest thing is a class with some virtual functions in it:

```cpp
class GPS {
public:
  virtual ~GPS() = default;
  virtual void addDestination(const std::string& destination) {}
  virtual Route getProposedRoute(int routeType) {}
};
```

I wrote mocking an interface on purpose. It's much easier to mock a virtual function than a non-virtual one. (*In this article I define interfaces using run-time polymorphism.*)

Let's start first with the *virtual*s.

## Mock a *virtual* function

Mocking a *virtual* function is easy in most cases, but there are a couple of things to pay attention to.

Let's start with mocking all the functions of the previously introduced `Car` class.

```cpp
class MockCar : public Car {
public:
  MOCK_METHOD(void, startEngine, (), (override));
  MOCK_METHOD(int, getTrunkSize, (), (const, override));
  MOCK_METHOD(void, addFuel, (double quantity), (override));
};
```

Let's break this down.

First, we create a class that inherits from the class we want to mock and prepend its name with `Mock` (the naming is just a convention).

Then in the public section, we start mocking the methods whose behaviour we want to change or monitor.

In earlier versions of *gMock*, there were a set of macros where the macro name included the number of function parameters and also the constness of the function, but since version 1.10.0, we can simply use the macro `MOCK_METHOD`.

Let's take the first example:

```cpp
MOCK_METHOD(void, startEngine, (), (override));
```

`MOCK_METHOD` takes the following parameters:
- In the first position, we pass in the return type of the function, in this case, `void`. 
- The second parameter is the name of the function we want to mock. 
- The third parameter is the list of parameters the function takes. They should be listed surrounded by parentheses, which seems natural. You can basically copy-paste the parameter list from the function signature - just remove the parameter names.
- The fourth and last parameter is a list (again surrounded by parentheses) of the qualifiers the function has. Ideally, all should be `override` as a mock function should mock the base class function. In addition, it takes the cv-qualifiers from the base class. Let's demonstrate it:

```cpp
MOCK_METHOD(int, getTrunkSize, (), (const, override));
```

But what does this macro do? Are we good yet?

No, we are not done yet. We should still provide a behaviour for the mocked methods. It doesn't matter whether a mocked function is defined in the base class or if it's abstract, `MOCK_METHOD` will provide an empty behaviour. The mocked function will do nothing and if the return type is not `void`, it will return the default constructed value.

If the return type has no default constructor and you don't provide a default action, *gMock* is going to throw an exception in the test body:

> *The mock function has no default action set, and its return type has no default value set."*

But how do we provide the default action?

## Stubs with *gMock*

As we discussed earlier, with *gMock*, we can create objects that are not only mocks, but stubs as well. And in fact, the way it's designed, stubs come first; a mocked function doesn't have a default behaviour, that's something we have to provide.

### Describe, but don't assert

We can use the `ON_CALL` macro to provide behaviour.

For the `ON_CALL` macro, we have to pass in at the first place an instance on which the behaviour has to be defined and at the second place, we have to pass in the function name and all the expected parameters.

But how do we pass in the parameter list? We don't pass the types, but the exact values!

Let's take `ON_CALL(c, addFuel(5.0))` as an example. This means that `addFuel` must be called with the value of `5.0` (implicit conversions are accepted), otherwise, the expectation will not be met.

If you don't know with what value `addFuel` should be called or if you don't care, you can use matchers!

Wildcards are often used, such as `_`: `ON_CALL(c, addFuel(::testing::_))`, but we can also express some more precise comparisons such as requiring that a parameter should be greater than a given value: `ON_CALL(c, addFuel(::testing::Gt(5)))`.

You can find more information on these pre-defined matchers [here](https://github.com/google/googletest/blob/master/docs/reference/matchers.md#wildcard).

After we set which function we provide with a behaviour, we have to set that action. We can do it with `WillByDefault()`. 

`WillByDefault()` can take many different parameters depending on what you want to achieve:

- To return a value, you can use `::testing::Return(value)`, e.g. `ON_CALL(c, getTrunkSize()).WillByDefault(::testing::Return(420))`
- To return a reference, you can use `::testing::ReturnRef(variable)`
- `Return` sets the value to be returned when you create the action, if you want to set the value when the action is executed, you can use `::testing::ReturnPointee(&vairable)`.

With `ON_CALL`, you have no other options to set the default behaviour than `WillByDefault()`. At the same time, you can use it after specifying different input parameters. This is completely valid:

```cpp
ON_CALL(o, foo(1)).WillByDefault(::testing::Return(42))
ON_CALL(o, foo(2)).WillByDefault(::testing::Return(66))
```

### Describe and assert

`ON_CALL` only describes what a method should do when it's called, but it doesn't make sure that it gets called. If we need more than that, if need to assert that a method gets called, maybe even with a given set of parameters, we need to use another macro, `EXPECT_CALL`.

Just like `ON_CALL`, an `EXPECT_CALL` expression can grow long, but I think in most cases it remains simple. Let's start with what it takes as parameters.

`EXPECT_CALL(c, getTrunkSize())` takes first the mocked object that it should watch and as a second one the method name, including its parameter list.

The parameters are passed the same way for `EXPECT_CALL` and `ON_CALL`.

`EXPECT_CALL(c, addFuel(5.0))` means that `addFuel` must be called with the value of `5.0` (implicit conversions are still accepted), otherwise, the expectation will not be met.

Matchers can be used to widen the range of accepted values.

Wildcards are often used, such as `_`: `EXPECT_CALL(c, addFuel(::testing::_))`, but we can also express some more precise comparisons such as requiring that a parameter should be greater than a given value: `EXPECT_CALL(c, addFuel(::testing::Gt(5)))`.

You can find more information on these pre-defined matchers [here](https://github.com/google/googletest/blob/master/docs/reference/matchers.md#wildcard).

But this is only the first part of the `EXPECT_CALL` macro. You can chain it with different optional clauses.

The first is often referred to as cardinality and it's expressed with `Times(n)`. `n` can be an exact number and in that case, if the given function is called more or fewer times - with the expected parameters - the test will fail.

We can also be less precise and write something like `AtLeast(n)` or `AtMost(n)`, or even `Between(n, m)`. You can find all the [options for cardinality here](https://github.com/google/googletest/blob/master/docs/reference/mocking.md#times-expect_calltimes). 

`EXPECT_CALL(c, addFuel(5.0)).Times(::testing::Between(1, 3));` would express that on instance `c`, `addFuel` with the parameter `5.0` should be called once, twice or even three times, but no more or fewer times.

As mentioned earlier, with mocks we can both observe how an object is used, but we can also define what it should do when it's called. We can define actions and we can do right after we set the cardinalities.

We have two options to define actions, we can use either `WillOnce` or `WillRepeatedly`. It's worth noting they can be also chained, `WillOnce` can be followed either by another `WillOnce` or `WillRepeatedly`.

These actions are self-evident, `WillOnce` will define the action to be taken for one call and `WillRepeatedly` for all the coming calls. What to pass them as a parameter?

- To return a value, you can use `::testing::Return(value)`, e.g. `EXPECT_CALL(c, getTrunkSize()).WillRepeatedly(::testing::Return(420))`
- To return a reference, you can use `::testing::ReturnRef(variable)`
- `Return` sets the value to be returned when you create the action, if you want to set the value when the action is executed, you can use `::testing::ReturnPointee(&vairable)`.

You saw in the previous example that I omitted to set the cardinalities - setting how many times we expect the function to be called. Setting the cardinalities is not mandatory and they can be deduced:
- With no action set, it's inferred as `Times(1)`
- If only `WillOnce` is used, it will be `Times(n)` where `n` is the number of times `WillOnce` is used
- If both actions are used, it will be `Times(AtLeast(n))` where `n` is the number of times `WillOnce` is used.

## Differences between ON_CALL and EXPECT_CALL

As mentioned, the biggest difference between `ON_CALL` and `EXPECT_CALL` is that `ON_CALL` doesn't set any expectations.

It might sound counter-intuitive, but because of the above difference, you should use `ON_CALL` by default.

With `EXPECT_CALL` you might overspecify your tests and they become too brittle. You might couple the tests too closely to the implementation. Think about [the problem of test contra-variance explained by Uncle Bob](https://blog.cleancoder.com/uncle-bob/2017/10/03/TestContravariance.html).

Use `EXPECT_CALL` only when the main purpose of a test is to make sure that something gets called, and even then you should think twice whether you want it to be tested at all.

## What if you don't want to provide a default behaviour?

In the previous sections, we saw what happens when we have a mocked interface and we provide the mocked behaviour with either `EXPECT_CALL` or with `ON_CALL`. But what happens if we forget or we don't want to provide an overridden behaviour? You might think it's not realistic, but if you mock lots of functions of an API - it should probably be a red flag, by the way - it might happen that you don't want to provide a mocked behaviour every time for every function.

Even if you fail to provide a mocked behaviour, it will be automatically provided under certain conditions:
- if the return type is `void`, the default action is a no-op. In other words, the mocked behaviour is to do nothing, instead of executing the original behaviour.
- if the return type is not `void`, a default constructed value will be returned, given that the return type can be default constructed.

If the return type is not *default constructible*, you'll get a runtime exception:

> *unknown file: Failure*
> *C++ exception with description "Uninteresting mock function call - returning default value.*
> *Function call: getTrunkSize()*
> *The mock function has no default action set, and its return type has no default value set."*

If you don't get the runtime exception and the default action is used, you'll get a runtime warning from the *gMock* framework:

> *Uninteresting mock function call - returning default value.*

It's quite straightforward, it doesn't require much explanation.

But how to get rid of it?

You have a couple of options:
- You stop mocking this method.
- You do provide a mocked behaviour.
- Instead of simply creating an instance of your `MockedClass`, use `::testing::NiceMock<MockedClass>` in order to silence such warnings. More about this next time.

But can we fall back to the original implementation?

Of course, we can do whatever we want! For this, we need a lambda:

```cpp
ON_CALL(c, startEngine()).WillByDefault([&c](){return c.Car::startEngine();});
```

As you can see, the lambda simply forwards the call to the underlying base class.

## Conclusion

Today we started to discover one of the most popular mocking frameworks for C++, *gMock*. In this first episode, we saw how to mock *virtual* functions, how to provide simplified behaviour for them, and how to make assertions on how many times and with what inputs a mocked function is called.

Next time we'll see how to mock non-virtual members and free functions. Stay tuned.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
