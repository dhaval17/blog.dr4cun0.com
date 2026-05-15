---
layout: post
title: "Stored XSS at exchange.onavo.com"
date: 2017-02-12 00:02:52 +0000
modified: 2017-02-12 00:04:05 +0000
author: Dhaval Chauhan
twitter: "@17haval"
description: "A stored XSS vulnerability found on exchange.onavo.com during security research"
---

Upon sub-domain enumeration I found this sub-domain `exchange.onavo.com`

It was some outdated app, and some fun for me

Though it was quite an app (while I didn't quite understand the purpose), All it had was simple feature of creating something called "Child"

Created a child with name : `[script>alert(1);</script>` 
And BAM ! XSS

But it's a self XSS and I didn't see any share feature in the app

Then I remembered a weird login process the app had. 
Login page had only one input field "email"

You enter an email id and you get an email in your inbox with a link 
You click the link and you get logged in !

Something like this : 
`http://exchange.onavo.com/signin/79ada86f556da28ab358dcea3e7d664579a52`

Anyways, This gave me an Idea

I could give this login link to anyone. 
When he visits it the XSS is triggered and I could steal the cookies scoped to `*.parse.com`

The XSS was fixed by Facebook Security Team
