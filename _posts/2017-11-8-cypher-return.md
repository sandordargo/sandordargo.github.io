---
layout: post
title: "Cypher tutorial: the RETURN keyword"
date: 2017-11-8
category: dev
tags: [database, graphs, cypher, neo4j]
header: "After <a href=\"/blog/2017/10/25/cypher-match\">having mentioned RETURN in the tutorial on Cyper's MATCH</a>, let's see what exactly it can do for us. This will not be a long post."
---
As usual, let's examine what its counterpart can be in SQL. In this case it is definitely `SELECT`, however `RETURN` is a bit simpler. Putting that aside, the most important visual difference between them is that in Cypher you end a query with `RETURN` while in SQL you start one with `SELECT`. But in both cases, you use them to define what you want to return from your query. In its simplest form you can return everything:

```
MATCH (wineRegion:WineRegion)-[r]->(otherNode), 
RETURN *
```

Most of the times, you will return nodes and/or relationships:

```
MATCH (wineRegion:WineRegion)-[r]->(grape:Grape)
RETURN wineRegion
```

```
MATCH ()-[relationship:GROWS_AT]->()
RETURN relationship
```

```
MATCH (grape:Grape)-[relationship:GROWS_AT]->(subregion:SubRegion)
RETURN grape, relationship, subregion
```

As you can see if you want to return multiple elements, it's really easy, you can just list the elements separated by commas.

If you don't want to return the whole node, just a single property, it's also really easy. You specify the name of the property after the node/relationship name separated by a dot:

```
MATCH (wineRegion:WineRegion)
RETURN wineRegion.name
```

This is what we more or less already saw in [the `MATCH` tutorial](/blog/2017/10/25/cypher-match). Now let's see what else we can do.

It is worth to mention that we can return strings including special characters and we can also use variable names with a space inside. All we have to do is to put that string/variable name within `\``'s. 
```
MATCH (`wine Region`:WineRegion)
RETURN `wine Region`.name
```

In SQL for a nicer, sometimes more meaningful visualization of results you can name the columns you want to return. In other terms, you can define an alias for each column using the `AS` keyword. Nothing fancy here, you can do the same in Cypher. If you want to use a one-word alias you just simply write it after the `AS` keyword.

```
MATCH (wineRegion:WineRegion)
RETURN wineRegion.name AS WineRegionName
```

If you want to use a nicer description, you have to use `\``-s around the alias, just for returning any uncommon character as mentioned before.

