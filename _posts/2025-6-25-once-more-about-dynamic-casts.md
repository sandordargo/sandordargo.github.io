---
layout: post
title: "Once more about dynamic_cast, a real use case"
date: 2025-6-25
category: dev
tags: [cpp, bug, undefinedbehaviour]
excerpt_separator: <!--more-->
---
I wrote a couple of times about `dynamic_cast` and I discouraged you from using it. In general, [it makes code worse in terms of readability](https://www.sandordargo.com/blog/2023/04/26/without-rtti-your-code-will-be-cleaner). When you get rid of `dynamic_cast`, either via self-discipline or by [turning RTTI off](https://www.sandordargo.com/blog/2023/03/01/binary-sizes-and-rtti), you'll have to rely on dynamic dispatching and better abstractions.

But there might be cases, when it's not possible or at least it's not meaningful to remove `dynamic_cast`, here is one, sent by one of you.

## Versioning with the help of `dynamic_cast`

They have an SDK that anyone can implement. As there are new features added every now and then, the API keeps changing. Not surprisingly, the owners of the SDK want to prevent their users' code from breaking. They achieve this by having different "versioned" interfaces for the same service where a new version inherits from the previous one.

Let's see a simplified example.

```cpp
class InterfaceForSomeService_v1 {
public:
	virtual void featureA() = 0;
	virtual void featureB() = 0;
};

class InterfaceForSomeService_v2 : public InterfaceForSomeService_v1 {
public:
	virtual void featureC() = 0;
};

class InterfaceForSomeService_v3 : public InterfaceForSomeService_v2 {
public:
	virtual void featureD() = 0;
};
```

So far so good, now the question is what it's the best way to choose between the interfaces, and how to know during runtime which are the interfaces that are implemented. By "best" we mean the most readable way that doesn't leak any implementation details.

Their solution is that they get the instance of the DLL and they always retrieve a `void*` (or something similarly basic type) pointer to stay compatible, but then they try to cast the object to all the different interfaces to know which are really implemented. If the cast is successful, they know they have found the right version.

```cpp
// https://godbolt.org/z/TK9WM65n1
#include <iostream>
#include <memory>

// sdk

class InterfaceForSomeServiceBase{
public:
    virtual ~InterfaceForSomeServiceBase() = default;
};

class InterfaceForSomeService_v1: public InterfaceForSomeServiceBase {
public:
	virtual void featureA() = 0;
	virtual void featureB() = 0;
};

class InterfaceForSomeService_v2 : public InterfaceForSomeService_v1 {
public:
	virtual void featureC() = 0;
};

class InterfaceForSomeService_v3 : public InterfaceForSomeService_v2 {
public:
	virtual void featureD() = 0;
};


class Server {
public:
    void handle(std::unique_ptr<InterfaceForSomeServiceBase> clientImplementation) {
        if (auto* p = dynamic_cast<InterfaceForSomeService_v3*>(clientImplementation.get()); p) {
            p->featureA();
            p->featureB();
            p->featureC();
            p->featureD();
        } else if (auto* p = dynamic_cast<InterfaceForSomeService_v2*>(clientImplementation.get()); p) {
            p->featureA();
            p->featureB();
            p->featureC();
        } else if (auto* p = dynamic_cast<InterfaceForSomeService_v1*>(clientImplementation.get()); p) {
            p->featureA();
            p->featureB();
        } else {
            std::cout << "unhandled version\n";
        }
    }
};


// client

class ClientServiceImplementation : public InterfaceForSomeService_v2 {
public:
	void featureA() override {
        std::cout << "ClientServiceImplementation(InterfaceForSomeService_v2)::featureA\n";
    }
	void featureB() override {
        std::cout << "ClientServiceImplementation(InterfaceForSomeService_v2)::featureB\n";
    }
    void featureC() override {
        std::cout << "ClientServiceImplementation(InterfaceForSomeService_v2)::featureC\n";
    }
};

// server

std::unique_ptr<InterfaceForSomeServiceBase> LoadFromDLL() {
    return std::make_unique<ClientServiceImplementation>();
}

int main() {
    std::unique_ptr<InterfaceForSomeServiceBase> clientImplementation = LoadFromDLL();

    Server s;
    s.handle(std::move(clientImplementation));
}
```

In this solution, it's important to start casting from the newest version and go towards the oldest one. If in the previous listing, we change `Server::handle` and accidentally try to cast to `InterfaceForSomeService_v1*` before `InterfaceForSomeService_v2*` then we end up in a different branch and miss calling `p->featureC()` which is not part of the *v1* API. As long as the API versioning is straightforward, it's relatively easy to pay attention to this rule.

## Is there another solution?

There is always another solution! Is it better? In this case, I'm not sure though. Remember, the goal is to avoid `dynamic_cast` so that we don't depend on RTTI and our code becomes cleaner as well.

Sadly (?), we cannot simply use different overloads to handle where there is an overload for each interface version.

```cpp
class Server {
public:
	/* ...*/

    void handle(InterfaceForSomeService_v1* p) {
    	// ...
    }

    void handle(InterfaceForSomeService_v2* p) {
    	// ...
    }

     void handle(InterfaceForSomeService_v3* p) {
     	// ...
    }

   int main() {
    std::unique_ptr<InterfaceForSomeServiceBase> clientImplementation = LoadFromDLL();

    Server s;
    s.handle(clientImplementation.get());
}

/*
<source>:120:7: error: no matching member function for call to 'handle'
  120 |     s.handle(clientImplementation.get());
      |     ~~^~~~~~
<source>:69:11: note: candidate function not viable: cannot convert from base class pointer 'pointer' (aka 'InterfaceForSomeServiceBase *') to derived class pointer 'InterfaceForSomeService_v1 *' for 1st argument
   69 |      void handle(InterfaceForSomeService_v1* p) {
      |           ^      ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
<source>:75:10: note: candidate function not viable: cannot convert from base class pointer 'pointer' (aka 'InterfaceForSomeServiceBase *') to derived class pointer 'InterfaceForSomeService_v2 *' for 1st argument
   75 |     void handle(InterfaceForSomeService_v2* p) {
      |          ^      ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
<source>:82:11: note: candidate function not viable: cannot convert from base class pointer 'pointer' (aka 'InterfaceForSomeServiceBase *') to derived class pointer 'InterfaceForSomeService_v3 *' for 1st argument
   82 |      void handle(InterfaceForSomeService_v3* p) {
*/
```

This is probably evident for many of you, but I thought it's still worth mentioning. We cannot get away from this problem that simply.

### Let's introduce a `ServiceVersion` tag

The way out of this situation is paved with a `ServiceVersion` tag. Let's add an abstract method called `getVersion` to the `InterfaceForSomeServiceBase` class that has to be defined by each interface version.

Then by querying that method, the server knows exactly which version it's dealing with and therefore it can `static_cast` the base class pointer to the right derived class pointer.

```cpp
enum class ServiceVersion {
    V1,
    V2,
    V3
};

class InterfaceForSomeServiceBase {
public:
    virtual ~InterfaceForSomeServiceBase() = default;
    virtual ServiceVersion getVersion() const = 0;
};

class InterfaceForSomeService_v1 : public InterfaceForSomeServiceBase {
public:
    ServiceVersion getVersion() const override {
        return ServiceVersion::V1;
    }
    // ...
};

// ...

class Server {
public:
    void handle(std::unique_ptr<InterfaceForSomeServiceBase> clientImplementation) {
        switch(clientImplementation->getVersion()) {
            case ServiceVersion::V1: 
                handle_v1(static_cast<InterfaceForSomeService_v1*>(clientImplementation.get()));
                return;
            case ServiceVersion::V2:
                handle_v2(static_cast<InterfaceForSomeService_v2*>(clientImplementation.get()));
                return;
            case ServiceVersion::V3:
                handle_v3(static_cast<InterfaceForSomeService_v3*>(clientImplementation.get()));
                return;
        }
    }
    // ...
};
``` 

This works well in a clean laboratory environment. The problem is that each derived class can override `getVersion` and a malicious or ignorant client might do this:

```cpp
class ClientServiceImplementation : public InterfaceForSomeService_v2 {
public:
    ServiceVersion getVersion() const override {
        return ServiceVersion::V3; // Oh, oh!
    }
    
    // ...
};
```

And now due to the `static_cast` and the version mismatch, we're going to have a segmentation fault! We cannot afford that. ([Here is the whole example.](https://godbolt.org/z/3sdbcjGhf))

Ideally, we should ban the `ClientServiceImplementation` from overriding `getVersion()`. That's what `final` is for, right? Making `getVersion()` `final` in `InterfaceForSomeService_v2` would solve the problem, right? Not exactly. Remember that in the original example (and that's a hard requirement in our scenario today), each new version inherits from the previous one:

