---
layout: post
title: "How I set up my Git"
date: 2018-10-17
category: dev
tags: [git, environment, tricks]
excerpt_separator: <!--more-->
---
This is post is not yet another one about Git aliases. Some like them, some don't. I use only a few. For the commands, I use the most and the ones I wouldn't forget anyway. In other words, I'm lazy to write `status` and `commit` all the time. These aliases are not worth a post. On the other hand, I modify my prompt in the terminal to give me a lot of information about the current state of the repository I'm at and I also use auto-completion for git commands.
<!--more-->

When I'm not in a git repository, my prompt is not altered. This is the default state of my prompt:

![Not in a git repo]({{ site.baseurl }}/assets/img/gitprompt-no-repo.png "Not in a git repo")

Let's create a new git repository by issuing the `git init` command right here on spot and see what's going to happen. As you can see on the below image we are on the master branch, plus we see a hashtag. That hashtag shows that we are still in the init stage, the repository initialization has not finished yet.

![After a git init]({{ site.baseurl }}/assets/img/gitprompt-after-init.png "After a git init")

Let's create a new file (i.e. _somefile.txt_) and commit it. Now our repository is in a clean state. We will see only the name of our current branch.

So if I'm in a git repository and it has a clean state, I'll simply see the name of the current branch:

![In a clean branch]({{ site.baseurl }}/assets/img/gitprompt-clean.png "In a clean branch")

Now, I'm going to modify _somefile.txt_ and let's see how the prompt changes. A star appears after the branch name. That `*` means that I have an unstaged tracked file.

![Having an unstaged file]({{ site.baseurl }}/assets/img/gitprompt-unstaged.png "Having an unstaged file")

Let's add another file (_newfile.txt_) and stage it immediately. Our prompt changed again, we have both a `*` and `+`. As you might have guessed that `+` is to show that we have a staged file.

![Having a staged and an unstaged file]({{ site.baseurl }}/assets/img/gitprompt-staged-unstaged.png "Having a staged and an unstaged file")

Now let's stage _somefile.txt_ as well, so we have only staged files. As we expect, the `*` disappears we are left only with the `+`.

![Having only staged files]({{ site.baseurl }}/assets/img/gitprompt-staged.png "Having only staged files")

In order to finish our cycle and to have a clean repo again, we have to commit. Our working area is clean again. There are no unstaged or staged files.

![In a clean branch again]({{ site.baseurl }}/assets/img/gitprompt-clean-again.png "In a clean branch again")

## Key Takeaways

To wrap it up, my prompt will always show the checked out branch name or commit hash. Besides it will display a `*` if we have any unstaged change on a tracked file and `+` if we have any staged files.

It also gives you some extra information during rebases and when you resolve merge conflicts.

This can save you issuing a lot a of `git branch` or `git rev-parse --abbrev-ref HEAD` to see in which branch you are and in addition you won't need some of the `git status` commands as well as you'll know just by looking at your prompt if you are in a clean state or if you still have to stage some files while preparing your next commit.

## Hot to get it

I don't want to take the credit for this. The credit goes to Carolyn and Sarah who have [this super course about Git on Udacity](https://classroom.udacity.com/courses/ud775). We started to use git in my company a couple years ago and after some time I realized that if I use only the push/pull/commit/diff/status quintuple I will keep having troubles and I'll like neither git, neither my work.

I enrolled to [this course](https://classroom.udacity.com/courses/ud775) and understood better many concepts. They also provided a nice style-guide and these scripts. I'll be always grateful to them.

Linux and Mac users will need to save [this file auto-completion](https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash) and [this file for checking the state of your repo](https://raw.githubusercontent.com/git/git/master/contrib/completion/git-prompt.sh), and [this](https://www.udacity.com/api/nodes/3341718587/supplemental_media/bash-profile-course/download?_ga=1.37232743.672083044.1467344711) helps you to set up your prompt using the previous two files.

If you are on Windows or if you want more details, check out the [course](https://classroom.udacity.com/courses/ud775), it's for free!