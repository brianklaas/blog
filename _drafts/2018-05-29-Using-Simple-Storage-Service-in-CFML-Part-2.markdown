---
layout: post
title:  "Using Simple Storage Service (S3) in CFML: Security, Control, and Caveats"
date:   2018-05-29 13:17:00 -0400
categories: AWS ColdFusion
---

This is the second of two posts about working with Simple Storage Service (S3) in CFML. In the last post, we looked at the basics of S3 and how simple it is to use S3 using native CMFL support.

### Security and Control with S3

S3, like most AWS services, provides a pretty amazing level of control through IAM. The [ColdFusion documentation](https://help.adobe.com/en_US/ColdFusion/9.0/Developing/WSd160b5fdf5100e8f-4439fdac128193edfd6-7f0a.html) explains how you can set access policies on files using native S3 support in CFML. CFML controls access to S3 folders and files within a S3 bucket via simple ACLs (access control lists). You can achive much finer-grained control via IAM. There are security and control features for accessing content in S3 that work outside of IAM, however.

You can make it so that only requests from your application can access the files on S3. You can generate URLs pointing to files on S3 that expire after a specific date/time. You can rename files on the fly, so that you can store them with random hashes as file names for better security. You can restrict requests to files to specific IP address ranges. You can even upload files directly to S3 from a Web browser.

All of these features require that you sign your requests with a HMAC-encoded version of the request and a set of IAM credentials, and then pass that signature in a HTTP call to S3.

This isn't a straightforward process, so I built a [S3 Request Signing utility component](https://github.com/brianklaas/ctlS3utils) and made it availble on GitHub. This utility component supports expiring URLs, changing the file name on the fly, specifying if the file should be accessed inline or as an attachment, and specifying the MIME-type of the file on a per-request basis.

### Caveats on Working with CFML's Native S3 Support

Native CFML implementations of S3 use an older method of passing your credentials (signing requests) to S3. Both Adobe ColdFusion and Lucee use the Version 2 of the [AWS request signature](https://docs.aws.amazon.com/general/latest/gr/signing_aws_api_requests.html). AWS currently uses (and encourages the use of) Version 4 of the request signature. This is handled automatically for you when you use the AWS Java SDK.

All regions created after January 30, 2014 require that you sign requests with the Version 4 signature. This means that you cannot use the native CFML S3 functionality in these regions (which include Paris, Seoul, China, and Ohio, among others). If you need to work in these AWS regions, you will need to use the AWS Java SDK to work with S3. (The AWS Java SDK also allows you to break huge, multi-gigabyte or terrabyte files into chunks and upload those chunks in parallel to S3 to speed file transfer. S3 knows how to reassemble those chunks into a single whole when all the parts have transferred.)

Next, Adobe ColdFusion does not support file operations for the cfpdf tag (and related functions) on S3. You'll need to pull those files off S3 to a local ColdFusion server, run the cfpdf commands locally, and then put the results back up on S3. Additionally, you cannot rename files from within CFML. You have to delete the current file and post a new copy with the new name. (This isn't a limitation of CFML. It's just how S3 works. There is no file rename command in S3.)

It's important to recognize that every time you perform a file operation with S3 from within your CFML application, you have to go across the wire to S3 to get the file. If you're running your CFML application in AWS on an [EC2 instance](https://aws.amazon.com/ec2/) (or an [ECS](https://aws.amazon.com/ecs/) or [EKS](https://aws.amazon.com/eks/) container), you don't have to go far. If your CFML application is running in a data center outside of AWS, you have to go across the Internet to get the file. This means that some operations &mdash; like looping through a 2GB .csv file &mdash; are going to incur major overhead. S3 is generally very fast, but you need to keep the overhead of running over the Internet to grab files in mind.

Finally, S3 is file storage, not a file system. You cannot perform traditional file system commands on objects in S3. You can get basic information about files in S3 by running the following:

{% highlight javascript %}
cfhttp( url="http://bucket.s3.amazonaws.com/path/to/file", method="head" );
{% endhighlight %}

### Go Do!

Hopefully you now see how easy it is to work with S3 from within your CFML applications. Even with the limitations of the service and the limitations of native CFML support for S3, it's still an incredibly valuable, robust, and inexpensive service. It's really nice to never have to worry about running out of storage space for user content or data. 

Amazon positions S3 as centralized storage for all of your data. With the recent introduction of services like [Athena](https://aws.amazon.com/athena/) and [S3 Select](https://aws.amazon.com/blogs/aws/s3-glacier-select/), S3 has moved beyond simple file storage to become a potential data lake for all of your application, monitoring, and log data. These are interesting new areas of exploration for me, and ones I'll hopefully write about in the future.