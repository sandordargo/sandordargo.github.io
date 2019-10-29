---
layout: post
title: "Keep calm and do less"
date: 2019-10-30
category: stoic
tags: [philosophy, stoic]
excerpt_separator: 
---
The last one of the [Five good emperors](https://en.wikipedia.org/wiki/Nerva%E2%80%93Antonine_dynasty#Five_Good_Emperors), Marcus Aurelius had a note in his [Meditations](https://amzn.to/2K2gt2L): _"If you seek tranquillity, do less."_ This is one of his thoughts that is the most applicable to software development.
<!--more-->
How many times did you break the code and had you no idea what went wrong? How many changes did you introduce? Would it have been different if you had introduced only one change?

"If you seek tranquility, do less", do only one change at a time. We can see this pattern by looking at different best practices. 

Test-Driven Development teaches you the same thing. You either write a small new test, you implement a tiny new feature to pass the test you just wrote, or you refactor your code while keeping your tests green. You do only one thing, but you do that one thing well. You keep your steps small and even if you break your code, you have no problem. You must know at which step you broke it and even if you don't manage to figure it out, you can go back to your previous commit by not losing much.

So regarding commits. Keep them small, but well described. Don't mix refactoring the legacy code and implementing a new feature in one commit. If something goes wrong, you will have hard times to localize the root cause. Keep your commits small and responsible for one thing.

Think about the Single Responsibility Principle. Think about high cohesion. A method or a function should not do much. It should do, it should be responsible for only one thing. Look at your code, most probably the majority of your units should do less. By having small classes with clear responsibilities, you will gain tranquility on the run as your code will adapt easier to changes.

Think about pull requests and code reviews. Let's stay with the previous example. If you create a pull request with refactoring old code and with the implementation of a new feature in it, your reviewers will have hard times to understand. Been there, done that. It might be difficult for them to understand which changes were made the code more flexible and which parts are for the change itself. 

Why not creating the pull requests? 

One for the refactoring and once it's merged, one for the change itself? Both your reviewers and your clients will be grateful thanks to more understandable changes and less shipped bugs.

Keep calm. Think big. Make small steps. Do one thing at a time. It will keep you calm.
