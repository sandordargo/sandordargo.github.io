---
layout: post
title: "The most important git practice"
date: 2020-12-2
category: dev
tags: [git, bestpractice, productivity]
excerpt_separator: <!--more-->
---
What do you think the most important git practice is?

## What is a practice?

But before you answer it, tell me what a practice is? How do you define the most valuable practices? What characteristics do they hold?
<!--more-->

The definition I use is borrowed from [Emergent Design by Scott Bain](). A most valuable practice carries 3 main attributes:

- It is something we can always do, we don't really have to make decisions whether we should do it or not.
- It is significant, something that helps a lot.
- It requires little or no extra work at all. We want the good, without the bad. We are greedy and we want the good without willing to pay a lot for it. Of course, there is always a cost, but the price of the most valuable practices will be negligible.

So what is the most valuable git practice that you can do without any extra effort, that will give you significant benefit and something that you can do without any thinking? 

Have you ever kept multiple clones of your fork? 

Have you ever created multiple forks of the same repository?

If you answered yes to any of the last two questions, you might have already learnt that you should...

## Keep your main branch clean!

Never work on master, or whatever should be your main branch! (From now on I use "master" and "main branch" interchangeably.)

For sure, people can argue whether the most important practice should be using small commits, writing meaningful commit messages, or something else. They are important and they might have a long term benefit, but they require efforts. A bit more thinking of structuring your commits, thinking about how to word your commit message. They are not mentally free.

On the other hand, avoiding working on your main branch is something you can do without any extra efforts and pays off with huge dividends given that you work on a bit bigger project where there can be parallel changes.

And unless you are working on a small side project, there is a fair chance of having to work on multiple changes at the same time

But what can go wrong if you work on master?

If you and your reviewers are fast, maybe nothing.

The problems start to arise when reality kicks in.

### Shifting priorities before pushing your changes

You started working on something, you progressed, you made some small commits, but you are not ready to create a pull request yet. You haven't even pushed to your fork. Then your boss comes to your - virtual - desk and tells you that there is an urgent problem to fix on the same repository and it's your new top priority - topping your other 10 most important tasks. You have to put things aside and shift your focus.

So what do you do?

If you don't work on master, you just switch back to there, you make sure that it's up to date with upstream and check out a new branch and off you go.

But if you already made your commits on master...

Maybe you'll simply clone to a new folder, but you know in the inside that it's wrong...

Otherwise, you'll have to create a new branch first where you park your already made commits, go back to master, remove your commits manually or simply forcing an overwrite from either your fork or from upstream directly (in case you have no automatic fork synching enabled between your fork and upstream).

While this is not a huge task, it's a waste of your time. And often people working on their main branch will be the ones not knowing how to remove commits or forcing an overwrite.

And this was only the simpler case.

### Subsequent pull requests

If working on master is not a taboo for you, the following case could have easily happened to you.

You commit a few changes on the master branch, you create a pull request, you go for a walk and when you come back, you realize that still no review has been done, your code is not merged yet. Your priorities are not necessarily the same as others'. 

But you'd like to start working on your next task - on the same repository. What will you do?

You will continue working as if nothing happened. You spend some time working and you'd like to create a new pull request. You realize that the first is still not merged. What do you do?

You might try to include the old changes in the new pull request. But most probably it would be bad if not disastrous.

- Maybe the changes are not related so your new pull request is not cohesive anymore.
- Maybe you want to create small pull requests so that you get more meaningful reviews.
- Or maybe you already received some comments that you must work on. The previous commit is simply not ready for any integration.

Anyway, you must somehow submit your new commits without the previous ones that are not yet integrated.

The process would be similar to the previous one, but you cannot simply overwrite your local commits with the ones from your fork, because your fork's master is already corrupted.

You cannot even simply clone from your fork, it'll contain the commits you don't want. Well, you might create a new fork...

But if you don't want a new fork, you have a couple of additional steps to do compared to the workaround we saw in the previous section.

You have to add (if you haven't done so) the forked repository - that is often referred to as upstream - as a remote and overwrite your local main branch from there. Obviously only after you parked all your local commits to a new branch. Then you should create a new feature branch - you really should! - and then cherry-pick from the parking branch the new ones you wanted to submit for your pull request #2.

How much easier would it have been just to create two different feature branches from your master and work on things separately in the first place?

It costs no more than `git checkout -b myFeatureBranch`.

## Conclusion

We have seen two simple cases when working on your main branch can cause trouble. They are not unmanageable and definitely, there are other ways to circumvent than the ones I presented. But if you know those commands, you're familiar with rolling back commits, working on detached heads, etc, most probably you know git better than committing on your main branch.

Starting each new development with refreshing the main branch of your local copy then checking out a new branch from there is the simplest and most valuable practice you can follow. It's essentially free, you have nothing to think about and it gives you the benefits of not having to use some extra commands to remove unwanted, unrelated commits in case of parallel changes.

What is your most valuable git practice?