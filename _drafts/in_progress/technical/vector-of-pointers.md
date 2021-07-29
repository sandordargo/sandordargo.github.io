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