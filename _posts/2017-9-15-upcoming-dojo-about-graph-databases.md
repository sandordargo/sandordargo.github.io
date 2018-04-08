---
layout: post
title: "Upcoming coding dojo about graph databases"
date: 2017-9-15
category: dev
tags: [databases, graphs, cypher, neo4j, java, dojo]
header: "On 5th October 2017 at <a href=\"http://www.telecom-valley.fr/5-octobre-soiree-test-logiciel/\">Telecom Valley</a> there will be an event dedicated to software testing where I will facilitate a dojo about test driven development with graph databases."
---
Let me give you some details to arouse your interest in joining this event if you are around [Sophia Antipolis](https://en.wikipedia.org/wiki/Sophia_Antipolis). 

I will briefly explain what a graph is and why they matter in IT. Then I will speak about the power of graph databases and also show an example of a [property graph](https://neo4j.com/developer/graph-database/). I will keep this part short in order to have more time for coding!

During the bigger time of the session we will work directly on a code kata I am still preparing. We will implement some services reading from a graph database.

The idea is that I make one service available in the starting code and the you will have to implement another two services reading from the underlying database and you will also have to evolve the first one. A very important aspect is that we're going to do this exercise in a test driven way. So you will have to write your tests first and I will show how you can instantiate an in-memory Neo4j instance. In other words in your test code you will use a lightweight non-persistent database while your production service will not care if it receives an in-memory database or access to a persistent data store.

I'm flexible about the format of the dojo. However I don't like people staring lonely at their screens. So depending on the number of participants we'll do either pair or mob programming, a.k.a. [the randori style](https://code.joejag.com/2009/the-coding-dojo.html). 

We'll wrap up by looking at what we did and I will show a practical use of these services visualizing the results.

If you are around Sophia Antipolis and interested in software testing, graph databases or any other topics of the event, don't hesitate to make your registration! It's free, but don't forget to [subscribe](https://www.eventbrite.fr/e/billets-soiree-du-test-logiciel-telecom-valley-37721625397) before 2nd October 2017. What you need for the dojo is a laptop with maven and Java 8 SDK installed.