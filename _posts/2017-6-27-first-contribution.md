---
layout: post
title: "I made my first open source contribution"
date: 2017-6-27
category: dev
tags: [open source, contributions, protocol buffer]
header: "I've been <strike>agonizing</strike> looking for a while for the possibility to make my first contribution to an open source repository other than mine."
---
Finally I got it!

In my new project we use a maven plugin to generate files from `protobuf` code with `protoc`. On the Windows machines the build worked fine, but on my Linux one the build failed.

After some investigation it turned out, that it is because of a certain maven plugin which is responsible for downloading protoc corresponding to your system, unzip it and generate the files.

First as I had to proceed with the setup I followed what one of my colleagues suggested me and cloned the [protobuf repository](https://github.com/google/protobuf), built the tool and put into my path. Of course it was not enough, but luckily the plugin lets you override from where you want to run protoc. If it's not defined, only then it will download it.

Fair enough, it worked. However it already distinguished my repo from the others' by specifying the "protoc command". As I had some time now, I started to investigate the source code of the plugin after cloning it. The unit tests were failing on my system becauase of a permission denied exception. First I thought that it is misleading as I do have all the rights on my `/tmp`. After some time it turned out that it was not misleading at all. My system has been setup according to the [Securing Debian Manual](https://www.debian.org/doc/manuals/securing-debian-howto/ch4.en.html#s4.10.1) and `/tmp` has been mounted as `noexec`. As such it is not possible to execute any file from that mount so the file generation failed as protoc was unzipped to `/tmp`.

Having identified the problem and understanding it made it evident that it's the time to make that contribution. I opened an issue on [github](https://github.com/) and offered my help.

My help were welcome, soon I opened my pull request. It turned out that my strategy was quite different from the repository owner's, so I had to modify my PR sometimes, but we managed to collaborate and recently the new version including our fix has been published. 

I wanted a really specific error handling with a bit longer but more expressive code. The error handling part would have run only if strict conditions fullfilled. This also included checking the attributes of `/tmp`. He wanted something else, he preferred much simpler code, but which attempts a recovery in more circumstances. In my opinion both has its pros and cons and we went into the direction he preferred.

Still I'm happy because the problem is fixed and I have the feeling of accomplishment and the happiness of giving something back for a free tool.

Long live the open source!