```
MATCH (wineRegion:WineRegion)
RETURN wineRegion.name AS `Wine region's name`
```

In certain cases, you might not be sure whether a certain property is there or not. It's not a problem. You can still select it and you will not have to face any nasty errors. If the property is not there, you will have a null.

Just like in SQL, also in Cypher there is a `DISTINCT` keyword. With it you can return only unique rows from your query. Do you remember _Eger_ which is both the name of a region and a subregion? If you query them without specifying the label, just mentioning the name itself and you return only the name, instead of two rows, only one will be returned.

```
MATCH (n)
WHERE n.name = "Eger"
RETURN DISTINCT n.name
```

The same can be done with relationships too. However there is something you can do with relationships, but not with nodes. To return their types. As we already it's possible to return relationships without specifying their type or direction. In this case from the `RETURN` clause using `type()` function you can get their type:

```
MATCH (region:WineRegion{name:'Eger'})-[rel:CONTAINS]->(subregion:WineSubRegion)
RETURN region, subregion, rel, type(rel)
```

Let's not forget that we can put some logic into our `RETURN`. We can use it to decide if a wine region is "big" for example. Here big means that it has more than 3 subregions.

```
MATCH (`wine Region`:WineRegion)-[]-(subregion:WineSubRegion)
RETURN distinct `wine Region`, count(subregion) > 3 as `Is it a big region`
```

The result will be this on my dataset:

| "wine Region"             | "Is it a big region" |
| ------------------------- | -------------------- |
| {"name":"Eger"}           | false                |
| {"name":"Sopron"}         | false                |
| {"name":"Pannon"}         | true                 |
| {"name":"Duna"}           | false                |
| {"name":"Észak-Dunántúl"} | true                 |
| {"name":"Tokaj"}          | false                |
| {"name":"Balaton"}        | true                 | 


But I could even list those relationships, but it's a bit hard to read the result:

```
MATCH (`wine region`:WineRegion)-[]-(subregion:WineSubRegion)
RETURN distinct `wine region`, count(subregion) > 3 as `Is it a big region`, (`wine region`)-[]-(subregion)
```


|"wine region"            | "Is it a big region" | "(`wine region`)-[]-(subregion)"                                      |
| ----------------------- | ------------------   | -------------------------------------------------------------------- |
|{"name":"Sopron"}        | false                |[{"start":{"identity":5,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Sopron"}},"end":{"identity":27,"labels":["WineSubRegion"],"properties"|
|                         |                      |:{"name":"Sopron"}},"segments":[{"start":{"identity":5,"labels":["Wine|
|                         |                      |Region"],"properties":{"name":"Sopron"}},"relationship":{"identity":20|
|                         |                      |,"start":5,"end":27,"type":"CONTAINS","properties":{}},"end":{"identit|
|                         |                      |y":27,"labels":["WineSubRegion"],"properties":{"name":"Sopron"}}}],"le|
|                         |                      |ngth":1}]                                                             |
|{"name":"Pannon"}        | false                |[{"start":{"identity":4,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Pannon"}},"end":{"identity":25,"labels":["WineSubRegion"],"properties"|
|                         |                      |:{"name":"Tolna"}},"segments":[{"start":{"identity":4,"labels":["WineR|
|                         |                      |egion"],"properties":{"name":"Pannon"}},"relationship":{"identity":18,|
|                         |                      |"start":4,"end":25,"type":"CONTAINS","properties":{}},"end":{"identity|
|                         |                      |":25,"labels":["WineSubRegion"],"properties":{"name":"Tolna"}}}],"leng|
|                         |                      |th":1}]                                                               |
|{"name":"Balaton"}       | false                |[{"start":{"identity":0,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Balaton"}},"end":{"identity":9,"labels":["WineSubRegion"],"properties"|
|                         |                      |:{"name":"Balaton-felvidék"}},"segments":[{"start":{"identity":0,"labe|
|                         |                      |ls":["WineRegion"],"properties":{"name":"Balaton"}},"relationship":{"i|
|                         |                      |dentity":2,"start":0,"end":9,"type":"CONTAINS","properties":{}},"end":|
|                         |                      |{"identity":9,"labels":["WineSubRegion"],"properties":{"name":"Balaton|
|                         |                      |-felvidék"}}}],"length":1}]                                           |
|{"name":"Pannon"}        | false                |[{"start":{"identity":4,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Pannon"}},"end":{"identity":23,"labels":["WineSubRegion"],"properties"|
|                         |                      |:{"name":"Pécs"}},"segments":[{"start":{"identity":4,"labels":["WineRe|
|                         |                      |gion"],"properties":{"name":"Pannon"}},"relationship":{"identity":16,"|
|                         |                      |start":4,"end":23,"type":"CONTAINS","properties":{}},"end":{"identity"|
|                         |                      |:23,"labels":["WineSubRegion"],"properties":{"name":"Pécs"}}}],"length|
|                         |                      |":1}]                                                                 |
|{"name":"Balaton"}       | false                |[{"start":{"identity":0,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Balaton"}},"end":{"identity":11,"labels":["WineSubRegion"],"properties|
|                         |                      |":{"name":"Nagy-Somló"}},"segments":[{"start":{"identity":0,"labels":[|
|                         |                      |"WineRegion"],"properties":{"name":"Balaton"}},"relationship":{"identi|
|                         |                      |ty":4,"start":0,"end":11,"type":"CONTAINS","properties":{}},"end":{"id|
|                         |                      |entity":11,"labels":["WineSubRegion"],"properties":{"name":"Nagy-Somló|
|                         |                      |"}}}],"length":1}]                                                    |
|{"name":"Pannon"}        | false                |[{"start":{"identity":4,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Pannon"}},"end":{"identity":26,"labels":["WineSubRegion"],"properties"|
|                         |                      |:{"name":"Villány"}},"segments":[{"start":{"identity":4,"labels":["Win|
|                         |                      |eRegion"],"properties":{"name":"Pannon"}},"relationship":{"identity":1|
|                         |                      |9,"start":4,"end":26,"type":"CONTAINS","properties":{}},"end":{"identi|
|                         |                      |ty":26,"labels":["WineSubRegion"],"properties":{"name":"Villány"}}}],"|
|                         |                      |length":1}]                                                           |
|{"name":"Balaton"}       | false                |[{"start":{"identity":0,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Balaton"}},"end":{"identity":8,"labels":["WineSubRegion"],"properties"|
|                         |                      |:{"name":"Balatonboglár"}},"segments":[{"start":{"identity":0,"labels"|
|                         |                      |:["WineRegion"],"properties":{"name":"Balaton"}},"relationship":{"iden|
|                         |                      |tity":1,"start":0,"end":8,"type":"CONTAINS","properties":{}},"end":{"i|
|                         |                      |dentity":8,"labels":["WineSubRegion"],"properties":{"name":"Balatonbog|
|                         |                      |lár"}}}],"length":1}]                                                 |
|{"name":"Eger"}          | false                |[{"start":{"identity":2,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Eger"}},"end":{"identity":16,"labels":["WineSubRegion"],"properties":{|
|                         |                      |"name":"Bükk"}},"segments":[{"start":{"identity":2,"labels":["WineRegi|
|                         |                      |on"],"properties":{"name":"Eger"}},"relationship":{"identity":9,"start|
|                         |                      |":2,"end":16,"type":"CONTAINS","properties":{}},"end":{"identity":16,"|
|                         |                      |labels":["WineSubRegion"],"properties":{"name":"Bükk"}}}],"length":1}]|
|{"name":"Duna"}          | false                |[{"start":{"identity":1,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Duna"}},"end":{"identity":13,"labels":["WineSubRegion"],"properties":{|
|                         |                      |"name":"Csongrád"}},"segments":[{"start":{"identity":1,"labels":["Wine|
|                         |                      |Region"],"properties":{"name":"Duna"}},"relationship":{"identity":6,"s|
|                         |                      |tart":1,"end":13,"type":"CONTAINS","properties":{}},"end":{"identity":|
|                         |                      |13,"labels":["WineSubRegion"],"properties":{"name":"Csongrád"}}}],"len|
|                         |                      |gth":1}]                                                              |
|{"name":"Észak-Dunántúl"}| false                |[{"start":{"identity":3,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Észak-Dunántúl"}},"end":{"identity":22,"labels":["WineSubRegion"],"pro|
|                         |                      |perties":{"name":"Pannonhalma"}},"segments":[{"start":{"identity":3,"l|
|                         |                      |abels":["WineRegion"],"properties":{"name":"Észak-Dunántúl"}},"relatio|
|                         |                      |nship":{"identity":15,"start":3,"end":22,"type":"CONTAINS","properties|
|                         |                      |":{}},"end":{"identity":22,"labels":["WineSubRegion"],"properties":{"n|
|                         |                      |ame":"Pannonhalma"}}}],"length":1}]                                   |
|{"name":"Balaton"}       | false                |[{"start":{"identity":0,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Balaton"}},"end":{"identity":10,"labels":["WineSubRegion"],"properties|
|                         |                      |":{"name":"Balatonfüred-Csopak"}},"segments":[{"start":{"identity":0,"|
|                         |                      |labels":["WineRegion"],"properties":{"name":"Balaton"}},"relationship"|
|                         |                      |:{"identity":3,"start":0,"end":10,"type":"CONTAINS","properties":{}},"|
|                         |                      |end":{"identity":10,"labels":["WineSubRegion"],"properties":{"name":"B|
|                         |                      |alatonfüred-Csopak"}}}],"length":1}]                                  |
|{"name":"Észak-Dunántúl"}| false                |[{"start":{"identity":3,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Észak-Dunántúl"}},"end":{"identity":20,"labels":["WineSubRegion"],"pro|
|                         |                      |perties":{"name":"Mór"}},"segments":[{"start":{"identity":3,"labels":[|
|                         |                      |"WineRegion"],"properties":{"name":"Észak-Dunántúl"}},"relationship":{|
|                         |                      |"identity":13,"start":3,"end":20,"type":"CONTAINS","properties":{}},"e|
|                         |                      |nd":{"identity":20,"labels":["WineSubRegion"],"properties":{"name":"Mó|
|                         |                      |r"}}}],"length":1}]                                                   |
|{"name":"Pannon"}        | false                |[{"start":{"identity":4,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Pannon"}},"end":{"identity":24,"labels":["WineSubRegion"],"properties"|
|                         |                      |:{"name":"Szekszárd"}},"segments":[{"start":{"identity":4,"labels":["W|
|                         |                      |ineRegion"],"properties":{"name":"Pannon"}},"relationship":{"identity"|
|                         |                      |:17,"start":4,"end":24,"type":"CONTAINS","properties":{}},"end":{"iden|
|                         |                      |tity":24,"labels":["WineSubRegion"],"properties":{"name":"Szekszárd"}}|
|                         |                      |}],"length":1}]                                                       |
|{"name":"Eger"}          | false                |[{"start":{"identity":2,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Eger"}},"end":{"identity":18,"labels":["WineSubRegion"],"properties":{|
|                         |                      |"name":"Mátra"}},"segments":[{"start":{"identity":2,"labels":["WineReg|
|                         |                      | ion"],"properties":{"name":"Eger"}},"relationship":{"identity":11,"sta|
|                         |                      |rt":2,"end":18,"type":"CONTAINS","properties":{}},"end":{"identity":18|
|                         |                      |,"labels":["WineSubRegion"],"properties":{"name":"Mátra"}}}],"length":|
|                         |                      |1}]                                                                   |
|{"name":"Eger"}          | false                |[{"start":{"identity":2,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Eger"}},"end":{"identity":17,"labels":["WineSubRegion"],"properties":{|
|                         |                      |"name":"Eger"}},"segments":[{"start":{"identity":2,"labels":["WineRegi|
|                         |                      |on"],"properties":{"name":"Eger"}},"relationship":{"identity":10,"star|
|                         |                      |t":2,"end":17,"type":"CONTAINS","properties":{}},"end":{"identity":17,|
|                         |                      |"labels":["WineSubRegion"],"properties":{"name":"Eger"}}}],"length":1}|
|                         |                      |]                                                                     |
|{"name":"Balaton"}       | false                |[{"start":{"identity":0,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Balaton"}},"end":{"identity":12,"labels":["WineSubRegion"],"properties|
|                         |                      |":{"name":"Zala"}},"segments":[{"start":{"identity":0,"labels":["WineR|
|                         |                      |egion"],"properties":{"name":"Balaton"}},"relationship":{"identity":5,|
|                         |                      |"start":0,"end":12,"type":"CONTAINS","properties":{}},"end":{"identity|
|                         |                      |":12,"labels":["WineSubRegion"],"properties":{"name":"Zala"}}}],"lengt|
|                         |                      |h":1}]                                                                |
|{"name":"Duna"}          | false                |[{"start":{"identity":1,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Duna"}},"end":{"identity":15,"labels":["WineSubRegion"],"properties":{|
|                         |                      |"name":"Kunság"}},"segments":[{"start":{"identity":1,"labels":["WineRe|
|                         |                      |gion"],"properties":{"name":"Duna"}},"relationship":{"identity":8,"sta|
|                         |                      |rt":1,"end":15,"type":"CONTAINS","properties":{}},"end":{"identity":15|
|                         |                      |,"labels":["WineSubRegion"],"properties":{"name":"Kunság"}}}],"length"|
|                         |                      |:1}]                                                                  |
|{"name":"Észak-Dunántúl"}| false                |[{"start":{"identity":3,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Észak-Dunántúl"}},"end":{"identity":21,"labels":["WineSubRegion"],"pro|
|                         |                      |perties":{"name":"Neszmély"}},"segments":[{"start":{"identity":3,"labe|
|                         |                      |ls":["WineRegion"],"properties":{"name":"Észak-Dunántúl"}},"relationsh|
|                         |                      |ip":{"identity":14,"start":3,"end":21,"type":"CONTAINS","properties":{|
|                         |                      |}},"end":{"identity":21,"labels":["WineSubRegion"],"properties":{"name|
|                         |                      |":"Neszmély"}}}],"length":1}]                                         |
|{"name":"Észak-Dunántúl"}| false                |[{"start":{"identity":3,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Észak-Dunántúl"}},"end":{"identity":19,"labels":["WineSubRegion"],"pro|
|                         |                      |perties":{"name":"Etyek-Buda"}},"segments":[{"start":{"identity":3,"la|
|                         |                      |bels":["WineRegion"],"properties":{"name":"Észak-Dunántúl"}},"relation|
|                         |                      |ship":{"identity":12,"start":3,"end":19,"type":"CONTAINS","properties"|
|                         |                      |:{}},"end":{"identity":19,"labels":["WineSubRegion"],"properties":{"na|
|                         |                      |me":"Etyek-Buda"}}}],"length":1}]                                     |
|{"name":"Tokaj"}         | false                |[{"start":{"identity":6,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Tokaj"}},"end":{"identity":28,"labels":["WineSubRegion"],"properties":|
|                         |                      |{"name":"Tokaj"}},"segments":[{"start":{"identity":6,"labels":["WineRe|
|                         |                      |gion"],"properties":{"name":"Tokaj"}},"relationship":{"identity":21,"s|
|                         |                      |tart":6,"end":28,"type":"CONTAINS","properties":{}},"end":{"identity":|
|                         |                      |28,"labels":["WineSubRegion"],"properties":{"name":"Tokaj"}}}],"length|
|                         |                      |":1}]                                                                 |
|{"name":"Balaton"}       | false                |[{"start":{"identity":0,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Balaton"}},"end":{"identity":7,"labels":["WineSubRegion"],"properties"|
|                         |                      |:{"name":"Badacsony"}},"segments":[{"start":{"identity":0,"labels":["W|
|                         |                      |ineRegion"],"properties":{"name":"Balaton"}},"relationship":{"identity|
|                         |                      |":0,"start":0,"end":7,"type":"CONTAINS","properties":{}},"end":{"ident|
|                         |                      |ity":7,"labels":["WineSubRegion"],"properties":{"name":"Badacsony"}}}]|
|                         |                      |,"length":1}]                                                         |
|{"name":"Duna"}          | false                |[{"start":{"identity":1,"labels":["WineRegion"],"properties":{"name":"|
|                         |                      |Duna"}},"end":{"identity":14,"labels":["WineSubRegion"],"properties":{|
|                         |                      |"name":"Hajós-Baja"}},"segments":[{"start":{"identity":1,"labels":["Wi|
|                         |                      |neRegion"],"properties":{"name":"Duna"}},"relationship":{"identity":7,|
|                         |                      |"start":1,"end":14,"type":"CONTAINS","properties":{}},"end":{"identity|
|                         |                      |":14,"labels":["WineSubRegion"],"properties":{"name":"Hajós-Baja"}}}],|
|                         |                      |"length":1}]                                                          |

That's it about the `RETURN` I hope you enjoyed this little tutorial. Stay tuned for the next episode where we will explore the 'MERGE' keyword.