---
layout: post
title:  "Utility for Creating Signed URLs for Amazon CloudFront Up on GitHub"
date:   2013-03-25 08:17:00 -0500
categories: AWS ColdFusion
---

I've taken the code I wrote for my previous two posts on creating signed URLs for protected content in Amazon CloudFront in ColdFusion and dynamically changing the file names of the files you serve from CloudFront and put it all in a little CFC which you may find useful if you're doing either of these two tasks.

In order for you to be able to do anything with this CFC, you're going to need an AWS account that uses CloudFront and have everything set up so you can serve protected content. You're also going to need a copy of the AWS SDK .jar file in your ColdFusion /lib directory. The two blog posts go into extensive detail about what needs to be done and the little gotchas you need to watch out for along the way.

**[CTL CloudFront Utility CFC on GitHub](https://github.com/brianklaas/ctlCloudFrontUtils)**
