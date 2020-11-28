---
layout: post
title: "Emergent Design: The Evolutionary Nature of Professional Software Development by Scott Bain"
date: 2020-11-28
category: dev
tags: [architecture, design, books, patterns, designpatterns]
excerpt_separator: <!--more-->
---
[Emergent Design](https://amzn.to/3pSFGSf) was published in 2008, but even after 12 years, I still found it extremely relevant in 2020. In the first part of the book, the author Scott Bain discusses about software development as a profession and what an occupation needs in order to be considered as a profession.
<!--more-->

Later he explains how patterns can help us in our jobs and how we should not feel discouraged by them even if they seem like a lot of boilerplate.

At the later chapters, he details some of the practices, principles and disciplines we might follow and how we sometimes misinterpret them.

While all the three are important, I was in particular interested in the parts where our profession is inspected. 

Do we have a profession after all?

## Software development as a profession

According to the author, software development should be considered and treated as a profession. Whatever organizers of few-weeks-long bootcamps claim, it requires extensive training and experience to do development well - that can come from other sources than formal education(!), but it's not a matter of weeks or a few months - and it's indeed a complex activity.

We have our own specialized language, have some practices and traditions, though as software development is something very new compared to law or medicine, we still have a lot to do in order to consider software development as a _well-established_ profession.

### Wash your hands and test your code

Wherever you start working as a doctor, you can be sure that certain practices will be expected and not challenged. It's out of question whether you should wash your hands before an operation. Doctors, lawyers, accountants have to pass trainings, get licenses in order to practice and follow various rules.

### Getting regulated?

In software development, you still can get into places where people will seriously doubt whether a developer should write tests or not. Even if lives depend on their products.

Should our profession be regulated, should it require more licences and so forth? We'll see what the future brings, but if you have a look at other professions, that's not an idea from the evil itself.

## Patterns are useful, but not the ultimate solution

Have you read [The Book from the Gang of Four](https://amzn.to/36VKyO2)? The classical book about design patterns? If so, I'm sure you admit that it's a great work and it's an important milestone. You also might felt a bit intimidated about it. Some patterns are pretty complex and they come with a lot of boilerplate.

### Sometimes patterns are overkill

Many of us after reading the book just want to implement some of these patterns. We don't care about whether it's the right place to use it, but we just want a factory here, a visitor there.

The inexperienced might not understand the code and the more seniors are horrified for a different reason. We looked for a foot for the shoe, not the other way around. And the foot is lost in the huge boot...

It doesn't have to be that way.

### Yet, often they bring some clarity

While certain out their fight a strange war to cancel patterns, they are important items in our toolbox. After our code hits a level of complexity, patterns do simplify the application, they bring a clearer architecture and mostly they do this by clearly communicating intentions.

Patterns are tools that many learnt, many understand and if these programmers see a visitor or a strategy pattern implemented, they understand what problems they are there to solve.

Often, it's not worth to start with them as they introduce a lot of boilerplate code. In the early stages of development when we are not even sure in which direction to code should evolve, it'd be too early to introduce some of the patterns. Instead of flexibility, they would give rigidity. But it doesn't mean either that we should throw them away or that we should implement them all at once when we are almost finished with our big bowl of spaghetti (code).

### We can get there little by little

The author proposes that little by little as the application evolves, architecture could emerge from something rudimentary towards well-known patterns through different steps. 

In his book, he presents different _"evolutionary paths"_. One of the simplest forms is moving towards a _[strategy pattern](https://refactoring.guru/design-patterns/strategy)_.

First, there is one behaviour and when a new one is introduced, construction is encapsulated and latter with additional behaviours, you implement the strategy pattern. But by that point, you already have the object construction encapsulated.

For more paths and explanation check out [Emergent Design](https://amzn.to/3pSFGSf).

## Practices Driven Development

"One of the values of a profession (or a craft, for that matter) is that it defines a set of practices that you can follow, practices that will reliably increase your success rate." Again, think about the doctors washing their hands. That's a simple practice that hugely increased the chance to succeed, the chance of survival.

### Automatic, significant and no extra effort

But what are the most valuable practices? They should be things that you can always do without having to decide, they should be almost automatic actions from your side. They should be significant yet at the same time require little or no extra work at all.

The author speaks about 4 practices he considers valuable:
- Performing commonality-variability analysis
- Encapsulating the constructor. _(This is something I think is debatable, but not in scope for the article.)_
- Programming by intention
- Using a consistent coding style. _(I spoke about this topic more [here](https://www.youtube.com/watch?v=CNDejB6Hg5A))_.

### Understand even if you don't go for something

Before wrapping up the book, the author writes about disciplines, such as unit testing, refactoring or TDD. 

He finishes the chapter on TDD, with a very important thought: **nobody can tell you what to think!** 

But, even if you don't think TDD is something you should follow, you can still get some value out of it, by knowing how tests work, what characteristics do testable classes have. 

And if you think beyond the very topic, it conveys a very important message. Even if you don't agree with someone or something, even if you think it's not for you, you'd better understand it well. Understand people, concepts, better than their supporters. It will give you both clarity on why you are against something (or maybe you will become a supporter yourself), and besides you will get a handle on the advantages of the techniques you dislike.  

## Conclusion

I found [Emergent Design](https://amzn.to/3pSFGSf) an extremely valuable book. The thoughts on software development as a profession are still ideas we have to discuss and in fact, what comes later in the book are the very items that might define our common practices just double booking keeping for accountants and - among many others -  washing their hands for doctors. 