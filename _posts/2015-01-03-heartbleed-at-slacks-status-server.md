---
layout: post
title: "Heartbleed at Slack's Status Server"
date: 2015-01-03
published_time: 2015-01-03T16:19:00.000Z
modified_time: 2015-12-13T09:34:51.486Z
author: Dhaval Chauhan
tags: [security, heartbleed, ssl, vulnerability, slack]
---

Slack has a bug bounty program on Hackerone 

One day I found this `status.slack.com`

This sub-domain looked different, maybe because this server was outside the main network. 

Though I could not find anything suspicious, lastly I checked the certificate and this was quite outdated 

I fired the metasploit module for heartbleed  
And BAM ! 

```
[*] 50.116.50.254:443 - Printable info leaked: @SE&kfT- zR]bjUsf"!98532ED/Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8Accept-Language: en-US,en;q=0.5Accept-Encoding: gzip, deflateCookie:...........' 
```

And got this as a response from server. 

I reported this to Slack and within few minutes got the reply 

> "Good catch - the status server sits outside of our main network and got skipped over. It's patched now with a new cert."

I was quite impressed by the response time !
