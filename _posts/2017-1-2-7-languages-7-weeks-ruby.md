---
layout: post
title: "Ruby is the first one coming"
date: 2017-1-2
category: books
tags: [7languages, ruby, books]
header: "I decided to read <a href=\"http://amzn.to/2xV90jo\">Bruce Tate's Seven Languages in Seven Weeks</a> and I just finished with the first one: Ruby!"
---


Why I wanted to read the [book](http://amzn.to/2xV90jo)? At work I use mostly 3 languages: C++, Java and Python, depending on my actual project/task. I tend to read a lot of technical books and articles, so I hear often about other languages, other paradigms which are quite unfimilar to me. I have been thinking more and more about learning something completely new (to me) such as functional programming.

Then I ran across [this book](http://amzn.to/2xV90jo) and I knew it is for me. It is challenging enough and should pay off quite well.

So I started to read, started to play with the examples and completing the exercises. 

The first language is Ruby, yes it is not FP yet, but Haskell will come too later.

I don't want to write a wrap up about Ruby. Many others did that already and there are great tutorials on the web. I'd rather mention a just few stuff I really liked:

**Open classes**

A class in ruby can be "opened up" and modified anytime. It means that if you want to attach some behaviour to any existing class, you can do it anytime.

**Numbers are objects**

Why is that so great? Because numbers can also have behaviour. So you can write `4.times {|i| puts i}` and it will count from 0 to 3. It is that simple!

**Method missing**

If you try to invoke a non-existing method of an object you will have to face with a `NoMethodError` exception. But you have a safety net, method_missing. It is given the symbol of the non-existent method, the array of the arguments and any block passed to it. Sounds like you can handle nasty events gracefully.

Not bad. But it is much more than that! It a very important tool of metaprogramming. It let's simplyfy things, creating more readable code or even Domain Specific languages.

Just don't get carried away, as always you should find a golden mean.

I just scrached the surface of Ruby, but I liked what I have seen so far. I will keep on practicing.

If you don't know it yet, I advise you to learn it a bit and practice. It's all about readable coding. If you get the taste while doing some Ruby, you might write more readable code in other languages too.

Good luck!
