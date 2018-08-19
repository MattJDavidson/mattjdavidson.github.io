---
layout: post
title: Creating a write only twitter client
---

A couple of years ago I read [Deep Work](https://www.goodreads.com/book/show/25744928-deep-work) by Cal Newport. More recently I've been identifying with the difficulty to be creative, wondering where my time is going and feeling generally unproductive. I don't believe there are any silver bullets in this space, but I do agree that Social Media, [and the drip feed of news is generally unhealthy](https://www.ted.com/talks/cal_newport_why_you_should_quit_social_media). I quite facebook years ago and do not miss it. 

I don't want to quit twitter, there are useful discussions here and it's a great way to keep in touch with an extended network of people involved in my workspace. However, I would like to mindlessly stop browsing it to fill 30 seconds of "boredom".

It's possible to remove the news feed by altering the css. I'm using the [stylebot](https://chrome.google.com/webstore/detail/stylebot/oiaejidbmkiecgbjeifoejpgmdaleoha) chrome extension. I add the following CSS to twitter:

```css
#timeline > div.stream-container.conversations-enabled {
    display: none;
}
```

This should help to:
1. Keep a twitter account
1. If I mindlessly wander to twitter, I'm less likely to spend 5 minutes aimlessly scrolling
1. Write intentional tweets on twitter's website
1. Interact with twitter notifications via email
1. Only read a weekly email summary of popular tweets (hopefully lessens the feeling of [fomo](https://en.wikipedia.org/wiki/Fear_of_missing_out) on twitter)
1. Have the confidence to delete twitter from my phone too 

P.S. something like [SelfControl](https://selfcontrolapp.com/) is very effective for reducing general procrastination website use, especially reddit!
