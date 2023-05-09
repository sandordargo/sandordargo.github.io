---
layout: post
title: "Is this dynamic_cast needed?"
date: 2023-5-10
category: dev
tags: [cpp, dynamiccast, virtual, polymorphism]
excerpt_separator: <!--more-->
---
I have already written a couple of times about `dynamic_cast`. I claimed that [if you can avoid using it and RTTI, you can get a smaller binary](https://www.sandordargo.com/blog/2023/03/01/binary-sizes-and-rtti). I also claimed that [without `dynamic_cast` your code will be cleaner](https://www.sandordargo.com/blog/2023/04/26/without-rtti-your-code-will-be-cleaner).

The first claim is new from my side, I didn't care about executable size earlier. The second one is less so, I read a long time ago [in the Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c146-use-dynamic_cast-where-class-hierarchy-navigation-is-unavoidable) that one should avoid using `dynamic_cast` whenever possible, but there are some cases when you cannot avoid it.

I've discussed this topic with a friend of mine who's been teaching C++ for a couple of years and I know he doesn't share these views that much. Or at least he's not so critical towards `dynamic_cast`. He thinks that it is a useful tool in many cases and there must be a reason why it's in the language and not removed.

It is true that is mentioned in the Core Guidelines that there can be some cases when it is needed. Besides, one of [the greatest superpowers of C++ is backward compatibility](https://www.youtube.com/watch?v=5P7fnH9cCR0). Removing `dynamic_cast` would break half of the world...

> *`dynamic_cast` safely converts pointers and references to classes up, down and sideways along the inheritance hierarchy* - according to [CppReference](https://en.cppreference.com/w/cpp/language/dynamic_cast).

This friend of mine sends me some piece of code every now and then saying that this might be a good example. He sent me an example a few weeks ago and maybe this was the good one. Maybe.

## A tree combining templates and virtuals

The example is from [Category Theory for Programmers](https://www.amazon.com/Category-Theory-Programmers-Bartosz-Milewski/dp/0464243874/ref=sr_1_1?crid=SBQNZW8VS5A&amp;keywords=Category+Theory+for+Programmers&amp;qid=1683655205&amp;s=books&amp;sprefix=category+theory+for+programmers+%252Cstripbooks-intl-ship%252C146&amp;sr=1-1&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=dd03bab283cd69a4cb78912fcad27448&camp=1789&creative=9325) written by [Bartosz Milewski](https://bartoszmilewski.com/) who also wrote [C++ In Action: Industrial Strength Programming Techniques](https://www.amazon.com/Action-Industrial-Strength-Programming-Techniques/dp/0201699486/ref=sr_1_1?crid=1BJUHHH2PO1L0&amp;keywords=C%252B%252B+In+Action%253A+Industrial+Strength+Programming+Techniques&amp;qid=1683655290&amp;s=books&amp;sprefix=c%252B%252B+in+action+industrial+strength+programming+techniques%252Cstripbooks-intl-ship%252C154&amp;sr=1-1&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=4818b2ac47747ebe901c75b163d8d3a6&camp=1789&creative=9325). He shows that we should be able to write/recognize some algebraic data structures in C++ and implement `fmap` for them.

> `fmap` is a higher-order function in functional programming that applies a given function to the elements of a container and returns a new container with the results.

And here is his implementation for a `Tree` and `fmap()`.

```cpp
template<class T>
struct Tree {
    virtual ~Tree() {};
};

template<class T>
struct Leaf : public Tree<T> {
   T _label;
    Leaf(T l) : _label(l) {}
};

template<class T>
struct Node : public Tree<T> {
   Tree<T> * _left;
   Tree<T> * _right;
   Node(Tree<T> * l, Tree<T> * r) : _left(l), _right(r) {}
};

template<class A, class B>
Tree<B> * fmap(std::function<B(A)> f, Tree<A> * t) {
    Leaf<A> * pl = dynamic_cast <Leaf<A>*>(t);
    if (pl)
        return new Leaf<B>(f (pl->_label));
    Node<A> * pn = dynamic_cast<Node<A>*>(t);
    if (pn)
        return new Node<B>( fmap<A>(f, pn->_left)
                          , fmap<A>(f, pn->_right));
    return nullptr;
}
```

Is that a good implementation? What is good anyway? The author explicitly writes that he omitted resource and memory management and in production code, one should use smart pointers. I think it's a meaningful simplification in a book or in a blog post.

Let's concentrate on `fmap()` first and just take note that it's probably an acceptable idea to return `nullptr` if both casts fail. I'd probably throw an exception instead if exceptions are allowed, otherwise, let's say that this is fine.

But what about the `dynamic_cast`?

My first idea was that hey, we should replace the one "big" `fmap()` with 2 overloads:

```cpp
template <typename A, typename B>
Leaf<B>* fmap2(std::function<B(A)> f, Leaf<A>* t) {
    return new Leaf<B>(f2(t->label));
}

template <typename A, typename B>
Node<B>* fmap2(std::function<B(A)> f, Node<A>* t) {
    return new Node<B>(fmap2<A>(f, t->_left),
                        fmap2<A>(f, t->_right));
}
```

But in this case, the compilation fails!

```
<source>: In instantiation of 'Node<B>* fmap2(std::function<B(A)>, Node<A>*) [with A = int; B = float]':
<source>:82:28:   required from here
<source>:68:32: error: no matching function for call to 'fmap2<int>(std::function<float(int)>&, Tree<int>*&)'
   68 |     return new Node<B>(fmap2<A>(f, t->_left),
      |                        ~~~~~~~~^~~~~~~~~~~~~
<source>:62:10: note: candidate: 'template<class A, class B> Leaf<B>* fmap2(std::function<B(A)>, Leaf<A>*)'
   62 | Leaf<B>* fmap2(std::function<B(A)> f, Leaf<A>* t) {
      |          ^~~~~
<source>:62:10: note:   template argument deduction/substitution failed:
<source>:68:39: note:   cannot convert 't->Node<int>::_left' (type 'Tree<int>*') to type 'Leaf<int>*'
   68 |     return new Node<B>(fmap2<A>(f, t->_left),
      |                                    ~~~^~~~~
<source>:67:10: note: candidate: 'template<class A, class B> Node<B>* fmap2(std::function<B(A)>, Node<A>*)'
   67 | Node<B>* fmap2(std::function<B(A)> f, Node<A>* t) {
      |          ^~~~~
<source>:67:10: note:   template argument deduction/substitution failed:
<source>:68:39: note:   cannot convert 't->Node<int>::_left' (type 'Tree<int>*') to type 'Node<int>*'
   68 |     return new Node<B>(fmap2<A>(f, t->_left),
      |                                    ~~~^~~~~
```

Of course! What if `t` is a `Tree<A>` and not a `Leaf` of a `Node`?! That shouldn't happen, but even in the original code we have a case to handle that, so let's just add:

```cpp
template <typename A, typename B>
Tree<B>* fmap2(std::function<B(A)>, Tree<A>*) {
    return nullptr;
}
```

Now the code compiles, but the executable returned *139*. In other words, we have a segmentation fault.

Right, `Node` has two pointers to `Tree` and instead of matching one of the overloads for `Leaf` or `Node`, the one for `Tree` is matched which returns a `nullptr` and when we try to print the labels we crash.

In case of function overloads the static type is matched. If we want runtime dispatching, we need `virtual` functions and overrides.

So what if we'd implement a *virtual* `clone` function? The initial idea might seem nice, but the problem is that we'd also have to either apply `f` on the member or just use its return type (marked by `typename B`). In order to do so, we could apply an extension of the prototype design pattern, where a *virtual* `clone()` method replaces a *virtual* constructor which doesn't exist in C++. The only problem is that we would need to extend the pattern by passing the transformation function (`f`) to the `clone()` as a parameter. But `f` is a template type and we cannot combine a virtual function with a template.

We'd need to somehow erase `f`'s return type by the time we reach `clone()`. But you have to know where to stop and when going down the rabbit hole is not worth it - in my opinion.

Let's just accept that `dynamic_cast` has its merit with *this* `Tree` implementation.

But do we really need it?

## Prefer composition over inheritance

Probably we have all heard in many places that we should prefer composition over inheritance.

We are also probably familiar with the KISS principle which means that we should *keep things simple, stupid*.

Now let's have a look at the above `Node` class once again:

```cpp
template<class T>
struct Node : public Tree<T> {
   Tree<T> * _left;
   Tree<T> * _right;
   Node(Tree<T> * l, Tree<T> * r) : _left(l), _right(r) {}
};
```

We have a derived class inheriting from a templated base class. This derived class stores two pointers to objects of the base class type. We use templates, composition and inheritance at the same time. That cannot be KISS.

I don't think that it's needed.

We do need templates given that we want the ability to store different types in the tree.

We also need composition, as in a node we want to store references to the underlying two trees.

To simplify the `Tree` class and also hope to remove the need for `dynamic_cast`, let's get rid of the class hierarchy.

*(I'm also omitting the issue of memory management as I want the two solutions to remain easily comparable)*

What does that inheritance give us anyway?

Just by looking at the type, we know whether we deal with an intermediary node or a leaf in the tree. But we can know that in other ways too.

```cpp
template<typename V>
class Tree {
 public:
    Tree(Tree* l, Tree* r): left(l), right(r) {}
    Tree(V l): label(l) {}

 private:
    Tree<V>* left = nullptr;
    Tree<V>* right = nullptr;
    std::optional<V> label = std::nullopt;
};
```

In the above `Tree` class we have three members. Two pointers two other `Tree`s and we also store an optional label.

We expose two constructors:
- the one taking two pointers to `Tree`s initializes an object that corresponds to the former `Node` class
- the other taking an instance of the template argument type initializes an object that will serve as a `Leaf`

As the members are private, we either use `Tree` one way or the other. By looking at the initialization, we'll know which way it is used. If you really want to know it during runtime, we could query an additional member `bool is_leaf;` which would be initialized in the constructor call. But I don't think that we need that knowledge. *(If the `label` cannot be `nullopt` in a leaf, we could also check the state of the optional `label`)*

Also, we'd need to expose certain getters to the members of this class, but we don't need them for the `fmap()` implementation. You might argue that if the accessors are non-`const` then people can misuse the class. And that's a valid concern. But it takes extra effort (*why would you do that?*) and it strikes out in a pull request, it's easy to spot such misuses (*why would you approve that?*).

The `fmap()` implementation is fairly simple:

```cpp
template<typename V, typename N>
Tree<N>* fmap(std::function<N(V)> f, Tree<V>* t) {
    if (!t) { return nullptr; }
    return new Tree(fmap(f, t->left),
                    fmap(f, t->right),
                    t->label ? std::optional<N>(f(*t->label)) : std::nullopt);
}
```

If `t` is `nullptr` then we stop the recursion by returning `nullptr`. Otherwise, we return a new `Tree` using the mapped type (`V -> N`).

In order to make this work, we need a new `Tree` constructor that can initialize all the members. But nobody else needs that so let's make it `private` and make `fmap()` a `friend` of `Tree`.

```cpp
template<typename V>
class Tree {
 public:
    Tree(Tree* l, Tree* r): left(l), right(r) {}
    Tree(V l): label(l) {}

 private:
    Tree(Tree* l, Tree* r, std::optional<V> v): left(l), right(r), label(v) {}

    template<typename V2, typename N>
    friend
    Tree<N>* fmap(std::function<N(V2)> f, Tree<V2>* t);

    Tree<V>* left = nullptr;
    Tree<V>* right = nullptr;
    std::optional<V> label = std::nullopt;
};
```

## Which one is better?

I don't have a clear answer.

In the original solution, you can see from the (dynamic) type whether you deal with a node or a leaf. But you also need a polymorphic structure and all the overhead that comes with it. You don't only see this characteristic, but each type represents a clear role. 

You might say that my solution is not complete because we need to expose the members, but there is a very high chance that the original `Tree` wouldn't stay a `struct` and the members would be `private` in that too. But even in that case, the second solution (if the accessors are non-`const`) might be misused. Even though it's not convenient and it is highly visible.

But what about performance?

We can expect that the second solution will have a bigger memory footprint as in every case each node of the `Tree` has 3 members. Two pointers and an optional `V`.

On the other hand, we can expect that the second solution will be faster, as there are no `virtual` functions, there is no dynamic dispatching of function calls and obviously no casting.

I created a small `Tree` of 8 integers and applied a function on it which multiplies each value by 2 and ran it 10,000 times. The static solution took a bit longer time to compile (~3%), and its executable was a tiny bit smaller (~2%), but it was executing much faster (~50%). Even though as expected, the memory consumption was 30% higher for the non-polymorphic version.

So the second solution is smaller, much faster, but it needs way more memory at runtime. It's simpler, but in a way, it's less expressive.

In most cases, there is no black or white. Only tradeoffs. In this case, we traded off memory for runtime. Simplicity for expressiveness.

## Conclusion

Today, we talked once again about when (not) to use `dynamic_cast`. We looked into a tree implementation where `fmap()` needed to use `dynamic_cast`. Then we looked into another where it was not needed. The `dynamic_cast` version needs more time to execute but much less memory than the non-polymorphic version.

Programming, just like life, is about tradeoffs. In this case, the solution with `dynamic_cast` has its merits, but you might go with another solution depending on your constraints.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!