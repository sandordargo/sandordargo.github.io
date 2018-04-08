---
layout: post
title: "How to use rules in JUnit"
date: 2017-8-8
category: dev
tags: [tdd, java, junit, rules]
header: "Have you ever wondered what JUnit rules are? No, because you have never heard about them? If you answered \"yes\" to any of these two question, you're just like I was. Let's explore rules together, they are not so complex after all."
---

Let's start from the beginning. How do you recognize rules? You'll see some public variables annotated with `@Rule` in your test class. Here is an example:

```
public class MyTest {

  @Rule
  public SomeRule someRule = new SomeRule()

  @Test
  public void returns42ToAllQuestions() {
    //...
  }
}
```

So if you see something annotated with `@Rule`, you have a rule in your test. Fine, but what the heck is that?!

Each JUnit rule will be applied before and after each test is executed. Better to say it is applied around a test case. In some sense it's like having a `@Before` and an `@After`, but a rule can be reused easily in multiple classes. 

You have to create a class implementing JUnit's `TestRule` interface. Then in you can instantiate it in your test classes as a rule. That's how you reuse it.

_Of course, JUnit also provides some predefined rules such as [`TemporaryFolder`](http://junit.org/junit4/javadoc/4.12/org/junit/rules/TemporaryFolder.html) which creates a temporary directory before each test and deletes it afterwards._

__How to create a custom rule?__

As I mentioned your rule class will have to implement the `TestRule` interface, you have to implement one method:

```
@Override
  public Statement apply(Statement base, Description description) {
    //...
  }
```

Normally this `apply` method will not do much, it will just return a new `Statement`, which is an abstract class in JUnit. 

In order to implement it, you only have to implement the abstract `evaluate()` method.

```
public abstract void evaluate() throws Throwable;
```

A dummy example would be:

```
@Override
public Statement apply(Statement base, Description desc) {
    return new Statement() {
        @Override
        public void evaluate() throws Throwable {
            System.out.println("Before executing method " + desc.getMethodName());
            base.evaluate();
            System.out.println("After executing method " + desc.getMethodName());
        }
    };
}
```

Your rule might expose other public methods which can be used in your test classes. Maybe directly in certain tests cases, maybe in some setup methods.

For example, let's have a quick look at [`EnvironmentVariables`](https://stefanbirkner.github.io/system-rules/apidocs/org/junit/contrib/java/lang/system/EnvironmentVariables.html). It has two public methods:

* the just described `apply()` returning a `Statement`

* and a `set()` method which takes two arguments, in fact a key-value pair

So this class will enable you to set environment variables within your tests. You create the rule, then you use the setter method in your test or in the setup and after each test the changes you made are reverted without you having to care about the revert.

__What happens if you need multiple rules?__

You can have as many rules in your test class as you need. But as always, you should find the balance, you are not building a rule database. And don't forget, the rules can be applied in any order regardless of the declaration order.

__What if a rule depends on another one?__

As I mentioned just before, rules can be executed in any order, you cannot and should not take the order of declaration as an order of execution. Instead, you can chain your rules! With the `@RuleChain` annotation it is straightforward. Let's look at the example taken from the [official documentation](http://junit.org/junit4/javadoc/4.12/org/junit/rules/RuleChain.html):


```
public static class UseRuleChain {
    @Rule
    public final TestRule chain = RuleChain
                           .outerRule(new LoggingRule("outer rule"))
                           .around(new LoggingRule("middle rule"))
                           .around(new LoggingRule("inner rule"));

    @Test
    public void example() {
        assertTrue(true);
    }
}
```

Assuming that you have a `LoggingRule` class which would log the message you passed to its constructor at rule application time, you will have the following lines in your logs:

```
Starting outer rule
Starting middle rule
Starting inner rule
Finished inner rule
Finished middle rule
Finished outer rule
```

That's it! Now you know enough to start using rules for managing your test resources, in addition to the `@Before`/`@After` `@BeforeClass`/`@AfterClass` annotations.
You should use rules when you see yourself repeating the same `@Before`/`@After` methods in many of your test classes.
