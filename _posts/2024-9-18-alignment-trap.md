---
layout: post
title: "Do it right or do the right thing: The Alignment Trap"
date: 2024-9-18
category: dev
tags: [agile, cleancode, projectmanagement, bestpractices]
excerpt_separator: <!--more-->
---
You might remember [one of my trip reports where I mentioned The Alignment Trap](https://www.sandordargo.com/blog/2021/11/17/trip-rerport-meetingcpp-2021). In this post, I'll dig a bit deeper in preparation for my future talk at [Meeting C++ 2024](https://meetingcpp.com/2024/).

First, I heard about The Alignment Trap at C++ On Sea 2021 where Phil Nash spoke about it in his talk called [Zen and the Art of Code Lifecycle Maintenance](https://www.youtube.com/watch?v=tjnFXS10jU0). The alignment trap originates from [Alan Kelly](https://x.com/allankellynet) who ran some studies at MIT to understand what makes a business successful in the mid/long term.

In particular, he examined two aspects of IT departments:
- whether IT is aligned with the business
- whether the IT department is efficient or not in what they do

To simplify, we can say he examined if the IT does the right thing (business alignment) and if they do their thing right (efficiency).

Interestingly, it turned out that having an effective IT department is more beneficial for the overall business than having IT aligned.

> To understand which business was doing better, IT spendings and business growth were examined over a span of three years.

![Do it right or do the right thing?]({{ site.baseurl }}/assets/img/alignment_trap.png)

There is no surprise in that companies where the IT was efficient and aligned with the business did better than anyone else. There are two surprises though.

Businesses where the IT is not efficient, but aligned (top left quadrant), did worse than those companies where the IT is efficient but not aligned (bottom right quadrant) and they also did worse than companies where the IT is not efficient and not aligned (bottom left quadrant).

So considering the above image, both companies in the "maintenance zone" and "well-oiled IT" quadrants made better than the ones in the top-left quadrant called the "alignment trap".

There is an important thing related to this study to get straight in order to avoid misunderstandings. The researchers don't say that you should NOT do the right thing on purpose.

It's something else. When you start a new business or a product, it's very difficult to know at the first shot what is right. In some cases it may even be impossible. You might figure it out by time. But to figure it out, you need multiple shots. You aim, you shoot, you miss and based on the feedback you got from your previous shot, you aim again.

In other words, you need quick and relatively cheap iterations.

If it reminds you of lowercase agility, you are not mistaken.

If you can iterate quickly, if you can change and adapt in a timely manner then what you do in a given moment is not so significant. You'll figure out what is right.

On the other hand, to change how you do things is bloody difficult.

Let's move to another scale and think about code reviews.

Usually, it's relatively easy to challenge your colleagues if you think they misunderstood the task, or the functionality to be implemented and therefore something should be changed. Maybe you convince them, maybe you realize that you misunderstood and you get convinced. Maybe you both figured out that there is no clear answer, both of your understandings are fine. These conversations usually go easy.

But when it comes to engineering practices, *"best practices"*, it can be a whole different story. People often become emotional and they can be really difficult to convince even if you - really - know it better. Even if you bring well-documented conventions and guidelines. They might even say they know better than what was written in the core guidelines of a language maintained by some of the most knowledgeable people of that language.

It's extremely difficult to change **how** people do something.

So with regards to the mentioned study, it's much more difficult to get from the top left corner to the top right than from the bottom right to the top right. It's more difficult to change how you do things than to change what you do.

Even doing the bad thing in a bad way leaves you with better chances to change, because at least there you know that you don't do the right thing and you have to change that, it might give you the incentive to change completely. But when you do the right thing the bad way, you are probably smug enough about the fact that you do the right thing.

## Conclusion

According to a study conducted by MIT Sloan and Alan Kelly, concerning mid-term IT spending and sales growth, it's more important to do your things right than doing the right thing. It's easier to change what you do than how you do it.

This is something that we can observe in other areas of life as well, for example directly in programming. It's easier to challenge someone about the functionality of their code than how it's engineered. If your plans are not only short-term, focus on small iterations and how you do things, instead of just trying to make something work at the first shot.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!