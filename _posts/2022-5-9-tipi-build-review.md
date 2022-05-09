---
layout: post
title: "Tipi, a new solution to build C++ projects easier"
date: 2022-5-9
category: dev
tags: [cpp, tipi, cicd]
excerpt_separator: <!--more-->
---
In this article, I'd like to share an initial review of [Tipi](https://tipi.build/), a C++ related cloud service. For your information, there might be a future collaboration between me and [Tipi](https://tipi.build/), but this article is not sponsored. I explicitly stated that I don't want to take any money for writing a review. Now let's get started.
<!--more-->

## How I learnt about [Tipi](https://tipi.build/)

I learnt about [Tipi.build](https://tipi.build/) at [CPPP 2021](https://www.sandordargo.com/blog/2021/12/15/trip-report-cppp2021). [Damien Buhl, Tipi CEO](https://twitter.com/daminetreg) delivered a presentation about their product, a "massively scalable C++ remote compiler cloud". I found the idea interesting and useful. I registered quickly an account using the promo code he shared at the conference, but I didn't do anything with it. I simply had too much on my plate around Christmas.

But something that really engaged my mind was this slide from Damien's presentation and I've been using it at several places.

![Carbon footprint]({{ site.baseurl }}/assets/img/carbon_footprint.png)

Writing software in PHP, Python, TS or Ruby increases CO2 emissions way more than software written in C++, C or Rust. As [Marek said](https://twitter.com/mrkkrj/status/1467798371670925315), the point of writing software in these high-level languages is out of “intellectual laziness”.

Then a few months later, Damien reached out to me about whether I'd write a review about [Tipi](https://tipi.build/). I said I'd do it with pleasure. This review didn't move forward as fast as I planned, because we identified some issues that they fixed first and I also needed a bit more time both to mitigate some technical issues on my side and to understand better how [Tipi](https://tipi.build/) works.

Then when I wanted to publish, I realized that my biggest pain point was fixed, but I didn't have the necessary time to try it before I went on a long vacation.

Finally, I finished my first review.

## What is it for?

Last year at one of the C++ conferences, someone asked how many languages you have to learn to code in C++. The answer was about 4 or 5. Obviously, you have to know some C++. You'll need some shell scripting on Linux and I guess Powershell on Windows. You need CMake or something similar to be able to build your project. Well, you might even have to know the makefile syntax and whatnot. Ok, I exaggerated. You can already get away with 3 languages.

That's the first place where [Tipi](https://tipi.build/) comes into the picture. It should reduce the need for two languages in most cases. C++ and shell. The need for our beloved C++ is obvious I guess and you also need a tiny bit of shell. You have to call [Tipi](https://tipi.build/) somehow, right? But you don't have to know much, so maybe we can say 1.5.

All the rest should be taken care of by [Tipi](https://tipi.build/). At least for the average user.

The promise is that you don't have to write your build scripts, [Tipi](https://tipi.build/) will take care of figuring out how to build your projects. 

That might be quite useful for many of us.

I've been coding in C++ for about 9 years and I spent the first 5-6 years incapable of compiling something on my own. I wouldn't have been able to exist outside of our in-house build management system. I simply didn't have the need and I didn't bother. Since then I came up with [Cmake Project Creator](https://github.com/sandordargo/cmake-project-creator) which also eases the creation of build scripts and dependency management, but it's just a pet project, nowhere near [Tipi](https://tipi.build/)'s capabilities.

Where [Tipi](https://tipi.build/) stands out is that it also takes care of dependencies and build environments. It doesn't just set up projects according to the environment you wish to build in (such as Linux, Mac, Windows), but you can also build in the cloud. You pass in as an argument what environment you want and the C++ standard and [Tipi](https://tipi.build/) will take care of the rest in the cloud. You don't have to worry about having the right environment.

That sounds really promising, right?

Let's see how far I got.

## The features I tried to use

First, let me just list what I tried to do. Everything in the list I tried both locally and in the cloud.

- compile a hello world project both 
- compile some small Github repositories with C++ code in them
- compile some random bigger libraries
- compile projects I generated with [Cmake Project Creator](https://github.com/sandordargo/cmake-project-creator)
- compile with its new [live build mode](https://tipi.build/explore/live-build)

I won't go through them one by one, but I'll rather share things that didn't work well and things that worked pretty well.

## The problems I faced

As of May 2022, when this article was originally published, [Tipi.build](https://tipi.build/) is a new product under heavy development. It still has some bugs and missing features. But the team is reactive and helpful, the product is improving. As I wrote earlier, by the time I finished my review, new features came and I decided to rewrite it.

Let me share the two biggest concerns I faced.

###  Installation

First of all, I couldn't install it on Ubuntu 18.04. It requires at least 20.04. That's a pity, but [Tipi](https://tipi.build/) plans to make it available on older versions too. So I went on with [creating a docker image](https://github.com/sandordargo/tipi-container) that I can use. [Tipi also provides one](https://hub.docker.com/r/tipibuild/tipi-ubuntu), but I wanted to learn a bit more about docker too and this was a good excuse. I ran into some issues along the way and for the [Tipi](https://tipi.build/) related ones as I asked the team and they always helped me out with some deep technical explanations included.

There are some minor usability issues, and I opened some tickets for them. By usability issues, I mean that sometimes the colours of the prompt are messed up after an unsuccessful exit or that when the CLI reminds you to update your [Tipi](https://tipi.build/) client then after the update it returns instead of doing what you originally asked for. These are unpleasant, but not severe and I'm sure they are going to fix them soon.

I was worried more about downloading all the build tools (\~7GB) whenever I instantiated my docker image. It made me lose quite some time every day when I started to play with [Tipi](https://tipi.build/). But it turned out that you can install those tools when you install the CLI which is might not important for those using [Tipi](https://tipi.build/) on their physical machine, but for those using an image it's a lifesaver.

Though I had to pay attention to one thing that Damien pointed out. I had to mount a volume on the TIPI_HOME_DIR, otherwise, I got every time a full download of the libraries and rebuild of the platform libs I depended on. The solution was to mount a docker volume on the TIPI_HOME_DIR (but that would mean our docker would be useless because the preinstalled state would be hidden and would be reinstalled again).

After all, this is how I ran my container.

```bash
export DOCKER_ID=$(docker run --rm --mount type=bind,source=/home/sdargo/.tipi,target=/home/tipi/.tipi -it -d my-tipi-image /bin/bash) && docker exec -it $DOCKER_ID /bin/bash
```
As such, I could start playing around right away whenever I felt like it.

### Unit tests

First, I used the [single](https://github.com/sandordargo/cmake-project-creator/blob/master/examples/single.json) blueprint to generate a project with CMake Project Creator. After declaring the dependency on GTest in `.tipi/deps`, there were some issues. It turned out that my tests were in a `tests/` directory, while `test/` was expected by [Tipi](https://tipi.build/). After changing the names, all worked fine.

I didn't find it very convenient, but when you start building a project with [Tipi](https://tipi.build/) and you are aware of the expected naming conventions, this is not an issue. And even better is that the team already fixed this. Now you can choose whatever name for your test directory. Thanks a lot for that!

```json
// .tipi/deps
{
"google/googletest"
 : { "u" : true, "packages": ["GTest"], "targets": ["GTest::gtest"] }
}
```

I tried [another blueprint](https://github.com/sandordargo/cmake-project-creator/blob/master/examples/nested_dual.json) where there are multiple libraries, with several `test/` directories. [Tipi](https://tipi.build/) couldn't pick up the tests when the directories were nested inside other directories. I think this is an important problem and the [Tipi](https://tipi.build/) team is already working on it.

## What I liked

Despite the initial difficulties which are partly because of my old setup, [Tipi](https://tipi.build/) is quite easy to use. You don't have to deal with the build scripts, you can just go ahead and build.

There is not much to say about that, it just works with simple structures and there is an ongoing development to make it work with more complex structures.

When I originally started to write this review, I had some trouble with the speed of [Tipi](https://tipi.build/). Once a project is synchronized with your vault, the build itself is fast. After all, depending on your subscription, you can have even 128 cores working on your build. But the initial setup is slow, which means that you'd need bigger projects to really benefit from [Tipi](https://tipi.build/).

Then I learned about a new feature, called [Live Build](https://tipi.build/explore/live-build). With the `--monitor` option, your Tipi client keeps monitoring the changes in your local directory and whenever there is a change, it reruns the build. If you also add the `--test all` option, it reruns the tests too. So basically, whenever you update a file, [Tipi](https://tipi.build/) will compile and if possible run the tests. It's very neat!

Sometimes it launches a bit too many builds, but this feature is still under development and when I reported it, it was clear that the team knows about it and is going to enhance the "smartness" of this very useful feature.

## Conclusion

I haven't finished with my experiments with [Tipi](https://tipi.build/), but I already played around with it enough to have an opinion on it. While [Tipi](https://tipi.build/) is at the beginning of its journey and still has a long road to go through, it's already clear it has the strength and stamina to walk that long road through if the team continues to deliver the fixes and features and stays so helpful.

[Tipi](https://tipi.build/) has a great potential to simplify how we build C++ projects both with its lack of explicit makefiles/CMakefiles and also with its ability to build in different parameters. With its new [Live Build](https://tipi.build/explore/live-build) features it's perfectly useable in everyday development. I'd love to try it in CI pipelines with Github actions. The development is still ongoing.

If the initial time needed for cloud builds could be shortened a bit, that would be just great.

Feel free to play around with it and let me know what you think.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!