---
layout: post
title: "Real-World Bug Hunting by Peter Yarowski"
date: 2020-4-15
category: books
tags: [books, hacking, security]
excerpt_separator: <!--more-->
---
I hold the role of a White Hat in an organization where being a white-hat doesn't imply that you an expert in security matters. It's more about coordinating software security-related matters.  It's a bit like being a manager - as far as I understand what it is like being a manager - you should understand what your team does, but you don't have to be an expert yourself. There might be teams where it's definitely not the case, but I know about other corporations where they nominate managers in a way that they can't even help with the daily job even if they wanted to.
<!--more-->

That's the situation I got into almost two years ago and it's not really for my taste. So little by little with the activities, I coordinate and preparing knowledge sharing sessions, I'm educating myself on how to find, recognize and exploit vulnerabilities.

The next step in my quest was reading [Real-World Bug Hunting](https://amzn.to/3aS0Ic9) by [Peter Yarowski](https://twitter.com/yaworsk).

Based on the title you could think it's about debugging, software testing, quality assurance among others. It's about hacking.

The book is divided into 20 chapters. There are three chapters not dedicated to a specific type of vulnerability, they cover some basic knowledge, how to find bug bounty programs (where you can attack websites without any possible bad consequences as long as you respect the rules) and how to write proper bug reports.

The other 17 chapters are dedicated to specific vulnerabilities. If you know the [OWASP Top 10 Project](https://owasp.org/www-project-top-ten/) you will be familiar with most of them.

Then each chapter is divided into 3 parts. There is a short introduction, then the author explains the vulnerability more in detail, the different subtypes, some technical details, etc. The third part is the longest and probably the most interesting one.

There are a couple of case studies of real-world reported and accepted bugs. When applicable you have a link to the original report and the amount of the payout. But obviously that is just the beginning, then the author goes into detail how the bug was found and how you can apply that knowledge when you are on the hunt.

I enjoyed reading this book. While it's highly technical, due to a lot of real-world examples and a bit of the human part of the bug bounties it became something very pleasant to read. It also proved to be an extremely useful material for creating content for my knowledge-sharing sessions.

I just want to emphasize one thing. Breaking applications, and see the results can be cool. But you must be aware of the consequences. Do you remember or have you ever heard about the [Myspace Samy Worm](https://en.wikipedia.org/wiki/Samy_(computer_worm))? A guy called [Samy Kamkar](https://en.wikipedia.org/wiki/Samy_Kamkar) became famous by exploiting a stored Cross-Site Scripting vulnerability of Myspace. He managed to store Javascript payload on his profile and whenever another logged-in user visited his profile, the viewer became Samy's friend, and updated his profile to do the same in addition to display the text: "but most of all, samy is my hero".

Samy became not just the hacked users' hero but also attracted the attention of the US Secret Servies and he was sentenced to 20 000 USD fine, 3 years probation and 720 hours of community service.

And he didn't even do anything harmful to the hacked users.

[Real-World Bug Hunting](https://amzn.to/3aS0Ic9) explains the technical details not just of this attack, but at least of another 50. I highly recommend you read it if you are interested in hacking.

But before you hack, think about the consequences and consider bug bounty programs.

Happy Hacking!