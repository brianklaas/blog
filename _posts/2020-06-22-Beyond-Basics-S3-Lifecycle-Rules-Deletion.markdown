---
layout: post
title:  "Beyond the Basics of Using AWS S3 in CFML: Use Lifecycle Rules to Automatically Delete Files After a Period of Time"
date:   2020-06-22 13:08:00 -0400
categories: AWS ColdFusion
---

In [the last post in this series](https://brianklaas.net/aws/coldfusion/2020/06/16/Beyond-Basics-S3-Lifecycle-Rules.html), we looked at Lifecycle Rules in S3, how they worked, and how you can use them to save money by automatically moving less-frequently used files to different (and cheaper) storage classes. There's another powerful aspect to Lifecycle Rules that can save you money: automatically deleting files after a specified period of time.

Many of the files that are added to S3 buckets are never used after a brief, initial period of time. Many companies put all of their log files in S3. Are those log files valuable after data analysis is complete? Are the storage costs worth the price after 180 days? After 365? Are there regulatory or compliance reasons to not store user-generated data for a long period of time?

Whatever the reason, do you want to do the work of tracking every file in S3 and then running tasks to determine if the file is ready for deletion? Wouldn't it be nice if all of that could be handled transparently for you? With S3 <em>expiration actions</em>, you don't have to do this work. Expiration actions are a type of Lifecycle Rule that tells S3 to automatically delete a file in a bucket after a specified period of time.

## Enabling Expiration Actions in CFML

The process of enabling expiration actions is the same as enabling transition actions. The same steps for setting up transition actions apply when setting up expiration actions. As always, the full code for this and all of my demos can be found in my [AWSPlaybox GitHub repo](https://github.com/brianklaas/awsPlaybox).

If you haven't reviewed the last post to [see the steps required in setting up transition actions](https://brianklaas.net/aws/coldfusion/2020/06/16/Beyond-Basics-S3-Lifecycle-Rules.html), please do so as I will not repeat them here.

The CFML code for enabling an expiration action is as follows:

{% highlight javascript %}
bucketLifecycleConfig = CreateObject('java', 'com.amazonaws.services.s3.model.BucketLifecycleConfiguration').init();
rule = CreateObject('java', 'com.amazonaws.services.s3.model.BucketLifecycleConfiguration$Rule').init();
// The rule Id just helps you to identify the purpose of the rule in plain text
rule.setId("Delete objects after 90 days");
rule.setExpirationInDays(90);
// Rules must be set to ENABLED to actually function on the bucket
rule.setStatus(bucketLifecycleConfig.ENABLED);
bucketLifecycleConfig.setRules([rule]);
s3.setBucketLifecycleConfiguration(s3BucketName, bucketLifecycleConfig);
{% endhighlight %}

This code is a bit simpler than the code for transition actions, because we're not enabling any transitions. We're simply setting an expiration action that applies to every file in the bucket. In this case, any file in the bucket will be automatically deleted after 90 days.

You can combine transition actions and expiration actions in the same rule. You can create a rule that says "Move files from S3 standard to S3 One Zone Infrequent Access after 90 days, and delete all files after 180 days." That's exactly what the example code in the [AWS Playbox app](https://github.com/brianklaas/awsPlaybox) does.

[As explained in the post on transition actions](https://brianklaas.net/aws/coldfusion/2020/06/16/Beyond-Basics-S3-Lifecycle-Rules.html), you can only have one rule set that applies to _all_ files in a bucket. You can have multiple filters based on object path prefixes or tags in a single bucket, and any combination of transition and expiration actions in those filters. 

Lifecycle Rules are a powerful way to reduce your overall S3 costs. If you're storing more than a few hundred gigabytes of data in S3, I encourage you to start using them. As always, if you have any questions about this, feel free to [message me on the Twitter](https://twitter.com/brian_klaas).