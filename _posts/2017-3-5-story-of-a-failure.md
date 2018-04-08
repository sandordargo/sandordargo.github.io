---
layout: post
title: "Story of a failure"
date: 2017-3-5
category: dev
tags: [failure, java, intellij, pairs]
header: "The first story of a failure on this page, but I'm sure it's not the last one."
---

My IntelliJ stopped working properly. When I clicked on run tests, it only compiled the code but never ran anything. I spent some time investigating the issue, I tried to fix this erroneous state, but nothing helped. Reboot (haha), reindexing, other JDK, new run configuration, removing all the settings. Nothing. No clue on stackoverflow either.

It was time to upgrade my Intellij anyway...

When I installed the new version on my Ubuntu machine, the compilation of my first Java project failed. Wow. The continuous integration is running fine and I changed nothing in local... So let's see. Cannot find javafx.util.Pair. Hm. Uhum.. No problem. The magical alt+enter didn't work. I checked the jars, the corresponding one was not added to the project.

While looking for the jar, I realized how strange it is that I cannot use a simple Pair class easily. Probably I should use another Pair. There must be something in java.util!

And yet nothing. Then I started to dig and I found on stackoverflow that [there is no pair in Java](http://stackoverflow.com/questions/156275/what-is-the-equivalent-of-the-c-pairl-r-in-java). Instead of fuzzy `first` and `second` references, I should write an own class for those two values I want to hold. It makes sense.

Fair enough, I wrote a new data container and removed all the references to javafx.util.Pair. My code compiled and my tests ran.

Bottomline is that if you are learning a new language, don't take anything granted. Not even a Pair. And doublecheck what you import.