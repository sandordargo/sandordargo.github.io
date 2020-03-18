---
layout: post
title: "The seniority trap"
date: 2020-3-18
category: dev
tags: [watercooler, career]
excerpt_separator: <!--more-->
---
Maybe you have been there, maybe you'll get into a similar situation soon. Maybe you don't work for the type of company where this could ever happen. Yet, most of us at least heard about highly knowledgeable, so-called senior developers who spend most of their time outside coding. Let's put it in another way, they go to the office and they don't do what they are best in.
<!--more-->

Some bosses could ironically say oh, you haven’t coded in 3 weeks, 2 days and 6 hours, Mr. Senior Developer? What a pity! You have more important things to do than being so geeky! By the way, have you already sent those TPS reports?

<div class="video-container">
<iframe src="https://www.youtube-nocookie.com/embed/Fy3rjQGc6lA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

You can reach this point in different ways.

* You looked for an opportunity to code less
* You were told that this is the way to get promoted
* It just happened to you

Let's have a quick look at those paths.

## You looked for an opportunity to code less

Maybe you’re attracted to non-programming activities because you find coding boring as you never learned it well. Or because you learned it too well and you wanted to get a break. It’s fine, you wanted it.

A funnier combination can be that you think you learned coding too well, but actually you did not. That can be dangerous because you will have false expectations regarding good engineers in terms of what it takes to deliver quality.

Once I was working on a bug in a relatively new system. It was the case of a complete rewrite of a mainstream application into something more modern. It didn't go so well as management only hired inexperienced people instead of making sure there is a desired healthy mixture of seniors and juniors. In addition, the project was always late and management just pushed, pushed, pushed for the delivery.

A tiny part of the end result was a 600 lines long function consisting full of complicated, sometimes contradicting, sometimes completely duplicated if statements.

Specs? There were, we managed to find them, but they were not aligned with the code.

Somebody already found a bug a couple of months ago, but he didn’t want to change this monster. So what did he do? He wrapped the function into another pile of ifs. Not surprisingly they were also buggy. Soon he got fed up and left the company.

So what could I do?

I wrapped the wrapped function into another one. 

Of course, not. I would have left before doing that.

I went down to the dungeon, extracted what I could into different classes and into a lot of small and - at least I hope - well-named functions. I unit tested everything and I also documented all the deviations from the specs.

When you see that the specs differ from what is implemented you have two choices. You fix the code risking that you change behaviour that users rely on. It can be dangerous. The safer option is that you separate the code and document it - preferably through unit tests.

This takes time.

A few days later the manager overseeing my activities came to my desk and gently asked about the status. I explained and showed him what I found. I was told that the product manager was complaining about the time it takes me to fix the bug, she used to be a developer too and she was sure it couldn’t take that much.

I stopped. I turned in my chair, facing George, the manager. I looked at his eyes and said:

_Maybe that’s why she’s not a developer anymore._

He tried his best to hide his smile and told me that he got the point. He won’t transmit what I said, he will just take care of her.

To me, she is the perfect example of someone who thinks she was too good but actually wasn't.  The whole team had similar problems with her. She was always looking for fast solutions, letting our codebase down. Our developers were too young and shy to push back. Looking at her code written years ago... You know, git never forgets... It was her way of delivering code. Luckily she changed positions.

## You were told that this is the way to get promoted

The second possibility why you might have ended up in the seniority trap of not coding so much is quite simple and straightforward. You were explicitly told that it is the way to get promoted to a senior developer and to be paid better.

People are motivated by fancy swags, bean bags, interesting technologies, and iMacs. Not by a good salary. 

What a peck of bullshit.

People do work for money and most of us will eventually feel bad if we are underpaid. Most employers are shortsighted and prefer to pay people just enough that they don’t leave immediately. Many people will ask at a certain point what they have to do for more money.

Hopefully, you will not be given a gun or asked something obscene.

Thinking more about it, maybe those wouldn't be the worst options.

You will be told that you have to coordinate this project, lead the activities on that stream and you might get recognized. In other words, you have to do the job of a (project) manager without getting paid like one. But you have a not so strong promise that one day if you do it long and well enough, the constellation of the stars are right and you satisfied all the gods of all religions, you might get a bit less than your manager, a little bit more than now.

Let’s be fair. You’ll learn important soft skills along the way and you can experience a bit how life would be on the dark side.

