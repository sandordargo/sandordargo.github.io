---
layout: post
title: "Which Is Harder To Find?"
date: 2018-4-25
category: stoic
tags: [philosophy, stoic]
header: "\"In my experience, men who respond to good fortune with modesty are harder to find than those who face adversity with courage.\" - said Cyrus the king of Persia."
---
In the field of software development, you will find more people who like building things from scratch than people who like to work on legacy code. Yet, you will find more developers who will actually work on maintaining code than programmers who build brand new applications. At least that’s my perception. I work in a big corporation so that’s kind of expected. Though, it might be different at smaller companies.

Anyway. Does this tell us anything about how people work on whatever type of projects they have? Not at all.

When you work on legacy code, you will find the following types of developers:

* _The copy-pasters_. They are afraid. They are afraid of failure. They are afraid of peer pressure or their bosses. Maybe they just don’t care. They will do the least effort possible to fix an issue. If it means that they copy the same two lines to dozens of functions because they don’t want to create a new one, they will do it. They might break things because sometimes they repeat the wrong lines of code at the wrong place. But most likely they will rarely break your application. They optimized the way they work to avoid immediate software failures. What is sure, that thanks to them the code will become less and less maintainable, they will let the code rot.

* Let’s call the second group _apprentices_. They are brave, but they lack experience, even knowledge sometimes, but the apprentices want to improve themselves and the code base they work on. Apprentices will extract till they drop if needed, they will create new functions, they will dare to reorganize code. They will add tests whenever they can. As they lack experience, they might not test the good thing. They might break the code, but they recognize that shit happens and the only people who don’t break things from time to time are those who don’t do anything. Depending on how persistent they are and what kind of environment they work in, they might end-up copy-pasters hating their own work, or they become masters.

* The _masters_ learnt to love working on a crap codebase in a supportive environment. They see big opportunities in turning shit not necessarily to gold, but to fertile soil. Masters will try to hold back the copy-pasters’ pull requests - almost every team has some of them. They will make serious efforts to educate them, to educate bosses, to educate anyone, especially the brave padawans. If they face too much negligence, they will eventually go to another team where change is welcome. When you really need them, they will appear because they look for a welcoming environment.

Now you might wonder, okay, here is this arbitrary and incomplete classification, but in the end who work on greenfield projects?

Of course the same type of people! 

Is the distribution of these developers different? 

I think so. In my opinion, it’s less likely that people will become masters working solely on greenfield projects. But why?

You will find the same copy-pasters and people who just don’t care about their craft. You will also find the same apprentices, who want to become better by learning and educating themselves.

The problem frequently is that in our fast-paced world these courageous developers with good intentions will not stay on the same code-base for a period of time that is long enough to see the effects of what they delivered.

It’s the next set of developers who will have to deal with the code they considered awesome. It’s the new developers who will have to face the memory leaks, the core dumps, the rigid structure, the premature optimization, the inadequate tests.

By the time these issues come up, the authors of that code will already work on the next project producing most probably better, but still suboptimal code quality. With a few notable exceptions of course. Keep in mind that this somewhat questionable code quality is still way better than what is produced by the copy-pasters.

Someone told a friend of mine who was looking for a new team that “writing maintainable code is nothing one can learn easily from a book or a training course at the corporation or the university. The best way to learn about it is to do it on an application that creates already value and to teach it to colleagues. ”

In his experience, men who respond to greenfield projects with careful design and with vigilant coding are harder to find than those who face legacy codebases equipped with proper refactoring techniques that they are willing to use with a whole heart.
