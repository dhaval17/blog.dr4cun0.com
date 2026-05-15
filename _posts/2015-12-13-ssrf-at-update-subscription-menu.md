---
title: SSRF at Facebook Update Subscription Menu
date: 2015-12-13 17:03:01 +0000
categories: [Security, SSRF]
tags: [facebook, ssrf, vulnerability]
author: Dhaval Chauhan
---

This blog is about the SSRF I found in Facebook.

A while ago I was fiddling with Facebook's Developer Section and I noticed **Update Subscription Menu**.

![Update Subscription Menu](/assets/images/ssrf-facebook/subscription-menu.png)

So, basically this menu allows you to enter a URL and Facebook pings and checks for the token at the entered URL.

## Testing Port 80

Let's ping some ports. I entered: `http://dr4cun0.com:80/`

![Test on Port 80](/assets/images/ssrf-facebook/test-port-80.png)

Test failed as expected.

## Testing Non-Existing Port

Now a non-existing port: `http://dr4cun0.com:1234/`

Test fails again but with a different response code.

## POC Time

Luckily, the content-length of the response was more or less constant:

- **Between 4363 – 4365**: Port Open
- **Between 4428 – 4430**: Port Closed

I automated the process with Burp Intruder, sorted the results with content-length, and reported it to Facebook Security. It took some time but the bug was fixed.
