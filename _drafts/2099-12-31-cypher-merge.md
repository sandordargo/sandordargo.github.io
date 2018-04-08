Let's get back to my Cypher tutorial and continue with MERGE after our having discussed CREATE, MATCH - OPTIONAL MATCH and RETURN.

When I first ran into MERGE I had a feeling that its name is strange, better to say, it's misleading. I still think it is. I don't write down what I thought it would do to not give you false ideas without any intention, but I guess you understand what I think about.

Instead, MERGE will make sure that a pattern exists in the queried graph. If the pattern described in the MERGE clause, fine, it not iw will be created.

A very important point is that you have to be very cautious if you don't want duplicate elements in your graph.

Think about the following pattern:

```
(:COMPETITION {name: `Dargo Wine Cup`})-[:LAST_WINNER]->(:WINE {name: `NiceSyrah 2016`})
```
If you run the followig query and the pattern exists in the graph, nothing will be created.

```
MERGE (competition:COMPETITION {name: `Dargo Wine Cup`})-[:LAST_WINNER]->(wine:WINE {name: `NiceSyrah 2016`})
```

On the other hand, image that both the `wine` and the `competition` exist in the graph, but there is no relationship so far between them. If you run the above query in such a situation both, the `competition` and the `wine` nodes will be created. So you will have duplicate nodes. 

In case you have some constraints ensuring unicity, an error will be thrown. 

I ran into this problem a few times.

How can you avoid this issue? 

Bound the variables before!

```
MATCH (competition:COMPETITION {name: `Dargo Wine Cup`})
MATCH (wine:WINE {name: `NiceSyrah 2016`})
MERGE (competition)-[:LAST_WINNER]->(wine)
```

Using the above query, no duplication will be created!

https://neo4j.com/developer/kb/understanding-how-merge-works/
http://neo4j.com/docs/developer-manual/current/cypher/clauses/merge/