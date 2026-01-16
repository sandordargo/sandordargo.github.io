---
layout: post
title: "Discovering observers - part 3"
date: 2025-9-17
category: dev
tags: [cpp, design patterns, observer]
excerpt_separator: <!--more-->
---
Over the last two weeks, we explored different implementations of the **observer pattern** in C++. [We began with a very simple example](https://www.sandordargo.com/blog/2025/09/03/observers-part1), then [evolved toward a more flexible, template- and inheritance-based design](https://www.sandordargo.com/blog/2025/09/10/observers-part2).

This week, let's move further — shifting away from inheritance and embracing **composition**.  

You might say (and you'd be right) that publishers and subscribers can just as well be used as members instead of base classes. In fact, with our previous implementation, handling changes inside subscribers felt more complicated than necessary.  

What we'd really like is this: subscribers that observe changes and trigger updates in their enclosing class *in a coherent way*. As one of the readers of Part 1 pointed out, this can be elegantly achieved if **subscribers take callables**.

## Step 1: Subscribers with callables

We start by letting `Subscriber` accept a `std::function`, which gets called whenever the subscriber is updated:

```cpp
// https://godbolt.org/z/5x4GP9dac

#include <functional>
#include <iostream>
#include <string_view>
#include <vector>

template <typename Callable>
class Subscriber {
   public:
    virtual ~Subscriber() = default;
    virtual void update(Callable updater) = 0;
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

    void notify(Message message) const {
        std::cout << "Sending an update to " << _subscribers.size()
                  << " subscriber(s)\n";
        for (auto* const subscriber : _subscribers) {
            notifyOne(subscriber, message);
        }
    }

    virtual void notifyOne(Subscriber<Message>* const,
                           Message message) const = 0;

   private:
    std::vector<Subscriber<Message>*> _subscribers;
};

using Setting1Message = std::pair<std::string, int>;
using Setting2Message = std::pair<std::string, bool>;

using Update1Message = std::function<void(Setting1Message)>;
using Update2Message = std::function<void(Setting2Message)>;

class Settings1Subscriber : public Subscriber<Setting1Message> {
   public:
    Settings1Subscriber(Update1Message func) : _func(func) {}
    void update(Setting1Message message) override { _func(message); }

    Update1Message _func;
};

class Settings2Subscriber : public Subscriber<Setting2Message> {
   public:
    Settings2Subscriber(Update2Message func) : _func(func) {}
    void update(Setting2Message message) override { _func(message); }

    Update2Message _func;
};

class SettingsPublisher : public Publisher<Setting1Message>,
                          public Publisher<Setting2Message> {
   public:
    void notifyOne(Subscriber<Setting1Message>* const subscriber,
                   Setting1Message message) const override {
        subscriber->update(message);
    }

    void notifyOne(Subscriber<Setting2Message>* const subscriber,
                   Setting2Message message) const override {
        subscriber->update(message);
    }

    void setSetting1(int value) {
        _setting1 = value;
        Publisher<Setting1Message>::notify({"setting1", _setting1});
    }

    void setSetting2(bool value) {
        _setting2 = value;
        Publisher<Setting2Message>::notify({"setting2", _setting2});
    }

    void subscribeSetting1(Subscriber<Setting1Message>* subscriber) {
        Publisher<Setting1Message>::subscribe(subscriber);
    }
    void unsubscribeSetting1(Subscriber<Setting1Message>* subscriber) {
        Publisher<Setting1Message>::unsubscribe(subscriber);
    }

    void subscribeSetting2(Subscriber<Setting2Message>* subscriber) {
        Publisher<Setting2Message>::subscribe(subscriber);
    }
    void unsubscribeSetting2(Subscriber<Setting2Message>* subscriber) {
        Publisher<Setting2Message>::unsubscribe(subscriber);
    }

   private:
    int _setting1{0};
    bool _setting2{0};
};

class SettingsUser {
   public:
    SettingsUser(SettingsPublisher& pub)
        : _pub(pub),
          _s1([this](Setting1Message m) { printSetting1(m.second); }),
          _s2([this](Setting2Message m) { printSetting2(m.second); }) {
        _pub.subscribeSetting1(&_s1);
        _pub.subscribeSetting2(&_s2);
    }

    ~SettingsUser() {
        _pub.unsubscribeSetting1(&_s1);
        _pub.unsubscribeSetting2(&_s2);
    }

    void printSetting1(int setting1) {
        std::cout << "setting1 is " << setting1 << '\n';
    }

    void printSetting2(bool setting2) {
        std::cout << "setting2 is " << std::boolalpha << setting2 << '\n';
    }

   private:
    SettingsPublisher& _pub;
    Settings1Subscriber _s1;
    Settings2Subscriber _s2;
};

int main() {
    SettingsPublisher pub;
    SettingsUser su(pub);
    pub.setSetting1(42);
    pub.setSetting2(true);
}
/*
Got a new subscriber
Got a new subscriber
Sending an update to 1 subscriber(s)
setting1 is 42
Sending an update to 1 subscriber(s)
setting2 is true
Someone unsubscribed
Someone unsubscribed
*/
```

In this solution:
- `SettingsUser` owns subscribers as members.
- The user class subscribes/unsubscribes automatically in its constructor/destructor.
- Each subscriber takes a callable, so it can directly update the enclosing class.

Pretty neat, right?

## Step 2: Removing inheritance from subscribers

After looking closely, I realized we don't need inheritance at all on the subscriber side. Subscribers can simply be template classes wrapping a callable:


```cpp
// https://godbolt.org/z/hrWK5bsfP
#include <functional>
#include <iostream>
#include <string_view>
#include <vector>

template <typename Message>
class Subscriber {
   using Callable = std::function<void(Message)>;
   public:
    Subscriber(Callable func) : _func(func) {}

    void update(Message message) {
        _func(message);
    }

    Callable _func;
};

template <typename Message>
class Publisher {
   public:
    using Callable = std::function<void(Message)>;
    virtual ~Publisher() = default;
    void subscribe(Subscriber<Message>* subscriber) {
        std::cout << "Got a new subscriber\n";
        _subscribers.push_back(subscriber);
    }

    void unsubscribe(Subscriber<Message>* subscriber) {
        std::cout << "Someone unsubscribed\n";
        std::erase(_subscribers, subscriber);
    }

    void notify(Message message) const {
        std::cout << "Sending an update to " << _subscribers.size()
                  << " subscriber(s)\n";
        for (auto* const subscriber : _subscribers) {
            notifyOne(subscriber, message);
        }
    }

    virtual void notifyOne(Subscriber<Message>* const,
                           Message message) const = 0;

   private:
    std::vector<Subscriber<Message>*> _subscribers;
};

using Setting1Message = std::pair<std::string, int>;
using Setting2Message = std::pair<std::string, bool>;

class SettingsPublisher : public Publisher<Setting1Message>,
                          public Publisher<Setting2Message> {
   public:
    void notifyOne(Subscriber<Setting1Message>* const subscriber,
                   Setting1Message message) const override {
        subscriber->update(message);
    }

    void notifyOne(Subscriber<Setting2Message>* const subscriber,
                   Setting2Message message) const override {
        subscriber->update(message);
    }

    void setSetting1(int value) {
        _setting1 = value;
        Publisher<Setting1Message>::notify({"setting1", _setting1});
    }

    void setSetting2(bool value) {
        _setting2 = value;
        Publisher<Setting2Message>::notify({"setting2", _setting2});
    }

    void subscribeSetting1(Subscriber<Setting1Message>* subscriber) {
        Publisher<Setting1Message>::subscribe(subscriber);
    }
    void unsubscribeSetting1(Subscriber<Setting1Message>* subscriber) {
        Publisher<Setting1Message>::unsubscribe(subscriber);
    }

    void subscribeSetting2(Subscriber<Setting2Message>* subscriber) {
        Publisher<Setting2Message>::subscribe(subscriber);
    }
    void unsubscribeSetting2(Subscriber<Setting2Message>* subscriber) {
        Publisher<Setting2Message>::unsubscribe(subscriber);
    }

   private:
    int _setting1{0};
    bool _setting2{0};
};

class SettingsUser {
   public:
    SettingsUser(SettingsPublisher& pub)
        : _pub(pub),
          _s1([this](Setting1Message m) { printSetting1(m.second); }),
          _s2([this](Setting2Message m) { printSetting2(m.second); }) {
        _pub.subscribeSetting1(&_s1);
        _pub.subscribeSetting2(&_s2);
    }

    ~SettingsUser() {
        _pub.unsubscribeSetting1(&_s1);
        _pub.unsubscribeSetting2(&_s2);
    }

    void printSetting1(int setting1) {
        std::cout << "setting1 is " << setting1 << '\n';
    }

    void printSetting2(bool setting2) {
        std::cout << "setting2 is " << std::boolalpha << setting2 << '\n';
    }

   private:
    SettingsPublisher& _pub;
    Subscriber<Setting1Message> _s1;
    Subscriber<Setting2Message> _s2;
};

int main() {
    SettingsPublisher pub;
    SettingsUser su(pub);
    pub.setSetting1(42);
    pub.setSetting2(true);
}
```
Notice:
- No inheritance is required for Subscriber.
- We didn't introduce extra template parameters.
- The alias for update functions is no longer needed—std::function is enough.

At this point, subscribers are simplified and self-contained.

## Step 3: Simplifying publishers too

Can we apply the same idea to publishers?

Looking at our code: every `notifyOne` call was just forwarding to `subscriber->update(message)`. That's boilerplate we don't need. By moving to composition, we can remove unnecessary overrides and forwarders.

```cpp
// https://godbolt.org/z/hGcvq9Pc6
#include <functional>
#include <iostream>
#include <string_view>
#include <vector>

template <typename Message>
class Subscriber {
    using Callable = std::function<void(Message)>;

   public:
    Subscriber(Callable func) : _func(func) {}

    void update(Message message) { _func(message); }

    Callable _func;
};

template <typename Message>
class Publisher {
   public:
    void subscribe(Subscriber<Message>* subscriber) {
        std::cout << "Got a new subscriber\n";
        _subscribers.push_back(subscriber);
    }

    void unsubscribe(Subscriber<Message>* subscriber) {
        std::cout << "Someone unsubscribed\n";
        std::erase(_subscribers, subscriber);
    }

    void notify(Message message) const {
        std::cout << "Sending an update to " << _subscribers.size()
                  << " subscriber(s)\n";
        for (auto* const subscriber : _subscribers) {
            subscriber->update(message);
        }
    }

   private:
    std::vector<Subscriber<Message>*> _subscribers;
};

using Setting1Message = std::pair<std::string, int>;
using Setting2Message = std::pair<std::string, bool>;

class SettingsPublisher {
   public:
    void setSetting1(int value) {
        _setting1 = value;
        setting1_publisher.notify({"setting1", _setting1});
    }

    void setSetting2(bool value) {
        _setting2 = value;
        setting2_publisher.notify({"setting2", _setting2});
    }

    void subscribeSetting1(Subscriber<Setting1Message>* subscriber) {
        setting1_publisher.subscribe(subscriber);
    }
    void unsubscribeSetting1(Subscriber<Setting1Message>* subscriber) {
        setting1_publisher.unsubscribe(subscriber);
    }

    void subscribeSetting2(Subscriber<Setting2Message>* subscriber) {
        setting2_publisher.subscribe(subscriber);
    }
    void unsubscribeSetting2(Subscriber<Setting2Message>* subscriber) {
        setting2_publisher.unsubscribe(subscriber);
    }

   private:
    Publisher<Setting1Message> setting1_publisher;
    Publisher<Setting2Message> setting2_publisher;
    int _setting1{0};
    bool _setting2{0};
};

class SettingsUser {
   public:
    SettingsUser(SettingsPublisher& pub)
        : _pub(pub),
          _s1([this](Setting1Message m) { printSetting1(m.second); }),
          _s2([this](Setting2Message m) { printSetting2(m.second); }) {
        _pub.subscribeSetting1(&_s1);
        _pub.subscribeSetting2(&_s2);
    }

    ~SettingsUser() {
        _pub.unsubscribeSetting1(&_s1);
        _pub.unsubscribeSetting2(&_s2);
    }

    void printSetting1(int setting1) {
        std::cout << "setting1 is " << setting1 << '\n';
    }

    void printSetting2(bool setting2) {
        std::cout << "setting2 is " << std::boolalpha << setting2 << '\n';
    }

   private:
    SettingsPublisher& _pub;
    Subscriber<Setting1Message> _s1;
    Subscriber<Setting2Message> _s2;
};

int main() {
    SettingsPublisher pub;
    SettingsUser su(pub);
    pub.setSetting1(42);
    pub.setSetting2(true);
}
```

Now:
- `Publisher`s are lightweight containers that just manage subscriber lists and notify them.
- `SettingsPublisher` composes two `Publisher` instances, one per setting.
- `Subscriber`s handle the logic of what to do when they're updated.

We've ended up with a clean, minimal, and flexible design.

## Conclusion

Over three parts, we've gone on a journey with the observer pattern in C++:

- [Part 1](https://www.sandordargo.com/blog/2025/09/03/observers-part1): A simple starting point, then generalized with inheritance and templates.
- [Part 2](https://www.sandordargo.com/blog/2025/09/10/observers-part2): Struggled with multiple message types and discovered the trade-offs of inheritance-heavy designs.
- [Part 3](https://www.sandordargo.com/blog/2025/09/17/observers-part3): Moved toward composition and callables, simplifying both publishers and subscribers, and removing boilerplate.

The final design is:
- **Type-safe**: different message types are still supported.
- **Composable**: publishers and subscribers can be members rather than base classes.
- **Minimal**: no unnecessary inheritance or virtual forwarding.

This design makes it easy to plug in new subscribers, keep publishers lean, and update enclosing classes coherently.

The observer pattern is a classic, but as we've seen, the implementation choices matter. By gradually refining, we moved from a rigid inheritance-based approach to a clean, modern C++ solution leveraging composition and lambdas.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!