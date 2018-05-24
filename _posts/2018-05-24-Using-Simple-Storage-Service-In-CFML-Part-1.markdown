---
layout: post
title:  "Using Simple Storage Service (S3) in CFML: Background and Basic Usage"
date:   2018-05-24 14:11:00 -0400
categories: AWS ColdFusion
---

In the last entry in the series, we looked at how you connect to AWS via the [AWS Java SDK](https://aws.amazon.com/sdk-for-java/) from your CFML code. That's the route you need to go if you want to work with AWS services from CFML &mdash; with one notable exception: [S3, or Simple Storage Service](https://aws.amazon.com/s3/).

Note: I had originally wrote one long post about each AWS service, but after getting feedback from the good people on Twitter, I have broken these posts apart into shorter pieces.

### S3 Background

I'll be completely upfront with you: I'm a huge fan of S3. It's fast, cheap, highly reliable storage for all of your data. If your users upload files to your app, store them in S3. If you've got PDFs or MP4s or MP3s, store them in S3. Put your HTML, CSS, and JavaScript there. Put your logs there. Put your .csv database backups there. Put everything there. 

S3 is [cheap](https://aws.amazon.com/s3/pricing/). S3 is fast. S3 is highly redundant and available. S3 offers a whole host of file lifecycle policies which you can set and forget.

If I could go back in time, I would put all of our non-executable data in S3. All of it.

My team started using S3 because of CFML's native support for the service. Adobe ColdFusion 9 and Railo 3 (now superceded by Lucee) were the first versions of CFML engines to support S3 as a native endpoint for file functionality in your CFML applications.

Here are the offical docs:

- S3 support in [Adobe ColdFusion](https://help.adobe.com/en_US/ColdFusion/9.0/Developing/WSd160b5fdf5100e8f-4439fdac128193edfd6-7f0a.html)
- S3 support in [Lucee](http://docs.lucee.org/guides/Various/file-system.html)

### Working with S3 from CFML

Essentially, all of the file-related CFML tags and functions support using S3 as an endpoint. So, instead of a file path starting with a local or network file share (ie; C:/ or /networkFilePath/), you point to S3 instead:

{% highlight javascript %}
<cffile action="read" file="s3://testbucket/test.txt" variable="data"/>
{% endhighlight %}

It's drop-dead simple.

You can use nearly all the file-related tags and functions with S3 by referencing s3://yourBucket/ in the file path.

As with all things AWS, you need to provide credentials to AWS to access S3. You have two options to do this.

1. In application.cfc, provide your IAM user account accessKey and secretKey as this.s3.accessKey and this.s3.secretKey.
2. Provide the credentials inline on each call to S3.

For the greatest flexibility and control, the second option is generally preferred. If you put the accessKey and secretKey in application.cfc, all calls to S3 in that application will use the same IAM user. If you pass the credentials inline, you have great control over what call has what permissions. This allows for much better security, particularly as your application grows more complex over time.

To pass the credentials inline, you specify them as follows:

{% highlight javascript %}
s3://accessKey:secretKey@[ absolute-path to the file on S3 ]
{% endhighlight %}

For example:

{% highlight javascript %}
<cffile action=“read” file=“s3://accessKey:secretKey@somebucket/somefile.txt” variable=“fileData” />
{% endhighlight %}

You may have noticed that I used the word "bucket" in the last two cffile examples. Buckets are root locations for files in S3. Buckets are like FTP sites, and you put files into buckets. CFML cannot create buckets for you. You need to do that in the [AWS Console](https://console.aws.amazon.com/console/home), via the [AWS command line tools](https://aws.amazon.com/cli/), or via the AWS Java SDK.

As I've mentioned before, AWS operates on the principle of least privilege. This means that when you set up a bucket, no one has access to it, not even you! You need to specify in your IAM user permissions that you should have access to that bucket (or to all buckets, if you like to live dangerously). You can set a bucket to be world-readable, which isn't a bad thing. You can also set a bucket to be world-writable, which *is* a bad thing. You'd be shocked how quickly you wind up hosting terrabytes of other people's data when you set your S3 buckets to world-writable.

To see the details of native CFML support for S3, take a look at the Adobe ColdFusion documentation linked above. Lucee supports the same set of functionality.

Next up: we'll look at security and control when working with S3 from CFML, and some caveats about CFML's native support for the service.