https://arne-mertz.de/2015/08/new-c-features-default-initializers-for-member-variables/

http://coliru.stacked-crooked.com/a/c0649ba6a57e59b4
https://en.cppreference.com/w/cpp/language/data_members
https://en.cppreference.com/w/cpp/language/default_initialization
default initialization is performed in three situations:
1) when a variable with automatic, static, or thread-local storage duration is declared with no initializer;
2) when an object with dynamic storage duration is created by a new-expression with no initializer or when an object is created by a new-expression with the initializer consisting of an empty pair of parentheses (until C++03);
3) when a base class or a non-static data member is not mentioned in a constructor initializer list and that constructor is called.




Default Member Initializers
https://abseil.io/tips/61

right there with the member ddeclaration
goes into all constructor, you can override them
but you don't have to

can you skip a few and let them do in the ctor?


is it still better to initialize something explicitely even with a default ctor?

+++
agregate init
https://www.fluentcpp.com/2018/06/15/should-structs-have-constructors-in-cpp/

default member init
And in C++11, like “real” constructors, their presence (even if for only one attribute) deactivates aggregate initialization (it’s no longer the case in C++14, thanks to Alexandre Chassany and chris for pointing this out).
The C++ Core Guidelines recommend their usage in guideline C.45: “Don’t define a default constructor that only initializes data members; use in-class member initializers instead”