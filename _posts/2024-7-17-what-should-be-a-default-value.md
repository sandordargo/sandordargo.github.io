---
layout: post
title: "What to do if you don't want a default constructor?"
date: 2024-7-17
category: dev
tags: [cpp, builds, dependencies, architecture]
excerpt_separator: <!--more-->
---
Do we need a default constructor? What does it mean to have a default constructor? What happens if we don't have one? Those are the questions we are going after in this article.

A default constructor is a constructor that takes no arguments and initializes - hopefully - all the members with some default values. If you define no constructors at all, it'll even be generated for you.

## Do we need default constructors?

It really depends. Let's first approach this question from a design point of view. Does it make sense to represent an object where the members are default initialized?

If you represent a tachograph, it probably makes sense to have such a default state where all the counters are initialized to zero.

> *The tachograph is the device that records driving times and rest periods as well as periods of other work and availability taken by the driver of a heavy vehicle.*

On the other hand, if you represent a person or a task identifier - which inspired me to write this article - it doesn't. A person with an empty name or a task ID with an empty ID doesn't make much sense.

Well, that's the case from a design point of view. What about the technical aspects? What happens if we don't have a default constructor?

You'll have a harder time using some standard library types.

```cpp
#include <map>
#include <string>
#include <vector>

class TaskID {
 public:
    TaskID(std::string uuid): m_uuid(uuid){};

    auto operator<=>(const TaskID&) const = default;

    void serialize(std::string &out_buffer) const {
        out_buffer.resize(sizeof(TaskID));
        memcpy(out_buffer.data(), reinterpret_cast<const char *>(this), sizeof(TaskID));
    }
 private:
    std::string m_uuid;
};

void foo(const TaskID& taskID) {
   // ...
   taskID.serialize();
}


int main() {
    std::vector<TaskID> tasks;
    
    // cannot compile, resize() needs a default constructor
    // tasks.resize(10);

    // this doesn't work as there is no default constructor
    // std::vector<TaskID> moreTasks(10);

    std::map<TaskID, std::string> tasksMap{ {TaskID{"ab12"}, "dummy"} };
    tasksMap[TaskID{"ab13"}] = "other dummy";


    std::map<int, TaskID> tasksMap2{{42, TaskID{"ab12"}}};
    // cannot use operator[], needs default constructor for the value type 
    // tasksMap2[4] = TaskID{"ab13"};

    foo(tasksMap.at(42));
}
```

Just to show you two examples, you'll have difficulties to use `std::vector` or `std::map` up to their full extents. At the same time, these limitations are not necessarily blocking.

On the other hand, there is another limitation that is harder to swallow.

Let's say that you have another class `Widget` that would hold `TaskID` by value. That's possible, sure, but that class cannot have an auto-generated default constructor because it would require that `TaskID` has a default constructor.

Let's assume that at the moment of construction, we cannot have the meaningful value `TaskID`, because we only can or want to figure that out later.

```cpp
#include <map>
#include <string>
#include <vector>

class TaskID {
 public:
    TaskID(std::string uuid): m_uuid(uuid){};

    auto operator<=>(const TaskID&) const = default;
 private:
    std::string m_uuid;
};


class Widget {
public:
    // ...
    TaskID getTaskID() const { return taskId; } 

private:
    TaskID taskId;
};

void foo(const TaskID& taskID) {
   // ...
   taskID.serialize();
}

int main() {
   // error: call to implicitly-deleted default constructor of 'Widget'
    Widget widget;

    foo(widget.getTaskID());
}
```

We can still provide a default constructor for that `Widget`, but how would you instantiate a default `TaskID`? Or any class that doesn't have a meaningful empty state?

## Still use a default constructor

We could assign some dummy values for members. For integers, we can often see that `-1` is used as a number to represent an invalid state. For strings that could be just an empty value. Let's be honest, most often people don't really think about these questions, they just let an object be created with the default values of the members.

If they see that the code doesn't compile, because the default constructor is missing for a class, they just add it.

If they are more careful, they probably define a member function like `isValid()` to check the validity of an object.

```cpp
class TaskID {
 public:
    TaskID() = default;
    TaskID(std::string uuid): m_uuid(uuid){};

    auto operator<=>(const TaskID&) const = default;

    bool isValid() const {
      return !m_uuid.empty();
    }
 private:
    std::string m_uuid;
};

void foo(const TaskID& taskID) {
   if (!taskID.isValid()) {
      return;
   }
   // ...
   taskID.serialize();
}
```

But what other options do we have if we care about not creating objects with meaningless default values?

## Wrap your object with `std::optional`

To avoid having a default constructor, but still be able to use it as a class member or to use it with the different standard containers, we can wrap `TaskID` with `std::optional`.

