---
layout: post
title: "Practice with a timer!"
date: 2018-2-6
category: dev
tags: [dojo, timer, roman numerals kata, potter kata]
header: "In the team that I joined a few weeks ago, we have regular coding dojos since the beginning of this year. We try to have one every week. Recently we were playing around with the <a href=\"http://codingdojo.org/kata/RomanNumerals/\">Roman Numerals kata</a>, which I had done a few times before, but never like this."
---
We defined some new rules. Our work had to be broken down into micro sprints. Not like in the [elephant carpaccio exercise](http://alistair.cockburn.us/Elephant+carpaccio) where you make a dedicated planning session in the beginning, here it was more ad-hoc and the notion of a sprint referred to a little bit different concept:

* At the end of each sprint, if your tests were not all green, you had to revert all the code you wrote in that sprint.
* In one sprint you either had to write a new test and implement the corresponding feature or you had to refactor.
* At the end of each sprint, you had to commit the changes you made.

Basically, it's the Test Driven Development cycle tied to a timer.

The length of one sprint was 5 minutes in the beginning and then it was reduced to 2 minutes.

Now you might think that okay if I write a crappy test, it will be green all the time, but remember, you don't come to a dojo to cheat, but to actually practice and learn something new.

However, at certain points, you have to work smart. Not everyone interpreted the rules the same way. Certain thought that you cannot finish a sprint before the timer goes off, but at the same time they were also afraid to start implementing a new feature worrying that they would lose their already implemented feature at the end of the sprint. On my side each time I implemented something new, I committed and reset the timer. To me, this worked pretty well and in fact usually, I had sprints shorter than two minutes. Sometimes I had sprints of about 45 seconds. I worked really in baby steps, didn't I?

As I mentioned earlier were working on the [Roman Numerals kata](http://codingdojo.org/kata/RomanNumerals/) in which we had to implement a converter that takes Arabic numbers and convert them to their Roman representations and it had to work properly up to around 3000.

As I already did this kata a few times, I knew there is a _"trick"_ meaning that: 
* you shouldn't use a long if-else block with a lot of branches but a simple map instead
* you shouldn't implement a complex logic to handle numbers like 'IV', 'IX', where a smaller Roman digit precedes a bigger one, but just map 4 to 'IV', 9 to 'IX', etc.

I had no major problems implementing the kata, but this was the point where I had to fall back two or three times at the end of the shortened sprints as I didn't remember to the code exactly, it did not just come right out of my fingers.

It turned out that others didn't even start the timer until they figured out in their head what they wanted to write in that sprint. It was a smart idea, I reused it the next week when we did the [Potter kata](http://codingdojo.org/kata/Potter/).

The [Potter kata](http://codingdojo.org/kata/Potter/) is slightly more complex. If you are not familiar with it yet, please read the [description](http://codingdojo.org/kata/Potter/) and think about it before you continue to read on.

When you start to implement the tests for several discounts, you have to prepare several smaller baskets of the big original input. Meaning that a list of `[0, 0, 1]` has to be split into `[[0, 1], [0]]`. By the time I reached this test, I was working with 2 minutes sprints. I knew what to do, I knew which features I wanted to use, but actually, I couldn't complete the feature in 2 minutes. I tried it a few times, but I had to realize that for me this short period of time is simply not enough. So I had to break down the feature into really small parts. I broke it into 4-5 baby features with the corresponding tests and it worked like a charm.

When I spoke about this problem of mine at the retrospective, it turned out that I overlooked a small change in the rules. Before each sprint it was up to us, how long should it take and only if I passed that time limit I set for myself I should have gone back to the base.

If I had used these tactics, I would have definitely set something like 5 minutes instead of 2 for this part, but I think it was more interesting that way I actually executed the exercise. I really had to break down the algorithm into small pieces and the small tests were leading me towards the solution.

If you found the idea of tying the circle of TDD to a timer, I encourage you to try it with any kata, but maybe with a simple one for the first time!
