---
layout: post
title: "How to automatically format your C++ code regardless the IDE you use"
date: 2018-5-30
category: dev
tags: [cpp, coding guidelines, clean code]
excerpt_separator: <!--more-->
---
If you follow me, you might already have noticed that [I'm a big fan of coding guidelines](/blog/2018/03/28/codereview-guidelines). Yet, I don't particularly enjoy commenting on formatting, such as indentation, [tabs vs spaces](https://www.youtube.com/watch?v=SsoOG6ZeyUI), whitespacing, etc... But I do and I keep doing it because it's an important part of readability.

<!--more-->

The more cohesive the code formatting is, the more readable, hence maintainable the code is.

In order to reduce the need for comments, debates and arguments on such items, we are introducing automated formatting to our source code.

In a previous project where we worked in Java, we already automated formatting checks by using the [Maven checkstyle plugin](https://maven.apache.org/plugins/maven-checkstyle-plugin/). Every time there was something not according to the rules we defined, the build failed, so nobody could check-in code that was not following certain rules.

In C++, we still had the good old code review validation. But this form of validation is not so efficient, as unfortunately, not everyone is strict enough.

Their time is up.

We are introducing [`clang-format`](https://clang.llvm.org/docs/ClangFormat.html) in our pipelines.

[`clang-format`](https://clang.llvm.org/docs/ClangFormat.html) is a tool to apply your formatting style to C/C++/Objectiv-C code, with lots of possibilities of customization. We are starting to use it in 3 steps.

## The mass update

We think that applying a new formatting style is best when the whole code base follows it. While this is unimaginable when you have to transform your code manually, it's an easy task with an automatic formatting tool.

So as a very first step, we run [`clang-format`](https://clang.llvm.org/docs/ClangFormat.html) on our repositories. Even for thousands of code files, this doesn't take more than a couple of seconds.

Right after, or maybe it's even better doing it right before, we introduce two validation steps in parallel.

## Format code in a pre-commit hook

We turn on a [pre-commit hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) in our local Git settings. Before committing, Git runs the [`clang-format`](https://clang.llvm.org/docs/ClangFormat.html) and applies the formatting style to the code you wish to commit. The time it takes is not significant as it checks only the changed code, but remember, even on the whole code base it was fast. 

If you wish not to have automatic reformatting, it's possible to only run the checks and failing the commit. In such cases, you will also have a report of where the checks failed.

This step needs a manual action because checking out a Git repository cannot automatically turn on any hooks. First, this was surprising to me. However, it makes perfect sense. It would be too dangerous. Imagine that I create a repository with a hook deleting all your files and folders... I can still add such hooks to an installation script, but it will not be automatically installed, but by you.

## Add checks to your continuous integration pipeline

In our Jenkins pipeline, we add a step in order to run [`clang-format`](https://clang.llvm.org/docs/ClangFormat.html) every time there is a new pull request. If there is any discrepancy, the pipeline fails and the build doesn't even start. This is just an extra security measure. If everyone turns on the hook on local, the pipeline should never fail because of styling issues. But better be prepared for human laziness and forgetfulness.

The key takeaway is that it's really easy to automate formatting for C++ codebases and you don't even have to force people to use the same IDE. That'd be a bad idea anyway. Either you just put some checks to your pipeline or you automate the formatting totally. What are your experiences with code formatting automation?