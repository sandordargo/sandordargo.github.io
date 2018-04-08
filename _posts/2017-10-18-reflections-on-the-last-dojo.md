---
layout: post
title: "Reflections on the dojo I facilitated at Soiree du Test Logiciel"
date: 2017-10-18
category: dev
tags: [database, graphs, cypher, neo4j, dojo]
header: "As <a href=\"/blog/2017/09/15/upcoming-dojo-about-graph-databases\">I mentioned before</a> I had the chance to facilitate a dojo at an <a href=\"http://www.telecom-valley.fr/5-octobre-soiree-test-logiciel/\">Evening about Software Testing</a>. This was the first dojo I organized outside of my work environment and it taught me some important lessons."
---
​I admit the dojo did not go well. Among the prerequisites communicated to the participants, there were only two things: Java 8 and maven. Still, almost none of the participants showed up with such an environment.

​I had been thinking about creating a docker image, but it seemed to be an overkill. I still think it would have been. In addition, you still need docker on your machine which I think is rarer than Java and maven.

​Probably I should have asked for the e-mail addresses of the registered participants of the dojo and contact them directly asking them to make sure that their environment is fine. And at this point, I should have communicated the GitHub address so that they might compile the [code](https://github.com/sandordargo/neo-wine-services) and run the tests before they come.

I could have also ported the code to Python to have even fewer requirements. I will do that for sure!

But this event was not only about the dojo, I also made a presentation about the basic ideas of graph databases. I mostly covered the topics of [this](/blog/2017/09/06/intro-to-graph-databases) and [this](/blog/2017/10/04/cypher-introduction) articles. Actually, this part went well. I think that my audience appreciated the presentation. Only one person was aware of graph databases before and they liked the concepts I introduced.

That's already a success, they had ideas to take away. On my side, I also learnt a lot and still gained some confidence!

If you'd like to do the kata yourself, go to this [GitHub repository](https://github.com/sandordargo/neo-wine-services) and get started!