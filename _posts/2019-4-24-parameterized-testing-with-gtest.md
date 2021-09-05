---
layout: post
title: "Parameterized testing with GTest"
date: 2019-4-24
category: dev
tags: [unit testing, cpp, gtest, parameterized tests]
excerpt_separator: <!--more-->
---
For one of the latest dojos in our department, we chose a relatively simple kata to help new people get on board. We were working on the [leap year kata](http://codingdojo.org/kata/LeapYears/) in Randori style meaning that we were using only one computer - there were 9 of us.

We also applied some extra constraints, such as if after every three minutes our tests were not green (except for the red phase when we had to write a failing test), we had to wipe out our changes with `git reset --hard`.
<!--more-->

Even with - some non-mentioned - extra constraints this kata doesn't take up one and half hours to implement, so we had extra time to try something new. As you could have already guessed based on the title, we were experimenting with [parameterized tests in GoogleTest](https://github.com/sandordargo/parameterizedTestExamplesCpp).

## How to make our tests less repetitive without parameterized tests?

The first question to answer is what parameterized tests are, but before let's see why we need them.

*If you want to go directly to parameterized tests, [jump to the next section](#parameterizedtests).*

Imagine that you have a couple of quite similar tests, like these:

```cpp
#include <gtest/gtest.h>

#include <LeapYearCalendar.h>

TEST(LeapYearTests, 1IsOdd_IsNotLeapYear) {
  LeapYearCalendar leapYearCalendar;
  ASSERT_FALSE(leapYearCalendar.isLeap(1));
}

TEST(LeapYearTests, 711IsOdd_IsNotLeapYear) {
  LeapYearCalendar leapYearCalendar;
  ASSERT_FALSE(leapYearCalendar.isLeap(711));
}

TEST(LeapYearTests, 1989IsOdd_IsNotLeapYear) {
  LeapYearCalendar leapYearCalendar;
  ASSERT_FALSE(leapYearCalendar.isLeap(1989));
}

TEST(LeapYearTests, 2013IsOdd_IsNotLeapYear) {
  LeapYearCalendar leapYearCalendar;
  ASSERT_FALSE(leapYearCalendar.isLeap(2013));
}
```

As you can observe there are only two things changing:
- the inputs
- and the expected results. 

Wouldn't it be great to [refactor a bit](https://amzn.to/2EhV5s8) and reduce the code repetition?

No doubt, it would be just awesome!

But how to do it?

You might start off in different directions.

### Using a fixture

One possible way to make the code DRYer is to create a fixture and get rid of the initialization of `Foo`.

```cpp

#include <gtest/gtest.h>

#include <LeapYearCalendar.h>

class LeapYearFixtureTests : public ::testing::Test {
protected:
    LeapYearCalendar leapYearCalendar;
};

TEST_F(LeapYearFixtureTests, 1IsOdd_IsNotLeapYear) {
    ASSERT_FALSE(leapYearCalendar.isLeap(1));
}

TEST_F(LeapYearFixtureTests, 711IsOdd_IsNotLeapYear) {
    ASSERT_FALSE(leapYearCalendar.isLeap(711));
}

TEST_F(LeapYearFixtureTests, 1989IsOdd_IsNotLeapYear) {
    ASSERT_FALSE(leapYearCalendar.isLeap(1989));
}

TEST_F(LeapYearFixtureTests, 2013IsOdd_IsNotLeapYear) {
    ASSERT_FALSE(leapYearCalendar.isLeap(2013));
}
```

This is a step forward, we don't need to instantiate `leapYearCalendar` anymore in each test, it's performed by the fixture. We might decide to change no more, but still, the code seems quite repetitive.

### The good old `for` loop

Another option is to create a list of years within the test case and iterate over it.

```cpp
#include <gtest/gtest.h>

#include <LeapYearCalendar.h>

TEST(LeapYearIterationTest, OddYearsAreNotLeapYears) {
    LeapYearCalendar leapYearCalendar;
    auto oddYears = std::vector<int>{1, 3, 711, 2013};
    for (auto oddYear :  oddYears) {
        ASSERT_FALSE(leapYearCalendar.isLeap(oddYear));
    }
}

```

In terms of repetitiveness, in my opinion, this code is better, it's denser, yet it is very readable. But it has a big flaw! A good unittest should have only one logical assertion - as always, some exceptions apply. On the other hand, in this case, we have multiple different assertions that should not be combined into one.

We might say this is a theoretical problem, but it has a practical issue too. Let's say that for our 2nd iteration the test fails. What happens then? Our tests are stopped and all the other values will not be tested. We miss the feedback for the other 4 values.

You might say that we can overcome this problem by using the macro `EXPECT_FALSE`, but the error message you'll get is not optiomal.

```
[ RUN      ] LeapYearIterationTest.OddYearsAreNotLeapYears
/home/sdargo/personal/dev/LeapYear/tests/TestLeapyearIteration.cpp:8: Failure
Value of: leapYearCalendar.isLeap(oddYear)
  Actual: true
Expected: false
[  FAILED  ] LeapYearIterationTest.OddYearsAreNotLeapYears (0 ms)
```

We don't even know which iteration failed!

## Parameterized tests, what are they? [parameterizedtests]

Can we combine the advantages of a DRY for loop with the ones of independent tests without the drawbacks?

Not completely. But using parameterized tests from GoogleTest is definitely an option you should consider..

We have two different ways to use this feature. One way is to build our tests from scratch and the other is to build them on the foundations of a [`FIXTURE`](https://github.com/google/googletest/blob/master/googletest/samples/sample5_unittest.cc) like the one we already saw when we introduced a common `leapYear` variable. Let's see the two options one by one.

### Write parameterized tests without a fixture

In this case, we have no existing fixture and we don't need one. 

Let's continue testing the [leap year kata](http://codingdojo.org/kata/LeapYears/).

First, we need to create our parameterized test class. Let's call it `LeapYearParametrizedTests` and it has inherit to from  `::testing::TestWithParam<T>`. `T` is a template parameter and it is going to be the type of the parameter or parameters we want to pass into each iteration. Let's start with a simple example, where the parameters will be of the type integer.

```cpp
class LeapYearParameterizedTestFixture :public ::testing::TestWithParam<int> {
protected:
    LeapYearCalendar leapYearCalendar;
};
```

Next, we need a test case with an assertion in it.

```cpp
TEST_P(LeapYearParameterizedTestFixture, OddYearsAreNotLeapYears) {
    int year = GetParam();
    ASSERT_FALSE(leapYearCalendar.isLeap(year));
}
```

While for a normal unittest we use the `TEST()` macro and `TEST_F()` for a fixture, we have to use `TEST_P()` for parameterized tests.
As the first parameter, we have to pass the name of the test class and as the second we just have to pick a good name for what our tests represent.

In order to retrieve the parameter from the list of values (that we are going to define in a few seconds), we have to use `GetParam()`.

So far, so good! Now we don't need anything else, but to call our use-case with - preferably - multiple inputs.

```cpp
INSTANTIATE_TEST_CASE_P(
        LeapYearTests,
        LeapYearParameterizedTestFixture,
        ::testing::Values(
                1, 711, 1989, 2013
        ));
```

Here we call the `INSTANTIATE_TEST_CASE_P` macro with first with a unique name for the instantiation of the test suite. This name can distinguish between multiple instantiations. In test output, the instantiation name - in this case, `LeapYearTests` - is added as a prefix to the test suite name `LeapYearParameterizedTestFixture`.

Last but not least, we have to list the different inputs we want to test with.

*Since, release 1.10 `INSTANTIATE_TEST_CASE_P` is replaced with `INSTANTIATE_TEST_SUITE_P`!*

Et voila, it's as easy as that! Here is the full example. I included a leap year implementation so you can run it easily if you have GTest available. You can also refer to [my GitHub repo](https://github.com/sandordargo/parameterizedTestExamplesCpp) for the code and instructions for compiling and running it.


```cpp
#include <gtest/gtest.h>

#include <LeapYearCalendar.h>

class LeapYearParameterizedTestFixture :public ::testing::TestWithParam<int> {
protected:
    LeapYearCalendar leapYearCalendar;
};

TEST_P(LeapYearParameterizedTestFixture, OddYearsAreNotLeapYears) {
    int year = GetParam();
    ASSERT_FALSE(leapYearCalendar.isLeap(year));
}

INSTANTIATE_TEST_CASE_P(
        LeapYearTests,
        LeapYearParameterizedTestFixture,
        ::testing::Values(
                1, 711, 1989, 2013
        ));

```

Let's have a look at the output:

```
[----------] 4 tests from LeapYearTests/LeapYearParameterizedTestFixture
[ RUN      ] LeapYearTests/LeapYearParameterizedTestFixture.OddYearsAreNotLeapYears/0
[       OK ] LeapYearTests/LeapYearParameterizedTestFixture.OddYearsAreNotLeapYears/0 (0 ms)
[ RUN      ] LeapYearTests/LeapYearParameterizedTestFixture.OddYearsAreNotLeapYears/1
[       OK ] LeapYearTests/LeapYearParameterizedTestFixture.OddYearsAreNotLeapYears/1 (0 ms)
[ RUN      ] LeapYearTests/LeapYearParameterizedTestFixture.OddYearsAreNotLeapYears/2
[       OK ] LeapYearTests/LeapYearParameterizedTestFixture.OddYearsAreNotLeapYears/2 (0 ms)
[ RUN      ] LeapYearTests/LeapYearParameterizedTestFixture.OddYearsAreNotLeapYears/3
[       OK ] LeapYearTests/LeapYearParameterizedTestFixture.OddYearsAreNotLeapYears/3 (0 ms)
[----------] 4 tests from LeapYearTests/LeapYearParameterizedTestFixture (0 ms total)
```

We can observe that each test name is composed of 3 parts:
- the suite name
- the test name
- number of each iteration starting from 0

If you want multiple test scenarios, you have to create a suite for each scenario as with `INSTANTIATE_TEST_CASE_P` each test in a suite will be triggered. We can safely assume that different tests would produce different results with the same inputs.

### Write parameterized tests based on an existing fixture

It might happen that you have already a test fixture available, like this one:

```cpp
class LeapYearTestFixtureToBeParameterized : public ::testing::Test
{
protected:
  LeapYearCalendar leapYearCalendar;
};
```

In this case, it is very simple, the fixture itself just helps to avoid declaring a leap year object in each different test case. It wouldn't be a big deal to lose it, but you might have a more complex setup.

As a reminder, here are the fixture tests that are really compact:

```cpp
TEST_F(LeapYearTestFixtureToBeParameterized, 1996_IsDivisibleBy4_ShouldBeALeapYear) {
  ASSERT_TRUE(leapYearCalendar.isLeap(1996));
}

TEST_F(LeapYearTestFixtureToBeParameterized, 1700_IsDivisibleBy100AndNotBy400_ShouldNotBeALeapYear) {
  ASSERT_FALSE(leapYearCalendar.isLeap(1700));
}

TEST_F(LeapYearTestFixtureToBeParameterized, 1600_IsDivisibleBy400_ShouldBeALeapYear) {
  ASSERT_TRUE(leapYearCalendar.isLeap(1600));
}
```

So first we decided to have a fixture and we could name our test cases well enough to document why something is a leap year and some others are not leap years. 

Then we thought that there are some use-cases which we'd like to test with many different values. Hm... What should we do?

We could create our parameterized tests here or in another file, it doesn't matter. But we wouldn't be able to access `leapYearCalendar`.

Put aside ugly global variables, what else can we do?

We can inherit from `::testing::WithParamInterface<T>` instead of `::testing::TestWithParam<T>`!

```cpp
class LeapYearTestFixtureToBeParameterized : public ::testing::Test
{
protected:
  LeapYearCalendar leapYearCalendar;
};

class LeapYearParametrizedTestFixtureBasedOnFixture :
  public LeapYearTestFixtureToBeParameterized,
  public ::testing::WithParamInterface<int> {
};

```

Of course, if you don't need the separate fixture, you can combine the two classes into one:

```cpp
class LeapYearParametrizedFixture :
        public ::testing::Test,
        public ::testing::WithParamInterface<int> {
protected:
    LeapYear leapYearCalendar;            
};
```

You might say that having a parameterized fixture does not make much sense. After all, we said that each test requires a different suite, so there is nothing to share, there will be no different tests.

Hence, inheriting from a fixture might make more sense. In the fixture, we removed some code duplication and in the parameterized suite we can benefit from the fixture's code.


```cpp
class LeapYearTestFixtureToBeParameterized : public ::testing::Test
{
protected:
    LeapYear leapYearCalendar;
};

TEST_F(LeapYearTestFixtureToBeParameterized, 1996_IsDivisibleBy4_ShouldBeALeapYear) {
    ASSERT_TRUE(leapYearCalendar.isLeap(1996));
}

TEST_F(LeapYearTestFixtureToBeParameterized, 1700_IsDivisibleBy100AndNotBy400_ShouldNotBeALeapYear) {
    ASSERT_FALSE(leapYearCalendar.isLeap(1700));
}

TEST_F(LeapYearTestFixtureToBeParameterized, 1600_IsDivisibleBy400_ShouldBeALeapYear) {
    ASSERT_TRUE(leapYearCalendar.isLeap(1600));
}

class LeapYearParameterizedTestFixture :
        public LeapYearTestFixtureToBeParameterized,
        public ::testing::WithParamInterface<int> {
protected:
    LeapYear leapYearCalendar;            
};

TEST_P(LeapYearParameterizedTestFixture, OddYearsAreNotLeapYears) {
    int year = GetParam();
    ASSERT_FALSE(leapYearCalendar.isLeap(year));
}

INSTANTIATE_TEST_CASE_P(
        LeapYearTests,
        LeapYearParameterizedTestFixture,
        ::testing::Values(
                1, 711, 1989, 2013
        ));
```

If you are wondering why we use `WithParamInterface<T>` instead of `TestWithParam<T>`, here is the answer. `TestWithParam<T>` inherits both from `Test` and `WithParamInterface<T>`. The fixture that we inherited from in the previous example already inherited from `Test`. So we inherited from `Test` trough both parents and it became an ambiguous base.

![Test Class Hierarchy]({{ site.baseurl }}/assets/img/test-class-hierarchy.jpg)

### How to pass multiple parameters to the same test case?

Let's say you have two inputs that you want to parameterize, or you want to pass both the input and the output! What can you do?

You cannot pass more than one template argument to `TestWithParam<T>`, but you can always pass a `std::pair`, or even better a `std::tuple` with as many members as you want. 

Here is an example:

```cpp
class LeapYearMultipleParametersTests :public ::testing::TestWithParam<std::tuple<int, bool>> {
protected:
    LeapYearCalendar leapYearCalendar;
};

TEST_P(LeapYearMultipleParametersTests, ChecksIfLeapYear) {
    bool expected = std::get<1>(GetParam());
    int year = std::get<0>(GetParam());
    ASSERT_EQ(expected, leapYearCalendar.isLeap(year));
}

INSTANTIATE_TEST_CASE_P(
        LeapYearTests,
        LeapYearMultipleParametersTests,
        ::testing::Values(
                std::make_tuple(7, false),
                std::make_tuple(2001, false),
                std::make_tuple(1996, true),
                std::make_tuple(1700, false),
                std::make_tuple(1600, true)));
```

In this case, `GetParam()` retrieves tuples. In order to obtain an element of a tuple we can use [std::get<T>](https://en.cppreference.com/w/cpp/utility/tuple/get). Or we could even use structured bidings starting from C++17:

```cpp
auto [year, expected] = GetParam();
```

Unit tests have multiple goals. On the one hand, they give you confidence when you change code. The higher your coverage, the more confident you are that your change will not introduce a bug.

On the other hand, unit tests also document your code, it gives the best possible documentation on how it should be used and how it behaves. Unlike written documentation, it cannot be stale, because it would not compile anymore.

The larger the tuples you pass in, the less your parameterized tests will document your code. With each new parameter, it gets more difficult to understand what you test at the moment and in case of a failure, it's more difficult to understand what went wrong. 

I don't say that parameterized tests are evil. I just say that it has its own compromises.

## Takeaway

In this article, we discovered how to write parameterized tests with the GoogleTest. Of course, GoogleTest is not the only library you can use to implement such tests in a simple way, [boost::unit_test](https://www.boost.org/doc/libs/1_64_0/libs/test/doc/html/boost_test/tests_organization/test_cases/param_test.html) and [Catch2](https://github.com/catchorg/Catch2/tree/master/docs) also have this nice feature. In later articles, I might show them.

Parameterized tests are a great tool to remove code duplication from your test suites. They come in handy when you want to test essentially the same behaviour for many different outputs.

As we saw, we can also parameterize the outputs, but then the main disadvantage of parameterized tests comes into play. The test suite has one name and for each set of parameters, it's going to be the very same name. If one fails, you don't have a hint from the test name.

Sometimes this is acceptable, sometimes you want to look for another solution.

You can download and experiment with the above examples from [this GitHub repository](https://github.com/sandordargo/parameterizedTestExamplesCpp).

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
