---
layout: post
title: "I broke production 3 times in 3 weeks - Part I"
date: 2021-11-24
category: dev
tags: [cpp, virtual, pointers]
excerpt_separator: <!--more-->
---
Are you a careful coder who barely introduces errors? How do you feel when still manage to bring production down? You might feel horrible, but I think you should take it as an opportunity.
<!--more-->
You can learn new things. 

You can practice responsibility.

You can enhance your team's processes.

You can do your best to ensure it won't happen again.

Recently I went on a spree. I caused 3 production issues in 3 weeks. If you consider that we load once a week, it is a remarkable performance.

I believe in the concept of [extreme ownership](https://devreads.sandordargo.com/extreme-ownership/). I must tell that they were all in my responsibility and in two cases I made big mistakes. The third one I consider more bad luck and a bit of negligence.

Whatever I'm going to write, bear in mind that I know I am the root cause of the incidents.

In general when faulty code gets delivered, I blame the reviewers. When you write an article, when you write documentation, God forbid a book, it's really difficult to spot your own errors. When you proofread your own work, you often don't read what is written there, but what you want to be there.

The reviewers don't have this bias.

You wrote something, obviously you think that it's right. The reviewers should assume that it's wrong and as it's not their code, it's easier for them to spot an error. 

Yet, when it's about my code, I assume that it's my fault. I cannot blame others for my failures.

Though sometimes the conclusion you draw should go beyond your responsibility.

When issues are not shown by any test campaign, when they don't appear in any test systems, something clearly went wrong and they should be fixed.

After all, the test systems are not there to slow down the delivery and the deployment process. They are in place to detect errors that humans committed.

In the coming weeks, I'm going to share with you 3 errors I made during recent times, 3 mistakes provoking fallbacks.

Let's start with some of the worst kind of bugs.

## Introducing undefined behaviour is never a great idea

I strongly believe in the boy scout rule:

> "Leave the campground cleaner than you found it."

I try to follow this principle both in my personal and professional life. To be fair, I'm more successful in this at work than at home.

What does this mean in practice?

When I fix a bug or when I add a new feature, I try to clean up a bit what is around. As I work on an application that has seen a lot during the last 30 years, there is always something to find.

Recently, I had to touch a big service class that had about 20 members and very long constructors.

The declarations were scattered through different `protected` and `private` blocks. Most of the members were initialized to always the same initial values, so in fact they didn't need to be defined in the constructor.

I started to remove the initializations from both the constructor's body and the constructor initialization list. I think this is a good idea, because when you initialize everything at declaration time, you cannot accidentally mess up with the orders and therefore introduce undefined behaviour.

```cpp
class A {
public:
  A();
  A(int m_foo, double m_bar);
private:
  int m_foo;
  double m_bar;
};

A::A() : m_bar(0.0), m_foo(0) {}

A::A(int foo, double bar) : m_bar(bar), m_foo(foo) {}
```

In this above example `m_foo` would be initialized after `m_bar`, whereas it was declared before and this is both undefined behaviour and a compiler warning.

Long story short, I prefer to see something like this:

```cpp
class A {
public:
  A(int m_foo, double m_bar);
private:
  int m_foo = 0;
  double m_bar = 0.0;
};

A::A(int foo, double bar) : m_foo(foo), m_bar(bar) {}
```
So that's what I did.

There were both value members and raw pointer members initialized to `0`, so I also updated the initial pointed values with `nullptr`. I prefer to move to a smart pointer in a different, dedicated step.

As mentioned, there were like 20 members scattered all over the place. I moved some of them together, so I ended up with one private and one protected section and...

And I missed to initialize one pointer to `nullptr` whereas it was initialized to `0`.

That's a bummer.

Is that a big problem?

It depends, but it's dangerous.

In my case, this pointer appeared in many different flows - the class never heard about the *[Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle)* - and in each case, it was initialized. In fact, it was simply assigned to a freshly allocated object on the heap, if there was anything assigned to the pointer before it was leaking.

It was used in a lot of flows and initialized, but it was not used in all the flows.

Obviously, the destructor was called in each case.

The peculiarity of this situation was that the only flow where it was not used was a timeout use case. We have three different timeouts and the third one is quite difficult to emulate in tests, so nobody did.

Therefore no test exercised this code and we didn't notice the problem until we hit production.

As deleting an uninitialized pointer is undefined behaviour, there is always a fair chance that the core dump will not reveal you the exact cause.

At least, it showed from which class it's coming from, that it's about a kind some destruction and in addition, in each core dump - believe me, there was many! - there was a timeout going on.

Easy-peasy, right?

It was a problem for sure. And by the time I discovered it, I had already another commit on top it, where I replaced the raw pointers by `std::unique_ptr`s.

The only problem was that we had nothing more than a hypothesis that this was the sole root cause of the core dumps as we also changed some callbacks in the same load item.

You might argue that such changes should not go together. I try not to put them in the same commit, but when you have one load per week, several commits are often packed into the next load.

What did I learn?

- Don't just double, but triple check critical changes
- It's not always worth going baby-steps. I separated the constructor simplification from the raw-pointer replacement on purpose. I wanted to be cautious. But introducing right away smart pointers would have been more cautious.
- Don't write huge classes. This problem could have been avoided if the class wouldn't have been so huge. Not every replacement class would have needed this pointer at all, and in addition smaller classes would have been easier to test.
- Test, test, test!

## Conclusion

In this mini-series I'm sharing a couple of code issues that reached production. In this first episode, I shared how undefined behaviour due to an uninitialized pointer got introduced to our software.

Next time, I'll share other 2 bugs. Stay tuned.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!

