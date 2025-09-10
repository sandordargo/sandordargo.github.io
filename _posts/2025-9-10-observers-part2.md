---
layout: post
title: "Discovering observers - part 2"
date: 2025-9-10
category: dev
tags: [cpp, designpatterns, observer]
excerpt_separator: <!--more-->
---
[Last week](https://www.sandordargo.com/blog/2025/09/03/observers-part1), we took the observer pattern from a very simple example and evolved it into a more flexible, template-based implementation in C++. We ended up with abstracted publishers and subscribers, a templated message type for flexibility, and publishers controlling when and how updates are pushed.
<!--more-->

This week, we're going to tackle the challenge of **supporting multiple message types in a single publisher**.

## First attempt: multiple inheritance of templated bases

It's tempting to provide **different specializations** for our templates so the same publisher can push different message types.

```cpp
// https://godbolt.org/z/9dWb5vabW
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

using Setting1Message = std::pair<std::string, int>;
using Setting2Message = std::pair<std::string, bool>;

class SettingsSubscriber : public Subscriber<Setting1Message>,
                           public Subscriber<Setting2Message> {
   public:
    void update(Setting1Message message) override {
        std::cout << "Subscriber is getting an update:\n"
                  << message.first << "=" << message.second << '\n';
    }
    void update(Setting2Message message) override {
        std::cout << std::boolalpha << "Subscriber is getting an update:\n"
                  << message.first << "=" << message.second << '\n';
    }
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

   private:
    int _setting1{0};
    bool _setting2{0};
};

int main() {
    SettingsPublisher pub;
    SettingsSubscriber s1, s2;
    pub.subscribe(&s1); // ERROR!
    pub.subscribe(&s2); // ERROR!
    pub.setSetting1(42); // ERROR!
    pub.unsubscribe(&s1); // ERROR!
    pub.setSetting2(true);
}
/*
<source>: In function 'int main()':
<source>:89:9: error: request for member 'subscribe' is ambiguous
   89 |     pub.subscribe(&s1);
      |         ^~~~~~~~~
<source>:16:10: note: candidates are: 'void Publisher<Message>::subscribe(Subscriber<Message>*) [with Message = std::pair<std::__cxx11::basic_string<char>, bool>]'
   16 |     void subscribe(Subscriber<Message>* subscriber) {
      |          ^~~~~~~~~
<source>:16:10: note:                 'void Publisher<Message>::subscribe(Subscriber<Message>*) [with Message = std::pair<std::__cxx11::basic_string<char>, int>]'
<source>:90:9: error: request for member 'subscribe' is ambiguous
   90 |     pub.subscribe(&s2);
      |         ^~~~~~~~~
<source>:16:10: note: candidates are: 'void Publisher<Message>::subscribe(Subscriber<Message>*) [with Message = std::pair<std::__cxx11::basic_string<char>, bool>]'
   16 |     void subscribe(Subscriber<Message>* subscriber) {
      |          ^~~~~~~~~
<source>:16:10: note:                 'void Publisher<Message>::subscribe(Subscriber<Message>*) [with Message = std::pair<std::__cxx11::basic_string<char>, int>]'
<source>:92:9: error: request for member 'unsubscribe' is ambiguous
   92 |     pub.unsubscribe(&s1);
      |         ^~~~~~~~~~~
<source>:21:10: note: candidates are: 'void Publisher<Message>::unsubscribe(Subscriber<Message>*) [with Message = std::pair<std::__cxx11::basic_string<char>, bool>]'
   21 |     void unsubscribe(Subscriber<Message>* subscriber) {
      |          ^~~~~~~~~~~
<source>:21:10: note:                 'void Publisher<Message>::unsubscribe(Subscriber<Message>*) [with Message = std::pair<std::__cxx11::basic_string<char>, int>]'
*/
```
As the errors show, calls to `subscribe`/`unsubscribe` are ambiguous: the compiler doesn't know which base `Publisher<T>` you mean.

We can disambiguate by qualifying the base, but it's pretty ugly:

```cpp
// https://godbolt.org/z/q4jnrj741

// Unchanged everything until main()
int main() {
    SettingsPublisher pub;
    SettingsSubscriber s1, s2;
    
    // Subscribe both subscribers to BOTH message types explicitly
    pub.Publisher<Setting1Message>::subscribe(&s1);
    pub.Publisher<Setting2Message>::subscribe(&s1);
    pub.Publisher<Setting1Message>::subscribe(&s2);
    pub.Publisher<Setting2Message>::subscribe(&s2);

    pub.setSetting1(42);

    // Unsubscribe s1 from BOTH channels
    pub.Publisher<Setting1Message>::unsubscribe(&s1);
    pub.Publisher<Setting2Message>::unsubscribe(&s1);
    
    pub.setSetting2(true);
}
```

This compiles — but the public interface became painful to use - to say the least.

## Attempt #2: separate API names per message "channel"

A mitigation is to give each "channel" its own explicit API on SettingsPublisher. This removes ambiguity but doesn't scale as settings grow:


```cpp
// https://godbolt.org/z/WaajsfYoE

// The rest is unchanged

class SettingsPublisher /* ... */ {
public:
    // ..
    void subscriberSetting1(Subscriber<Setting1Message>* subscriber);
    void unsubscriberSetting1(Subscriber<Setting1Message>* subscriber);

    void subscriberSetting2(Subscriber<Setting2Message>* subscriber);
    void unsubscriberSetting2(Subscriber<Setting2Message>* subscriber);
}

int main() {
    SettingsPublisher pub;
    SettingsSubscriber s1, s2;
    pub.subscribeSetting1(&s1);
    pub.subscribeSetting2(&s1);
    pub.subscribeSetting1(&s2);
    pub.setSetting1(42);
    pub.unsubscribeSetting1(&s1);
    pub.setSetting2(true);
}

```

The problem is that we lost the nice interface of the publisher with unified names, but maybe this is what we wanted. Imagine, if instead of `setting1`, `setting2`, etc. we have `volume`, `speed`, `subtitles`, etc. You might take it as verbose, but it's expressive.

## Different subscriber classes per setting

Notice that we implemented a `SettingsPublisher` supporting several publishers - through base classes. Maybe we want to make our intentions more explicit through different subscriber classes. For example, we could have:
- a class subscribing only to `setting1` changes
- a class subscribing only to `setting2` changes
- a class subscribing to all changes

We could already instantiate `SettingsSubscriber` and subscribe to individual changes, but maybe we prefer to have different `Subscriber` implementations:

```cpp
// https://godbolt.org/z/nEbx4qq8v
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

using Setting1Message = std::pair<std::string, int>;
using Setting2Message = std::pair<std::string, bool>;

class Settings1Subscriber : public Subscriber<Setting1Message> {
   public:
    void update(Setting1Message message) override {
        std::cout << "Subscriber is getting an update:\n"
                  << message.first << "=" << message.second << '\n';
    }
};

class Settings2Subscriber : public Subscriber<Setting2Message> {
   public:
    void update(Setting2Message message) override {
        std::cout << std::boolalpha << "Subscriber is getting an update:\n"
                  << message.first << "=" << message.second << '\n';
    }
};

class AllSettingsSubscriber : public Subscriber<Setting1Message>,
                              public Subscriber<Setting2Message> {
   public:
    void update(Setting1Message message) override {
        std::cout << "Subscriber is getting an update:\n"
                  << message.first << "=" << message.second << '\n';
    }
    void update(Setting2Message message) override {
        std::cout << std::boolalpha << "Subscriber is getting an update:\n"
                  << message.first << "=" << message.second << '\n';
    }
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

int main() {
    SettingsPublisher pub;
    Settings1Subscriber s11, s12;
    Settings2Subscriber s21;
    AllSettingsSubscriber sa1;
    pub.subscribeSetting1(&s11);
    pub.subscribeSetting1(&s12);
    // pub.subscribeSetting1(&s21); // ERROR wrong subscriber for setting1
    pub.subscribeSetting2(&s21);
    pub.subscribeSetting1(&sa1);
    pub.subscribeSetting2(&sa1);
    pub.setSetting1(42);
    pub.unsubscribeSetting1(&sa1);
    pub.setSetting2(true);
}
/*
Got a new subscriber
Got a new subscriber
Got a new subscriber
Got a new subscriber
Got a new subscriber
Sending an update to 3 subscriber(s)
Subscriber is getting an update:
setting1=42
Subscriber is getting an update:
setting1=42
Subscriber is getting an update:
setting1=42
Someone unsubscribed
Sending an update to 2 subscriber(s)
Subscriber is getting an update:
setting2=true
Subscriber is getting an update:
setting2=true
*/
```

Where are we?

We have to implement a class template for each setting type - not necessarily for each setting, several settings can use the same message type. We have `string` key to identify the modified setting. At our examples, we only use the key in our print, but not much imagination is needed to see different usages.

Here is an example with one message type, but several settings.


```cpp
// https://godbolt.org/z/3a5GhY9Th
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

using SettingMessage = std::pair<std::string, int>;

class SettingsSubscriber : public Subscriber<SettingMessage> {
   public:
    void update(SettingMessage message) override {
        std::cout << "Subscriber is getting an update:\n"
                  << message.first << "=" << message.second << '\n';
    }
};

class SettingsPublisher : public Publisher<SettingMessage> {
   public:
    void notifyOne(Subscriber<SettingMessage>* const subscriber,
                   SettingMessage message) const override {
        subscriber->update(message);
    }

    void setSetting1(int value) {
        _setting1 = value;
        Publisher<SettingMessage>::notify({"setting1", _setting1});
    }

    void setSetting2(bool value) {
        _setting2 = value;
        Publisher<SettingMessage>::notify({"setting2", _setting2});
    }

    void subscribeSetting1(Subscriber<SettingMessage>* subscriber) {
        Publisher<SettingMessage>::subscribe(subscriber);
    }
    void unsubscribeSetting1(Subscriber<SettingMessage>* subscriber) {
        Publisher<SettingMessage>::unsubscribe(subscriber);
    }

    void subscribeSetting2(Subscriber<SettingMessage>* subscriber) {
        Publisher<SettingMessage>::subscribe(subscriber);
    }
    void unsubscribeSetting2(Subscriber<SettingMessage>* subscriber) {
        Publisher<SettingMessage>::unsubscribe(subscriber);
    }

   private:
    int _setting1{0};
    int _setting2{0};
};

int main() {
    SettingsPublisher pub;
    SettingsSubscriber s11, s12;
    SettingsSubscriber s21;
    pub.subscribeSetting1(&s11);
    pub.subscribeSetting1(&s12);
    pub.subscribeSetting2(&s21);
    pub.setSetting1(42);
    pub.setSetting2(true);
}
/*
Got a new subscriber
Got a new subscriber
Got a new subscriber
Sending an update to 3 subscriber(s)
Subscriber is getting an update:
setting1=42
Subscriber is getting an update:
setting1=42
Subscriber is getting an update:
setting1=42
Sending an update to 3 subscriber(s)
Subscriber is getting an update:
setting2=1
Subscriber is getting an update:
setting2=1
Subscriber is getting an update:
setting2=1
*/
```

The above example reveals another limitation. `Subscriber`s are effectively subscribed to all keys carried by `SettingMessage`. To subscribe per-key, we'd need extra bookkeeping (per-key subscriber lists), duplicating what the base `Publisher` already manages.

## A pragmatic split: one publisher per setting

The simplest clean cut is to have one publisher per setting (still using a common message shape), which keeps APIs small and intent clear:

```cpp
// https://godbolt.org/z/az1ffsq1r
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

using SettingMessage = std::pair<std::string, int>;

class SettingsSubscriber : public Subscriber<SettingMessage> {
   public:
    void update(SettingMessage message) override {
        std::cout << "Subscriber is getting an update:\n"
                  << message.first << "=" << message.second << '\n';
    }
};

class Settings1Publisher : public Publisher<SettingMessage> {
   public:
    void notifyOne(Subscriber<SettingMessage>* const subscriber,
                   SettingMessage message) const override {
        subscriber->update(message);
    }

    void setSetting(int value) {
        _setting1 = value;
        Publisher<SettingMessage>::notify({"setting1", _setting1});
    }

   private:
    int _setting1{0};
};

class Settings2Publisher : public Publisher<SettingMessage> {
   public:
    void notifyOne(Subscriber<SettingMessage>* const subscriber,
                   SettingMessage message) const override {
        subscriber->update(message);
    }

    void setSetting(bool value) {
        _setting2 = value;
        Publisher<SettingMessage>::notify({"setting2", _setting2});
    }

   private:
    int _setting2{0};
};

int main() {
    Settings1Publisher pub1;
    Settings2Publisher pub2;
    SettingsSubscriber s11, s12;
    SettingsSubscriber s21;
    pub1.subscribe(&s11);
    pub2.subscribe(&s12);
    pub2.subscribe(&s21);
    pub1.setSetting(42);
    pub2.setSetting(true);
}
/*
Got a new subscriber
Got a new subscriber
Got a new subscriber
Sending an update to 1 subscriber(s)
Subscriber is getting an update:
setting1=42
Sending an update to 2 subscriber(s)
Subscriber is getting an update:
setting2=1
Subscriber is getting an update:
setting2=1
*/
```

By splitting publishers per setting, we avoid ambiguity, keep per-setting subscriber lists naturally separated, and reduce API noise. Notice how we're drifting from deep inheritance toward composition of simple publishers, which often produces cleaner code.

We'll explore that composition route next week.

## Conclusion

Supporting multiple message types under one publisher sounds convenient, but in practice it introduces API ambiguity, extra ceremony, and fragile coupling:

- Multiple inheritance of `Publisher<T>` bases leads to ambiguous calls (subscribe/unsubscribe), forcing awkward qualification.
- Per-channel methods (e.g., `subscribeSetting1`, `subscribeSetting2`) remove ambiguity but don't scale with the number of settings.
- Collapsing to a single "keyed" message type centralizes notifications but requires per-key subscription bookkeeping, duplicating what the base publisher already manages.

A pragmatic, maintainable compromise is to use one small publisher per setting (or per channel). This keeps responsibilities tight, APIs simple, and subscriber lists accurate. From here, the natural next step is to replace inheritance-heavy designs with composition. That's where we'll pick up next time.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
