---
layout: post
title: "Class templates versus constructor templates"
date: 2022-12-21
category: dev
tags: [cpp, templates, tmp]
excerpt_separator: <!--more-->
---
I realized that this simple but important difference should be covered twice during the last year. Once when I wrote about [how shared and unique pointers take their deleters](https://www.sandordargo.com/blog/2022/06/08/smart-pointers-and-deleters), and once when I read [Template Metaprogramming with C++ by Marius Bancila](https://www.sandordargo.com/blog/2022/10/28/template-metaprogramming-with-cpp-by-marius-bancila).

## Class templates

A class template serves as a blueprint for creating a class. The relationship is similar to that of a class and an instance. You (optionally) pass in a certain number of parameters to create an instance of a class. In the case of a class template, you pass in a certain amount of types to create a class.

Defining a class template is straightforward.

```cpp
#include <iostream>


template <typename T>
class ClassTemplateWithRegularConstructor {
public:
  ClassTemplateWithRegularConstructor(T data);
  
  T getData() const;
private:
  T m_data;
};

template<typename T>
ClassTemplateWithRegularConstructor<T>::ClassTemplateWithRegularConstructor(T data) : m_data(data) {}

template<typename T>
T ClassTemplateWithRegularConstructor<T>::getData() const { return m_data; }

int main() {
    ClassTemplateWithRegularConstructor<int> a{42};
    std::cout << a.getData() << '\n';
}
```

We define the template parameter on a class level and then we can use it throughout the class. Unless it can be deduced, you have to define it at instantiation time. What is also important to note is that for each function definition that takes place outside of the class, you must type `template <typename T>` and you have to list the template parameters after the class name.

## Constructor templates

Just as it's possible to write class templates, we can write function templates.

But what about the special functions? Is it possible to templatize them?

It's not possible to templatize the destructor as it doesn't take any parameter and there can be only one destructor in a class - [unless you use concepts to constrain them](https://www.sandordargo.com/blog/2021/06/16/multiple-destructors-with-cpp-concepts). But it's totally possible to have a constructor template - even without having a class template.

The syntax is simple. Instead of using the usual template syntax on the class, you apply it to the constructor. 

```cpp
class RegularClassWithConstructorTemplate {
public:
  template<typename T>
  RegularClassWithConstructorTemplate(T data) {
    std::cout << "RegularClassWithConstructorTemplate called with " << data << '\n';
  }
private:
};
```

The important difference to keep in mind is that the template type `T` cannot be used outside the constructor.

So how to use it then?

## Convert types with a constructor template

You might want to convert some value of any type to a certain type that you store then. To have it more effective, you might want to even constrain the accepted types.

```cpp
#include <concepts>
#include <string>

template<typename T> 
concept HasToString = requires (T t) {
  { t.to_string() } -> std::same_as<std::string>;
};

template <typename T>
concept ConvertableToString = std::constructible_from<std::string, T>;

class RegularClassWithConstructorTemplate {
public:

  template<std::integral T>
  RegularClassWithConstructorTemplate(T number) : m_text(std::to_string(number)) {}

  template<HasToString T>
  RegularClassWithConstructorTemplate(T input) : m_text(input.to_string()) {}
  
  
  template<ConvertableToString T>
  RegularClassWithConstructorTemplate(T input) : m_text(std::string(input)) {}

private:
  std::string m_text;
};

class Wrapper {
public:
  Wrapper(int data) : m_data(data) {}

  std::string to_string() const {
    return std::to_string(m_data);
  }
private:
  int m_data;
};

int main() {
    RegularClassWithConstructorTemplate ct1{Wrapper{42}};
    RegularClassWithConstructorTemplate ct2{"c-string"};
    RegularClassWithConstructorTemplate ct3{41};
}
```

In the above example, we provided 3 different constrained constructors to take care of the conversion of the input parameter.

We could simplify these constructors, using the [abbrevieated function template syntax](https://www.sandordargo.com/blog/2021/02/17/cpp-concepts-4-ways-to-use-them), completely removing the `template` keywords. No matter the syntax, the constructors would still remain templated ones.

```cpp
RegularClassWithConstructorTemplate(std::integral number) : m_text(std::to_string(number)) {}

RegularClassWithConstructorTemplate(HasToString input) : m_text(input.to_string()) {}
  
RegularClassWithConstructorTemplate(ConvertableToString input) : m_text(std::string(input)) {}
```

## Use a constructor template with a nested class template

Another occasion to use a constructor template without a class template is when you need to use the type parameter with an internal class template that inherits from a non-template class. With the help of the non-template base type, you can store a reference or pointer of the templatized child without relying on the type parameter.

I know this might be a bit overwhelming.

In other words, you have to use such a constructor template in the type erasure design patterns.

Let's see an example:

```cpp
#include <memory>
#include <vector>

struct Shape {
   template <typename T>
   Shape(T&& obj) :
      m_shape(std::make_shared<ShapeModel<T>>(std::forward<T>(obj))) {}
   
   void rotate() {
      m_shape->rotate();
   }
   
   struct ShapeConcept {
      virtual void rotate() = 0;
      virtual ~ShapeConcept() = default;
   };

   template <typename T>
   struct ShapeModel : public ShapeConcept {
      ShapeModel(T& shape) : t(shape) {}
      
      void rotate() override { t.rotate(); }
     private:
      T& t;
    };
private:
   std::shared_ptr<ShapeConcept> m_shape;
};


void rotate(std::vector<Shape>& shapes) {
   for (auto& shape : shapes) {
      shape.rotate();
   }
}

int main() {}
```

There can be different implementations of the type erasure design pattern, I borrowed this one from [Template Metaprogramming with C++ by Marius Bancila](https://www.sandordargo.com/blog/2022/10/28/template-metaprogramming-with-cpp-by-marius-bancila). Another day, we'll dig deeper in the topic.

So in this example, we can see how the template constructor's template parameter is used to instantiate the internal class template `ShapeModel` which inherits from the non-class template `ShapeConcept`. As `ShapeConcept` is not templatized, we can (only) store a non-template pointer to `ShapeConcept`, pointing to a `ShapeModel<T>`.

## Conclusion

In this article, we saw that we cannot only write class templates, but we can also write classes which are not templatized, yet their constructors are templates. We saw that we can use a constructor template to provide different conversions for inputs and that it's also useful if we have a nested class template that inherits from a non-template base.

How and why do you use constructor templates?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!