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

One fine night when I was editing my Blogger Account I found an awkward behavior. This caught my eye and upon digging a bit... BOOM!

## POC

The vulnerable link was:

```
https://www.blogger.com/switch-profile.g?switchProfileSource=3&continue=/home
```

![Vulnerable Link](/assets/images/xss-blogger-vulnerable-link.png)

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

![XSS Executed](/assets/images/xss-blogger-executed.png)

## Snapshots

![Blogger XSS Screenshot 1](/assets/images/xss-blogger-bx-1.jpg)

![Blogger XSS Screenshot 2](/assets/images/xss-blogger-bx-2.jpg)

## Outcome

Anyways, I wasn't rewarded a penny because of Browser Issue (nothing new with Google) and Same Origin Policy and some typical requirements and the list goes on and on.

Thanks to Google Security Team for fixing the bug in matter of days and listing me in Hall Of Fame!
