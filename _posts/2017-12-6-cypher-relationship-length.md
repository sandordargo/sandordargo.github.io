---
layout: post
title: "Cypher tutorial: Relationships with a varying length"
date: 2017-12-6
category: dev
tags: [database, graphs, cypher, neo4j]
header: "<a href=\"/blog/2017/11/22/cypher-optional-match\">In one of my previous posts</a> I wrote about the OPTIONAL MATCH clause in Cypher. I briefly mentioned that in your queries you can match paths with a varying length. Let's see how."
---
Let's suppose we have a data model where we store all the winners of wine competitions. I mean virtually. There are nodes with the type of `:COMPETITION` and for each competition, we store the winning `:WINE`s in a linked list as such:

```
(:COMPETITION {name: `Dargo Wine Cup`})-[:LAST_WINNER]->(:WINE {name: `NiceSyrah 2016`})->[:PREVIOUS_WINNER]->(:WINE {name: `Great Cabernet Franc 2015`})->[:PREVIOUS_WINNER]->(:WINE {name: `Awesome Kadarka 2014`})
```

For the sake of simplicity, we ignore the other attributes of competitions and wines here.

Retrieving the winner of the last year is really simple:
```
MATCH (:COMPETITION {name:`Dargo Wine Cup`})-[:LAST_WINNER]->(winner)
RETURN winner
```

Retrieving the winner of the previous year is still an easy task:

```
MATCH (:COMPETITION {name:`Dargo Wine Cup`})-[:LAST_WINNER]->()->[:PREVIOUS_WINNER]->(winner)
RETURN winner
```

Retrieving winners of the previous years can be done using brute force when you write your queries, but it becomes tedious. Please, don't do this:

```
MATCH (:COMPETITION {name:`Dargo Wine Cup`})-[:LAST_WINNER]->()->[:PREVIOUS_WINNER]->()->[:PREVIOUS_WINNER]->(winner)
RETURN winner
```

Instead, you can explicitly declare the desired relationship length, so that you don't have to sketch multiple relationships of the same type connecting to each other:

```
MATCH (:COMPETITION {name:`Dargo Wine Cup`})-[:LAST_WINNER]->()->[:PREVIOUS_WINNER*2]->(winner)
RETURN winner
```

This means that from the node connected with `:LAST_WINNER` relationship to our competition we have to hop through wines twice, connected to each other by the `:PREVIOUS_WINNER` relationship.

We can easily infer that when you don't define the relationship length by putting a number following a `*` after the relationship type, there is an implicitly defined default length: `*1`.

Fixed multiple relationship length is already a nice feature, but there is even more. We can not just return something in a given distance, but we can define a range. If we want to retrieve all the winners of _Dargo Wine Cup_ from the last three years, we can do it as such:

```
MATCH (:COMPETITION {name:`Dargo Wine Cup`})-[:LAST_WINNER]->(last_winner)->[:PREVIOUS_WINNER*2..3]->(previous_winner)
RETURN last_winner, previous_winner
```

Let's say you want to return all the winners of `Dargo Wine Cup`, but you don't know when the competition first took place. Either you query that information and pass it to the previous query as a relationship length, or you just say that you want them all.

It is really easy, you just have to omit the number after the `*`:

```
MATCH (:COMPETITION {name:`Dargo Wine Cup`})-[:LAST_WINNER]->(last_winner)->[:PREVIOUS_WINNER*]->(previous_winner)
RETURN last_winner, previous_winner
```

Please keep in mind that this is quite a dangerous option. In a production environment, it can seriously harm your user's experience in a big enough database. So it's better to limit your query in a way that in extreme conditions it can still provide an answer in a tolerable time frame.

We have already seen that instead of the default length of 1, you can use bigger lengths, ranges, even infinite lengths, but I haven't mentioned that you set the length to zero.

Imagine that in our data model there are a lot of competitions including relatively new ones. By relatively new, I mean that they have only the last winner, they don't have winners from the previous years.

If you want to list all the winners of the last three years, you might want to use the following query:

```
MATCH (:COMPETITION)-[:LAST_WINNER]->(last_winner)->[:PREVIOUS_WINNER]->(previous_winner)->[:PREVIOUS_WINNER]->(previous_previous_winner)
RETURN last_winner, previous_winner, previous_previous_winner
```

This will return you all the winner wines from competitions where there are actually previous winners, but we skip all the winners of competitions which are not old enough to have winners from the last three years. Here we could use `OPTIONAL MATCH`, but it's even easier to use the 0 length:

```
MATCH (:COMPETITION)-[:LAST_WINNER]->(last_winner)->[:PREVIOUS_WINNER*0..2]->(previous_winner)
RETURN last_winner, previous_winner
```

_Source: https://graphaware.com/graphaware/2015/05/19/neo4j-cypher-variable-length-relationships-by-example.html_