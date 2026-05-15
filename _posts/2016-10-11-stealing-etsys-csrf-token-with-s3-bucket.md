---
layout: post
title: "Stealing Etsy's CSRF Token with S3 Bucket"
date: 2016-10-11 15:50:08
modified_date: 2016-10-11 16:13:03
author: Dhaval Chauhan
twitter: "@17haval"
description: "TLDR; S3 bucket was mis-configured while crossdomain.xml was lenient. So the chain of vulnerabilities threw CSRF Token out of the window."
---

**TLDR;** S3 bucket was mis-configured while crossdomain.xml was lenient

So the chain of vulnerabilities threw CSRF Token out of the window.

Here's how it went down:

## Discovery

Upon basic recon I find a suspicious S3 bucket

First thing first:
```bash
aws s3 ls s3://suspicious.etsy
```

But nothing except `images` folder. Who cares if I can access few images?

## Write Access

What more:
```bash
aws s3 cp file.html s3://suspicious.etsy
```

```
upload: ./file.html to s3://suspicious.etsy/file.html
```

So I am able to write files to this S3 bucket but who cares about XSS at S3.amazonaws.com?

## Subdomain Enumeration

Now what? More recon.

Next I find a suspicious sub-domain which points to this S3 bucket. Now my XSS is worth something.

But still session cookies are scoped to www.etsy.com

## Crossdomain.xml Analysis

Let's see crossdomain.xml of www.etsy.com:

```xml
<cross-domain-policy>
  <site-control permitted-cross-domain-policies="master-only"/>
  <allow-access-from domain="etsy.com"/>
  <allow-access-from domain="*.etsy.com"/>
</cross-domain-policy>
```

So I can send HTTP requests to www.etsy.com with ActionScript.

## CSRF Token Extraction

So I upload SWF file and fetch www.etsy.com. And this is what I get:

```html
<meta name="csrf_nonce" content="TOKEN">
```

That's all folks!
