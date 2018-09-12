---
layout: post
title: "My first Flutter app"
date: 2018-9-12
category: books
tags: [dart, flutter, mobile development]
excerpt_separator: <!--more-->
---
If you are following my posts, you might remember that I went to [Riviera DEV 2018](/blog/2018/06/01/i-went-to-riviera-dev-2018) recently where I attended a really interesting talk about Flutter by [Florian Loitsch](https://github.com/floitschG). [Flutter](https://flutter.io/) is a framework to develop native interfaces for Android and iOS devices, yet you keep only one code base.
<!--more-->

I've developed a couple of Android apps mostly for my (and my wife's) amusement, and for my own education. I was amused by the ease with which Loitsch created a new Flutter app in front of us during his session. I wanted to give it a try soon.

I just needed an idea. While I was home ~~enjoying~~ (house)working through the paternity leave I got for my second child, once when I was going to the local DIY store, some soundingly very harsh lady was speaking on the radio quite outraged that Frech people including the French president himself are drinking way too much alcohol. She elaborated on the topic, citing the president, Macron who said that he drinks one glass of wine for every lunch and dinner. Thinking about a glass of 100 millis it's already 14 units a week, even if he doesn't drink anything else.

According to [some new study](https://www.bfmtv.com/societe/dix-verres-d-alcool-par-semaine-la-nouvelle-limite-fixee-par-les-experts-1157512.html), you shall not drink more than 10 units of alcohol a week and you should have two days without any alcoholic drinks. Sorry, I didn't find it in English. How lucky non-French speakers are, you still have a [recommendation of 14 units](https://www.drinkaware.co.uk/alcohol-facts/alcoholic-drinks-units/how-much-is-too-much/) :D (Sidenote: [English](https://en.wikipedia.org/wiki/Unit_of_alcohol) and [French](https://fr.wikipedia.org/wiki/Unit%C3%A9_d%27alcool) calculate units differently. Apparently, 14 English units correspond to 11.2 French units. Still. That's a small glass of wine.)

I was thinking that having a glass of wine for a meal is actually not too much. Now, this lady is saying that it is too much. Hm... How much do I drink after all? I should measure it. Bamm!

And the idea came. Of course, I could have downloaded an app... But it's way more interesting to build your own, isn't it?

So I started.

First and most important step, I wanted to save data about the drinks I consume. For that, an SQLite database seemed just perfect.

In the beginning, I wanted something simple, so I had two screens.

1. List all the drinks I had (saved).
2. Add a new drink. A drink has a name, a date of consumption, a volume, a strength, units of alcohol calculated from the previous two and an optional remark.

At this point, I only used the application for testing because there was a too important chance of data loss. I needed some synchronization.

But before, I added some small features to make the existing parts more comfortable, I added an edit and a delete function to the application.

![BoozeTracker List of Drinks]({{ site.baseurl }}/assets/img/boozetracker-list-small.png "BoozeTracker List of Drinks")

It was high time to get back to the synchronization part. First, I had to come up with a solution. So I had to clarify my needs.

I wanted not to lose data in case I have to install the app on another phone. I wanted to store the data I described earlier, the data of one single SQL table.

My data can be easily represented in a CSV format. In order not to lose data I can simply dump the data into a CSV file and send it via email. Not the most innovative solution, but I've already seen it in popular apps.

So as a first iteration, I exported the data to a CSV file and I e-mailed it to myself. As a next step, I wanted to be capable of importing that data.

I [published the csv](https://support.google.com/docs/answer/183965?co=GENIE.Platform%3DDesktop&hl=en) on my GoogleDrive and I read it from there simply with an HTTP GET request. Once you have the data in memory, it's just about updating your database. Yes, at this point, if I imported I simply replaced the whole content of my local data.

As you can feel this solution is okay to save data, but it's not comfortable at all, because I had to manually update my published spreadsheet from the latest available backup each time I wanted to perform an import.

Using Google Drive seemed to be a free and relatively comfortable solution so I started to discover its API. Unfortunately for Flutter, there is no native API available so far, thus I had to use their [REST API](https://developers.google.com/drive/api/v2/about-sdk). In the end, this might be a good thing as it's something cross-platform/cross-language.

By the end of this step, I stopped publishing the CSV file and I update and read the file through Google Drive API.

This solution still could be improved as it is still invoked manually (with a button) and one still completely overwrites the other. But it is good enough for my needs, so I went to implement other features.

My AppBar on the top of the screen was a bit overwhelming with a lot of buttons, so I implemented a [_drawer_](https://flutter.io/cookbook/design/drawer/) in order to ease the navigation between the different activities.

I also implemented some widgets (in Flutter everything is a widget) to provide statistics on alcohol consumption.

One, called `MainStats` provides the overall consumption and dry days of the last X days. It colours the values based on some limits defined by the user. (Green if you're below the limit, red if above.) You will also get the day when you drank the most - well if you could keep track of it... If you don't drink with moderation, it's hard to use any apps, I guess.

![BoozeTracker Statistics]({{ site.baseurl }}/assets/img/boozetracker-stats-small.png "BoozeTracker Statistics")

The big statistics page adds some more details, including a chart showing your drinking trends. I remember that I found it quite difficult last year to create a chart in Android, it took much more time than it took with Flutter. Maybe it's just a change in my experience, but probably it's more than that. The learning curve with [charts lib](https://google.github.io/charts/flutter/gallery.html) was quite short. I'll definitely write a more detailed post about it.

At this point, the application is good enough for me. I'm going to take some time on refactoring because I was focusing too much on the features. And while I generally like TDD, I haven't taken the time get familiar with the ways I can practice it with dart and Flutter, so I'll discover how to add tests.

Then I have to think about publishing. If I want to publish it, I'll have to make some additional changes.
- Implementing an automatic synchronization 
- Adding imperial/metric unit settings
- Preferably I have to add logic to prefill data (like the level of alcohol based on the name)

I gave time to myself until the end of September. At that point, I'll take on another mini-project.