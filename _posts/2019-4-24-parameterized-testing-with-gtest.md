---
layout: post
title: "Parameterized testing with GTest"
date: 2019-4-24
category: dev
tags: [unit testing, cpp, gtest, parameterized tests]
excerpt_separator: <!--more-->
---
For one of the latest dojos in our department, we chose a relatively simple one to help new people get on board. We were working on the [leap year kata](http://codingdojo.org/kata/LeapYears/) in Randori style meaning that we were using only one computer - there were 9 of us.

We also applied some extra constraints, such as if after every three minutes our tests were not green (except for the red phase when we had to write a failing test), we had to wipe out our changes (`git reset --hard`).
<!--more-->

Even with - some non-mentioned - extra constraints this kata doesn't take up one and half hours to implement, so we had some time to try something new. As you could have already guessed based on the title, we were experimenting with [parameterized tests in GTest](https://github.com/sandordargo/parameterizedTestExamplesCpp).

## How to make our tests less repetitive?

The first question I might have to answer is what parameterized tests are?!

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

As you can observe the only things that change are the inputs and the expected results. Wouldn't it be great to [refactor a bit](https://amzn.to/2EhV5s8) and reduce the code repetition?

No doubt, it would be just awesome!

But how to do it?

You might start off in different directions.

### Using a fixture

One possible way to make the code DRYer is to create a fixture and get rid off the initialization of `Foo`.

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

This is nice, we might decide to stay here, but still, the code seems quite repetitive.

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
In terms of repetitiveness, in my opinion, this code is better, it's denser, yet it is very readable. But it has a big flaw! A good unittest should have only one logical assertion - some exceptions as always apply. On the other hand, in this case, we have multiple different assertions that should not be combined into one.

We might say this is a theoretical problem, but it has a practical issue too. Let's say that for our 2nd iteration the test fails. What happens then? Our tests are stopped and all the other values will not be tested. We miss the feedback for the other 4 values.

Maybe our code would fail with all the other values, but we will have to discover these failures one by one. What a pity!

## Parameterized tests, what are they?

Can we combine the advantages of a DRY for loop with the ones of independent tests while removing the problems they have?

Of course, we can! Let's use parameterized testing from GTest.

We have two different ways to use this feature. One way is to build our tests from scratch and the other is to build them on the foundations of a [`FIXTURE`](https://github.com/google/googletest/blob/master/googletest/samples/sample5_unittest.cc) like the one we already saw when we introduced a common leapYear variable. Let's see the two options one by one.

### Write parameterized tests from scratch

In this case, we have no existing fixture and we want our parameterized tests to exist on their own, without combining them with one.

In the coming example, I'm going to assume that we are building tests for the [leap year code kata](http://codingdojo.org/kata/LeapYears/).

First, we need to create our parameterized test class. Let's call it `LeapYearParametrizedTests` and it will inherit from  `::testing::TestWithParam<T>`. `T` is a template parameter and it is going to be the type of the parameter or parameters we want to pass in. Let's start with a simple example, where the parameters will be of the type integer.

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
As the first parameter, we have to pass the name the test class and as the second we just have to pick a good name for what our tests represent.

So far, so good! Now we don't need anything else, but to call our use-case with - preferably - multiple inputs.

```cpp
INSTANTIATE_TEST_CASE_P(
        LeapYearTests,
        LeapYearParameterizedTestFixture,
        ::testing::Values(
                1, 711, 1989, 2013
        ));
```

Here we call the `INSTANTIATE_TEST_CASE_P` macro with first with a random name, then the test class and last but not least with our list of inputs.

Et voila, it's as easy as that! Here is the full example. I included a leap year implementation so you can run it easily if you have GTest available. You can also refer to my repo for the code and instructions for compiling and running it.


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


If you want multiple test scenarios, you have to create a class for each scenario.

_Also note that about GTest v1.8.1 you should use `INSTANTIATE_TEST_SUITE_P` instead of `INSTANTIATE_TEST_CASE_P`._

### Write parameterized tests based on an existing fixture

It might happen that you have already a test fixture available, like this one:

```cpp
class LeapYearTestFixtureToBeParameterized : public ::testing::Test
{
protected:
  LeapYearCalendar leapYearCalendar;
};
```

In this case, it is very simple, the fixture itself just helps to avoid declaring a leap year object in each different test case. Thus we can keep them really compact:

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

After this, we basically do the same as in the previous example. We have our parameterized tests and the fixture to sharing some code, so yay, we removed some code duplication, we keep distinct names for the fixture test cases and one for the parameterized tests.

### How to pass multiple parameters to the same test case?

Let's say you have to inputs that you want to parameterize, or you want to pass both the input or the output! What can you do?

Well, you cannot pass more than one template arguments to the demonstrated classes, but you can always pass a `std::pair`, or even better a `std::tuple` with as many members as you want. 

Here is a full example:

```cpp
#include <tuple>

#include <gtest/gtest.h>

#include <LeapYearCalendar.h>


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

It's worth to note two things.
* As values you pass tuples, as I already suggested.
* You have to wrap `GetParam()` with `std::get<POSITION>()` in order to get the desired parameter in a TestCase. As usual, in programming, we start counting from zero.

## Takeaway

In this article, you could learn about how to write parameterized tests with the GTest library. GTest is not the only library you can use to implement such tests in a simple way, [boost::unit_test](https://www.boost.org/doc/libs/1_64_0/libs/test/doc/html/boost_test/tests_organization/test_cases/param_test.html) and [Catch2](https://github.com/catchorg/Catch2/tree/master/docs) also have this nice feature. In later articles, I might show them.

You can download and experiment with the above examples from this [this GitHub repository](https://github.com/sandordargo/parameterizedTestExamplesCpp).