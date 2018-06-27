---
layout: post
title: "Essential Skills for the Agile Developer by Alan Shalloway et al."
date: 2018-6-27
category: books
tags: [books, agile, design]
header: "The subjects covered in this book stand somewhere between the topics of <a href=\"https://amzn.to/2qSewxm\">Code Complete</a> and <a href=\"/blog/2018/06/13/architecture-patterns\">Patterns of Enterprise Application Architecture</a>. Maybe if I wanted to compare it to another book I shall choose Clean Architecture. However, as I don't consider myself an authority to judge, I will not compare them."
---
I only wanted to mention these books so that you can see what is covered in [Essential Skills for the Agile Developer](https://amzn.to/2qSbnxo). It will not go into low-level details about what is the best way to write a for loop, but it will not cover design patterns in details either. On the other hand, it will give you some practical pieces of advice, some guidelines to follow in order to design your application, your classes and even methods. Or when to reach out for the GoF book.

In the foreword of the book, it's written that "somewhere along the line, agile methods stopped including technical practices. Fortunately, they are coming back." I wish it was true, but at least, this thought has a promise of a good understanding of agile by the authors. I think they fulfilled their promise.

They open with and put a great emphasis throughout the book on __[programming by intention](https://en.wikipedia.org/wiki/Intentional_programming)__. You might wonder what that is. Let's say it's a mindful way to write your code. You should always think about what you want to achieve with your code. You should always keep in mind how you want to test your code, how you want to validate its behaviour. If you code by intention, you will not just accidentally end up with long functions and classes that do so many things and are impossible to test. Instead, you will write something that we call nowadays clean code.

To achieve clean code, you need tests. And yes, the authors write a lot about the importance of test-driven development and in general the importance of defining your acceptance tests before you implement the application.

This book also helps to understand such trivial concepts, like encapsulation, cohesion and coupling.

It also treats in detail a technique to design your domain which is Commonality and Variability Analysis, that I'm going to cover in another article.

Even if it is not my favourite book about design, it is a valuable one which is worth its hours of reading. If you think you need some extra knowledge or some refreshments of the above topics, don't hesitate to grab and read [this book](https://amzn.to/2qSbnxo).