```cpp
#include <map>
#include <optional>
#include <string>
#include <vector>

class TaskID {
 public:
    TaskID(std::string uuid): m_uuid(uuid){};

    auto operator<=>(const TaskID&) const = default;
 private:
    std::string m_uuid;
};

void foo(std::optional<TaskID> taskID) {
   if (!taskID) {
      return;
   }
   // ...
   taskID.serialize();
}

int main() {
    std::vector<std::optional<TaskID>> tasks;
 
    tasks.resize(10);
    std::vector<std::optional<TaskID>> moreTasks(10);

    std::map<TaskID, std::string> tasksMap{ {TaskID{"ab12"}, "dummy"} };
    tasksMap[TaskID{"ab13"}] = "other dummy";


    std::map<int, std::optional<TaskID>> tasksMap2{{42, TaskID{"ab12"}}};
    tasksMap2[4] = TaskID{"ab13"};

    foo(tasksMap2[42]);
}
```

What does that mean for us?

From a practical point of view, we have to validate if a `TaskID` is present to avoid segmentation faults or `bad_optional_access`. In other words, we have an extra layer. That can be cumbersome.

From a semantic point of view, this means that a `TaskID` is either present or not. It's optionally present. I'd argue that it's only partially what we want to communicate. What we want to say in most cases is that the `TaskID` is not yet available or it's already there. At the same time, `std::optional`'s communication capabilities stop here.

Let's look into another option with more communication capabilities.

## Use `std::variant`

`std::variant<Ts...>` is a bit like `std::optional<T>` but on steroids. Instead of `std::nullopt` or `T`, it can hold practically any kind and number of different types. Therefore, instead of `std::nullopt`, we might say that it can hold a `TaskID`, `InvalidTaskID`, `UninitialziedTaskID`, etc. It's enough to define those other options as empty `struct`s.

```cpp
class TaskID { /* as before*/ };
struct InvalidTaskID{};
struct UninitialziedTaskID {};

std::variant<UninitialziedTaskID, InvalidTaskID, TaskID> taskID;
```

I put `UninitialziedTaskID` in the first place in the template argument list because by default variables will be initialized to that.

Compared to `std::optional` its usage is not painfully complicated, though probably you'll need to use quite a few `std::holds_alternative<T>` and `std::get<T>` in your code which can hinder its readability.


```cpp
void foo(std::variant<UninitialziedTaskID, InvalidTaskID, TaskID> taskID) {
   if (!std::holds_alternative<TaskID>(taskID)) {
      return;
   }
   // ...
   taskID.serialize();
}

void bar(std::variant<UninitialziedTaskID, InvalidTaskID, TaskID> taskID) {
   try {
      std::get<TaskID>(taskId).serialize()
   } catch (std::bad_variant_access const& ex) {
        std::cout << ex.what() << ": taskID didn't contain TaskID\n";
   }
}
```

## Use `std::variant` combined with an `enum`

There is an interesting option we haven't experimented with yet. We could use `std::variant` with only two types, `TaskID` in the second place and an enum in the first place which can hold the different invalid states.

```cpp
#include <string>
#include <variant>
#include <vector>

enum class InvalidTaskIDStates {
InvalidTaskID,
UninitialziedTaskID,
};

class TaskID {
 public:
    TaskID(std::string uuid): m_uuid(uuid){};
 private:
    std::string m_uuid;
};

void foo(std::variant<InvalidTaskIDStates, TaskID> taskID) {
   if (std::holds_alternative<InvalidTaskIDStates>(taskID)) {
      return;
   }
   // ...
   taskID.serialize();
}

void bar(std::variant<InvalidTaskIDStates, TaskID> taskID) {
   if (std::holds_alternative<InvalidTaskIDStates>(taskID)) {
      switch (std::get<InvalidTaskIDStates>(taskID)) {
         case InvalidTaskIDStates::InvalidTaskID:
            std::cout << "InvalidTaskID\n";
            return;
         case InvalidTaskIDStates::UninitialziedTaskID:
            std::cout << "UninitialziedTaskID\n";
            return;
      }
   }
   // ...
   taskID.serialize();
}

int main() {
    std::variant<InvalidTaskIDStates, TaskID> myvar;
    std::vector<std::variant<InvalidTaskIDStates, TaskID>> tasks;
    tasks.resize(10);
    tasks.push_back(TaskID{"ab12"});
    foo(tasks.back());
    bar(tasks.front());
}
```

On the positive side, we can say that if we are interested in invalid states, we only have one variant to check. On the other hand, if we are interested in which invalid state a variable holds, we'll need to mix two kinds of grammar. We need to use the API of `std::variant` and the syntax related to `enum`s as well.

## Conclusion

Today, we discussed what options we have if we don't want our type to have a default constructor because it doesn't make sense. We've seen that even in those cases having a default constructor might make sense, initializing the object to a kind of an invalid state. Even though it's really not a best practice.

Types without a default constructor are sometimes hard to use with containers. Our options are also limited when we need to compose other types that are default constructible, but one of the members have no default constructor. In such cases, we can wrap these types into `std::optional` or `std::variant` based on our needs.

Of course, we could use (smart) pointers as well, but that's not an option we discussed, because dynamic memory allocation is clearly not something we need to solve the original problem.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
