---
layout: post
title: "Meet Neo4j again"
date: 2017-7-11
category: dev
tags: [databases, graphs, neo4j]
header: "Not much before my daughter was born, in the beginning of 2015 an intern at Amadeus presented about graph databases, namely about Neo4j. I was really impressed by the idea. I decided once I'll have a bit of time, I'll experiment with it a bit."
---
Then when my daughter arrived to our world, I spent a month at home. Even though being with a new born is a tough job, I could dedicate a little bit of time to learn, to stretch my mind every day. First I started to import and visualize geographical data such as countries, cities, airports and related properties. It was interesting, but I didn't follow up on it as I did not see it's imminent value in a world of Google Maps and OpenStreetMap.

However I started to work on another topic. I was thinking that representing a family as a graph would be a good use case. Much more convenient then storing data in tables with "relationships", foreign keys here and there. It was great, it made me learn more about [Neo4j](https://neo4j.com/), a little bit about [flask](http://flask.pocoo.org/) and [Bootstrap](http://getbootstrap.com/) as well. It was a nice full stack project. I didn't publish it as a service, only the code on [github](https://github.com/sandordargo/family-tree), but I did use it for my own. (N.B. this was before I started to learn about clean code) I think its biggest issue is the representation of the family tree, mostly because in my model a tree composed of multiple families' trees.

Then for quite some time I did not spend time on Neo4j, however I never forgot about it. Like a world traveller who left a lover in a distant city, I was waiting for a return.

Recently I saw an internal page advocating about Neo4j and looking for people who are interested in it. I think I was the very first one to like the page and I directly contacted Thomas, the author. Soon we had a cafe and he briefly introduced what we try to achieve with Neo4j (due to confidential reason, I can't write about that).

Thomas also told me that he'd like to have someone from Neo4j to make a presentation for Amadeus employees. And last week, [Rik](https://twitter.com/rvanbruggen) came and made a nice presentation about the graph databases and Neo4j. For me the presentation's technical part didn't reveal a lot of new things, but I really liked his presentation about [his beer dataset](https://neo4j.com/blog/fun-with-beer-and-graphs/) and the smaller workshop what we had afterwards about specific use cases and more techinal questions/requirements.

For me the most important outcome of this event was that I got inspired again to work a bit with Neo4j. I like Belgian beer, just like Hungarian or French (or any good) wine. I am just a laic and I think it is quite difficult to navigate in the world of wines and grapes and to find one you might like much more than something else. So I decided I am going build up a wine database starting with Hungarian wines.

First while I'm working on my previous pet project, I just collect data and build the database. For the moment, I populate the database, with wine regions, related grapes, then later with wineries, specifics of bottles and the specifics, typical aromas of grapes. And I'd also like to build a nice UI around it.

By the end of September it should be online.
