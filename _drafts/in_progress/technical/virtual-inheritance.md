What is virtual inheritance in C++ and when should you use it?

Virtual inheritance is a C++ technique that ensures only one copy of a base class's member variables are inherited by grandchild derived classes. Without virtual inheritance, if two classes B and C inherit from a class A, and a class D inherits from both B and C, then D will contain two copies of A's member variables: one via B, and one via C. These will be accessible independently, using scope resolution.

Instead, if classes B and C inherit virtually from class A, then objects of class D will contain only one set of the member variables from class A.

As you probably guessed, this technique is useful when you have to deal with multple inheritahnce and you can solve the infamoud diamond inheritance.

In practice, virtual base classes are most suitable when the classes that derive from the virtual base, and especially the virtual base itself, are pure abstract classes. This means the classes above the "join class" (the one in the bottom) have very little if any data.

Consider the following class hierarchy to represent the diamond problem, though not with pure abstracts.

```cpp
struct Person {
    virtual ~Person() = default;
    virtual void speak() {}
};

struct Student: Person {
    virtual void learn() {}
};

struct Worker: Person {
    virtual void work() {}
};

// A teaching assistant is both a worker and a student
struct TeachingAssistant: Student, Worker {};

TeachingAssistant ta;
```

As declared above, a call to `aTeachingAssistant.speak()` is ambiguous because there are two `Person` (indirect) base classes in `TeachingAssistant`, so any `TeachingAssistant` object has two different `Person` base class subobjects. So an attempt to directly bind a reference to the `Person` subobject of a `TeachingAssistant` object would fail, since the binding is inherently ambiguous:

```cpp
TeachingAssistant ta;
Person& a = ta;  // error: which Person subobject should a TeachingAssistant cast into, 
                // a Student::Person or a Worker::Person?
```
To disambiguate, one would have to explicitly convert `ta` to either base class subobject:

```cpp
TeachingAssistant ta;
Person& student = static_cast<Student&>(ta); 
Person& worker = static_cast<Worker&>(ta);
```
In order to call `speak()`, the same disambiguation, or explicit qualification is needed: `static_cast<Student&>(ta).speak()` or `static_cast<Worker&>(ta).speak()` or alternatively `ta.Student::speak()` and `ta.Worker::speak()`. Explicit qualification not only uses an easier, uniform syntax for both pointers and objects but also allows for static dispatch, so it would arguably be the preferable method.

In this case, the double inheritance of `Person` is probably unwanted, as we want to model that the relation (`TeachingAssistant` is a `Person`) exists only once; that a `TeachingAssistant` is a `Student` and is a `Worker` does not imply that it is a `Person` twice (unless the `TA` is skyzophenic): a `Person` base class corresponds to a contract that `TeachingAssistant` implements (the "is a" relationship above really means "implements the requirements of"), and a `TeachingAssistant` only implements the `Person` contract once. 

The real world meaning of "is a only once" is that `TeachingAssistant` should have only one way of implementing `speak`, not two different ways, depending on whether the `Student` view of the `TeachingAssistant` is eating, or the `Worker` view of the `TeachingAssistant`. (In the first code example we see that `speak()` is not overridden in either `Student` or `Worker`, so the two `Person` subobjects will actually behave the same, but this is just a degenerate case, and that does not make a difference from the C++ point of view.)

If we introduce `virtual` to our inheritance as such, our problems disappear.

```cpp
struct Person {
    virtual ~Person() = default;
    virtual void speak() {}
};

// Two classes virtually inheriting Person:
struct Student: virtual Person {
    virtual void learn() {}
};

struct Worker: virtual Person {
    virtual void work() {}
};

// A teaching assistant is still a student and the worker
struct TeachingAssistant: Student, Worker {};
```

Now we can easily call `speak()`.

The `Person` portion of `TeachingAssistant::Worker` is now the same `Person` instance as the one used by `TeachingAssistant::Student`, which is to say that a `TeachingAssistant` has only one, shared, `Person` instance in its representation and so a call to `TeachingAssistant::speak` is unambiguous. Additionally, a direct cast from `TeachingAssistant` to `Person` is also unambiguous, now that there exists only one `Person` instance which `TeachingAssistant` could be converted to.

This can be done through `vtable` pointers. Without going into details, the object size increases by two pointers, but there is only one `Person` object behind and no ambiguity.

You must use the `virtual` keyword in the middle level of the diamond. Using it in the bottom, doesn't help.

You can find more detail at the [Core Guidelines](https://isocpp.org/wiki/faq/multiple-inheritance)
Addition reference [here](https://en.wikipedia.org/wiki/Virtual_inheritance).



Should we always use virtual inheritance? If yes, why? If not, why not?

The answer is definitely no. The base of an idiomatic answer can be that the foundation of C++ is that you only pay for what you use. And if you don't need virtual inheritance, you should rather not pay forit.

Virtual inheritance is almost never needed. It addresses the diamond inheritance problem that we saw just yesterday. It can only happen if you have multiple inheritance, in case you avoid it, you cannot have it.

Other than that, it has some drawbacks.

Virtual inheritance causes troubles with object initialization and copying. Since it is the “most derived” class which is responsible for these operations, it has to be familiar with all the intimate details of the structure of base classes. Due to this, a more complex dependency appears between the classes, which complicates the project structure and forces you to make some additional revisions in all those classes during refactoring. All this leads to new bugs and makes the code less readable.

Troubles with type conversions may also be a source of bugs. You can partly solve the issues by using the expensive  dynamic_casting wherever you used the static_cast, but it is too slow, and if you have to use it too often in your code, it means that your project’s architecture is probably very poor. 

Project structure can be almost always implemented without multiple inheritance. After all, there are no such exotica in many other languages, and it doesn’t prevent programmers writing code in these languages from developing large and complex projects.

