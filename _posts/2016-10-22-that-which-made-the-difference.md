---
layout: post
title: "That (.) Which Made The Difference"
date: 2016-10-22
published_time: 2016-10-23T03:13:32.227Z
modified_time: 2016-10-23T03:18:18.632Z
author: Dhaval Chauhan
tags: [security, domain, dns, subdomain-takeover]
---

**TL;DR** Help Scout **did** distinguish between regular domain name and trailing dot domains which lead to trivial low risk sub domain takeover

After reading [The story of EV-SSL, AWS and trailing dot domains](https://labs.detectify.com/2016/10/05/the-story-of-ev-ssl-aws-and-trailing-dot-domains/) it felt the issue shouldn't be just limited to Amazon because there are lot of other services which allow you to host the content in custom domains using CNAME. One of which pretty frequently comes up in `dig` is [HelpScout](https://www.helpscout.net/).

This time it was [Trello](https://trello.com/), Trello used Help Scout for the [Trello Help](http://help.trello.com/).  

```
; <<>> DiG 9.9.5-3ubuntu0.8-Ubuntu <<>> help.trello.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40237
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;help.trello.com.        IN  A

;; ANSWER SECTION:
help.trello.com.    299 IN  CNAME   trellohelp.helpscoutdocs.com.  
trellohelp.helpscoutdocs.com. 59 IN    CNAME   pdocsweb-513455410.us-east-1.elb.amazonaws.com.  
pdocsweb-513455410.us-east-1.elb.amazonaws.com.    59 IN A 52.204.90.23  
pdocsweb-513455410.us-east-1.elb.amazonaws.com.    59 IN A 54.164.242.232  
```

So CNAME pointing to `trellohelp.helpscoutdocs.com`

Randomly I inserted . at the end of the domain and instead of obvious help page I get and Help Scout error saying the page wasn't found.

So trailing dot does make a difference to HelpScout

## Next Steps

1. Register in HS  
2. Try to create a page with `Custom Domain` set to `help.trello.com.`

## So ?

Yes, It worked. 
My page was loading at [http://help.trello.com./](http://help.trello.com./)

So it looks everyone using HS via CNAME can be at risk.

## Further Steps

1. Mail HelpScout regarding the issue  
2. Report to Trello, Harvest via H1 incase I never hear from HS.

## Issue:

1. Trailing dot should not have made a difference.  
2. Allow the registration of already registered domain

## End Result:

1. HS accepted the issue and then never turned up even after fixing the issue  
2. Trello got in touch with HS from there side as well, Bug Fixed. Tada.  
3. Myself closed the Harvest Bug report because made no sense once the HS problem was fixed  
4. Profit ? Yep

That's about it I guess
