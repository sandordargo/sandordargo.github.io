---
layout: post
title: "Ramping up a new team: let's mob"
date: 2017-11-1
category: dev
tags: [efficiency, office practices, tips, mob programming, pair programming]
header: "Now that I've finished a five-month-long start-up like project, it's time for some reflections. Okay, I'll keep most of those for myself, but there is one thing I want to discuss in public."
---
In our team of eight, there were six of us with a developer background. But the background is one thing and experience/knowledge is yet another. We arrived with very different experiences language-wise, mentality-wise, etc.

For this project our main language was Java, luckily Java 8 which enabled us [using its functional features](/blog/2017/09/27/railway_oriented_programming). However, two of our developers never worked in Java before. One didn't even develop brand new code in previous assignments, but only maintenance. This used-to-be maintenance developer told us that in that team they were encouraged to just make the code work, better with copy-paste, than with some refactoring. Because refactoring when the original code is not tested might break things. And breaking things was seen as a shame there. It doesn't sound like a place where I would be happy to spend more than a few seconds...

On the other hand, we also had more medior people and a real expert too. How did we manage to work together? We ended up collaborating pretty well! I think we did two important things. One mostly in the beginning and another one during the whole project.

#### Mob and pair programming

These tools of extreme programming were really useful during the first few sprints. I have to admit that we didn't do mob programming according to the book most of the time. We forgot to change roles frequently enough. But still, this was the single most useful practice to balance the difference between people's knowledge. It was a great way to share knowledge about [Spring](https://projects.spring.io/spring-framework/), unit testing, test driven development. Much better than doing a meeting and consider the "training" done.

Its disadvantage is that it's more expensive in the beginning and some more experienced people might get bored sometimes. But everyone has to understand that it is more useful to have people afterward who can work on their own. Besides educating less experienced developers should be everyone's obligation.

#### Code reviews

I worked in teams where code reviews were important for quality control and knowledge sharing. I also worked in teams where code reviews were just there to approve them as soon as it's opened and then merge the pull request. 

In this team from the very beginning, we emphasized a lot on the importance of quality code reviews. Not everyone treated it on the same level, but everyone considered important to read pull requests and comment on them both for tiny typos and for bigger desired design changes. 

One of our more experienced guys said that he never worked in a team, where code reviews were so important for the people. What a great compliment to the team!

What I particularly liked were the _"Work in progress"_ pull requests for bigger changes or for introducing non-trivial features. Having an open pull request from the very beginning lets the people immediately see what directions you are taking and they can comment on it when a change is still cheap rather than just saying at the end of a big check-in that _hey, you should have done it differently, but now we don't have enough time to change_.

#### Meetings

Yes, meetings. I hate them. [Most of them are just a waste](http://sandordargo.com/blog/2017/07/20/time-waster-meetings), a nice way to rob money from your company.

But sometimes we saw some really interesting ideas in pull requests which we didn't understand right away, but still, we felt it made sense. In such cases, we asked the author to schedule a quick meeting and introduce us the concept used in that given change. These were the most useful meetings of the project. For example, it helped me to learn about [Railway Oriented Programming](/blog/2017/09/27/railway_oriented_programming).

To sum up, I think mob and pair programming are really good tools to form a new team out of developers with a heterogeneous skill set. Code reviews should be part of any team's daily practices no matter what. It's almost as important as automated unit testing in catching bugs as early as possible.