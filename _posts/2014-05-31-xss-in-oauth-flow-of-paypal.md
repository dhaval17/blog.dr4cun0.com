---
layout: post
title: XSS in OAuth flow of Paypal
date: 2014-05-31 18:40:00
modified: 2014-10-02 10:35:23
author: Dhaval Chauhan
description: One fine night while tinkering with Paypal's REST API lead to XSS in their OAuth flow. Paypal REST API provides a simple payment solution, but analyzing the authentication revealed it was only client-side validation.
tags: []
categories: []
permalink: /blog/xss-in-oauth-flow-of-paypal/
---

Again, One fine night while tinkering with Paypal's REST API lead to XSS in their OAuth flow.  
I quickly made a working POC and sent to paypal.

## POC

Paypal REST API provides a simple payment solution.  
So, basically one needs to create an application in developers section and integrate it with the website.

But what lured me was "Return URL" section:

![Paypal Return URL Configuration](/assets/images/xss-paypal-return-url.png)

I tried few vectors but there was some authentication which required "HTTP:// or HTTPS://" in the beginning of the string.

But analysing the authentication it was just **Client Side Authentication**.

![Client Side Authentication Check](/assets/images/xss-paypal-client-auth.png)

## Exploitation

So, I fired up burp suite and edited the "oauth_return_url_live" parameter:

![Burp Suite Request Modification](/assets/images/xss-paypal-burp-request.png)

Bingo, the payload was successfully injected!

![XSS Payload Injected](/assets/images/xss-paypal-payload-injected.png)

Lastly, all I did was put the things in order:

![Final XSS Exploit](/assets/images/xss-paypal-final-exploit.png)

## Impact

So, Victim visits your site and as soon as he pays you with his paypal account.  
His account becomes your account.

This vulnerability worked in Opera and Older versions of Firefox.  
I was rewarded a decent bounty for this by Paypal Security.
