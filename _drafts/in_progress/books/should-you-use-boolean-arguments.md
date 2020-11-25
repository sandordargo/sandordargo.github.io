**Why shouldn't we use boolean arguments?**

Because it reduces readability on client side.

Let's say you have such a function signature:

```
placeOrder(std::string shareName, bool buy, int quantity, double price);
```

In the implementation of this functions, it will be all fine, you might have something like this somewhere in it:

```
//...
if (buy) {
  // do buy
  // ...
} else {
  // do sell
  // ...
}
```

That's maybe not ideal, but it is readable.

On the other hand, think about the client side:

```
placeOrder("GOOG", true, 1000, 100.0);
```

What the hell true means? What if we send false? You don't know without jumping to the signature. While modern IDEs can help you with tooltips showing the function signature even the documentation - if available -, we cannot take that for granted, especially for C++ where this kind of tooling is still not at the level of other popular languages.

And even if you look it up, it's so easy to overlook and mix up true with false. 

Instead, let's use enumerations:

```
enum class BuyOrSell {
  Buy, Sell
};

placeOrder("GOOG", BuyOrSell::Buy, 1000, 100.0);
```

Using enumerations instead of booleans slightly increases the number of lines in our codebase (well in this case with three lines), but it has a much bigger positive effect on readability. Both on API side and on client side, the intentions become super clear with this technique.

Reference:
- [Matt Godbolt: Correct by Construction​](https://www.youtube.com/watch?v=nLSm3Haxz0I)