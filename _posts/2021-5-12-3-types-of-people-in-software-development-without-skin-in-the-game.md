---
layout: post
title: "3 types of people in software development without skin in the game"
date: 2021-5-12
category: dev
tags: [career, watercooler, management, books]
excerpt_separator: <!--more-->
---
I read the really valuable [Skin in the game, by Nassim Taleb a few months ago](https://dev.to/sandordargo/skin-in-the-game-hidden-asymmetries-in-daily-life-by-nassim-taleb-9i). If you haven't read it yet, you might ask *who are people without skin in the game?* 
<!--more-->

There are people who take no real responsibility for making the wrong calls. We mentioned non-founder CEOs, football coaches. When they fail, they just dance around finding their next similar position. Often after taking a hefty severance package.

In the comments section, [raddevus](https://dev.to/raddevus) mentioned that there are developers without skin in the game as well. People who produce some features and then off they go, he called them **Mic Drop Devs**.

Inspired by [raddevus](https://dev.to/raddevus), I collected a set of roles in our industry where you'll have a higher chance to find people without *skin in the game*.

Beware of them!

I'll start from the lower levels.

## Drop The Mic Devs

They are the ones [raddevus](https://dev.to/raddevus) mentioned. They work on real projects, they produce real code, sometimes usable code, but they are never taking responsibility for maintaining it.

I identified 3 subtypes of *Drop The Mic Devs*.

### Devs with one leg out

There are often some colleagues who don't enjoy working for the company and [concentrate on leaving](https://dev.to/sandordargo/how-not-to-quit-1hmo). Maybe they already started to do some interviews, or maybe they are already on the notice period and you just don't know about it. In certain countries, the length of the notice is up to 3 months. It's way more than necessary for a proper handover, so obviously they are expected to perform some real work during those months. 

Due to some reasons of conscience, most of them will do some work, but they don't have a skin in the game anymore.

Whatever long term consequences their code will have, that will be the problem of someone else. It doesn't mean that everyone will produce crap on their notice period. Not all of them will produce crap, but there is a fair chance that they won't do their best. Who could blame them?

### Job hoppers

The case for job hoppers is similar, but there are some key differences. I don't want to clearly define who is a job hopper based on time spent in a job. Someone who changes every 6 months? Maybe 12? Maybe 24? I don't know and I don't care. That's not the point here.

Regardless of the time, they enter a job without even giving the mental possibility for staying there for a longer undefined period of time. They don't plan to make long term effects on the project they work or to deeply influence the company they work for.

They only join to build their CV to be able to add a few sexy names and logos. They come to take, not to give. They are interested in getting some experience in certain technologies, to try some patterns, but they are not interested in how the product will be maintained. Of course not, they will be soon somewhere else.

One could argue that their reputation will suffer by producing code that is difficult to maintain. But with the pace people move around between positions or jobs, it's hardly the case. It's very difficult to identify individuals who contributed above average to make a product too difficult to maintain.

### Feature team developers

The last group of people among the *Drop The Mic Devs* are the feature team developers. 

What do I mean by a feature team?

In certain organizations you will find teams who are solely responsible for developing new features for a product, but they don't take care of their maintenance.

Usually, these teams take responsibility for problems identified before the production activation. Once it's in production, they wash their hands and start delivering the next feature.

Why should they make maintainability their main concern? What would motivate them?

Apart from hoards of maintenance people with sticks, pitchforks in their hands...

Maybe a rotation between the feature and the maintenance teams... Often there is no such rotation or it's very limited on purpose. Due to the context switches people would be less productive, they'd have to learn different processes and in addition people ~~might~~ do have strong preferences.

Another solution is to have teams owning the product they work on during its whole lifecycle. If you don't have such teams, you'll easily end up with people not taking responsibility for the long term maintainability of the product.

This doesn't mean that the delivered product will be subpar with full of bugs. Maybe it will seem quite good at the time of delivery. Yet there is a fair chance that with the evolution of the product, the maintenance costs will be much higher than optimal as it was not an important aspect for the developers.

## Non-coding Architects

After the Drop The Mic Devs let's mention one of returning characters of Uncle Bob's works. The infamous non-coding architect.

A person with a long and usually aged experience of building software projects. He (or she) is in a distant position from the coding teams, his hands-on coding skills are rusty.

He's responsible to design the architecture of complex systems, yet I say often he has no skin in the game. He is rarely an owner of a project. He is definitely not someone who is responsible for making the system lucrative, and he is not responsible for delivering the product either.

He delivers blueprints and based on those documents the (feature) teams have to build the product.

If the product doesn't sell it will be the failure of either the business or - if the implementation was low quality - of the implementation teams. They did the manual labour and if the results are crappy it's their fault instead of the person who made bad plans...

With enough time, with enough projects, probably the people around could realize what's going on, but it might be too late or simply nobody has such a long visibility on the non-coding architect's work to draw such consequences.

## (Non-tech) Agile coaches

Agile coaches are usually the ultimate people without skin in the game. There are good agile coaches for sure, I've also worked with one, but the vast majority based on my experience didn't do anything special, and didn't care that much about the projects.

Let's put it this way. They were like not so experienced martial arts students who know a couple of moves and know only those therefore they use only those. The agile coaches I met with knew a few things about agilish project management, they learnt a few techniques to popularize and teach them and that's all they knew, therefore that's all they did.

Unlike martial arts students, these agile coaches are not really learning new things. When someone asks them about techniques they don't know about, they fend off the questions with the professionalism of a politician who is asked some cumbersome questions.

As the scope of their coaching is limited, as the length of the coaching is way shorter than the projects themselves they will not be held responsible for any failures.

In the case of successful projects for sure, they will be invited to the celebrations but when things don't go well they will not be held responsible. After all, they didn't do the plannings, they didn't do the code... They have no skin in the game.

## Conclusion

Don't feel attacked if you belong to any of these groups. You can still have skin in the game or maybe you prefer not to have any skin in the game... 

I used to be part of a feature team myself, and I do sympathize with coaching roles. Yet these groups will give a home to more people without skin in the game because of the reasons I explained above.

If you are someone who is fully owning (as part of a team at least) a product for its full lifecycle, it's hard not to have any skin in the game.

People leaving or wanting to leave, people who are not part of the maintenance or maybe not even the implementation are usually people who don't face the consequences of bad design/implementation decisions. They have *no skin in the game*.

Do you?
