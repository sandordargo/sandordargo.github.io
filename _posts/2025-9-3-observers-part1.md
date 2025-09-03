---
layout: post
title: "Discovering observers - part 1"
date: 2025-9-3
category: dev
tags: [cpp, designpatterns, observer]
excerpt_separator: <!--more-->
---
The goal of this mini-series is to explore the **Observer Design Pattern** in C++, walking through different implementations and weighing their pros and cons.

First, let's briefly recap what the observer pattern is. It belongs to the family of **behavioral design patterns**.

> As a reminder: design patterns are usually grouped into three categories: **creational**, **structural**, and **behavioral**.

You might also have encountered the observer under other names such as *listener*, *event subscriber*, or *publisher-subscriber*.

The central idea is simple: instead of repeatedly querying an object for information, the querying object (the *observer*) gets notified automatically when the information holder (the *subject*) changes. For example, imagine an orchestrator object that needs the latest value of a user setting. Instead of polling the setting every `n` milliseconds, it can *subscribe* to value changes and receive notifications whenever a new value is set.

Using the common terminology: there is typically **one publisher** and **one or more subscribers**. Subscribers register for events or changes, and whenever an update happens, the publisher notifies them.

## A Simple Example

```cpp
// https://godbolt.org/z/Krs81Kr8x
#include <iostream>
#include <string_view>
#include <vector>

class Subscriber {
   public:
    void update(std::string_view message) const {
        std::cout << "Subscriber is getting an update:\n" << message << '\n';
    }
};

class Publisher {
   public:
    void subscribe(Subscriber* subscriber) {
        std::cout << "Got a new subscriber\n";
        _subscribers.push_back(subscriber);
    }

    void unsubscribe(Subscriber* subscriber) {
        std::cout << "Someone unsubscribed\n";
        std::erase(_subscribers, subscriber);
    }

    void notify() const {
        std::cout << "Sending an update to " << _subscribers.size()
                  << " subscriber(s)\n";
        for (const auto* subscriber : _subscribers) {
            subscriber->update("here is a new version");
        }
    }

   private:
    std::vector<Subscriber*> _subscribers;
};

int main() {
    Publisher pub;
    Subscriber s1, s2;
    pub.subscribe(&s1);
    pub.subscribe(&s2);
    pub.notify();
    pub.unsubscribe(&s1);
    pub.notify();
}
/*
Got a new subscriber
Got a new subscriber
Sending an update to 2 subscriber(s)
Subscriber is getting an update:
here is a new version
Subscriber is getting an update:
here is a new version
Someone unsubscribed
Sending an update to 1 subscriber(s)
Subscriber is getting an update:
here is a new version
*/
```

In this simple version:
- We have a `Publisher` that can subscribe/unsubscribe `Subscriber`s.
- `Subscriber`s get notified with a message.
- `Subscriber::update` is marked as `const`, but in real use it probably needs to update the subscriber's internal state and therefore be non-`const`. For now, we'll keep it, and revisit this later.

So what's the problem here?

There are a few:
- We used generic names but the implementation is tightly coupled.
- We can't easily reuse the logic for different kinds of publishers/subscribers.
- The message type is fixed (`std::string_view`).

Let's start improving things.

## Adding Inheritance

One option is to introduce inheritance and abstract base classes for publishers and subscribers.

