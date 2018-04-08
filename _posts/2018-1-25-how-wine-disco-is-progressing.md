---
layout: post
title: "How Wine-Disco is progressing?"
date: 2018-1-25
category: dev
tags: [open-source, wine-disco, neo4j, graphs]
header: "As <a href=\"/blog/2017/07/11/meet-neo4j-again\">I wrote in July</a> I had plans to write an application to browse information collected about Hungarian wines."
---
At the time I was building a Neo4j database including the wine regions and sub regions in Hungary, just like rhe different types of grapes grown at each. Now I kind of done with this and each week I dedicate a one or two pomodori to gather Hungarian wineries.

Well, [Uncle Bob](http://blog.cleancoder.com/) writes totally against this in his book [Clean Architecture](http://amzn.to/2DL4hj7). He says that such decisions, like one about the database should be deferred as much as possible. That's how he built FitNesse for example. I think he is right. But in my case, I was looking for an application to support my learning about Neo4j.

I said that I want this application to be online by the end of September. It's been up and running since the 29th of October, so I missed my target a month ago. It was already working on my local, but that doesn't count.

I had some issues with finding free hosting for my application. I had to find a place for the database and for the web app itself. Prefereably at the same place but it didn't work out. The first place I tried was Python Anywhere. I liked the platform, but the free plan didn't work out well for me. It doesn't host databases and although it supports connections to [graphenedb](https://www.graphenedb.com/), but only via http and I wanted to use [bolt](https://neo4j.com/blog/neo4j-3-0-language-drivers/). In fact I was lazy and didn't want to modify those few classes. 

I ended up using [Heroku](https://www.heroku.com/home), which I think is also fine. In the beginning I found a bit more difficult to deploy my application, but once I learnt how to do it, it's simple. Now it's even connected to the hosting github repository, so I have continuous deployment. Though continuous integration is missing, I should think about that.

I host my database at [grapehedb](https://www.graphenedb.com/) which has some free plans for small databases and so far so good for me. 

I feel that these free services doesn't provide high availabitity and high speed, but it doesn't really matter for the moment. I have an application, it's online and available.

Even though now it provides quite some information, it is quite ugly. So my next steps are to give it a nicer looks and a bit better navigation between the nodes. Even tough I think that the navigation is not that bad since I removed a few bugs and I added navigation though hyperlinks from the details section.

Please leave a note here or send me a mail if you have any constructive criticism!