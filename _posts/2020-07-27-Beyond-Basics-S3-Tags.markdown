---
layout: post
title:  "Beyond the Basics of Using AWS S3 in CFML: The Why and How of Adding Tags to Your Files"
date:   2020-07-27 8:51:00 -0400
categories: AWS ColdFusion
---

As anyone who has worked in any of the major cloud service providers will tell you, there will be ever-increasing sprawl the more you use a cloud provider: more files, more servers, more databases, more accounts, more services, more everything. This is especially the case with object storage like S3. You can easily end up with tens or hundreds of thousands of files in a month. If you were to look at any one of the files in a bucket, could you say who put that file there? Could you look at all the files and be able to charge each file owner for the storage cost? Not without the power of tags.

## Tags in AWS

Tags are metadata-like key-value pairs you can apply to nearly anything in AWS. You determine the key and you determine the value and then apply the tags to a resource in AWS. You can then use those tags when you search for resources (including files in S3) inside AWS. You can even enable a special kind of tag &mdash; a [cost-allocation tag](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) &mdash; that will be used to break down individual "ownership" costs of each service on your monthly AWS bill. Most people use tags to determine things like the:

- Department which owns the resource ("department" : "marketing")
- Environment in which the resource runs ("environment" : "dev")
- Application to which the resource belongs ("application" : "order fulfillment")
- Security level of the resource ("compliance" : "hippa")

Tags can represent anything, so it's important that you:

a. start tagging your resources as soon as possible<br>
b. have a strategy for coming up with tags

AWS sees how hundreds of thousands of customers tag their resources and have come up with [a really useful guide to tagging strategies](https://aws.amazon.com/answers/account-management/aws-tagging-strategies/). It's worth a read when you're coming up with your own plan for tagging.

## Adding Tags to Files in S3

Adding tags to files in S3 is very much like adding tags to any resource in AWS. You create tags, and then add them to a request to create or modify a resource in AWS. In the case of S3, you create your tags, add them to an ObjectTagging object, and then add that ObjectTagging object to the putObject request that actually uploads your file to S3.

{% highlight javascript %}
s3 = application.awsServiceFactory.createServiceObject('s3');
javaFileObject = CreateObject('java', 'java.io.File').init(fileLocation);
putFileRequest = CreateObject('java', 'com.amazonaws.services.s3.model.PutObjectRequest')
     .init(bucketName, fileName, javaFileObject);

// Here's where we add our tag
tag = CreateObject('java', 'com.amazonaws.services.s3.model.Tag').init(tagKey, tagValue);
fileTagging = CreateObject('java', 'com.amazonaws.services.s3.model.ObjectTagging').init([tag]);
putFileRequest.setTagging(fileTagging);

s3.putObject(putFileRequest);
{% endhighlight %}

The code above should be fairly self-explanatory, but here's a quick rundown. First, you create a putFileRequest using a standard Java File object. ([Read more about uploading files to S3 via the AWS Java SDK](https://brianklaas.net/aws/coldfusion/2020/05/08/Beyond-Basics-S3-Upload-File-via-AWS-Java-SDK.html).) Next, you create a Tag object, assigning it the key and value with which you want to tag the file. Next, the tag(s) are added to an ObjectTagging object, which is a standard object throughout the AWS Java SDK. Finally, you set the tagging on the putFileRequest object using the ObjectTagging object. When the file is actually uploaded to S3, the tags are added to the file.

It's really straightforward to add tags to the objects you put in S3, so start doing it! In the next post in this series, we'll look at editing object tags, and the differences between tags and object metadata in S3.