---
layout: post
title: "How to share data in Jenkins?"
date: 2020-9-23
category: dev
tags: [cicd, jenkins, groovy, docker]
excerpt_separator: <!--more-->
---
I've spent quite some time during the last two months on Jenkins pipelines and I had to solve an interesting problem that I'm going to share with you in this article.
<!--more-->

We perform regular source code scans and binary scans for our applications, but the triggering process has been somewhat manual. I already automated both scans as much as I could, as much as I had ideas, but it was far from optimal, even if it was much better than in the other departments I know about.

Unfortunately, there was no way to put them into the regular CI/CD pipelines as one of the scans is really slow and with such a delay, devs would have done to me the worst of the worst and the other scan is asynchronous and it was not accepted to be triggered by the regular CI/CD.

Let's say that my automation was good enough for what I needed it.

But requirements and expectations changed and we needed something better.

As I already had some experience with these jobs, I volunteered for piloting the new solution. At that time, I didn't understand that piloting in this case would mean that I have to build most of it.

## Problem statement

* We needed a scheduled pipeline in Jenkins that executes two scans for various software components for several applications. 
* The scans should be parallelized as much as meaningful
* It should be easy if not effortless to add new components to the scan

What we built, in the end, was a scheduled job that at each run creates other one-off jobs and executes them immediately. One job that creates them all, and one job for each component of an application.

The scheduling workflow (a scheduled job capable of creating other jobs) was developed already by someone at another division, who guided me through with the rest. Benjamin was a great help and taught me a lot about Jenkins pipelines.

I only had to modify the scheduling workflow a little and that little part is what I'm focusing on today.

## Share data between docker images

I had to share a generated file between my Jenkinsfile and the workflow it was calling. As the file was generated on the fly, it was not possible to simply pass a Bitbucket URL as a parameter from one to the other. If I committed the file, I could have just passed the URL. This is something that I already made working as a first iteration, but I clearly didn't want to go with committed files as there can be many of them.

Unfortunately, I couldn't simply just look at the place in the filesystem where I generated the file, because the different parts of workflows were running on different docker images and I had no right to change this behaviour.

The file generation couldn't be delegated either, due to two reasons. I have no right to modify the scheduling workflow just anyhow and more importantly, it wouldn't have made any sense to give it a responsibility that doesn't belong there.

Sharing the content of the generated as a (quite long) text parameter could have been an option and I kept it as such - one last option. An option that I didn't want to try until I had any hope to resolve the problem in any other way. I don't find it elegant, even though Groovy's 65536 characters limit to Unicode strings would have been more than enough even in the long run.

## `stash`/`unstash` the Jenkins way

I was advised to use `stash`/`unstash`. First I was puzzled. What should I do with git here? But okay, let's give it a try. Then I realized that it's also [a pair of commands in Jenkins terminology](https://www.jenkins.io/doc/pipeline/steps/workflow-basic-steps/#stash-stash-some-files-to-be-used-later-in-the-build), it's simply not so widely known as its git counterpart. After all, fewer people configure Jenkins pipelines than work with git.

The usage seemed fairly simple, an easy way to share jobs between jobs in the same pipeline. What is important is that we are in a node context, but it was granted in my case. The main thing to pay attention to is that somehow I still have to share the `stash` name with the other workflow which would call the `unstash` command with the same name. 

This means, there is an observable difference compared to git un/stash, where you can simply `unstash` the lastly stashed items. In Jenkins, you must provide the name. And if you provide a wrong name, there is an exception thrown.

But the name is short, and it was easy to share it as a context parameter between my main job and the scheduling pipeline.

I went along the examples I found and I had no exception! 

Champagne?! 

Not yet.

I needed to copy the within the scheduling workflow, or in fact, rename the file would have been enough.

Again, I have to emphasize that I'm no way a Jenkins expert. So I went to Stackoverflow for copy-pasta and checked how to copy a text file:

```java
  src = new File('src.txt')
  dst = new File('dst.txt')
  dst << src.text
```

It didn't work! In fact, an exception was thrown because the source file didn't exist. Why? I didn't understand at that point. I couldn't understand where it can be, where it disappeared.

After all, `unstash` was successful, but the file was nowhere. I tried to list the current directory, I tried to list files a level higher, a lever deeper, nothing. I was in despair.

```java
// Listing all the files and subdirectories in a given path. 
// I didn'have the rights to call the eachFileRecurse call
local = new File('.')
local.eachFile {
    println(file)
}
```

Then I contacted my colleague once more. He told me that depending on whether I use Jenkins commands or Java/Groovy classes my current working directory can be completely different. And as `stash`/`unstash` are part of Jenkins, not Groovy, and I should try to use `readFile` and `writeFile` instead to make the copy.

Lo and behold, it worked right away. I could read in the file without a problem, and I could write into a new one without permission issues.

```java
 writeFile(file:fileName, text:readFile("unstashedFile"))
```

## Conclusion

The most important takeaway for me is that good cooperation is indispensable and we should always rely on the knowledge of people more experienced with a given technology. Even if what they say seems quite surprising. Like who would have thought that the local directory is not the same if I read a file with the `File` class and if read it with `readFile()`. In fact, most probably that's the moment - when they say something surprising - when we should pay even more attention.

Thinking about key learning from a technical aspect, I learned that I can share files in a Jenkins pipeline between different jobs, between different docker images trough `stash` and `unstash`, and I only have to make sure that the name is something that I can share between the different parts of the pipeline or something that we can compute or we know before.

The most interesting I learned that when in Jenkins, I should use Jenkins verbs as much as I can and not mix them with native Java and Groovy classes unnecessarily, especially when it comes to the filesystem. Otherwise, I might end not being able to read files, or writing them to somewhere else compared to where intended them to write.

All in all, I've been enjoying this project and my understanding of what is possible within the realms of CI/CD reached the next - still not to high - level.