```cpp
class InterfaceForSomeService_v1 : public InterfaceForSomeServiceBase { /* ... */ };
class InterfaceForSomeService_v2 : public InterfaceForSomeService_v1 { /* ... */ };
class InterfaceForSomeService_v3 : public InterfaceForSomeService_v2 { /* ... */ };
```

If we add the `final` qualifier to `InterfaceForSomeService_v2::getVersion()` then `InterfaceForSomeService_v3` has no means to override it.

### Hiding details from the clients

As I shared my concerns with the engineer who shared this example, he came up with a better solution. He would even hide the `ServiceVersion` `enum`. In this solution, the `getInteraceServiceVersion()` method returns a new type `InterfaceForSomeServiceVersion` which is only forward declared and its definition is not distributed to the clients. This type effectively wraps `ServiceVersion`.


```cpp
// sdk.h

// Client cannot see the implementation of this class
class InterfaceForSomeServiceVersion;


class InterfaceForSomeServiceBase {
public:
    virtual ~InterfaceForSomeServiceBase() = default;
    virtual InterfaceForSomeServiceVersion getInteraceServiceVersion() const = 0;
};

class InterfaceForSomeService_v1 : public InterfaceForSomeServiceBase {
public:
    virtual void featureA() = 0;
    virtual void featureB() = 0;

    InterfaceForSomeServiceVersion getInteraceServiceVersion() const override;
};
// and below would come the rest of the different versions
```

