---
layout: post
title: "Getting familiar with Kafka"
date: 2017-6-15
category: dev
tags: [kafka]
header: "I've already started to play with Kafka in my new project. It's great that it's not just a plain promise that will be faced to new techs.
"
---
Even though my feature team won't communicate directly with Kafka, we'll use a wrapper organized by another feature team, it is important to understand what is underneath.

Hence first I just drafted a small app which first produces some records, then goes to sleep and then consumes them. The production was easy. I could see them through the CLI consumer, but couldn't consume them through my app. I tried a bunch of stuff, I checked tutorials, I upgraded my Kafka version. Nothing. In fact not nothing. Sometimes I managed to consume the messages, but then for a lot of restarts of the app, nothing again. 

Then I ran into another article where the groupid was not a constant but it was a UUID.

`configProperties.put(ConsumerConfig.GROUP_ID_CONFIG, UUID.randomUUID().toString());`

That was finally something different, so I tried it. And it worked like a charm. 

Only one instance of a consumer group will receive a message. Put it differently the messages are load balanced between the instances of a consumer group. The more I restarted the app the more instances I got in the same group I think. Even though the other processes were stopped, I think they still left some marks behind them, thus my new instance barely received any message. Of course when all my consumers started to belong to a unique group, this problem vanished.

So at that point I split the one dummy app, into two dummies. One produced and one consumed and on my two screens I could see both running at the same time. Great!

Next step is to do the same on OpenShift.