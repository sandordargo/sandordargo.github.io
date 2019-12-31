---
layout: post
title: "The Art of Unit Testing by Roy Osherove"
date: 2020-1-1
category: books
tags: [books, watercooler, tdd, testing]
excerpt_separator: <!--more-->
---
[The Art of Unit Testing](https://amzn.to/2S3gRVr) is useful for both beginner unit testers and for those who already have a bit of experience. While the edition I read is with C# examples it is useful and understandable for people who work in other languages. People like me.
<!--more-->

It is practical, probably as long as you work with statically typed languages, not like Python or Ruby. In dynamic languages, you can extend basically anything at any point in time. You can change behaviour, replace implementations, hence design for testability is not a concern - as confirmed by the author, [Roy Osherove](https://osherove.com/).

In the beginning, the author spends quite some time on defining what is unit testing and he iterates over several definitions reaching the final one:

> "A unit test is an automated piece of code that invokes the unit of work being tested, and then checks some assumptions about a single end result of that unit. A unit test is almost always written using a unit testing framework. It can be written easily and runs quickly. It’s trustworthy, readable, and maintainable. It’s consistent in its results as long as production code hasn’t changed."

This also means that if a test uses the real system time, filesystem or a real database, it's not a unit test anymore, but rather an integration test.

I had a discussion about this with other devs, and apparently there are many who accept certain "integration" tests as unit tests as long as they are deterministic and fast enough.

While the author discusses a lot [Test Driven Development](https://en.wikipedia.org/wiki/Test-driven_development), [Solid principles](https://en.wikipedia.org/wiki/SOLID) and so, he claims that we can and should speak about three different areas:
* unit testing
* code design
* test-driven development

According to the author, while test-driven development is useful and has a lot of advantages including more testable code, it won't lead automatically to better architecture. The above mentioned three areas are three different skills that one has to learn. The book focuses only on the first one, unit testing.

I found some interesting ideas in the book. At some points, I was a bit surprised and I thought that the content might be outdated. But maybe it was not out-of-date, just a bit language-specific. As an example, I prefer much more dependency injection compared to introducing inheritance just for the sake of unit testability. In C++, methods are not virtual by default and I prefer not to declare something virtual just in order to make it replaceable. Osherove is much more permissive with introducing inheritance for testing purposes. Again, this might depend on the language.

The book doesn't only help you learn about unit testing, but also gives advice on where to start unit testing in legacy code - you should definitely read [Your Code as a Crime Scene](https://amzn.to/38NwJBn) - and on how to introduce it in an organization, how to convince people. Though, these are not the main topics. If you want to get more detailed information on introducing changes, I'd advise you to read [Driving Technical Change by Terrence Ryan](http://sandordargo.com/blog/2019/07/31/driving-technical-change).

On the other hand, on unit testing best practices it really goes into details and if you have hard times to convince your peers about some techniques that you heard before here and there, now you have the perfect reference!

I'd mention one idea from the book, that I have never considered before, not even when I worked with Java and JUnit.

I thought that such a test for asserting that you expect an exception is great:

```java
@Test(expected = IndexOutOfBoundsException.class)
public void testIndexOutOfBoundsException() {

    List emptyList = new 
    Object o = emptyList.get(0);

}
```

According to [Osherove](https://osherove.com/), it's not.

The exception can come from any instruction, not just from the call that you actually want to test. So he prefers the following format instead.

```java
@Test
public void doStuffThrowsIndexOutOfBoundsException() {
  Foo foo = new Foo();

  IndexOutOfBoundsException e = assertThrows(
    IndexOutOfBoundsException.class, foo::doStuff);

  assertThat(e).hasMessageThat().contains("woops!");
}
```

The point is that your assertion should consider only the call that you expect to throw, no more.


All in all, I liked [The Art of Unit Testing](https://amzn.to/2S3gRVr) for its detailed insights. If you work with C#, most probably it's the _goto_ book on unit testing, but even for other languages, it is more than useful. Read it, adapt it to your language and help your team to unit test better!

Happy testing!