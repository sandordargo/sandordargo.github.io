---
layout: post
title: "The big STL Algorithms tutorial: partitioning operations"
date: 2021-1-20
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we cover the partitioning operations - except for ranges which will be covered in a different series.
<!--more-->

* `is_partitioned`
* `partition`
* `partition_copy`
* `stable_partition`
* `partition_point`

## `is_partitioned`
`std::is_partitioned` checks whether a range is partitioned by a given predicate. But what does _partitioned_ mean?

Let's say that you have a list of cars and each car - among others - has an attribute of transmission. A car's gearbox is either manual or automatic. If a range of cars is considered partitioned, then all the manual cars will appear before all automatic. Or the other way around, depending on how the predicate is written.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

enum class Transmission {Automatic, Manual};

struct Car {
  int horsePower;
  Transmission transmission;
};

int main() {
  std::vector unpartitionedCars {
    Car{100, Transmission::Automatic},
    Car{80, Transmission::Manual},
    Car{120, Transmission::Automatic},
  };

  std::vector partitionedCars {
    Car{80, Transmission::Manual},
    Car{100, Transmission::Automatic},
    Car{120, Transmission::Automatic},
  };

  auto isManual = [](const Car& car ){ return car.transmission == Transmission::Manual;};
  std::cout << std::boolalpha;

  std::cout << "unpartitionedCars is_partitioned? " << std::is_partitioned(
    unpartitionedCars.begin(), unpartitionedCars.end(), isManual) << '\n';
  std::cout << "partitionedCars is_partitioned? " << std::is_partitioned(
    partitionedCars.begin(), partitionedCars.end(), isManual) << '\n';
}
/*
unpartitionedCars is_partitioned? false
partitionedCars is_partitioned? true
*/
```
As you can see, the usage is simple, first, you pass in the range by the usual begin/end iterator pairs, then your predicate as a lambda, functor or function pointer.

You'll always get a simple boolean as an answer.

## `partition`

`partition` is a solicitation. Calling `partition` means that you ask for your range to be partitioned.

Just as for `is_partitioned`, you pass in two iterators defining a range and a unary predicate, but this time your range might be modified.

All the items satisfying the passed in predicate will be moved to the front and the non-satisfying items will come only after. It's worth to note that the original order between the satisfying/non-satisfying items is not necessarily kept. If you need that, you should use `stable_partition`.

As a result, you'll get an iterator pointing at the first element of the second group, so pointing at the first element not satisfying the predicate.

Let's see an example:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

enum class Transmission {Automatic, Manual};

struct Car {
  int horsePower;
  Transmission transmission;
};

int main() {
  std::vector cars {
    Car{100, Transmission::Automatic},
    Car{80, Transmission::Manual},
    Car{250, Transmission::Manual},
    Car{120, Transmission::Automatic},
  };

  auto isManual = [](const Car& car ){ return car.transmission == Transmission::Manual;};
  auto printCar = [&](const Car& car ){ std::cout << "Car: " << car.horsePower << " " << (isManual(car) ? "manual" : "automatic" ) << '\n';};
  
  std::cout << std::boolalpha;
  std::cout << "Cars:\n";
  for_each(cars.begin(), cars.end(), printCar);

  std::cout << '\n';
  std::cout << "cars is_partitioned? " << std::is_partitioned(
    cars.begin(), cars.end(), isManual) << '\n';
  
  std::cout << '\n';
  std::partition(cars.begin(), cars.end(), isManual);
  
  std::cout << "Cars:\n";
  for_each(cars.begin(), cars.end(), printCar);  
  std::cout << '\n';
  std::cout << "cars is_partitioned? " << std::is_partitioned(
    cars.begin(), cars.end(), isManual) << '\n';

}
/*
Cars:
Car: 100 automatic
Car: 80 manual
Car: 250 manual
Car: 120 automatic

cars is_partitioned? false

Cars:
Car: 250 manual
Car: 80 manual
Car: 100 automatic
Car: 120 automatic

cars is_partitioned? true
*/
```

## `partition_copy`

`partition_copy` has a very similar functionality compared to `partition`. The only difference is that it leaves the original input range intact and instead it copies the partitioned element into another range.

In fact, into two other ranges and it makes this algorithm quite interesting and requires a bit more attention.

The first two parameters are defining the inputs, then there are two other iterators taken. 

The first output iterator (third parameter) should point at the beginning of the range where you want to copy the elements satisfying the predicate (the predicate is to be passed as a fifth parameter.)

The second output iterator (fourth parameter) points at the beginning of the range where you want to copy the elements not matching the predicate.

There are a couple of things you have to make sure
- as usual, the output ranges are defined by only their beginning. You either have to make sure that they are big enough to accommodate all the items that will be copied into them, or you pass an inserter iterator (`std::back_inserter`)
- the other noticeable items is that we have to output ranges and we have to make sure that there is no overlap between them. As we don't pass containers but iterators, we can easily pass iterators pointing to the same container, but if you don't like trouble, it's better to just create two different containers for the matching and non-matching elements and use them.

`partition_copy` returns a pair of iterators with the first pointing after the last matching copied element and the other pointing similarly after the last non-matching copied element.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

enum class Transmission {Automatic, Manual};

struct Car {
  int horsePower;
  Transmission transmission;
};

