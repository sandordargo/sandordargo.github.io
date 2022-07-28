Last year, as the usage of our services grew sometimes by 20 times, we had to spend significant efforts on optimizing our application. Although these are C++ backed services, our focus was not on optimizing the code. We had to change certain things, but removing not needed database connections I wouldn't call performance optimiziation. It was rather fixing a bug.

In my experience, while perofrmance optimization is an important thing, often the bottleneck is about latency. It's either about the network or the database.

Checking some of our metrics, we saw some frontend queueing every hour.

Long story short, it was about a materialized view. We introduced it for better performance, but seemingly it didn't help enough.

What could we do?

The view was refreshed every hour. A refresh meant that the view was dropped, then in a few seconds a new was built. The few seconds of downtime was enough to build up a queue.

We founf a setting to *********** With that the new view was built up while the old one was still in use. Then once ready, Oracle started to use the new view and drop the old.

The queueing vanished.

We traded some space for time.

The idea is not exclusive to databases obviously. In C++, there is a similar concept, an idiom, called copy and swap.