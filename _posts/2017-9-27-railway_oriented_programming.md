---
layout: post
title: "Programming on rails: Railway Oriented Programming!"
date: 2017-9-27
category: dev
tags: [code, design patterns, monads, java]
header: "<a href=\"https://twitter.com/mathbruyen\">A good colleague of mine</a> created a pull request in which he spoke about monadic behaviour, an alternative way of error handling, so we called him for a quick meeting where he explained something which he called Railway Oriented Programming."
---
As explaining is one of the best ways to understand, let me give a try here.

Many times when you start writing a program, first you will consider that things would go well (do TDD instead!), so you end up designing something which you might call the happy path later on. Let's take an example of a user who wants to update his phone number in a system in which user data is stored. What is happening on the server side?

```
Recieve request with phone number
Validate the format of the phone number
Update existing phone number in the user's profile
Send verification message
Return the result to the user
```

Then either because of some discovered bugs or because you just did think about it, you start adding some error handling. First returning a status here...
```
...
boolean isFormatValid = validateRequest(request);
if (!isFormatValid) {
  return "Invalid format";
}
...
```

Then adding a try-catch block there...

```
try {
  updateStatus = db.updateRecordFor(request)
} catch(DBError error) {
  return "DB error: coulnd't update record";
}

```

Pretty soon you are going to obfuscate totally your code and especially your readers' minds. All of this just by iteratively adding the "unhappy path". It happened to me. Then I started to refactor the code, following Uncle Bob's Extract till you drop number one hit, but it was still ugly.

Are there other ways? Yes, there are.

Imagine that you have two parallel tracks. You have a bunch of railroad switches. Which means you can go from one track to the other. In our case it would mean that usually you could go from the happy path to the unhappy path, but sometimes even the other way around.

How to do it if you don't want your code to look like a huge pile of if-is-valids?

Here comes the monadic approach into picture.

What you want is a block that can be responsible for holding and handling the actual state of your process either if it's on the happy or on the unhappy side of the tracks. What you don't want is `if`s all around the business logic.

Now think about Java's `Optional<>`. It's a similar concept. You have an object which might hold something, or it might hold a `null`. In other languages maybe it is called `Maybe` - people hate me because of puns. You can define the behaviour in any case easily as such:

```
Optional<Record> record = db.findRecordBy(key);
return record.orElseThrow(throw new RecordNotFound());
```

So it can be just:
```
return db.findRecordBy(key).orElseThrow(throw new RecordNotFound());
```

Instead of having:

```
Record record = db.findRecordBy(key);
if (record == null) {
  throw new RecordNotFound();
}
return record;
```

Let's call our new object `Try<>`. It exists in other languages such as `Result` or `Try` as well, etc... It has two members, one for the happy path and one for the unhappy path. The happy member is generic, while the other one is an `Exception`. So in Java it'd look somehow like this:

```
public final class Try<T> {

  private final Exception exception;
  private final T value;

  private Try(T value, Exception exception) {
    this.value = value;
    this.exception = exception;
  }
  //...
}

```

With a bunch of query methods you can know whether you are in a success or in an error case and you can get either the exception or the happy value.

So far so good, but it's more like just something which makes other people's lives more miserable around your code while you might appear a smart guy. Where are rails or railguns anyway? We haven't seen them.

Railguns are pretty cool weapons which make a big damage, but it takes quite some time for them to reload and their usage needs high precision, but if you are handling them well, you are a master in Quake, but you cannot rocket jump while... oh wait... that's another article... a quite old one...

So as we mentioned there, can be switches on rails or better to say on a track. In fact imagine that you have the two tracks running parallel and there is a segment missing from both. You can fit in three different kinds of elements. Let's check them one by one.

## map()
  
A map will keep you on track, there is no switch in it. This element will keep you on the happy path, whatever happens. (Put unexpected exceptions aside). As an input it will take an object implementing the [Function interface](https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html) (referred simply as a function from now on) which will transform the happy variable into another happy variable - their type can change: imagine a juicer which will take an orange and will give you orange juice on the other side, it cannot really give you a rotten orange.

```
  public <S> Try<S> map(Function<T, S> transform) {
    if (isError()) {
      return castError();
    } else {
      return new Try<S>(transform.apply(value), null);
    }
  }
```

## flatMap()

Now this is a railway switch. It might lead you to the unhappy path but it can also keep you on the happy path depending on the outcome of the transformation it performs. Even if staying on the happy path, the types can change. Imagine that orange juicer in a worse edition. You give it an orange and normally it will give you its juice. But if something goes, wrong it will cut in your juice some of the skin too. That latter one is the unhappy path.

```
  public <S> Try<S> flatMap(Function<T, Try<S>> transform) {
    if (isError()) {
      return castError();
    } else {
      return Objects.requireNonNull(transform.apply(value));
    }
  }

```

## orElse()

This can also be considered as a kind of a switch, but it's rather a join. It will take you from the unhappy path if you are there and join you in the happy path. Otherwise it just keeps you on the happy path. At the end of your juicer you have a filter, so the skin won't end up in yout juice - but the fibers will, this is a great filter!

```
  public T orElse(Function<Exception, T> restore) {
    if (isError()) {
      return restore.apply(exception);
    } else {
      return value;
    }
  }
```

Putting it altogether this monadic approach of error handling is a useful concept in complex scenarii where you can fail at many different points of the execution flow. It adds some complexity to your code as you have to write your `Try` monad and the transformator functions too, you have to understand a few concepts with which hopefully this article helps you a bit, but on the other hand can simplify your business logic a lot. Up to you to decide whether you use `if`s, try-catch blocks and a messed up, really error-prone business flow, or a lot of boiler plate and a simple business logic. My best guess is that it really depends on the size and complexity of the project which one is better to choose.