It is useful, the main problem is that it is still the only way forward in many corporations.

The good thing about this second option is that it’s not a surprise. It’s exactly what is promised. You are given the two pills and it’s up to you to decide. 

## It just happened to you

It’s not the case for the last option when it just happens. You just accidentally walk into the seniority trap without noticing it before it's too late.

If you are a young and keen software engineer with a bit of a character and not very impatient regarding your promotions and raises, your job will change little by little until you are caught in the trap.

You will learn to code better and better, you will probably have ideas on how to be more efficient and how others should be more productive too.

Management will be happy at first, but soon you will realize that they are not that much interested in doing anything in order to help you implement those ideas. Unless you managed to sell your idea as theirs. But probably you are not so canny yet at this point in your career.

So if you really want a better life, you will have to take the lead.

Once you did that and you didn’t completely mess up everything, the next similar tasks will find you fast.

You will be asked to coordinate this activity, make a presentation there, be the ear on that other meeting, etc. 

Soon you will realize that you haven’t coded in a while because you are only coordinating projects and do the reporting for your boss, who thinks he is great as he delegated so much of his tasks to his unsuspecting subordinates.

The end result is pretty much the same in all the three cases. You’ll end up as a senior developer who barely codes. You finish almost as uncle Bob’s famous non-coding architect, just with a worse salary.

## The way out

So what is the solution? 

It depends on what you want. 

The most important thing, like always in your life, you have to know what you want.

If what motivates you is leaving programming while staying in IT, don’t go this way. Just move forward and do the necessary to become a manager of developers or a product owner, something like that. There is nothing wrong with that. Just don’t be the developer who barely codes, does it in a degrading quality and doesn’t make as much money as she could as a real manager. 

If you know what you want, better to be single-minded on it and skip a few steps on the ladder, just let your management know and go for it.

What if you like programming and you don’t want to end up in a non-coding developer position? 

If you are in a corporation where a developer’s advancement is not tied to coding but to other activities... well... it’s not easy, but you must keep focused.

Don’t forget that you’re a developer and you want to become a better one.

Do the needful to keep your manager happy, but try to look for side activities that help you become technically better.

Don’t get me wrong, soft skills are important and it’s also useful to know how to use Excel, but that’s not your main goal.

In my case, helpful side-activities were things like organizing coding-dojos, knowledge sharing sessions and helping ramp up our in-house technical mentoring program.

If certain activities you would be interested in are not ongoing yet, you have to check if they would be welcome. If so, go for them. If you’ll be the one who will start those activities in your department or in the whole corporation, the possible rewards are even higher.

Keep one thing in mind. Whatever you do, you have to communicate your priorities to your boss. You have to say no. Frequently. That will be your most important skill. To refuse opportunities gently, but firmly. Focus on your essentials. 

If your priority is, for example, to keep C++ coding in at least half of your time, don’t accept a project with an undefined length where you’d be writing Ansible scripts, even if you find it interesting.

If you really cannot bear with this, I understand. Maybe you have to specialize in a niche and work as a consultant, or simply work for smaller companies where these ladders are not created yet and where developers can focus on development.

At least that's what you think. Don’t forget that in smaller companies, often you have to do many things that are done by separate departments in a huge corporation. Like maintaining your servers, or the cloud, or a VM within another VM. Whatever.

Again, it’s not a problem if you have to do those things. You just have to understand your needs and the available constraints.

You can also reject all these activities leading to fancier titles and higher salaries and focus on becoming a better programmer. You won’t be a star in the eyes of your management and you won’t get hefty raises, but by the time you’ll be a brilliant developer.

Is it worth it? It is, but maybe sooner or later you’ll have to change jobs. While we don’t work only for money, we don’t like to be underpaid either, right?

## Conclusion
Today we saw a few different ways to end up in a senior developer position where you barely code. I call this the seniority trap.
What to do if you are in a trap? 
Think about what you want. Then change your official position if that's what you want, keep doing it to get the promotion or just simply tell your management what you like and what you don't like, what are your goals and dreams.

After all, we speak about your goals and time, not about your boss’. You are the one responsible for your career, not anyone else. You will have to accept some trade-offs, that’s fine. But things should not just happen to you. You should understand what’s going on and what is under your control. 

Do everything that you can, share your thoughts and needs, work hard and accept whatever you cannot change.