- `Subscriber::update` becomes a pure virtual function.
- The non-virtual `Publisher::notify` delegates to a virtual function notifyOne, following [the Template Method Pattern and the Non-Virtual Idiom](https://www.sandordargo.com/blog/2022/08/24/tmp-and-nvi).

```cpp
// https://godbolt.org/z/371EzsGns
#include <iostream>
#include <string_view>
#include <vector>

class Subscriber {
   public:
    virtual ~Subscriber() = default;
    virtual void update(std::string_view message) = 0;
};

class Publisher {
   public:
    virtual ~Publisher() = default;
    void subscribe(Subscriber* subscriber) {
        std::cout << "Got a new subscriber\n";
        _subscribers.push_back(subscriber);
    }

    void unsubscribe(Subscriber* subscriber) {
        std::cout << "Someone unsubscribed\n";
        std::erase(_subscribers, subscriber);
    }

    void notify() const {
        std::cout << "Sending an update to " << _subscribers.size()
                  << " subscriber(s)\n";
        for (auto* const subscriber : _subscribers) {
            notifyOne(subscriber, "here is a new version");
        }
    }

   private:
    virtual void notifyOne(Subscriber* const,
                           std::string_view message) const = 0;

    std::vector<Subscriber*> _subscribers;
};

class SettingsSubscriber : public Subscriber {
   public:
    void update(std::string_view message) override {
        std::cout << "Subscriber is getting an update:\n" << message << '\n';
    }
};
class SettingsPublisher : public Publisher {
   public:
    void notifyOne(Subscriber* const subscriber,
                   std::string_view message) const override {
        subscriber->update(message);
    }
};

int main() {
    SettingsPublisher pub;
    SettingsSubscriber s1, s2;
    pub.subscribe(&s1);
    pub.subscribe(&s2);
    pub.notify();
    pub.unsubscribe(&s1);
    pub.notify();
}
/*
Got a new subscriber
Got a new subscriber
Sending an update to 2 subscriber(s)
Subscriber is getting an update:
here is a new version
Subscriber is getting an update:
here is a new version
Someone unsubscribed
Sending an update to 1 subscriber(s)
Subscriber is getting an update:
here is a new version
*/
```

This version is better: reusable, extensible, and type-safe.

But it still assumes all messages are `std::string_view`.

That's better.

## Introducing Templates

What if we want different message types? For example:
- `std::string_view` for text updates
- `std::pair<std::string, int>` for key-value updates

We can generalize by turning both `Publisher` and `Subscriber` into templates parameterized by the message type:

```cpp
// https://godbolt.org/z/ocnjf83P7
#include <iostream>
#include <string_view>
#include <vector>

template <typename Message>
class Subscriber {
   public:
    virtual ~Subscriber() = default;
    virtual void update(Message message) = 0;
};

template <typename Message>
class Publisher {
   public:
    virtual ~Publisher() = default;
    void subscribe(Subscriber<Message>* subscriber) {
        std::cout << "Got a new subscriber\n";
        _subscribers.push_back(subscriber);
    }

    void unsubscribe(Subscriber<Message>* subscriber) {
        std::cout << "Someone unsubscribed\n";
        std::erase(_subscribers, subscriber);
    }

    void notify() const {
        std::cout << "Sending an update to " << _subscribers.size()
                  << " subscriber(s)\n";
        for (auto* const subscriber : _subscribers) {
            notifyOne(subscriber, "here is a new version");
        }
    }

   private:
    virtual void notifyOne(Subscriber<Message>* const,
                           Message message) const = 0;

    std::vector<Subscriber<Message>*> _subscribers;
};

template <typename Message>
class SettingsSubscriber : public Subscriber<Message> {
   public:
    void update(Message message) override {
        std::cout << "Subscriber is getting an update:\n" << message << '\n';
    }
};

template <typename Message>
class SettingsPublisher : public Publisher<Message> {
   public:
    void notifyOne(Subscriber<Message>* const subscriber,
                   Message message) const override {
        subscriber->update(message);
    }
};

int main() {
    SettingsPublisher<std::string_view> pub;
    SettingsSubscriber<std::string_view> s1, s2;
    pub.subscribe(&s1);
    pub.subscribe(&s2);
    pub.notify();
    pub.unsubscribe(&s1);
    pub.notify();
}
/*
Got a new subscriber
Got a new subscriber
Sending an update to 2 subscriber(s)
Subscriber is getting an update:
here is a new version
Subscriber is getting an update:
here is a new version
Someone unsubscribed
Sending an update to 1 subscriber(s)
Subscriber is getting an update:
here is a new version
*/
```
This gives us flexibility, but it reveals an already - existing issue: `Publisher::notify` hardcodes the message. That's not great.

## Pushing Messages from the Publisher

A better design is to let the publisher provide the message, rather than requiring it externally. 
- We'll make `notify(Message)` protected so only the publisher itself can decide when and how to push updates.
- Also, we'll let the derived publishers and subscribers to hardcode the message type.


```cpp
//https://godbolt.org/z/Mv7YMM3aK 
#include <iostream>
#include <string_view>
#include <vector>

template <typename Message>
class Subscriber {
   public:
    virtual ~Subscriber() = default;
    virtual void update(Message message) = 0;
};

template <typename Message>
class Publisher {
   public:
    virtual ~Publisher() = default;
    void subscribe(Subscriber<Message>* subscriber) {
        std::cout << "Got a new subscriber\n";
        _subscribers.push_back(subscriber);
    }

    void unsubscribe(Subscriber<Message>* subscriber) {
        std::cout << "Someone unsubscribed\n";
        std::erase(_subscribers, subscriber);
    }

   protected:
    void notify(Message message) const {
        std::cout << "Sending an update to " << _subscribers.size()
                  << " subscriber(s)\n";
        for (auto* const subscriber : _subscribers) {
            notifyOne(subscriber, message);
        }
    }

   private:
    virtual void notifyOne(Subscriber<Message>* const,
                           Message message) const = 0;

    std::vector<Subscriber<Message>*> _subscribers;
};

using SettingsMessage = std::pair<std::string, int>;
class SettingsSubscriber : public Subscriber<SettingsMessage> {
   public:
    void update(std::pair<std::string, int> message) override {
        std::cout << "Subscriber is getting an update:\n"
                  << message.first << "=" << message.second << '\n';
    }
};

class SettingsPublisher : public Publisher<SettingsMessage> {
   public:
    void notifyOne(Subscriber<SettingsMessage>* const subscriber,
                   SettingsMessage message) const override {
        subscriber->update(message);
    }

    void setSetting1(int value) {
        _setting1 = value;
        notify({"setting1", _setting1});
    }

    void setSetting2(int value) {
        _setting2 = value;
        notify({"setting2", _setting2});
    }

   private:
    int _setting1{0};
    int _setting2{0};
};

int main() {
    SettingsPublisher pub;
    SettingsSubscriber s1, s2;
    pub.subscribe(&s1);
    pub.subscribe(&s2);
    pub.setSetting1(42);
    pub.unsubscribe(&s1);
    pub.setSetting1(51);
}

/*
Got a new subscriber
Got a new subscriber
Sending an update to 2 subscriber(s)
Subscriber is getting an update:
setting1=42
Subscriber is getting an update:
setting1=42
Someone unsubscribed
Sending an update to 1 subscriber(s)
Subscriber is getting an update:
setting1=51
*/
```

Now the publisher controls when updates happen, and subscribers automatically get the right type of message.

At this stage, we have:
- Generic, reusable templates for publishers and subscribers.
- Customizable message types (string, pair, or anything else).
- A publisher-driven notification system.

This design is quite usable.

But there's still a limitation: what if we want one publisher to handle multiple different message types (e.g., int for one setting and `bool` for another)?

We'll explore this next week.

## Conclusion

We've taken the observer pattern from a very simple example and evolved it into a more flexible, template-based implementation in C++. Along the way, we:

- Abstracted publishers and subscribers.
- Introduced templates for message flexibility.
- Ensured publishers control when and how updates are pushed.

Next time, we'll tackle the challenge of supporting multiple message types in a single publisher.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
