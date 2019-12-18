---
layout: post
title: "The big STL Algorithms tutorial: transform's undefined behaviour"
date: 2019-12-18
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In the last article on the series on the STL algorithms, we discussed `std::transform`. For not the first time, we saw an interface where the user has to pass in two ranges with the help of three parameters.
<!--more-->
The first range is defined by its beginning and the end, while the second only by its beginning.

Why so? To have a more compact signature, I think.

On the flip side, the second range must contain at least as many elements as the first one. It is the user's full responsibility to respect this requirement. The algorithm(s) won't do any checks!

So what happens if the user is a naughty little fellow - and sends in a smaller second range?

Let's see it through an example!

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main () { 

auto values = std::vector<int>{1,2,3,4,5};
auto otherValues = std::vector<int>{10,20,30};
auto results = std::vector<int>{};
std::transform(values.begin(), values.end(), otherValues.begin(), std::back_inserter(results), [](int number, int otherNumber) {return number+otherNumber;});

std::for_each(results.begin(), results.end(), [](int number){ std::cout << number << "\n";});
return 0;
}
```

Here are the results:
```
11
22
33
4
5
```

So the elements are automatically zero-initialized or... To me, this looked strange, to say the least, so I wrapped my integers and scattered the standard output with a bunch of messages.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

class T{
public:
  T() {
    std::cout << "Empty constructor " << "\n";
  }

  T(const T& other) {
    std::cout << "Copy constructor with _number: " << other.getNumber() << "\n";
  }

  T(int number) : _number(number) {
    std::cout << "Default constructor with number: " << number << "\n";
  }

  ~T() {
    std::cout << "Destructor " << _number << "\n";
  }

  int getNumber() const { return _number; }
private:
  int _number;
};

int main () { 

  auto values = std::vector<T>{T{1},T{2},T{3},T{4},T{5}};
  auto otherValues = std::vector<T>{T{10},T{20},T{30}};
  auto resutls = std::vector<int>{};
  std::transform(values.begin(), values.end(), otherValues.begin(), 
  std::back_inserter(resutls), [](T number, T otherNumber) {return 
  number.getNumber() + otherNumber.getNumber();});

  std::for_each(resutls.begin(), resutls.end(), [](int number){ std::cout << number << "\n";});
  return 0;
}
```

I don't copy here the output as it is long, you can run everything [here](http://coliru.stacked-crooked.com/a/d4ae736a454083b0).

The results are different, all number 6 everywhere in terms of results. While that is interesting, I was more motivated to find the root cause.

There is such a section:
```
Default constructor with number: 10
Default constructor with number: 20
Default constructor with number: 30
Copy constructor with _number: 10
Copy constructor with _number: 20
Copy constructor with _number: 30
Destructor 30
Destructor 20
Destructor 10
Copy constructor with _number: 0
Copy constructor with _number: 0
```

This is the first time in the logs that we see some instances with `0` in them. How have they appeared?

I mean in order to copy some object where there are zeros inside, we must have created those objects they were copied from. But we don't have any such logs even though we logged everything. I double-checked.

For curiosity, I even marked the default constructor deleted. (`T() = delete;`) Still, the behaviour has not changed at all.

Then I asked for a second pair of eyes and with some changes to the code, it became more understandable. There are two ways to proceed.

We either create the first container __much__ bigger or we store the variable on the heap instead of the stack (so we store pointers).

As the output of the second one is smaller, let's do that!

So [here](http://coliru.stacked-crooked.com/a/436ae3e23b81156f) is the new code:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

class T{
public:
  T() {
  std::cout << "Empty constructor " << "\n";
  }

  T(const T& other) {
  std::cout << "Copy constructor with _number: " << other.getNumber() << "\n";
  }

  T(int number) : _number(number) {
    std::cout << "Default constructor with number: " << number << "\n";
  }

  ~T() {
    std::cout << "Destructor " << _number << "\n";
  }

  int getNumber() const { return _number; }
private:
  int _number;
};

int main () { 

  auto values = std::vector<T*>{new T{1},new T{2},new T{3},new T{4},new T{5}};
  auto otherValues = std::vector<T*>{new T{10},new T{20},new T{30}};
  auto resutls = std::vector<int>{};
  std::transform(values.begin(), values.end(), otherValues.begin(), 
  std::back_inserter(resutls), [](T* number, T* otherNumber) {
    std::cout << "number: " << number->getNumber() << ", another number: " << otherNumber->getNumber() << std::endl;
    return number->getNumber() + otherNumber->getNumber();
  });

  std::for_each(resutls.begin(), resutls.end(), [](int number){ std::cout << number << "\n";});
  return 0;
}
```

Now we don't have those zeros anymore, we have something much better a segmentation fault, yes!

So why did we have zeros before?

When we create a vector, it automatically reserves enough size for the items we put into it at creation time, plus some. How much is that _"some"_? Well, it depends on the compiler implementation.

That memory is empty and cleaned up.

So when in our previous example we went beyond the size of the second container it was just reading zeros.

When we store things on the heap, we don't have anymore a continuous memory area, but we use random places in the memory. And at random places, we have random things and we can easily end up in a segmentation fault.

I said there are two ways to show this root cause.

If we had a much longer first container, that container would have been allocated to a bigger memory area as the second one. When we have 5 vs 3 values, like in our [original example](http://coliru.stacked-crooked.com/a/d4ae736a454083b0), most likely the two vectors occupy the same space in memory.

This means that after a certain point during the transformation, for the second container we'll touch memory that was never allocated to the second vector and will have random values just like in the case of storing pointers.

[Here](http://coliru.stacked-crooked.com/) you can find such an example with way more interesting numbers than 0, such as `29764816` or `455072427`.

## Conclusion

In this article, we have seen what dangers hide behind the way we pass two containers to `std::transform` (and to other containers). The second container is defined only by its start point without the endpoint and besides, there are no runtime checks to verify if it's at least as long as the first one.

In some simple situations, we might get away with this without being severely punished, but it would be still only by accident. 

Through using pointers and vectors that hugely differ in size we saw how and why this undefined behaviour manifests.

The takeaway is that when you read the documentation and you read some warnings, like the second container should be always at least as large as the first one, take them seriously and do your homework.

Next time we continue with the replace algorithms. Stay tuned!