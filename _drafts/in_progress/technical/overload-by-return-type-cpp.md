A friend of mine asked me if it's posible to return different types from a function depending on an input parameter.

My gut asnwer was that it's not possible to overload a function based on its return type. But as this friend f mine is teaching C++ at a university, I was cautios. He must have something going on in his mind.

But the definition is crystal clear. You can overload functions only based on:
- parameters
- cv qualifiers
- and ...?

He told me he knows, but there must be a way around. Maybe with some generic function object involved, maybe with some templates and...

And as usual where is a will, there is a way.

In fact, there are multiple ways, all going to ~~Rome~~ to function overloads by return type.

My first step was returning lambdas.

```cpp

#include <iostream>
#include <string>

class A {
public:
 auto get() const {
     return [](){
         return 42;
     };
 }
   auto get(int i) const {
     return [](){
         return "42";
     };
 }
 
   
};

int main() {

}
```
Do we really need C++14? What if we store them as members and use decltype?
We can in fact use immediately invoked lambdas. Then we don't even need type deduction.

http://coliru.stacked-crooked.com/a/231faed66311aa80

We indeed cheated a bit. Seemingly we overloaded on the return yype, but in fact, it's not the case. We oerloaded, on the the parameter.

We can take different parameters and return different types.

But we cannot take different values of the same type and return different types. 

If  you want to do that in one function, you will get a compilation error about different return types (inconsistent deduction). If you want to do that in different function, you have the well known error that you cannot overload based on the return type.

Will function templates help us?
http://coliru.stacked-crooked.com/a/15b9ce174bd39985
Still we have different input types.


If we specialize, we can.
http://coliru.stacked-crooked.com/a/661863fd2bf3344c

But we need constexpr.
http://coliru.stacked-crooked.com/a/48576d347cc78310

We cannot solve this...
MyClassInterface* Factor(int p1, int p2, int p3) {
  if (p1 == 0 && p2 == 0 && p3 == 0)
    return new MyClass<0,0,0>();
  if (p1 == 0 && p2 == 0 && p3 == 1)
    return new MyClass<0,0,1>();
  etc;
}