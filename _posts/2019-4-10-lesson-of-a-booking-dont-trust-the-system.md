---
layout: post
title: "Lesson of a booking: Don't trust the system!"
date: 2019-4-10
category: dev
tags: [ux, error-handling, travelling, story]
excerpt_separator: <!--more-->
---
We were going on holidays recently and we were going far away. We were travelling to Vietnam. The last time before my daughter would go pre-school and my son would start walking. In other terms, the last time before our schedule would become more fixed and the last time we can buy an infant ticket to my son and saving some money.
<!--more-->

We had an 18-hour long connection (thanks to our choice) somewhere in the middle and I wanted to book a hotel room. Even better, a room for free as it was a long connection and our airline offered some related promotions.

Have you ever felt in your life that you had to cope with contradictory processes? 

It was definitely the case with this booking.

The normal process - without taking into account the promotions - says that you should do the booking when you apply for the transit visa. Pretty straightforward!

In the promotion, it is written that you should make your booking until the end of January if you travel before the end of April.

Still fine, even though it might raise some questions. We were to travel between the end of February and the middle of March. Three long and adventurous weeks. Let's apply for the visa in January then and everything will be just fine and I could repress my questions.

At the _manage my booking_ section, I clicked on the addition of a transit visa. Suddenly, I got an error message that it was too early to do so! Visa requests should be made no earlier than 28 days before the travel. Hm. Okay. I'd check it at the end of the month, just like 28 days before the flight. 

But in fact, which flight?

The outbound or the inbound? Hm... We had our long connection during our inbound one, so when we were coming back from Vietnam to France. 28 days before that would have meant mid-February. If I had to reserve the hotel room when applying for the visa, it would have meant that I couldn't do it before the end of January so I couldn't benefit from the promotion. It did not really make sense. 

As my wife pointed a bit later, I was thinking about a non-existent problem.

The error message simply said that it is too early to apply for a visa, so I didn't even think that I don't it.

As the need for a visa depends on your nationality - which doesn't change very frequently -, to me it would make more sense to have some form of validation like this one:

```
if (travellerNeedsVisa()) {
  if (isTimeWindowForVisaApplicationOpen) {
    // ...
  } else {
    alert('too early to apply, come back later');
  }
} else {
  alert('no need for a visa, enjoy your travel');
}

```

Instead, they probably had something more like this:

```
if (isTimeWindowForVisaApplicationOpen) {
  if (travellerNeedsVisa()) {
    // ...
  } else {
    alert('no need for a visa, enjoy your travel');
  }
} else {
  alert('too early to apply, come back later');
}

```

Maybe this latter form has its own good reasons. Something I couldn't really think of right now. Anyway, it's as it is and that logic tricked me a bit.

As soon as my wife told me that we simply don't need a visa, I moved on with the booking.

In the conditions, it was clearly written that in one room there can be no more than two adults and one child accommodated. I feel it a bit strict, but I'm okay with that. On the other hand, there was nothing written about what should be done in other cases. As we're travelling with two kids, I'd have been pretty much interested in those _other cases_.

I dropped them a mail explaining what I would like and what information I'd need.

Luckily, they replied fast. Unluckily, they didn't seem to actually read my mail properly and they only said that even an infant is considered a child, but they didn't tell me if I could book two rooms with the same booking number.

New round.

Finally, I've got the information I needed. For the two adults on the same booking but a different eTicket number, I could book two rooms.

Then I made a mistake in the search. For each room, I indicated one guest, only the kids. After clicking on the search button, I didn't receive an error message or an empty list of hotels, but rather an empty page where even the map was not displayed. I could only see three grey boxes.

I clearly had no idea what was going on.

Then I figured out that I should have only declared the kids as our guests but ourselves too as the guests of the hotel. This was clearly my silly mistake. But the grey boxes of total emptiness didn't give me any indication of what happened, which is a pity in terms of error handling and user experience.

Finally, I managed to make the booking and interestingly only one eTicket number had to be given and the two rooms came free. The hotel was fabulous we had a good time during those 18 hours.

I want to avoid any consequences, hence I don't share screenshots nor even the city's name, not to mention the airline itself.

The goal of this article was to show how improper error handling and lack of information can influence user experience. Sidenote: the same site can be used for non-free bookings.
