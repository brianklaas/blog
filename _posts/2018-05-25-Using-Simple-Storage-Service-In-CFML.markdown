---
layout: post
title:  "Using Simple Storage Service (S3) in CFML"
date:   2018-05-21 13:17:00 -0400
categories: AWS ColdFusion
---

In the last entry in the series, we looked at how you connect to AWS via the [AWS Java SDK](https://aws.amazon.com/sdk-for-java/) from your CFML code. That's the route you need to go if you want to work with AWS services from CFML &mdash; with one notable exception: [S3, or Simple Storage Service](https://aws.amazon.com/s3/).

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

### Security and Control with S3

S3, like most AWS services, provides a pretty amazing level of control through IAM. The ColdFusion documentation linked above explains how you can set access policies on files using native S3 support in CFML. CFML controls access to S3 folders and files within a S3 bucket via simple ACLs (access control lists). You can achive much finer-grained control via IAM. There are security and control features for accessing content in S3 that work outside of IAM, however.

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