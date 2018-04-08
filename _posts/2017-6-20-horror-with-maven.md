---
layout: post
title: "Horror with Maven - dependency management vs dependencies"
date: 2017-6-20
category: dev
tags: [maven, java, dependencies]
header: "Even though I have written some Java code, but I did not have to really deal with Maven. IntelliJ took care of most of the things. But this time it was different. I had to use a maven plugin and create my own build settings."
---

In fact we want to upload our code to a local OpenShift instance with the help of Fabric8 plugin. Maybe there is an IntelliJ plugin, but I did not go into that direction.

The first time I ran `mvn install`, everything went fine. Compilation succesful, packaging as well, I could deploy my server on Openshift in a minute. Great. There I had some problems though, my server didn't start up.

I changed some settings in the `pom.xml` to remove some unsupported dependencies and run `mvn` again. Nothing. I mean nothing but failure. It couldn't fine some dependencies. I did not change anything regarding those and it used to work...

Hm... I was checking my maven home, checking if the packages are available in our company repository, but I found no clues. Everything seemed fine. Another guy who says he has some experience with maven checked my `pom.xml`, but couldn't find anything.

I tried to generate `pom.xml` by maven, but surprisingly it didn't fix anything.

Long story short, I found that my dependencies were only declared in the `<dependencyManagement>` section which is useful if you have child projects which inherit dependencies and their versions. But it is for version control, you still have to declare your dependencies in the `pom.xml`, otherwise it's not going to work. I only had the dependencies listed in the `<dependencyManagement>` section. After I listed them only in `<dependencies>`, it started to work.

Good that I learnt something.