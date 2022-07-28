Last year, as the usage of our services grew sometimes by 20 times, we had to spend significant efforts on optimizing our application. Although these are C++ backed services, our focus was not on optimizing the code. We had to change certain things, but removing not needed database connections I wouldn't call performance optimiziation. It was rather fixing a bug.

In my experience, while perofrmance optimization is an important thing, often the bottleneck is about latency. It's either about the network or the database.

Checking some of our metrics, we saw some frontend queueing every hour.

Long story short, it was about a materialized view. We introduced it for better performance, but seemingly it didn't help enough.

What could we do?

The view was refreshed every hour. A refresh meant that the view was dropped, then in a few seconds a new was built. The few seconds of downtime was enough to build up a queue.

We founf a setting to *********** With that the new view was built up while the old one was still in use. Then once ready, Oracle started to use the new view and drop the old.

The queueing vanished.

We traded some space for time.

The idea is not exclusive to databases obviously. In C++, there is a similar concept, an idiom, called *copy-and-swap*.


But are the motivations the same?


Not exactly.

Even though I can imagine a situation where there is a global variable that can be used by different threads and it's cruical the time spent updating that variable, replacing it's value.

There is something more important.

It's about the safety of copy assignment. What is copy assignment about? You create a new object and assign it to an already existing variable. The object that was held by the existing variable gets destroyed.

So there is a construstion and a destruction. The first might fail, but a destruction must not.

Is that really the case in practice?

Not necessarily.

What often happens is that the assignment is performed from member to member.

```cpp
class MyClass {
 public:
  MyClass(int x, int y) : m_x(x), m_y(y) {}

  MyClass& operator=(const MyClass& other) {

    if (this != &other)
    {
      //Copy member variables
      m_x = other.m_x;
      m_y = other.m_y;
    }

    return *this;
  }

  // ...

 private:
  //Member variables
  int m_x;
  int m_y;
};
```

The problem is that what if it fails? Here we deal with simple POD members, but it could easily be something more complex. Something more error-prone. If the copies fail, if constructing any of those members fail, our object which we wanted to assign to remains in an inconsistent state. 

That is basic exception safety at best. Even if all the values remain valid, they might different from the original.

If we want strong exception safery, the copy-and-swap idiom will help us achieving that.

The constructions might fail, but destruction must not.

> [The C++ standard library provides several levels of exception safety (in decreasing order of safety)](https://en.wikipedia.org/wiki/Exception_safety#Classification):
> 1) No-throw guarantee, also known as failure transparency: Operations are guaranteed to succeed and satisfy all requirements even in exceptional situations. If an exception occurs, it will be handled internally and not observed by clients.
> 2) Strong exception safety, also known as commit or rollback semantics: Operations can fail, but failed operations are guaranteed to have no side effects, leaving the original values intact.[9]
Basic exception safety: Partial execution of failed operations can result in side effects, but all invariants are preserved. Any stored data will contain valid values which may differ from the original values. Resource leaks (including memory leaks) are commonly ruled out by an invariant stating that all resources are accounted for and managed.
> 3) No exception safety: No guarantees are made.
 



Of these three functions, the copy assignment function is a bit nuanced. If that function fails, it may leave the original object in an inconsistent state, thus violating strong exception safety.

The copy-and-swap idiom lets us implement the copy assignment operator with a strong exception safety guarantee.





Assignment, at its heart, is two steps: tearing down the object's old state and building its new state as a copy of some other object's state.

Basically, that's what the destructor and the copy constructor do, so the first idea would be to delegate the work to them. However, since destruction mustn't fail, while construction might, we actually want to do it the other way around: first perform the constructive part and, if that succeeded, then do the destructive part. The copy-and-swap idiom is a way to do just that: It first calls a class' copy constructor to create a temporary object, then swaps its data with the temporary's, and then lets the temporary's destructor destroy the old state.
Since swap() is supposed to never fail, the only part which might fail is the copy-construction. That is performed first, and if it fails, nothing will be changed in the targeted object.





- tranmogrify example

https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom

https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Copy-and-swap