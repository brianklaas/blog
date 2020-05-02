---
layout: post
title:  "New Series: Going Beyond the Basics of Using AWS S3 in CFML"
date:   2020-05-04 09:47:00 -0400
categories: AWS ColdFusion
---

Amazon Web Services [Simple Storage Service (S3)](https://aws.amazon.com/s3/) is an amazing service. Super fast, super reliable, and cheap, it's often referred to as the &quot;file storage of the Web.&quot; On an average day and in a single AWS [region](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html), S3 will handle 60 _terrabytes_ of data transfer per second. It's the backbone &mdash; and data lake &mdash; of many companies. CFML engines have long made it simple to utilze S3 for file storage. I've written about [reading files from and writing files to S3 from CFML](https://brianklaas.net/aws/coldfusion/2018/05/24/Using-Simple-Storage-Service-In-CFML-Part-1.html) before. 

There is so much more to S3 than just file storage, however. This new series will cover how to utilize some of that additional functionality from within our CFML runtimes. In this series, I'll cover:

- Uploading a file to S3 via the AWS Java SDK (not CFML tags or functions)
- IAM permissions when using the AWS Java SDK vs. built-in CFML tags and functions
- Making S3 buckets private and using signed URLs to access otherwise inaccessible content
- Encrypting objects at rest in S3
- Using different storage classes in S3
- Enabling object lifecycle flows in S3
- Enabling object versioning on a bucket
- Adding AWS Tags to individual objects

To do any of this, we'll need to use the AWS Java SDK as this functionality isn't currently available in our CFML runtimes. If you don't already have the AWS Java SDK added to your CFML runtime, please see my post on [what AWS .jar files to add and where to add them in your CFML runtime](https://brianklaas.net/aws/coldfusion/2018/12/10/Update-On-Using-AWS-Java-SDK-With-ColdFusion-2018.html).

The code for all of these posts can be found in my [AWSPlaybox GitHub repo](https://github.com/brianklaas/awsPlaybox).

Finally, it's important to note that with the upcoming release of Adobe ColdFusion 2020, you will be able to do all of the above items natively within Adobe ColdFusion 2020. If you're running Adobe ColdFusion 2018, 2016, or Lucee, you'll need to use the AWS Java SDK to do the things I'll talk about in this series. 