int main() {
  std::vector cars {
    Car{100, Transmission::Automatic},
    Car{80, Transmission::Manual},
    Car{250, Transmission::Manual},
    Car{120, Transmission::Automatic},
  };

  auto isManual = [](const Car& car ){ return car.transmission == Transmission::Manual;};
  auto printCar = [&](const Car& car ){ std::cout << "Car: " << car.horsePower << " " << (isManual(car) ? "manual" : "automatic" ) << '\n';};
  
  std::cout << std::boolalpha;
  std::cout << "Cars:\n";
  for_each(cars.begin(), cars.end(), printCar);

  std::cout << '\n';
  
  
  std::vector<Car> manualCars;
  std::vector<Car> automaticCars;
  std::partition_copy(cars.begin(), cars.end(), std::back_inserter(manualCars), std::back_inserter(automaticCars), isManual);
  
  std::cout << "manual Cars:\n";
  for_each(manualCars.begin(), manualCars.end(), printCar);  
  std::cout << '\n';

  std::cout << "automatic Cars:\n";
  for_each(automaticCars.begin(), automaticCars.end(), printCar);  
  std::cout << '\n';
}
/*
Cars:
Car: 100 automatic
Car: 80 manual
Car: 250 manual
Car: 120 automatic

manual Cars:
Car: 80 manual
Car: 250 manual

automatic Cars:
Car: 100 automatic
Car: 120 automatic
*/
```

I found no guarantees, but it seems (not only based on the above example) that the relative order of the elements is preserved. That's something that was explicitly not guaranteed for `partition`

## `stable_partition`

What was clearly said for `partition`, namely that the relative order of the elements partitioned into their categories is not kept, `stable_partition` has this guarantee.

If two items belong to the same category, their relative order will be the same before and after partitioning.

Apart from that, there is no difference between `partition` and `stable_partition`, there is no difference in the way you have to use them.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

enum class Transmission {Automatic, Manual};

struct Car {
  int horsePower;
  Transmission transmission;
};

int main() {
  std::vector cars {
    Car{100, Transmission::Automatic},
    Car{80, Transmission::Manual},
    Car{250, Transmission::Manual},
    Car{120, Transmission::Automatic},
  };

  auto isManual = [](const Car& car ){ return car.transmission == Transmission::Manual;};
  auto printCar = [&](const Car& car ){ std::cout << "Car: " << car.horsePower << " " << (isManual(car) ? "manual" : "automatic" ) << '\n';};
  
  std::cout << std::boolalpha;
  std::cout << "Cars:\n";
  for_each(cars.begin(), cars.end(), printCar);

  std::cout << '\n';
  std::cout << "cars is_partitioned? " << std::is_partitioned(
    cars.begin(), cars.end(), isManual) << '\n';
  
  std::cout << '\n';
  std::stable_partition(cars.begin(), cars.end(), isManual);
  
  std::cout << "Cars:\n";
  for_each(cars.begin(), cars.end(), printCar);  
  std::cout << '\n';
  std::cout << "cars is_partitioned? " << std::is_partitioned(
    cars.begin(), cars.end(), isManual) << '\n';

}
/*
Cars:
Car: 100 automatic
Car: 80 manual
Car: 250 manual
Car: 120 automatic

cars is_partitioned? false

Cars:
Car: 80 manual
Car: 250 manual
Car: 100 automatic
Car: 120 automatic

cars is_partitioned? true
*/
```

If you check the results of the example with the provided results of `partition` you can also observe that the relative order was not kept before, but  
now it is.

## `partition_point`

`partition_point` as its name suggests will return you the dividing point between the matching and non-matching points.

In other words, `partition_point` comes with a contract asking for already partitioned inputs. As usual, calls with invalid arguments are subject to undefined behaviour.

`partition_point` returns an iterator past the end of the first partition, or the last element if all elements match the predicate. Just like `partition` or `stable_partition`.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

enum class Transmission {Automatic, Manual};

struct Car {
  int horsePower;
  Transmission transmission;
};

int main() {
  std::vector cars {
    Car{100, Transmission::Automatic},
    Car{80, Transmission::Manual},
    Car{250, Transmission::Manual},
    Car{120, Transmission::Automatic},
  };

  auto isManual = [](const Car& car ){ return car.transmission == Transmission::Manual;};
  
  std::cout << std::boolalpha;

  std::cout << '\n';
  std::cout << "cars is_partitioned? " << std::is_partitioned(
    cars.begin(), cars.end(), isManual) << '\n';
  
  std::cout << '\n';
  auto partitionResult = std::partition(cars.begin(), cars.end(), isManual);
  auto partitionPoint = std::partition_point(cars.begin(), cars.end(), isManual);
  
  std::cout << "cars is_partitioned? " << std::is_partitioned(
    cars.begin(), cars.end(), isManual) << '\n';
  std::cout << "partitionResult == partitionPoint: " << (partitionResult == partitionPoint) << '\n';
}
/*
cars is_partitioned? false
cars is_partitioned? true
partitionResult == partitionPoint:true
*/
```

## Conclusion

Today, we learned about partitioning algorithms. They allow us to separate elements of a container based on any predicate we might want to define. Next time we are going to discuss sorting algorithms. Stay tuned!

## Connect deeper

If you found interesting this article, please [subscribe to my personal blog](http://eepurl.com/gvcv1j) and let's connect on [Twitter](https://twitter.com/SandorDargo)!