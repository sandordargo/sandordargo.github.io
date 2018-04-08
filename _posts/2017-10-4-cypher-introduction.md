---
layout: post
title: "Introduction to cypher"
date: 2017-10-4
category: dev
tags: [database, graphs, cypher, neo4j]
header: "When you start working with a database for the first time, maybe one of the most important things is the ability to query it."
---
Put it differently, there are many ways to start exploring a database. You can learn about the paradigms behind that I think is the most important. It is essential to understand it on a high level how your data is stored and organized. For [Neo4j](https://neo4j.com/) I already covered this in [the introduction to graph databases](/blog/2017/09/06/intro-to-graph-databases). 

But I don't think it's important at the beginning of your journey to understand how the data is organized on a low level. What data structures are used in order to store tables, or documents, nodes, and relationships. How these structures point to each other. That can come later.

I think it's better to jump right in where the fun is and start exploring your data and query it many different ways.

Then as a next step you can experiment with the different drivers from your preferred language(s) and after or in parallel with that you can explore the underlying systems from a closer aspect.

For [Neo4j](https://neo4j.com/) - the graph database of my choice - the query language is called [Cypher](https://neo4j.com/developer/cypher-query-language/). As I assume that if you are already interested in querying a graph database you know things about SQL. I will not be afraid of making connections between Cypher and SQL in this series.

I am going to write a series of articles about the concepts and different keywords of Cypher.

Let's make a little recap before we start.

Neo4j is graph database implementing the labeled property graph data model. A lot of words, let's cover them one by one:

* Graph:
It consists of nodes and relationships between the nodes. The relationships are always directed, they have a type, a start, and an end node. There are no broken relationships, that means each relationship must have both a start and an end point.

* Property graph:
A previously described graph can hold some attributes. An attribute means basically a key-value pair. Both nodes and relationship can have such attributes/properties. As examples, imagine Joe who has a birthday attribute: `{birthDay: 25/01/1985}`. And for a relationship, let's say that there is a `:MANAGES` relationship between Jim and Joe. The key of an attribute on `:MANAGES` can be `since` and the value is `2017`.

* Labeled property graph:
On the top of this, nodes can have labels. Labels are tags which can represent different roles within your domain. Jim can a have a label of `:EMPLOYEE` and R&D  can be `:DEPRATMENT`. A node can be tagged with multiple labels.

To finish this post, let's see how easy it is to write down a relationship in Cypher.

Jim who is an employee is member of R&D department since 2017. The department has a size of 250 people.

`(jim:EMPLOYEE {name: "Jim"})-[:MEMBER_OF {since: 2017}]->(rnd:DEPARTMENT {name: "R&D", size:250})`

Some characteristics that you can see immediately:
* nodes are described between parentheses
* relationships are written between brackets
* labels of a node and the type of a relationship will come after a colon
* attributes are put between braces in a JSON style
* the direction of the relationship is visible through the direction of the arrow

Next time I'm going to [introduce the `CREATE` keyword](/blog/2017/10/11/cypher-create) to you so that we can create our first small graph.

Stay tuned and if you want to get notified of the latest articles follow me on [Twitter](https://twitter.com/SandorDargo)!