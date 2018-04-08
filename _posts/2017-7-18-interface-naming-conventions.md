---
layout: post
title: "Naming conventions for interfaces in Java"
date: 2017-7-18
category: dev
tags: [java, interfaces, conventions]
header: "Recently in a code review I spotted out that someone used ISomeService as an interface name and SomeService as a class name. I didn't feel okay about it and we went into a long discussion.
"
---
Although the guy was open to change the code, he didn't really consider using names with an I prefix a bad habit.

I was not so happy sith a such an outcome, even though it would have already resulted in a change of code. I wanted more, I wanted persuasion. So I went on, read articles, asked people, looking for their opinions.

I posted a question including my own opinion on an internal coding guidelines board. There many people might have read my comment, but I received no answers, just two likes in about a week. Slight support.

We also have a chat room for the company's Software Craftsmanship community, where I asked what the people think about the usage of names such as `ISomeInterface`. I received 7 opinios just before/after lunch time. One guy said he doesn't really care if he sees that in a code review, all the rest was in favor of removing the leading I.

In the following few paragraphs, I try to summarize the opinions I got, which also resonates with my opinion.

The leading I's are considered as a code smell. If we see such names we might think that the author did not spend enough time on finding meaningful names, or maybe an interface is not even necessary. (This might be the case when there is only one implementation.)

The I prefix is a legacy of bygone times when developers had no powerful IDEs, couldn't simply hover on names, types to see information on them, but had to rely on the names of the types, variables to get as much information as possible. This might ring a bell for you. Yes, this is the (in)famous [Hungarian notation](https://en.wikipedia.org/wiki/Hungarian_notation), which was useful at a time, but luckily those times are gone.

Even worse these leading I's show an incosistency. Most of the people will not call their methods mSomething and their classes CSomething. Why only the interfaces? Why?

If it still happens to you that you cannot come up with a good name for the interface and/or for the class, use the ugly name for the implementation, not for the interface, which you are more likely to reveal to your clients.

Bottom line is don't use I prefixed interface names, just as you don't use C prefixed class names, take your time and come up with some meaningful names.
