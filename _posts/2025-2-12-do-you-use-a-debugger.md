---
layout: post
title: "Do you use a debugger?"
date: 2025-2-12
category: dev
tags: [cpp, debugging, bestpractices]
excerpt_separator: <!--more-->
---
You might laugh at this question because your answer is _of course, who wouldn't!_ Especially in the world of C++. Equally, you might laugh at this question, because your answer is _obviously no!_ You write clean code that reads like well-written prose and the behaviour of your code is clearly documented in automated tests.

If your answer is the latter, good for you. I mean it. I used to believe in being in that group.

In the earlier days of my developer career, I didn't use a debugger. The reason was fairly simple, I didn't know how to.

So what did I do?

I did like everyone else, I put "debug statements" along the suspected execution path. Preferably comments that stand out in the logs, or at least they are easily searchable, but not too embarrassing in case I accidentally include them in a pull request.

That was fine.

Sometimes someone asked me if I used a debugger. I usually revealed my dirty little secret with a bit of shame. The reaction was always compassionate. Most people I knew barely used the debugger. At least, not for more than just setting a few breakpoints.

Years passed by and I became familiar with clean code and test-driven development.

Finally, I got an excuse for not using a debugger. 

A fair excuse!

My code indeed became better, I delivered code with more tests and fewer bugs. For the rest, debug statements were usually enough. I also switched departments and I ended up with a more complex codebase. My approach was still enough even for existing bugs.

But our ecosystem and documentation also evolved and it turned out that attaching the debugger to tests was relatively easy so from time to time I did it, but barely for the code that I wrote. Given the sometimes long, yet straightforward flows of our codebase, my approach was enough.

After changing jobs I didn't even write feature code or fix bugs for nearly 2 years. I dealt with architectural questions. Debugging was not needed.

Lately, I joined a new team and I was lucky enough to meet some of the folks in person soon. I sat down to pair with one of the engineers to learn about our codebase and we had a conversation similar to this:
- Please just build the desktop app and attach a debugger.
- I don't know how to. Can you show me?
- I don't know how it works with your IDE, but wait... If you don't use a debugger, then how do you debug?
- I don't need to. I use the tests.

I received some eyes for sure.

During the coming months, I realized that I can be confident or even smug around using tests, but it's not enough. I used to work with an 800kLOC codebase where I could strive without using a debugger. But now, with a much smaller codebase, I must use it. To understand what goes on with bugs reported, debug statements are simply not enough. Given the architecture and the tons of callbacks, it's often very difficult to figure out where execution flows go.

Instead of throwing debug statements all around, it's much easier to set a breakpoint near the entry point and step through the codebase.

Thinking about those saying that they haven't used a debugger in decades, I have to scratch my head.
- Maybe they didn't have to work with so complex systems.
- Maybe they were around from the inception of such complex systems and made sure that everything was well documented.
- Something else?

I don't know, but nowadays I have nobody around me without stepping through code with the debugger regularly.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)