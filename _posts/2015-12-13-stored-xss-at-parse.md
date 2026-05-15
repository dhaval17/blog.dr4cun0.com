---
layout: post
title: "Stored XSS at Parse"
date: 2015-12-13 17:25:34 +0000
modified: 2015-12-13 17:32:03 +0000
author: Dhaval Chauhan
twitter: "@17haval"
description: "A stored XSS vulnerability found in Parse Gallery"
---

This blog is about the stored XSS at Parse

Not very long time ago Parse had [Parse Gallery](https://www.facebook.com/parseit/posts/152861228171239) where you could [submit](https://parse.com/questions/how-do-i-add-my-app-to-the-parse-gallery) your application

It's settings menu `https://www.parse.com/apps/YOUR_APP/edit#settings`

So, I tried to make submit the form with the `javascript:alert(1)` in URL parameter and it worked

Here's how it work, 
You create an app, insert malicious link and publish it

But the moderators need to validate your app before it is listed. But you can create a app with valid credentials and when it's listed change the URL with `javascript:alert(1)`

**Timeline:**

- Jan 5 2014 – Report Sent
- Jan 6 2014 – Report confirmed and sent for further investigation
- Jan 16 2014 – Fix confirmed
- Jan 18 2014 – Bounty Awarded
