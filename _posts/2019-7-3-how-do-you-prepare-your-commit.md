---
layout: post
title: "How do you prepare your commits?"
date: 2019-7-3
category: dev
tags: [git, hooks, productivity, showdev]
excerpt_separator: <!--more-->
---
What is your process to create a new commit? Is it just `git commit -am`? Or is it more sophisticated?
<!--more-->

Mine used to be something like this:
```
for file in files returned by git status:
  git diff file
  if like it:
    git add file
  else:
    change the file
    continue //so you will take it again from git st
```

Writing those `git add`s and `git diff`s were tedious even if I used copy-paste and the command history a lot. Especially for bigger changes, it was really cumbersome.

Then I thought it would be cool to simplify it a bit and combine `diff` and `add` and I came up with an alias that I call for simplicity `da` (_diff & add_).

If you call `git da myFile`, it will first show the diff of `myFile`, then it asks back whether you want to stage it or not and as a courtesy, as a third option it offers you patching in case you want to stage only part of the changed lines.

If you are interested in using it, feel free take it from [this gist](https://gist.github.com/sandordargo/ce3a55be6bd794be1826391ebe95718b).

If you interested in how I wrote this piece of code, please read on.

## Calling shell in a git alias

The first important problem I faced when I wanted to write `git da` was that in the second git command (git add) I'd have to use the first one's (git diff) input. How to do that?

The best way seemed to be if I just pass the filename as a parameter to both commands and chain them.

This is very easy, just like in a shell script you can reference an input parameter by ${POSITION_STARTING_FROM_1}. Because... it's a shell script that we need if we have to chain two commands.

But how to call a shell script in a git alias?

It's very easy`!` I mean it `!`

You have nothing to do, just start the content of your new alias with `!` (bang)! That's it.

```
[alias]
  da = "! git diff $1 && git add $1"
```
How would this work?

Well, it would add the passed file unconditionally and that's not what we want! Why did we do a diff then?

We have to pop up a question asking if we want to call `git add` or not.

Which git command can do that? I'm not aware of any. Patching (git add -p) is quite similar, show you a diff and asks you whether you want to stage it, but it goes hunk by hunk, and by default, it doesn't show all the changes at once if you made a more complex change.

I'm not a shell guru and _I like to copy and paste from Stackoverflow_, so I was looking around on the net and customized a bit and came up with something like this:

```
[alias]
    da = "! addprev() { while true; do \
          read -p "Do you wish to add this file? ([Y]es, [N]o, [P]atch)" yn ; \
          case $yn in \
          [Yy]* ) git add $1; break;; \
          [Pp]* ) git add -p $1; break;; \
          [Nn]* ) exit;; \
          * ) echo "Please answer yes, no or patch.";; \
          esac \
        done } ; \
        git diff $1 ; addprev $1"
```

In `addprev()` the shell just asks you what do you want to do and maps it the corresponding comment. It keeps repeating the question as long as you don't answer with one of the supported options or you can reboot, but com'on this is not vim!

## Wait, but where are you?

This version worked like a charm on my little local sample repo, then I started to use it at work in a much bigger repo. Usually, I have to modify one component and I don't even want our build management system to check the other components, so I launch the compilation in a given subdirectory, in a given component.

Aaaand my alias didn't work... After some googling around, I learnt that aliases are always executed from the root of the repository. This means that if you are in `/home/auser/myrepo/mycomponent` and you call `git da myFileInMyComp`, it will be searched in `/home/auser/myrepo` and it will ruthlessly fail.

I also found that the variable `${GIT_PREFIX}` holds your current path, so I simply prefixed my chain of commands with change directory to where ${GIT_PREFIX} points  to:

```
cd ${GIT_PREFIX} && git diff $1 && addprev $1"
```

### Things are often relative

This solution was working as long as I was calling `git da` from a subcomponent but it stopped working from the repository root. After all, I was in the exact opposite situation than before. But why... - I asked myself.

First, let's see what is `GIT_PREFIX`. It is set as returned by `git rev-parse --show-prefix` from the current directory. Which means that it will return the path relative to your root repo.

But what if you are in the root? Then your relative path is just nothing. But what is nothing in programming? Well, it can be many things. Zero, an empty string, a null pointer, etc.

In our case, `${GIT_PREFIX}` is simply not defined and if you do `cd ${UNDEFINED_VARIABLE_NAME}` you will end up in your home directory.

Like this, it's straightforward why my solution didn't work.

### Let's fix it

Now I understood that I only have to change directories if I'm not in the root. I wrote a small function to make that happen:

```
if [ -n "${GIT_PREFIX}" ]; then \
  cd ${GIT_PREFIX} ; \
fi \
} ; \
```

This is working fine and I'm using it both for side projects and for work. Here is my complete solution:

```
[alias]
    da = "! addprev() { while true; do \
          read -p "Do you wish to add this file? ([Y]es, [N]o, [P]atch)" yn ; \
          case $yn in \
          [Yy]* ) git add $1; break;; \
          [Pp]* ) git add -p $1; break;; \
          [Nn]* ) exit;; \
          * ) echo "Please answer yes, no or patch.";; \
          esac \
        done } ; \
        gotoUsedDirectory() { \
        if [ -n "${GIT_PREFIX}" ]; then \
          cd ${GIT_PREFIX} ; \
        fi \
        } ; \
        gotoUsedDirectory && git diff $1 && addprev $1"
```

## Conclusion

In this post, I shared my workflow of how I create my commits so that I'm my very first code reviewer before creating the commit. I also showed how I eliminated some repetitive commands from this workflow by crafting an interactive git alias.

The most important takeaway is that you can access shell any time which from git aliases giving you endless possibilities to create the alias you want.

## Call to action

If you like the idea and you think it would enhance your workflow, feel free to take [this small gist](https://gist.github.com/sandordargo/ce3a55be6bd794be1826391ebe95718b) and start using it - copy pasting from this page might now work due to character escape issues. If you'd like to be notified of my new posts, follow me on [Twitter](https://twitter.com/SandorDargo).