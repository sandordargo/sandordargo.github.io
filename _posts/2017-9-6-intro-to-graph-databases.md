---
layout: post
title: "Introduction to graph databases"
date: 2017-9-6
category: dev
tags: [databases, graphs, cypher, neo4j]
header: "Let's start exploring the world of graph databases. Before I go into details regarding language drivers of Neo4j and the Cypher query language, let's see what a graph database is."
---
Or maybe let's start from a more generic point. What is a database? It is a datastore. In IT it is a set of data held in a computer. The most known datastores are still the good old relational databases which were invented in the 70's at IBM. Hm... Relational... Seems nice! A datastore created to easily handle relations, right?! Not exactly, but let's not run forward that much.

In a relational database we store data in different tables. Imagine that in a table we store employees and in another we store the teams of the company. How should we represent that an employee is part of a division? Most probably in the employee table we will create a column in order to store the team id for each employee and it will serve as a foreign key to the team table. Neat!

It's not even hard to visualize and really not difficult to write a query:

`SELECT employee.name, team.name FROM employee, tables WHERE employee.team_id = team.id`

It will get worse if our company does not only have employees and teams, but departments, divisions, agile tribes, God forbid squadrons. Now imagine you want to get the employees from a specific division. Here is a potential implementation:

`SELECT emplyee.name, divison.name FROM employee JOIN team ON emplyee.team_id = team.id JOIN departement ON team.departement_id = departement.if JOIN division ON department.divison_id = divison.id WHERE division.name = "R&D"`

This is not just ugly, but inefficient in a large enough dataset. The system will have to deal with a huge set of data. It will be slow. It's unreadable.

It would be great to store it differently, right? That's where graph databases come into place.

In math a graph is composed of nodes and edges and it will be the same in our domain as well.

Nodes represent records and edges represent relationships among them.

If we assume a similar data structure, we could visualize that Jim is part of R&D division as such:


![Simple relationships]({{ site.baseurl }}/assets/img/intro-to-graph-dbs-simple-relationships.svg)

And how would we query it:

`MATCH (jim:Employee)-[:MEMBER_OF]->(team)-[:PART_OF]->(department)-[:PART_OF]->(divison:Division) WHERE jim.name = 'Jim' AND division.neme = 'Division I.' RETURN jim, divison`

Great, as you can see, we store the data pretty much as we would imagine, visualize it. This is one of the greatest powers of graph databases. Development is relatively easy, because what you code and what you draw are pretty much the same.

But easy development and visualization means nothing if it will be as impractical as multiple joins in the realm of relational databases.

In math if you want to explore a graph, you will traverse it from node to node through edges between the corresponding nodes. That's what you'll do in a graph database too. You will jump from one node to another. Sounds like a need for pointers, right? Indeed, in an index-free adjacent graph there will be pointers between the nodes, that's why it's extremely cheap to traverse a graph.

One of the other big advantages of a graph database is that it's easy to change it. Most of the time it's cheap to shape it to your actual needs. You design a relational database for years or decades and you try to change it the least possible. That's not the case with graph databases. Let's imagine that we have a big need of querying someone's department. We can just create a new relationship in our database and simplify our query to:

`MATCH (jim:Employee)-[:MEMBER_OF_DIVISION]->(divison:Division) WHERE jim.name = 'Jim' AND division.neme = 'Division I' RETURN jim, divison`

Our graph would look like this after introducing this new realtionship:

![Multiple relationships]({{ site.baseurl }}/assets/img/intro-to-graph-dbs-multiple-relationships.svg)

Of course you can do the same thing in a relational database, but you will barely see people creating temporary relations and the dropping them.

Using a graph database is not always the best solution. In a later article I will explore this question more in details. Considering the current state of the technoligies, you won't necessairly want to use graph databases with a small dataset. You will enjoy the benefits of a graph database mostly when you deal with read-intensive, highly connected data, such as social networks, topologies, recommendation systems, etc.

I am going to post more and more articles about graph databases and mainly about [Neo4j](https://neo4j.com/). I am going to show you how to use [Cypher](https://neo4j.com/developer/cypher-query-language/) which is the query language for [Neo4j](https://neo4j.com/) and other graph databases. Then I will intorduce some of the driver which can be used for development with it and then later, we'll discover Test Driven Development with these drivers. Stay tuned.