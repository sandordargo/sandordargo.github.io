---
layout: post
title: "Why clean code is not the norm?"
date: 2019-8-28
category: dev
tags: [clean code, discuss, craftsmanship, technical debt]
excerpt_separator: <!--more-->
---
Why is it that #CleanCode is still the exception and not the norm in so many companies?

A very interesting question from [Marcus Biel](https://twitter.com/MarcusBiel), who was the Director of Developer Experience at RedHat at that time posted on [LinkedIn](https://www.linkedin.com/feed/update/urn:li:activity:6481563683385278464). That moment, I didn't have the time to go deep on this topic, but I reread the answers a couple of months later.

<!--more-->
## It's them!

People pointed at two groups mostly:
* project managers
* junior devs

Project managers are guilty because they want no more than one thing: kicking new features and projects out on the software factory door. They don't care about clean code at all, they don't even understand what is technical debt and without that deeper understanding _"clean code"_ (anyhow we define it) is expensive.

Besides, "[the amount of new developers doubles every five years](http://blog.cleancoder.com/uncle-bob/2014/06/20/MyLawn.html)" and they obviously don't know to how code in a clean way and the math implies that there is simply not enough senior developers to mentor them.

While these are all true, they simply blame others and the circumstances. These explanations are childish as they lack self-responsibility.

But I also found some more interesting answers.

## It's us!

### We have to say no

Eloy mentioned that in his opinion, sometimes it's better to say no as a developer. Or at least gently warn people that if they continue like that they'd shoot themselves in the leg. I can't agree with him more. Not surprisingly a conversation was spurred about the lack of mandate to say no as a developer, as an individual contributor. Again, it's easier to blame the circumstances and others.

But as both Eloy and another person pointed out, there is more than one way to say no. One is to simply step aside and move to another project if you don't feel that your expertise is welcome. Or you can be a bit harsher and remind your management that if they don't trust you, they are free to fire you. If you are a seasoned developer, how difficult would it be to find another job? [It's not so easy](http://sandordargo.com/blog/2019/05/22/it-is-so-easy), but still it's better than staying at a position where you are not valued and trusted.

I liked this subthread a lot because it emphasizes the responsibility of the individual, it brings up professionalism. In my opinion, if you are a real professional, you must be able to say no. More than that it's your duty to say so when you feel it's appropriate! I know it's difficult for the first time. But a good boss will value it. Even if due to some circumstances he will order you to take the dirty road, he would understand your reasons and not punish you for them. Well, if he is not a good boss maybe he'd be just pissed of. I already said. There are always other projects/companies to work for.

### Clean code is difficult

Speaking about professionalism and difficulties, another reply I liked was about saying that #CleanCode is hard. As it requires serious efforts (at least in the beginning) and discipline not to accept anything but the best. It's not in the mainstream. What I like in this answer is that it doesn't toss responsibility away. People - we - are lazy and disorganised. People - we - tend to go towards the least resistance and if mediocre performance is accepted by the team, the individuals - we - will just perform on a mediocre level.

If you want to perform constantly on a higher level, you have to set up a framework that will force you to meet higher standards. Most of the people are around average - no surprise, it's fact by definition. If you can help others improve even better, but you are responsible for yourself. You have to [push yourself to be better](). 

We have to [drive the technical change](http://sandordargo.com/blog/2018/02/28/setting-yourself-up-to-succeed) and maybe the herd will follow you. Or at least a couple of them.

## Let's do our job!

While it's true that many project managers don't even understand technical debt or clean code, I don't see it as a problem. Do they understand inheritance, immutability or the difference between different types of memory allocations? Some surely do, but many don't and it is not a problem. We [do our job](http://sandordargo.com/blog/2019/02/27/do-your-job) and when we have to then we explain and answer questions.

As professionals, we have to argue and say no when we think so. Hopefully, you are hired to give your best, to share your expertise, not to just act a coding monkey. Read how [Yegor Bugayenko despised his hairdresser](https://www.yegor256.com/2015/02/23/haircut.html) just because he did nothing but tried to please him.

I find that many people are afraid of acting as professionals. Sometimes it's because of negligence. They simply don't care. Do you remember [Office Space](https://en.wikipedia.org/wiki/Office_Space)? [Will I make another dime?](https://www.youtube.com/watch?v=_iiOEQOtBlQ) It's somewhere understandable, but it most probably means that you are either not real professional or you are at the wrong place.

Sometimes, it's due to impostor syndrome, which is normal. But you have to push through it. Others wrote about this in so [captivating words](https://dev.to/search?q=impostor%20syndrome).

Sometimes, it's not negligence, just fear and lack of experience. Seniors have to show the way. Have to show that you can step up for your opinion. People with experience have to foster a safe environment which is safe to speak and stand up for your professional opinion. Sometimes, introducing tech debt is acceptable, but if you just keep feeling that your opinion is not respected and the debt is endlessly growing, probably you have to step aside.

And while it's understandable that project managers don't understand all the details about what we do and they don't even care - why should they? - but it's our responsibility to translate our points to their language. It's not about ugly variable names, messed up indentations in vim. It's also not about lines of code in a class or the cyclomatic complexity of a function. It's solely about maintenance costs. [Code Simplicity: The Fundamentals of Software](https://amzn.to/2ShCI9I) makes a very good job explaining this. 

It's difficult, I've failed and will fail at it so many times, but we should give it a try and then keep trying. It's our responsibility to make the management understand if they spend a little bit more, in the beginning, they will spend much less in the long run. Of course, it's market pressure and stuff. Sometimes it's more applicable, sometimes a little bit less. But do they like when their product hit the ground because of poor code? What if it means millions of dollars lost in revenue and market value? What if it means hundreds of lives?

## Conclusion

It's difficult to define what clean code is. But probably it's like hardcore pornography. You can't really define it, but you know it when you see it. It's difficult to describe clean code properly, but if you encounter a clean pull request you recognize it with a smile of satisfaction.

What we have to keep in mind that the responsible is always us. Never them. We have to raise our voices, we have to explain and educate. And in case, we have to step aside. That's the only way towards more maintainable and reliable code bases. You think it's an exaggeration today, but tomorrow you might understand that it can be a matter of life and death, not just professional pride!
