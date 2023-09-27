---
layout: post
title: "Be a good gardener of your scripts"
date: 2023-9-27
category: dev
tags: [automation, programming, scripting, watercooler]
excerpt_separator: <!--more-->
---
I wanted to share a few words on why you should care about your scripts.

Programming is a science. Programming is art. Programming is a craft. Programming is...

## Programming is like gardening

Different people categorize programming differently. Let's add another category - the idea is not mine! - programming is like gardening.

The code grows organically in different directions and your responsibility as a programmer is to cut back the different sprouts and branches where you feel the need to do so. You have to cut back the different parts so that your code grows in the directions you desire.

If you are a good gardener, you promote the healthy growth of your plants by removing the excess. You make sure that each plant and each branch receive sufficient resources to thrive.

What does this have to do with your scripts?

We often have tedious and boring tasks which are sometimes on their own as a form of preventive gardening. You remove the weed so that the rest can grow healthily. You have to remove old, unused files of blocks of code so that they don't use our scarce mental resources. Sometimes those boring tasks are parts of another project. You have to prepare the terrain so that it can accommodate new plants. You have to refactor the code so that adding new features becomes an easier task.

## Your scripts also deserve the attention

Sometimes you'll have no other option and you have to do these tasks manually. But sometimes you can write a script that helps you in your tedious, boring tasks.

You could push through migrating a hundred tickets from one tool to another saying that it'll be a one-off task and automation makes no sense there. If you spend a few hours automating it while it would also take just a few hours to complete the migration, why would you do it?

I think automation is more fun.

![A programmer move from KnowYourMeme]({{ site.baseurl }}/assets/img/programmer-move-from-knowyourmeme.png "A programmer move from KnowYourMeme")

And I also think that you should handle your scripts as small plants and use a version control system as their soil. Save them, name them well and organize them. Maybe they will grow into something useful, or maybe you'll have to cut them out later so that they don't steal precious attention.

## Old scripts as a source of future productivity

I don't advocate for useless automation. At the same time, what seems like a one-off task, will end up being a recurring activity or at least it will reappear in different forms. Let me give you an example.

We had to move from one project management tool to another. From the old one, we could export all our tickets in a csv format. Besides a ~~nice and easy-to-use~~ UI, the new one also has a REST API.

So we could break down the migration into two easy tasks here:
- read data into a decent format from a csv file we downloaded from the old software
- call the REST API of the new one to create the tickets

That's a matter of maybe half a day if you know Python.

But guess what? 

Probably 3 years before, I had a task where I had to transform one csv into another format while also manipulating the data following a few rules. That had three parts:
- read data into a decent format from a csv file
- manipulate the data
- write the data to disk in a different format

So the first part of ticket migration was essentially done, I had to update a few column names and of course create a new value class, like this one:

```py
class Ticket:
    def __init__(self, summary, description, epic=None, 
                 status=None, assignee=None, reporter=None, 
                 due_date=None, ticket_type=TicketType.TASK, parent=None) -> None:
        # Summary must be less than 255 characters.
        self.summary = summary
        self.description = description
        self.epic = epic
        self.status = status
        self.assignee = assignee
        self.reporter = reporter
        self.due_date = due_date
        self.ticket_type = ticket_type
        self.parent = parent
```

But 2 years ago I also faced a problem where I had to create tickets in the same project management tool on a mass level based on some static code analyzer's (SCA) findings. That had two parts:

- query the data from the SCA REST API
- call the project management tool REST API and create some tickets!

Bingo! I have already done the second part of the current migration task!

While none of the parts matched the current needs 100% and I didn't write them in a way that was aimed at the highest possible level of reusability, I had the most relevant parts ready and separated well enough so that I could come up with a new tool in a matter of an hour or two.

Don't get me wrong, I'm not bragging about it. That's not my goal.

My goal is to tell you that if you take the time to automate certain things, you should
- save your "one-off" scripts
- organize them well, so that you can find them again if you need them
- in your scripts, separate the concerns. Don't over-engineer them, but do at least the bare minimum. In our example that minimum is to separate the csv reading part from the REST API handling part.

## Old scripts buy you time

If you garden your scripts well and take on some automatable tasks, you'll observe that after a few such tasks, you'll become much faster. You'll have many different parts in your hands and you can combine them in various ways. As such, for so many [boring tasks](https://www.sandordargo.com/blog/2023/08/30/the-value-of-boring-tasks) you'll need so little time compared to others.

Imagine that you have already written a few scripts that:
- extracts data from your SCA
- creates tickets on your project management platform
- reads in a csv
- parses through a directory looking for patterns
- updates files by searching and replacing some texts in it

First of all, by now you know how to do them and second, you already have the parts available. Now you have to tailor and combine them. Your collection of small scripts might make you much more productive in such tasks compared to others.

But the point is not having an advantage over others, but being able to perform otherwise boring and tedious things faster. Then it's up to you how you communicate and use the extra time you gain.

You might just say that you have all this extra time because you had the script and you are ready to take on the next task. You might reserve some time before getting back to the team to further garden your scripts so that on the next occasion you'll have an even easier time. Or you might do as if you had no automation and you act like those who automated their jobs but never told anyone about it.

There might be teams, companies, or situations where this latter approach is your best option, but in general, I don't think you should do that. Usually people like those who automate things, in other words, those who solve problems in a smart way have a better reputation. Besides, there is a fair chance that others think about it too. So if you don't say that you automated something, someone else will do the same thing you already did.

I'm personally for option two. There might be situations where option one is preferable because you are so much in a hurry... Otherwise, I would say reserve some of the time you gained and invest it into gardening your scripts. I don't have an exact recommendation, but if you saved 10 hours for your team because you have well-gardened scripts, take one or two hours so that next time, it'll be even easier to find and use the necessary parts.

Who knows, maybe by the time you have so many parts on a specific topic, you release something on Github or internally...

## Conclusion

In this article, I claimed that programming is like gardening. As a gardener, you have to make sure that your code grows in the intended direction. I claim that your scripts deserve similar attention than your main projects.

If you take care of your scripts and make sure they are easy to find, and different parts are easy to dissect and reuse, they will buy lots of time. They will not only save you from doing too many manual, boring tasks, but they will also make you a better coder and your reputation will grow. 

How do you deal with one-off scripts?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!