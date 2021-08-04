This article is about the problem of storing vectors in a container and a bug I faced recently.

Many would quickly find a conclusion that you should not store raw pointers, but you should work with smart pointers. I think they are right. When you have issues with dangling pointers, with lifetime and ownership, that's an indication that you should have chosen a smarter way to manage your pointers.

Many would argue that you also have architecture issues, if you face such problems. Again, they are right.

Meanwhile, when you are working on a huge and old codebase, you don't necessarily have the freedom of updating dozens of components to meet such expectations.

Let's assume that we have a container of pointers. We add elements to it not at construction time, just to emulate a realistic scenario where pointers are added later: 

```cpp
#include <vector>
#include <iostream>


int main() { 
  std::vector<int*> numbers;
  
  int* a = new int{42};
  numbers.push_back(a);
  
  int* b = new int{51};
  numbers.push_back(b);
  
  int* c = new int{66};
  numbers.push_back(c);
  
  for (auto* n : numbers) {
    std::cout << *n << '\n';
  }
}
```

What can go wrong?

Many things and we'll see some simplistic examples.

Let's say we want to delete one of the pointers, and you delete it.

```cpp
#include <vector>
#include <iostream>


int main() { 
  std::vector<int*> numbers;
  
  int* a = new int{42};
  numbers.push_back(a);
  
  int* b = new int{51};
  numbers.push_back(b);
  
  int* c = new int{66};
  numbers.push_back(c);
  
  delete numbers[1];
  
  for (auto* n : numbers) {
    std::cout << *n << '\n';
  }
}
/*
42
585960360
66
*/
```
We still have three outputs and `585960360` is not exactly what we wanted.

You might add a guard statement in the for loop to skip an iteration, in case you get a `nullptr`, but it won't help.

```cpp
for (auto* n : numbers) {
  if (n == nullptr) { continue; }
  std::cout << *n << '\n';
}
```

After deletion, we didn't set the pointer to `nullptr`.

```cpp
#include <vector>
#include <iostream>


int main() { 
  std::vector<int*> numbers;
  
  int* a = new int{42};
  numbers.push_back(a);
  
  int* b = new int{51};
  numbers.push_back(b);
  
  int* c = new int{66};
  numbers.push_back(c);
  
  delete numbers[1];
  numbers[1] = nullptr;
  
  for (auto* n : numbers) {
    if (n == nullptr) { continue; }
    std::cout << *n << '\n';
  }
  std::cout << '\n';
  std::cout << numbers.size() << '\n';
}
```

Now it's better, we indeed skipped the second number, but from our last line we can still see that even though we deleted a pointer, the size of the vector haven't changed.

We deleted a number, but not the element of the vector.

To complete the removal, if that's what we wanted, we have to erase the pointer from the vector:

```cpp
  delete numbers[1];
  numbers[1] = nullptr;
  numbers.erase(numbers.begin()+1);
```
Note, that `erase` doesn't accept an index, it takes an iterator. If we run the full example, we can see that know size of our vector is down to 2.

The takeaway?

If you want to delete an item of a vector, don't forget to set it to `nullptr` after the destruction so that you can detect in other parts of the code that it got deleted. If you also want to remove it from the container, don't forget to erase it.

Now let's go to another sort of problematic. Let's have a look at a part of our example.

```cpp
std::vector<int*> numbers;
  
int* a = new int{42};
numbers.push_back(a);
```

So we store raw pointers. Who owns those raw pointers? Well, nobody knows. Maybe the same entity that owns the `numbers` vector, maybe the same who created the pointers. The in the above example it's the same function, it's not necessarily the case.

What if a pointer is deleted not through a vector but by the original owner?

To skip a couple of rounds, let's assume that we don't forget about setting the pointer to `nullptr` and that we have the `the nullptr` guard in our for loop.

```cpp
#include <vector>
#include <iostream>

int main() { 
  std::vector<int*> numbers;
  
  int* a = new int{42};
  numbers.push_back(a);
  
  int* b = new int{51};
  numbers.push_back(b);
  
  int* c = new int{66};
  numbers.push_back(c);


  delete b;
  b = nullptr;

  for (auto* n : numbers) {
    if (n == nullptr) { continue; }
    std::cout << *n << '\n';
  }
  std::cout << '\n';
  std::cout << numbers.size() << '\n';
}
```

What do you think the results are?

It's something like this:

```
42
148114344
66
```

Which means that if you delete the original pointer, in the vector we don't know about it.

It makes perfect sense.

```cpp
#include <iostream>

int main() { 
  int* n = new int{66};
  int* n2 = n;
  
  std::cout << std::boolalpha;
  std::cout << "n is nullptr? " << (n == nullptr) << '\n';
  std::cout << "n2 is nullptr? " << (n2 == nullptr) << '\n';

  delete n;
  n = nullptr;
  
  std::cout << "n is nullptr? " << (n == nullptr) << '\n';
  std::cout << "n2 is nullptr? " << (n2 == nullptr) << '\n';

}
/*
n is nullptr? false
n2 is nullptr? false
n is nullptr? true
n2 is nullptr? false
*/
```












#include <algorithm>
#include <iostream>
#include <set>
#include <limits>
#include<functional>

class A {
public:
  A(int i) : n(i) {}
  int getN() const {return n;}
private:
  int n=0;
};

int main() {
//   const uint64_t nullPointerId = std::numeric_limits<uint64_t>::max();
  std::set<std::reference_wrapper<A*>> s;
  
  A* a1 = new A{42};
  A* a2 = new A{51};
  A* a3 = new A{66};
  
  A* a4 = a3;
  
  std::cout << a3->getN() << '\n';
  std::cout << a4->getN() << '\n';
  
//   delete a4;
//   a4 = nullptr;
//   if (a4 == nullptr) {
//       std::cout << "a4 is nullptr" << std::endl;
//   }
//   if (a3 == nullptr) {
//       std::cout << "a3 is nullptr" << std::endl;
//   }
  
//   std::cout << a3->getN() << '\n';
//   std::cout << a4->getN() << '\n';
  
//   /*
  s.insert(std::ref(a1));
  s.insert(std::ref(a2));
//   s.insert(std::ref(nullptr));
  s.insert(std::ref(a3));
  
  
//   delete s[1];
  delete a2;
  a2 = nullptr;
  if (a2 == nullptr) {
      std::cout << "a2 is nullptr" << std::endl;
  }
  
  for(auto it = s.begin(); it != s.end(); ++it) {
      if ((*it) == nullptr) {
          std::cout << "we have a nullptr\n";
      } else {
        std::cout << (*it).get()->getN() << '\n';
      }
  }
  
  for(const auto pa: s) {
      if (pa == nullptr) {
          std::cout << "we have a nullptr\n";
      } else {
        std::cout << pa.get()->getN() << '\n';
      }
  }
//   */
}










#include <iostream>
#include <vector>


int main() {
  std::vector<int*> numbers;
  
  int* a = new int{42};
  numbers.push_back(a);
  
  int* b = new int{51};
  numbers.push_back(b);
  
  int* c = new int{66};
  numbers.push_back(c);
  
  for(auto* n : numbers) {
    std::cout << *n << '\n';
  }
}