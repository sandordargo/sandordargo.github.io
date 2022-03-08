---
layout: post
title: "Mocking non-virtual and free functions with gMock"
date: 2022-3-9  
category: dev
tags: [cpp, virtual, mocking, gmock]
excerpt_separator: <!--more-->
---
[Last time we started to discover `gMock`](https://www.sandordargo.com/blog/2022/03/02/mocking-non-virtual-and-free-functions) and we went into details regarding how we can mock `virtual` functions. We saw how to indicate that a function is to be mocked, how to provide a canned behaviour for them and how to make assertions on whether they are called or not and with what inputs.
<!--more-->

Today, we are going to continue our quest by mocking non-`virtual` members and free-standing functions.

I must mention before we discuss the details that I try not to repeat lots of information from the [previous article](https://www.sandordargo.com/blog/2022/03/02/mocking-non-virtual-and-free-functions). In particular, I don't share again how to build up `ON_CALL` or `EXPECT_CALL` commands. Those work the same both for `virtual` and non-`virtual` functions. Please visit the [previous article](https://www.sandordargo.com/blog/2022/03/02/mocking-non-virtual-and-free-functions) if you are interested in those parts.

Let's get down to business!

## How to mock a non-virtual function?

Now that we know how to mock a `virtual` function, let's discuss whether we can mock a non-`virtual` one. While the [gmock cook book](https://github.com/google/googletest/blob/master/docs/gmock_cook_book.md#mocking-non-virtual-methods-mockingnonvirtualmethods) says it can be easily done, I tend to disagree with the *easily* part. At least it's far from convenient.

The great thing about mocking `virtual` functions is that you don't need to change the production code at all- unless they are private. It's not the case for non-`virtual`s.

Let's assume that we have the same interface as before, but without the methods being `virtual` and of course without any abstract functions:

```cpp
class Car {
public:
  ~Car() = default;
  void startEngine() {
    // some implementation
  }
  
  int getTrunkSize() const {
    // some implementation
  }
  
  void addFuel(double quantity) {
    // some implementation
  }
};
```

We have to create the mocked class the very same way as before except for the `override` specifier and also we don't inherit from any class. Given that we have no `virtual`, there is nothing to override:

```cpp
class MockCar {
public:
  MOCK_METHOD(void, startEngine, (), ());
  MOCK_METHOD(int, getTrunkSize, (), (const));
  MOCK_METHOD(void, addFuel, (double quantity), ());
};
```

So what we have now are two completely unrelated classes (no inheritance!) with the same signatures, same interface. We have to relate them somehow! We have to be able to tell the code which implementations are to be used and without the virtual dispatching. We have to do this at compile time.

The [cookbook suggest templatizing our code](https://github.com/google/googletest/blob/main/docs/gmock_cook_book.md#mocking-non-virtual-methods-mockingnonvirtualmethods). This is far from being an easy and comfortable solution to me.

We have to extract the code where mocked methods are used and replace them with forwarding calls to the implementation that is passed as a template argument.


```cpp
template <typename CarImpl>
class CarWrapper {
public:
  CarWrapper(C carImpl): _carImpl(carImpl) {}

  void startEngine() {
    _carImpl.startEngine();
  }
  
  int getTrunkSize() const {
    return _carImpl.getTrunkSize();
  }
  
  void addFuel(double quantity) {
    _carImpl.addFuel();
  } 
private:
  CarImpl _carImpl;
}
```

Now that we wrapped the implementation, what is rest is to replace all the calls to `Car` in production code with the instantiation of the wrapper:

```cpp
CarWrapper<Car> c;
```

And then the calls can stay the same.

In the unit tests, we have to do the same, but with `MockedCar`:

```cpp
CarWrapper<MockedCar> c;
```

I wouldn't say that this is a complex technique, but it does require some modifications, you have to add a new templatized wrapper to your codebase and you also have to change all the places where the wrapped object is used.

What you gain though is not introducing inheritance and vtables. You have to put everything on the balance and decide if it's worth it in your case.

This implementation is not exactly what the cook book suggests, though it's very similar. In the cook book, the calls on the class under test were not exactly forwarded, but the calls and the surrounding code were wrapped into functions with a different name compared to the existing functions in the original object. 

I think that suggestion goes too far. Templatizing the to be mocked functions and extracting code at the same time is a mixture of two steps. 

I would rather suggest taking two steps:
- replace the to be mocked object with its wrapper
- do the code extractions at your will, but not in the class template

This will help you to go in baby steps and to keep your changes small. Your code will be also clearer at the end.

### How to mock a free or a static function

Mocking a free or `static` function also requires changes. You can choose the direction you take.

If you want easy mocking, you can turn a free or a static function into a virtual member function. For free functions, this requries even to create a class around them.

The other way around is wrapping these functions with a templated layer as we saw for in the previous section. It's worth noting that with C++20 and with [the introduction of concepts](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations) and requires expressions, it's easy to communicate and enforce the types that can be used with a given template.

In most cases, I'd go with the templatization to avoid introducing a new class when it's not needed. Moreover to avoid introducting virtual tables when it's clearly not necessary.

## Some common pitfalls to avoid

While you are learning to use mocking in your unit tests, you will run into problems. Here is a collection of some common mistakes to avoid. Comment yours with your solutions and I'll keep enriching this list.

### Stating your expectation after exercising the code

A regular unit test generally follows the *AAA* pattern:
- Arrange
- Act
- Assert

This means that first, you *arrange*, you set up all the necessary objects that you need to *act*, to *execute* your code. And finally, you *assert* the result.

When it comes to mocking, it's a bit different. After making your *arrangements*, you have to set either your expectations and reactions (corresponding more or less to the *assert* part). And only then you should execute your code (*act*).

Otherwise if you *act* before you arrange, *gMock* will not be able to match the expectations. The expectation will stay unsatisfied and active.

```cpp
TEST(CarMockTest, testStatementOrder) {
  ::testing::NiceMock<MockCar> c;
  c.startEngine();
  EXPECT_CALL(c, startEngine()).Times(1);
}

/*
[----------] 1 test from CarMockTest
[ RUN      ] CarMockTest.testStatementOrder
/home/sdargo/personal/dev/LeapYear/tests/LeapYearFixtureTests.cpp:64: Failure
Actual function call count doesn't match EXPECT_CALL(c, startEngine())...
         Expected: to be called once
           Actual: never called - unsatisfied and active
[  FAILED  ] CarMockTest.testStatementOrder (0 ms)
[----------] 1 test from CarMockTest (0 ms total)
*/
```

Make sure that you make your expectation first and your test will work as intended:

```cpp
TEST(CarMockTest, testStatementOrder) {
  ::testing::NiceMock<MockCar> c;
  EXPECT_CALL(c, startEngine()).Times(1);
  c.startEngine();
}
/*
[----------] 1 test from CarMockTest
[ RUN      ] CarMockTest.testStatementOrder
[       OK ] CarMockTest.testStatementOrder (0 ms)
[----------] 1 test from CarMockTest (0 ms total)
*/
```

Probably this sounds too obvious, but in my experience it's a common mistake that I also often made in the early days.

### Don't return dangling pointers

The normal rules of thumb of C++ apply during mocking too. If you want the mock to return a pointer, you have to make sure that it points to a valid location in memory.

It happens that when you have to do the same setup for multiple test cases, you extract the code that arranges the test scenario into its own function.

In this case, you have to make sure that if a pointer or reference is returned, it's not pointing to a local object as the same restrictions apply as otherwise.

```cpp
class CarMockTest : public ::testing::Test {
protected:

  MyInt Setup() {
    auto size = MyInt{420};
    EXPECT_CALL(c, getTrunkSize()).Times(2).WillRepeatedly(::testing::ReturnPointee(&size)); // returning a dangling pointer
  }

  MockCar c;
};
```
The above case is erroneus, as due to `Setup()`, `getTrunkSize()` will return a something that got already destroyed. `ReturnPointee` returns a value pointed at by a pointer, and in this case it's just a local variable, therefore it is destoryed by the time it gets called.

You have 3 ways to fix this:
- don't extract the setup
- don't use `ReturnPointee` - in any case, if not needed, just use `Return`
- with `ReturnPointee` use something that lives as long as the fixture, like a `std::unique_ptr` declared as a member

### Scattering your results with uninteresting mock calls

This might happen when you have a bit too many mocked methods. You mock many methods in the same fixture that got often called, but as you are not interested in all of them in all your test cases, you don't set any expectations on them.

Then, when running your test that calls something you didn't define a behaviour for, you might get something like this:

```
GMOCK WARNING:
Uninteresting mock function call - returning default value.
    Function call: getTrunkSize()
          Returns: 0
NOTE: You can safely ignore the above warning unless this call should not happen.  Do not suppress it by blindly adding an EXPECT_CALL() if you don't mean to enforce the call.  See https://github.com/google/googletest/blob/master/googlemock/docs/cook_book.md#knowing-when-to-expect for details.
```

You have 2 ways to get rid of this.

The first one is to fix your tests in a way that you don't call unnecessary mocked methods. This can be achieved by making sure that those unnecessary methods are not called or by actually providing a behaviour for them. But this latter is indeed superfluous as the test worked already without. I would go with simplifying the tests.

The other way is to not use a regular mock object, but a `NiceMock`. `NiceMock<T>` and `StrictMock<T>` are class templates, wrappers that you use when you create your mocked objects. They modify the behaviour in case of uninteresting function calls.

By default, as we saw a few paragraphs before, *gMock* emits warnings. With `NiceMock` you don't receive any such warning while `StrictMock` will fail your test for any uninteresting function call.

## Conclusion

Today, in this second article on mocking we discussed how we can mock a non-`virtual` member function or a free function. We saw what changes we have to make in our code to make them testable.

Once we turned them into testable code, their mocking goes the same way as explained in the [previous article](https://www.sandordargo.com/blog/2022/03/02/mocking-non-virtual-and-free-functions).

We also saw a couple of common pitfalls that we must avoid when we try to mock our classes.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!