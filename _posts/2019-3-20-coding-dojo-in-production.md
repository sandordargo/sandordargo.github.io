---
layout: post
title: "Production code as a playground?"
date: 2019-3-20
category: dev
tags: [dojo, self-learning]
excerpt_separator: <!--more-->
---
With one of my colleagues, we've been working a lot to create and foster a culture of learning, an environment where constant self-improvement and knowledge sharing is highly valued. Let's say that we are part-time developer advocates. We've achieved some of our goals, but there is still a long way ahead of us. The _"us"_ applies both to the developers and the management.
<!--more-->

Recently at a team retrospective, two people complained a bit that they don't find the dojos challenging enough. They also cannot apply the knowledge to our code base, because of the massive technical debt and architectural deficiencies we own. They even proposed to work on production code so that we have time to refactor it, to do something that we don't have time for otherwise.

These are interesting points, let's dive deep!

## The katas are not challenging enough

Code katas are made for practice. They exist in a lot of different levels of difficulty. I am considered one of the more senior developers on the team, but I don't find them too easy. There is always a place for practice, or for trying out different approaches. I admit, sometimes we work on the easier katas. I believe they can still teach you more even if you are senior. Besides, it's not only katas that are heterogeneous. People too. We have a lot of junior developers - and [that's a good thing](https://dev.to/isaacandsuch/if-you-dont-hire-juniors-you-dont-deserve-seniors-48kb). We cannot immediately or always work on the most difficult katas.

When I started to attend coding dojos, I was not junior per se anymore, yet I really enjoyed working on the simpler katas with some of the best devs I know in the company. Every time, I learnt so much from them. I am really grateful for this, Christian, Cyrill, Gary, just to name a few of you - and for Alessandro who made it happen!

## We cannot apply the knowledge

There are different possibilities here. What can be the reason for me not being able to use the knowledge I got?

### I'm not allowed to

I've been there as a database admin. Due to strict processes, many features were simply not allowed to use. But here it is not the case. Nobody forbids you the usage of the [STL](https://dev.to/sandordargo/the-big-stl-algorithms-tutorial-introduction-295a-temp-slug-7151635) and [lambdas](https://dev.to/sandordargo/lambda-expressions-in-c-4pj4) for example. Nobody forbids you to clean up a little bit the code base that you have work on. If you claim that you don't have time to refactor, think about it again. It's not necessarily about having months of time to refactor a whole application. It can also be about just cleaning up the surrounding lines or the function, God forbid the enclosing class.

Baby steps. Baby steps, Luke.

### Lack of knowledge

This is more an option. There are different levels of knowledge. You might think you understand something and you can maybe solve the prepared exercises. And then you receive a problem you haven't seen before and you have no idea how to make the first steps. Do you remember from your studies? I do! Was that a problem with the exercise? Of course! Not! It was me who was not prepared enough.

What could have been the solution? Let me pass and send me to work on real-world issues. Well. Maybe not. I was sent back to study more and prepare better.

So maybe, if we cannot apply the knowledge who think we have on production code, we should try to understand better the techniques we want to apply.

## Let's work on production code

Myself, I haven't tried to organize coding dojos on production code. I don't consider myself an exceptionally smart person, yet I try to learn from others' mistakes. And according to some of the more senior devs I know, it didn't end up well. Trying to learn something completely new on the job such as testing, refactoring, TDD, etc, is very difficult.

There are many reasons for that, let's pick one here.

The goal of refactoring production code is to succeed in making the existing code base better. On the other hand, doing a (refactoring) kata is not about succeeding. Its goal is to learn and you learn the most by failure. Briefly, while you refactor production code, you are expected to deliver. When you are at a dojo, you expected to fail and learn. These goals are quite contradictory.

![Refactoring horror]({{ site.baseurl }}/assets/img/refactor-man.png "Refactoring horror")

## How to move forward

Even though, we - the organizers - don't agree that dojos should happen on production code if there are multiple voices complaining. we must change something. Either the structure of our dojos, the communication, the education or everything a little bit.

Most probably we are going to keep the dojos as-is, maybe having more _"advanced"_ dojos. Besides at our knowledge sharing sessions we are going to focus more on teaching the different techniques described by Martin Fowler in his book [Refactoring](https://amzn.to/2WwtOa6) and by Michael Feathers in [Working Effectively with Legacy Code](https://amzn.to/2sPgdwM). Probably we should take the time to find some examples for each in our production code and show how it is possible to apply those techniques in our code. It will take much more time, but as long as management doesn't complain, we should do it. And even if they complain. We should all understand that simply by delivering new code all the time without thinking much about how we do it, we will not become better in what we do.