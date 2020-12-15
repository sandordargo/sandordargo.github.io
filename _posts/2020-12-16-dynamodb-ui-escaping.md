---
layout: post
title: "A story of nasty a bug: AWS DynamoDB UI special character escaping"
date: 2020-12-16
category: dev
tags: [aws, cpp, database, python]
excerpt_separator: <!--more-->
---
Recently, I've built [Daily C++ Interview](https://www.dailycppinterview.com/) and since the beginning, I kept something important in mind. I need to provide value, and I don't need a perfect implementation for that. I don't need to automate a process if I barely have to perform it, especially if it's simple and doesn't take many efforts.
<!--more-->

It's different when your purpose is practising automation or just practising building tools. Then you might want to automate everything just for the sake of automation. It's _l'art pour l'art_. But in this case, I just want to deliver valuable content to my audience.

Mostly I just glued services together, it's almost a no-code service from my perspective, I had some interesting bugs to fix and I'd like to share one of them with you.

One of the activities I have to perform every day is adding a new question (and of course the answer) to my database. I'm using [AWS DynamoDB](https://aws.amazon.com/dynamodb/) and I didn't create any custom admin tools to add my data, I simply used DynamoDB's UI for at least a month, until...

## I discovered a problem with my content

One thing, I'm really prudent of is that I only share valid information and that my code examples compile out of the box. Hence usually even if takes more space, I include the header inclusions and the `main()` function to my C++ sample code.

I also have a friend who is - among his other activities - teaching C++ at a university and he free access to my [Pro membership](https://www.dailycppinterview.com/checkout/) and when he has time, he reviews the mails I'm sending out.

One evening he contacted me saying that it's strange that when I share the original and the desired output of a code sample, both are the very same.

I checked and indeed he was right. So the easiest thing for me was to open the code-sample in [coliru](http://coliru.stacked-crooked.com/) and make the necessary changes to get the output I wanted to share.

It didn't compile. I forgot to add the template type to a vector:

```cpp
std::vector v {1,2,3};
```

And while this is valid code in C++20, I don't use yet that version in my samples. Moreover, a simple `#include` without specifying the included header file will never be valid, and my code sample started like this

```cpp
#include
#include

int main() {
  //...
}
```

I felt a bit peeved on how sloppy I was when I saved this piece of content. I didn't quite understand how I could do that, but well, we all make mistakes, maybe I was just too tired at the end of the day.

A few days later, another person contacted me that I have a syntax issue in an exercise. 

What the hell?

I checked and he was also right. It was the very same type of problem. It's impossible that I made these mistakes in a row.

I was sure that something was broken in my pipeline.

## Code as HTML tags

In order to format the raw content into something I can send to my subscribers, I'm using two free services:
- [markdowntohtml.com](https://markdowntohtml.com/) to convert markdown texts to HTML code
- [Hilite.Me](http://hilite.me/) to convert raw code samples into well-formatted HTML content.

So for example these lines:

```cpp
#include <iostream>

int main() {
  std::cout << "Hello world\n";
}
```

Would be formatted into something like this:

![Code foramatted with hilite.me]({{site.baseurl}}/assets/img/hiliteme.png "Code foramatted with hilite.me")

Notice how `#include <iostream>` is escaped and transformed into `#include &lt;iostream&gt;`.

So I take the different parts from [Markdowntohtml](https://markdowntohtml.com/) and from [Hilite.me](http://hilite.me/), assemble them into one HTML document and I just copy-paste the big string into DynamoDB UI.

When I copied my content all seemed fine. 

Then my e-mail was sent, and instead of `#include <iostream>` only `#include` was there. Instead of `std::vector<int>` only `std::vector` could be read.

When I checked the source code of the page with my e-mail on it (`Ctrl+U` in Chrome), I saw the missing `<iostream>` and `<int>` in it.

It started to become clear. The includes and template parameters are handled as - never closed - HTML tags. But where is the problem? Is there a bug in [Hilite.me](http://hilite.me/)? In Chrome? Or...

In fact, the problem was with DynamoDB UI when I clicked on saving my new item, all the escaped HTML special sequences were transformed into plain HTML tags. So after the data was fetched and sent out in an e-mail, the browser handled included headers (`<iostrea>`) and template parameters (`<int>`) as HTML tags not as code.

## Do it yourself and save time

I spent some time looking for some DynamoDB UI settings but in fact, I found none.

So as a solution, I implemented a small helper script that connects to the database and uploads my content. Luckily then the escaped characters are kept escaped.

The script is very simple as you can see:

```py
import boto3

def add_question(title, question, answer, id, teaser):
    client = boto3.client('dynamodb', aws_access_key_id='<YOUR ACCESS KEY ID>',
                          aws_secret_access_key='<YOUR SECRET ACCESS KEY>', region_name='us-east-1')

    client.put_item(TableName='questions',
                    Item={'title': {'S': title},
                          'question': {'S': question},
                          'answer': {'S': answer},
                          'id': {'N': id},
                          'teaser': {'S': teaser}})

```

Then you just to call `add_question` with the right parameters.

The key is that with `boto3.client` you make the connection to DynamoDB and with `put_item` you can upload the data into your table.

As simple as that.

## Conclusion

There are a couple of morals to this story.

If you build content, keep an eye on what reaches your users and if you can ask someone else to have a look at well. Don't just read what you upload, but check what gets published. Even if you are sure that what you write is of high quality, make sure that your end-users receive the same content.

This doesn't mean that you have to reinvent the wheel just for the sake of writing some code and tools, but make sure that you have all the tools that you need.

In my case, writing such a small and easy-to-use helper script even eased how I add content to my database. Updating a couple of parameters and hit on _Run_ is more convenient than going to the UI.

Let me know what kind of strange bugs you encountered when you built your side-projects!

## Connect deeper

If you found interesting this article, please [subscribe to my personal blog](http://eepurl.com/gvcv1j) and let's connect on [Twitter](https://twitter.com/SandorDargo)!