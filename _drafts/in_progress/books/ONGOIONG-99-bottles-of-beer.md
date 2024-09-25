this book is about finding the right abstractions

Therefore, one lesson to be gleaned from this solution is that you should name methods after the concept they represent rather than how they currently behave. However, notice that even if you e





Fake It style TDD may initially seem awkward and tedious, but with practice it becomes both natural and speedy. Developing the habit of writing just enough code to pass the tests forces you to write better tests. It also provides an antidote for the hubris of thinking you know what’s right when you’re actually wrong. Although it sometimes makes sense to skip the small steps and 39


Learning the art of transforming code one line at a time, while keeping the tests passing at every point, lets you undertake enormous refactorings piecemeal. This small problem is a good place to practice this technique, in preparation for later tackling bigger ones.
124

Your goal is to optimize for ease of understanding while maintaining performance that’s fast enough. Don’t sacrifice readability in advance of having solid performance data. The first solution to any problem should avoid caching, use immutable objects, and treat object creation as free. This results in speedy development of simple code, which leaves plenty of time to identify and correct the real performance problems.
134


Your goal is to minimize costs, and costs are determined by the situation at hand. There’s no hard and fast rule about what’s best. It just depends.
182


his "depend on abstractions, not on concretions" definition distills the essence of the more verbose original DIP definition from the May 1996 issue of C++ Report, which explained it like this:
1. High-level modules should not depend upon low-level modules. Both should depend upon abstractions.
2. Abstractions should not depend upon details. Details should depend upon abstractions.
208/222



It cannot be emphasized strongly enough that most classes deserve their own explicit unit tests. This should be your default point of view. But just as every rule is meant to be broken, occasionally the most useful thing to communicate about an object is that it is so small, simple, or invisible that testing it individually would raise costs rather than lower them. Every now and then it makes sense to test an object in conjunction with its enclosing unit. Doing so creates a test that leaks into the space between integration and unit, but if you think of the smaller objects as being private, you can be justified in calling the whole thing a unit test.
[...]
If you subscribe to the principle that applications should have 100% test coverage, you might need to reexamine your definition of that rule. Perhaps it means "100% of the code should be exercised during unit tests," rather than "100% of the public methods should have their own personal tests."
233/247
