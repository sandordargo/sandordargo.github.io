---
layout: post
title: "The Phoenix Project by Gene Kim"
date: 2021-X-X
category: books
tags: [books, watercooler, management, biography]
excerpt_separator: <!--more-->
---
[The Phoneix Project](https://www.amazon.com/gp/product/1942788290/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=sandordargo-20&creative=9325&linkCode=as2&creativeASIN=1942788290&linkId=fc9c78132b98763cc56dca36a783a5f4) is a fantastic book that is both a professional book aiming to guide teams and a "a **novel** about it, devops, and helping your business win".

It's written in a style I love the most and I think we have a clear lack of this type of books. It offers both what I'd look for in fiction and in non-fiction books. The author is a great storyteller, and he makes you not want to put the book down, not only because you want to learn about devops and how to organize properly a modern organization, but also because you are interested in how the story unfolds. Whether the CISO commits suicide or just leaves the city without notifying anyone from the organization or if the main character actually finds another job instead.

The only book of similar style is [Code Ahead by Yegor Bugayenko](https://www.amazon.com/gp/product/1982063742/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=sandordargo-20&creative=9325&linkCode=as2&creativeASIN=1982063742&linkId=a09eb2af4520bbf25642d58904093001). Let me know in the comments if you know any similar book!

You probably already guessed that I more than recommend reading The Phoenix Project. Let me share a couple of details from the book.

## Overtime, overtime, overtime

The Phoenix Project is about an entreprise where the business is not happy. They lost their competitive edge and they feel that they are kept hostage by their IT organization.

IT is not happy either because nothing works well, their opinions are always neglected, CIOs come and go and whenever there is a deployment problems are guaranteed.

As Erik Reid a board candidate described, the relationship between a CEO and a CIO (or business and IT) is often like a dysfunctional marriage. Both sides feel both powerless and held hsotage by the other.

Being hostage often manifests in the book by constant overtime. All nighters, working through the weekends, skipping festivities with your spouse, with your kids.

I was reading the book and shaking my head. People would really do that? If they would, why? For money? There are plenty of jobs...

Of course the book also shares some of the reasons. You have a calling, you or your spouse appreciate a high salary and you don't want to move for another opportunity, etc. 

Anyway, shit happens and there are definitely unanticipated events when someone has to solve the problem immediately and not during the next business day. There are emergencies and I think employees can be flexible in such cases, just like employers should be flexible with personal emergencies. 

But when it happens every month or every week, it's not an emergency anymore. It's just expected unplanned work. It's work that is in fact normal.

Leaving your life behind should not be normal.

[Don't burn out.](https://www.sandordargo.com/blog/2021/06/09/3-ways-to-prevent-micro-burnouts)

## The need of mentoring

Although the author doesn't write about mentoring explicitely, if you follow the story of Bill Palmer, the new VP of IT Operations, you will notice that there are two key factors to his success. The first one is his attitude. He doesn't accept things as they are but he does whatever he can to change whatever he can to the better.

As a former Navy officer, he is willing to fight back, he is willing to share his opinion and he doesn't only have preferences, but he has principles. When he realized that his opinion doesn't matter, he gave his resignation.

The other key element is Erik, whom he first described as a *raving madman*. He turned out to be his most important help. He became his mentor who did not solve problems on behalf of Bill, but instead, he gave some ideas, some clues that Bill could follow upon.

He couldn't have been succeeded the way he did without his mentor Erik. We all need help, we all need teachers, mentors who don't feed us the solutions on a silver spoon, but who help us to discover the right thing to do on our own - with the initial spark.

Ideally, you don't only have one mentor or at least not the sameone  trhough your career. On different levels, you'll need different mentors. Maybe even in different areas of your lives.

And how to find them?

As Lao Tzu said, *"when the student is ready the teacher will appear. When the student is truly ready, the teacher will disappear."*

So don't worry. Learn, be open and you'll find your mentor. Just like Bill did.

## The 3 ways 

Erik keeps telling Bill that IT work, IT operations is like plant operations and there are essentially 3 ways to be mastered in order to successfully support development and after all, the business.

The first way, is the traditional left to right flow of work. From development, the work flows to IT operations so that it can be delivered to the customer. The question is how to make this flow effective.

In plant operations, the key is to keep the amount of Work In Process small. In IT it's not different. Big deployments suck. Reviewing big pull requests sucks. You need small changes, small batches of changes, contiunous integration, continuous delivery and deployment to limit the work in process.

The second way is about having fast feedback from right to left at all of work. This is needed so that teams can improve, so that the same failures are not happening again an again. If development makes things more difficult for deploying and operating a system, but they don't get the feedback for it, they cannot improve.

Ideally, if something is broken the production system must be stopped until the error is fixed. In a plant this might mean to stop the assembly lines. In software, on a smaller scale, this means that when you break a unit test, you cannot integrate your change until all the tests are fixed. On a bigger scale, it might mean a complete change freeze,

The second way is about sending feedback back from each step to the previous ones. One can only improve with feedback. If errors are made at a workcenter, but they are never told so, they cannot fix their processes.

The third way is what makes the first two happen and in a sustainable way. It's the right culture. You need a team culture, a company culture that cultivates experimentation, as such it doesn't punish for failures as long as you learn from them.

You need a culture where people care about their craft, where they hone their skills, they practice towards mastery and as a result they dare to take some risks to experiment to innovate, to get better.

You need a culture where you don't have to hide feedback, where it's taken into account, where problems are not swept under the carpet but where they are welocme because they are opportunities to make things better.

If you want a healthy organization, these 3 ways have to be mastered.

## The 4 types of work

Finally, let me share the 4 types of work that are explained in the book. Erik told Bill that there are 4 categories of work and if he really wants to manage IT operations well, he must understand the different types.

Bill recognized the first 3 types quite easily, the last one was a bit more difficult whereas often that's the most common one.

First, you have *business projects*. This is quite straightforward. New feature requests coming from the sponsoring business units as new projects.

Then you have *internal IT projects*, such as modernizing the infrastructure, improve the pipeline or even to refactor the code to accommodate changes easier. Even if these activities are not budgeted directly by business units, it's important to keep track of the money, the resources they take so that you have a clear understanding of capacities for the other types of work.

The third category is a broad one, *changes*. Changes are often generated from the first two categories and they are usually tracked in ticketing systems.

The last category is *unplanned or recovery work*. If you have an unhealthy organization, this is going to be the most common one.

It is always urgent, otherwise it can be planned and deferred as a change. Unplanned work comes to the expense of the first three categories and as such it's distruptive, it makes plans inaccuriate and if it's too frequent, even makes planning impossible.

Building a predictible organization requires that you limit the possibilities of unplanned work appearing out of nowhere.

## Conclusion

[The Phoneix Project](https://www.amazon.com/gp/product/1942788290/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=sandordargo-20&creative=9325&linkCode=as2&creativeASIN=1942788290&linkId=fc9c78132b98763cc56dca36a783a5f4) is a mix of novel and a cookbook for implanting a successful devops mindset. More than that a mindset that will help you turn a dysfunctional IT - business relationship into prospering collaboration. 

Yet, I found the first part of the book unusually depressing. Maybe because I felt strong sympathy for Bill Palmer.

Speaking of Bill, he has a great personality as a leader and reflects traits that can be useful for many of us. He relentlessly asks questions and looks for solutions to problems isntead of complaining. And he doesn't stop until he understnads what goes on.

I already tried to put in place some items I learnt from him.

The Phoenix Project is a must read for everyone who works in IT, and probably even to those who work **with** IT.

I can't wait for [*The Unicorn Project*](https://www.amazon.com/gp/product/1942788762/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=sandordargo-20&creative=9325&linkCode=as2&creativeASIN=1942788762&linkId=9e9b64c0b50993a4d56f1989ddf25908).

Happy reading!