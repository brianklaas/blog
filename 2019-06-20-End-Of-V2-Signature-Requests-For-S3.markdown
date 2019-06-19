---
layout: post
title:  "End of the Road for Adobe ColdFusion's Built-In Support for S3 and New S3 Buckets"
date:   2019-06-20 15:51:00 -0400
categories: AWS ColdFusion
---

One of the things that got me started using AWS is [CFML's built-in support for S3](http://brianklaas.net/aws/coldfusion/2018/05/24/Using-Simple-Storage-Service-In-CFML-Part-1.html) &mdash; the Simple Storage Service. CFML's built-in support makes storing files in S3 instead of your local file system as easy as changing a "C:\" path to "S3://" It's drop-dead simple and is available on nearly all file functions in CFML.

One of the things that CMFL's built-in support for S3 masks is the process of digitally signing each and every request to the S3 service. All requests to all services within AWS have to be digitally signed. These signatures include a combination of the actual API request itself, and a hash of your credentials for accessing AWS. CFML transparently takes care of this request signing process for you.

The AWS developer community was abuzz this last week about yet another change to how AWS will support (or, rather, not support) version 2 signatures when making requests to S3. [This blog post details the changes in detail](https://aws.amazon.com/blogs/aws/amazon-s3-update-sigv2-deprecation-period-extended-modified/), but the basics are as follows:

- Version 2 of the AWS Signature was deprecated in 2018.
- AWS had planned on ending all support for Signature Version 2 in June, 2019.
- AWS instead will only end support for Signature Version 2 on *new* buckets created on or after June 24, 2019.

What does this have to do with CFML?

**All versions of Adobe ColdFusion prior to [ColdFusion 2018, Update 2](https://helpx.adobe.com/coldfusion/kb/bugs-fixed-coldfusion-2018-update-2.html) use Signature Version 2 for request signing.** This means that if you try to use ColdFusion's built-in support for S3 on buckets created on or after June 24, 2019, and you are using any version of Adobe ColdFusion prior to ColdFusion 2018, Update 2, those requests will fail.

If you are using ColdFusion 9, 10, 11, or 2016, and you're doing something like this:

{% highlight javascript %}
<cffile action="read" file="s3://testbucket/test.txt" variable="data"/>
{% endhighlight %}

That request will fail and your CFML engine will throw an error.

Again, the request will fail if the following conditions are met:

- Running a version of Adobe ColdFusion prior to ColdFusion 2018, Update 2
- Working with a S3 bucket created on or after June 24, 2019

Requests to buckets created *before* June 24, 2019 will continue to work. It is highly likely that AWS will set a final, hard deadline for ending support for Signature Version 2 requests on *all* S3 buckets within the next year. You should plan for that accordingly.

If you are still running ColdFusion 9, 10, 11, or 2016, and need to work with buckets created on or after June 24, 2019, you have a couple of options:

1. Upgrade to ColdFusion 2018, Update 2 or later.
2. Use the AWS Java SDK, which uses Signature Version 4 by default, for all of your S3 requests.
3. Use a CFML library that supports Signature Version 4 for your S3 requests. [JC Berquist's aws-cfml library](https://github.com/jcberquist/aws-cfml) is very good.

