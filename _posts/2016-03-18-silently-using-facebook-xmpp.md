---
layout: post
title: Silently Using Facebook XMPP
date: 2016-03-18 11:19:03
modified: 2016-03-19 09:55:30
author: Dhaval Chauhan
description: XMPP login was not listed in Sessions Tab. Facebook could be accessed via XMPP and can be used as messenger. Using Jabber client you can connect at chat.facebook.com:5222
tags: []
categories: []
permalink: /blog/silently-using-facebook-xmpp/
---

TLDR; XMPP login was not listed in [Sessions](https://www.facebook.com/settings?tab=security&section=sessions&view) Tab

Facebook could be accessed via XMPP and can be used as messenger. Using Jabber client you can connect at chat.facebook.com:5222

## Bug

But this active login was not listed in Setting > Security > Sessions

So X can login into Y's account (Provided X know the credentials) X can chat without Y being able to see the active session

Additional bug that existed along side was if you login into Facebook using XMPP and forget to logout you will have to reset your password and check "Logout other sessions" to logout remotely.

## The Fix

The bug was fixed and chat section was added in Sessions Tab

![Facebook Sessions Fix](/assets/images/silently-using-facebook-xmpp.png)

## Timeline

- Bug reported - June 18, 2014
- Report rejected - June 18, 2014
- More info sent - June 18, 2014
- Bug triaged - June 19, 2014
- Bug Fixed - August 15, 2014