The definition of `ServiceVersion` and `InterfaceForSomeServiceVersion` is part of an internal header that is not distributed:

```cpp
// This file is not distributed

enum class ServiceVersion {
    V1,
    V2,
    V3
};

class InterfaceForSomeServiceVersion {
public:
    constexpr InterfaceForSomeServiceVersion(ServiceVersion version) : version(version) {}

    constexpr ServiceVersion getVersion() const { return version; }

private:
    ServiceVersion version;
};
```

The implementation of the public `sdk.h` header would also not be distributed and it contains the definitions of `getInteraceServiceVersion()` for the different interface versions, such as this one:

```cpp
InterfaceForSomeServiceVersion InterfaceForSomeService_v1::getInteraceServiceVersion() const
{
    return { ServiceVersion::V1 };
}
```

On the server side, the same handling goes on as in the previous version, each call can be dispatched to the right version with the help of `static_cast`, given that we know exactly which type we are dealing with. You can [check out the full solution here](https://godbolt.org/z/WGq7o96a5).

In this solution, the `ServiceVersion` is completely hidden from the client and if they want to try something malicious, they would have to define their own version of `InterfaceForSomeServiceVersion` which would violate the [One Definition Rule](https://en.wikipedia.org/wiki/One_Definition_Rule) and still likely make the server crash.

### Move forward with double inheritance

I took a slightly different approach and ended up with code that is sadly less readable. My goal was to restrict client implementations from overriding `getVersion` while new interface versions can still do it. In order to do that, I expanded the inheritance tree.

In order to define new interface versions, let's use classes whose definitions are not available for clients. I mark them by appending *Private* to their names:

```cpp
class InterfaceForSomeService_v1Private : public InterfaceForSomeServiceBase { /* ... */ };
class InterfaceForSomeService_v2Private : public InterfaceForSomeService_v1Private { /* ... */ };
class InterfaceForSomeService_v3Private : public InterfaceForSomeService_v2Private { /* ... */ };
```

These classes don't make the `getVersion()` method final.

On the other hand, for each version, there is a non-private counterpart and they do make the `getVersion` method `final` so that client implementations cannot override the version values.

Here is one pair of classes:

```cpp
class InterfaceForSomeService_v2Private : public InterfaceForSomeService_v1Private {
public:
    virtual void featureC() = 0;
};

class InterfaceForSomeService_v2 : public InterfaceForSomeService_v2Private {
    ServiceVersion getVersion() const final override {
        return ServiceVersion::V2;
    }
};
```

Now it's impossible for the client implementation (inhering from a non-"private" class) to override `getVersion()` so as far as I can tell, our solution is safe. [You can have a look at the full code here](https://godbolt.org/z/oPW8YEnvq).

Now you might tell me that this solution is not more readable than the original one based on RTTI and `dynamic_cast`s. And you are perfectly right about that. Maybe we have found a case where using `dynamic_cast` gives probably the better solution.

At the same time, it's also worth remembering that RTTI incurs a bigger binary size and if that is something you cannot afford, you might have to deal with such occasional increased complexities.

## Conclusion

Over the last years, I claimed a couple of times that if you give up RTTI and you restrict yourself from using `dynamic_cast` you'll have not only smaller but also more readable code. I still think that it's true in general, but we have been shown e a case where `dynamic_cast` makes things easier and probably even safer. Yet, if you cannot use RTTI, we saw some alternative solutions.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
