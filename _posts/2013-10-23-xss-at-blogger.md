---
layout: post
title: XSS at Blogger
date: 2013-10-23 21:53:00
modified: 2015-12-13 09:37:34
author: Dhaval Chauhan
description: One fine night when I was editing my Blogger Account I found an awkward behavior. Upon digging a bit, I discovered an XSS vulnerability that could be exploited using data URIs [...]
tags: []
categories: []
permalink: /blog/xss_at_blogger/
---

One night when I was editing my Blogger Account I found an weird behavior. This caught my eye and upon digging a bit... BOOM!

## POC

The vulnerable link was:

```
https://www.blogger.com/switch-profile.g?switchProfileSource=3&continue=/home
```

Now, let's change this "continue"'s value to:

```
https://www.google.com/
```

And it redirects to:

```
https://www.google.com/
```

Let me try something else with data URI:

```
data:text/html;base64,......
```

AND... BOOM!

## Snapshots

![Blogger XSS Screenshot 1](/assets/images/xss-blogger-bx-1.jpg)

![Blogger XSS Screenshot 2](/assets/images/xss-blogger-bx-2.jpg)

## Results

I wasn't rewarded a bounty because of it being a browser issue and Same Origin Policy which prevents exploitation.
Thanks to Google Security Team for fixing the bug in matter of days and listing me in Hall Of Fame!