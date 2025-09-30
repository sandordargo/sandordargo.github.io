This post is a good reminder for me that we always keep learning and C++ can be overly complicated.

Let's talk about `const` return types. Again. I say again, [because we already talked about them back in 2020](https://www.sandordargo.com/blog/2020/11/18/when-use-const-3-return-types#returning-const-objects-by-value). Let's reiterate over what I wrote.

When an object is returned from a function by value, the caller gets an own copy. They can do whateveer they want with it and there is no reason to make the return type const. Make your own copy const.

We also said that if the returned type supports move semantics, then returning by const value might even hurt your performance as it will prevent the compiler from performing return value optimization.

I also made a note that *"there are important books advocating for returning user-defined types by const value. They were right in their own age, but since then C++ went through a lot of changes and this piece of advice became obsolete."*



A few weeks ago I was reviewing some code and we got into a discussion where I learned why someone still might want to return a const type value. (It was not the case though in the code reviewed.)