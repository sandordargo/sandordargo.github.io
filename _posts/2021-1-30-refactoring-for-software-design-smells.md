---
layout: post
title: "Refactoring for Software Design Smells"
date: 2021-1-30
category: books
tags: [books, architecture, design, watercooler]
excerpt_separator: <!--more-->
---
I was looking for some books for these months taking into consideration that I should improve my architecture and design skills. So [I recently read Emergent Design](https://devreads.sandordargo.com/emergent-design/) and now [Refactoring for Software Design Smells](https://amzn.to/36rDKbh). I don't remember where I saw it recommended, but when I started to browse the book (I use O'Reilly), I saw it was recommended by the ACM fellow, Grady Booch. The person who authored [Object-Oriented Analysis and Design With Applications](https://amzn.to/2MBmvNU) and who said that code should read like well-written prose. It seemed to be a good recommendation.

Was it worth to read the book? Let's find it out together.
<!--more-->

## What is technical debt?

In the part of the book, the authors discuss what is technical debt, what kinds of technical debts are out there and what are their root causes.

### The well-known metaphor

The authors use the well-known metaphor, originally introduced by Ward Cunningham, the one that compares technical debt to financial debt.

You might invest in something that you don't have money for at a given point in time, by taking on some loans. That's possible and sometimes even desirable. But if you do so, you have to soon start paying back your debts, you have to pay the interests. If you don't do that, if you postpone paying back your dues, the amount of interest will rise. They will keep rising until a point where they become unbearable and you have to declare bankruptcy and lose your home, your investment, etc.

Technical debt is like that. You can introduce some sloppy solutions to hit the market faster, to get faster feedback, but that doesn't come for free.

You either pay back your technical debt, by cleaning up your code, by refactoring it, or the inevitable changes will be more difficult to implement. They can become so difficult that you simply wouldn't even date to touch the code.

### Four kinds of technical debt

The authors differentiate among 4 different categories of technical debt:

- code debt
- design debt
- test debt
- documentation debt

I think that test documentation debt are self-explaining categories. You either don't write tests or documentation and therefore your level of trust in the code, your level of confidence to change it will decrease (test code), or it will be simply difficult to get relevant information about the code, about the features as you lack documentation.

The line between code and design bet can be thin, but in general code debt, code smells refer to much lower-level problems such as how you write a loop, how your functions are structured within a class, while design debt is more about how classes and even modules interact with each other. 

The book focuses on the design debt, the design smells and how to remove them from a technical point of view.

### Why do we have technical debt?

There are different reasons why technical debt is introduced. And just like for financial debt, there can be good and wrong reasons.

One cause is __schedule pressure__. If it's constant and happens all the time, then it's obviously bad and most probably you won't take the time to fix the introduced smells. But from time to time, it can be crucial to get out to the market as fast as possible, and you can win big with it.

The other three reasons identified by the authors can hardly be categorized as good reasons. It's simply bad people or bad technical management:

- lack of good/skilled designers 
- lack of application design principles
- lack of awareness of design smells and refactoring

Management either doesn't hire the right people for the job / the composition of the team is not a good match for the problems to tackle.

This book teaches about the principles of good design and makes the readers aware of the relevant design smalls and the available refactoring techniques. Though if you need more details on the refactoring techniques, you'll have to follow up with other sources.

## Design smells

Before jumping into the details of the many different design smells, the authors lay down the foundations by stating what they mean by design smells and what characteristics they have.

> "Design smells are certain structures in the design that indicate violation of fundamental design principles and negatively impact design quality."

### Affected attributes of a software

Design smells can hurt software in many different ways. In the [book](https://amzn.to/36rDKbh), there are seven quality attributes listed.

- Understandability: how easy it is to understand the design of an application?
- Changeability: how easy it is to change the functionality of an application, or will it just break at unexpected places? 
- Extensibility: in case of the need for new functionality arises, will it be easy to implement or...?
- Reusability: can you reuse some parts of the software at other places?
- Testability: is the design welcoming for tests or will it be a difficult challenge resulting that people will eventually stop testing?
- Reliability: with the design in place is easy to implement the requirements correctly? Will it help to avoid runtime issues?

Later in the book, for each design smell, it's studies which quality attributes they affect.

### What are the main design principles?

The authors enumerate 4 design principles and they use those to categorize the smells too:

- Abstraction: eliminate the details and generalize
- Encapsulation: hide the details
- Modularization:  Abstractions should be cohesive and loosely coupled
- Hierarchy: organize abstractions in clear hierarchical structures

### Why are the principles violated?

Sometimes, _we simply don't know about them_ so we cannot take them into consideration. Other times, _we misunderstand them_. We try to create something according to a pattern we read about, but we lack the experience and we implement it in an incorrect way.

Often it happens that _we don't think in an object-oriented way_ and we come up with implementations that are procedural. It might be because we worked in a language where that was enforced and _there were strong language limitations_ preventing the devs from using modern designs. 

Or we simply don't care for any reason...

## So many design smells

In the biggest part of the book, you will find 25 design smells discussed in details. The scope of this book review obviously doesn't let me introduce you all of them, so I picked 3 design smells that I want to mention.

### Imperative Abstraction

I choose this one because that's something I regularly do myself. I turn operations into classes.

You can recognize this smell when there is only one - public - method defined within the class. Even worse, the class name and the method name might be the same. In C++, sometimes I just override the `operator()` to get rid of this problem.

As a solution, the authors suggest to move such an operation to the abstraction that uses it, often there is only one user of this class/method.

By that, the design becomes cleaner and less complex.

When I offend this role, it's often because I have to fix some legacy code and I have no time, to do major a refactoring, but I want to extract some behaviour, in fact, an operation, so that I can easily test it.

I understand it would be better to extract the data along with the behaviour so that they constitute a cohesive entity, even though it takes more effort to refactor that way.

### Multifaceted Abstraction

This is something that happens a lot. We see it everywhere and we also contribute with our fair share to worsen the situation.

A multifaceted abstraction has a too wide API, it has too many responsibilities. In fact, it's an offence against the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle).

The authors bring many examples, even from the core of Java SDK libraries.

The solution is quite straightforward, but of course, not so easy to carry out all the time: extract classes, divide them into smaller, cohesive abstractions.

### Deficient Encapsulation

Do you automatically create a pair of getters and setters for the members of a class? Some IDEs will even help you do that.

With that, you'll leak many implementation details. Details that the clients of the class should not depend on.

The problem is that it's quite difficult to change the situation. You might also know this as [Hyrum's law](https://www.hyrumslaw.com/):

> With a sufficient number of users of an API,
it does not matter what you promise in the contract:
all observable behaviours of your system
will be depended on by somebody. 

Often, it's compelling to make internal methods publicly accessible for the sake of testability, but if you have to do that, it only means that your design is bad.

You have to take into consideration the effects. Often, if you work in an internal codebase, it will be an acceptable compromise. You expose a method, you make it testable and you know that the external world will have no access to it. But if you'd it a library used by many, you should really look for another solution.

## Conclusion

In [Refactoring for Software Design Smells](https://amzn.to/36rDKbh) the authors explain what 4 design principles should we use to achieve a high-quality design and what are the main reasons behind failure to achieve it.

Then they analyze 25 design smells categorized by the 4 design principles. For each we read about how they appear, what quality attributes they hurt and what can we do about them.

If you are looking for levelling up your software design knowledge, I think this book is worth to read. You'll realize how many smells you also introduce to your code with the best intention and you'll also get some recipes on how to avoid them. 

Happy reading!