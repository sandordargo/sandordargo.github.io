---
layout: post
title: "Sane office environment with code review guidelines"
date: 2018-3-28
category: dev
tags: [guidelines, code reviews, rules, efficiency]
header: "In my new team we are working on several guidelines, rules and process improvements. Why do we think these are so important? If things are well documented, it's easier for a newcomer to start delivering value. It reduces the possibilities to err for everyone. It removes lots of possibilities for arguments. And we all know that <a href=\"http://lesswrong.com/lw/j6o/according_to_dale_carnegie_you_cant_win_an/\">one cannot win an argument</a>, we should avoid them at all costs."
---
For a more detail vision about the importance of guidelines, please check out [this article](https://medium.com/@SandorDargo/zuckerbergs-gray-t-shirt-and-coding-guidelines-caef9079ba7e), that I'll revisit soon by the way.

This time I'm going to focus on code reviews and on the corresponding guidelines.

## The aim of the code review

Reviewing a pull request is an important and sensitive task. In my opinion, it is at least as important as writing the code. Besides, reviewing someone else's code is a not just a technical task, it's also a human one. That gives most of its delicateness.

So let me start with the most important rule that you should always have in mind whenever you start to review a pull request or whenever you open up a review you received:

_**No comment should be personal. No comment should be made about the author or the reviewer. A review always must always be about the code!**_

The aim of a code review is to make the code better, to detect bugs before merging and delivering, and in addition, to improve maintainability of a given code base.

## Items to be checked in a code review

Reviewing code is difficult, and in fact, it's a very broad task. According to my bosses, I'm considered a good code reviewer, but still, I think my effectiveness could be improved a lot. I think that following checklists, just in most of the cases, can be a huge help.

Now obviously, some of those checklists and/or tasks will be language specific. However, what makes a review a good one is most of the time the same concepts, regardless of the language used.

These lists are mostly here to give you some ideas, they are far from complete. Feel free to use them, update them, personalize them or just let them inspire you to come up with completely new ones.

I think that one reviewer shouldn't use them all, but maybe just a few. But if you have separate checklists, it's easy to share the tasks.

Please, consider that not all the checklists are there to be used for all the code reviews. If the pull request is a really small bugfix, just correcting an off-by-one in a condition, it will not require checking the design of the whole domain.

### Full process checklist

This one focuses on some foundational characteristics of a pull request. I didn't add to the list, but you should make sure that the new commits don't break the compilation or the tests. Your Continuous Integration pipeline should take care of this, but in case not. Don't forget about it. Otherwise, check these.

* Are new unit/regression tests added?
* Are there new compiler warnings?
* Does the change functionally make sense?
* Are there a lot of dependencies?
* Are the commit message clean?
* ...

### SOLID (object-oriented design) principles checklist
In order to verify the sanity of the design, it's worth to go through the SOLID principles. Most probably it's worth to expand these items, into sublists helping to verify each principle.

* Single responsibility principles
* Open/closed principle
* Liskov substitution principle
* Interface segregation principle
* Dependency inversion principle
* ...

### Security checklist
Your application might or might not be security-critical. As soon as it's hacked once or it fails because of some messy input, it will become one... This checklist if usable, should be heavily language dependent, I give you one for C++. A colleague of mine extracted mainly from this [talk on secure programming practices at the NDC Security Conference at 2018](https://www.youtube.com/watch?v=Jh0G_A7iRac)

* Is external input handled properly?
* Are C-style interfaces used?
* Is the `new` operator superfluously used instead of stack allocation?
* Are there lots of (error-prone) size calculations?
* Are pointers used a lot?
* Are shared_ptrs used a lot?
* Are there any threads?
* ...

### Testing best practices checklist
I hope we all agree that testing is part of a developer's job and if we had a discussion on testing it would be about the different way to do it, not whether we should do it or not. The bad news is that there is no one way fits for all - still I'd advise you to follow the cycle of Test Driven Development. Good news is that hopefully on a project there is a common understanding on at least what should be done. If there is none, step in and advocate for testing, gather articles, studies and convince. You'll be much more respected.
Here a few points to clarify in regards to the testing part:

* Are there enough unit tests?
* Are there enough non-regression tests?
* Do tests test one thing?
* Do they have assertions? (A test might have multiple assertions, still logically they assert one thing)
* Are they readable?
* How dates are used? (Fixed vs. generated)
* ...

### Code readability checklist
We - developers - are all authors. If we do an impeccable job, [our code will read like a prose](https://www.goodreads.com/quotes/7029841-clean-code-is-simple-and-direct-clean-code-reads-like). I don't say that always reach this goal for the whole codebase, but we should aim for that. The code reviewer has a huge responsibility here. If you are reading a pull request, please think about the following questions:

* Are names meaningful?
* Are classes/functions small enough?
* Does the code "read like a prose"?
* Is the code well-formatted?
* Is there duplicated code?
* ...

### Resource handling checklist, a.k.a. [RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization)
This last one is rather language specific. It's not only for C++, but mostly. If you are a C++ developer and you ever fought against dangling pointers, memory leaks and nasty core dumps. You know what I mean. For a non-expert, it can be really difficult to spot these issues, but following a helpful checklist might help you both in pointing out the problematic lines and both in developing the RAII expertise.

* Is object ownership clarified?
* Are objects properly destroyed/ is the memory correctly deallocated?
* Are new fields properly handled?
* Are Fields correctly initialized in the constructors?
* Are comparison operators updated?
* ...


## The code of conduct for code reviewers

As stated before, commenting on someone else's code is also a human task, be nice to your fellow developers. Here are some pieces of advice, following them will highly decrease the chance that developers cry or throw chairs to each other in the office. (Just to mention, I have never seen the latter. So far...)

### Don'ts
* Don't refer to personal traits and don't judge (e.g. refer from saying you/your code is stupid...)
* Don't make demands (at least put there a please and explain why you ask for a change)
* Don't be sarcastic, even if you are buddies, other reviewers/readers might find some comments inappropriate
* Never say never, nor always. There will always be exceptions. So treat this rule with care...
* Avoid selective ownership of the code, i.e. don't use "mine", "not mine", "yours"...

### Dos
* Ask questions.
* Ask for clarification.
* Be explicit. Remember people don't always understand your intentions online.
* Seek to understand the author's perspective.
* If discussions turn too philosophical or academic, move the discussion offline 
* Identify ways to simplify the code while still solving the problem.
* Communicate which ideas you feel strongly about and those you don't. If you just express your preference, say that it's only your preference.
* Educate. If you suggest something, share proofs why it's better. Articles, studies, books, etc.

## Rules for the authors
* Be humble and honest about the submitted code. Mistakes happen every day, the process is there to support you.
* Remember that you shouldn't take it personally. The review is of the code, not of you.
* Explain why the code exists.
* Follow guidelines.
* Seek to understand the reviewer's perspective.
* Be grateful for alternative suggestions and keep the discussion technical. Try to learn from the different perspective.

## Call for action

* Make thorough code reviews. You will learn a lot, just like your fellow developers.
* Emphasize the importance of proper code reviews in your teams and if necessary educate your colleagues how to review code.
* Check and star [this repository](https://github.com/sandordargo/code-review-guidelines) where I collected some checklists and ideas and feel free to contribute, to add what you have found important! 