---
layout: post
title:  "Beyond the Basics of Using AWS S3 in CFML: File Versioning in S3 Buckets"
date:   2020-07-13 10:01:00 -0400
categories: AWS ColdFusion
---

When editing files of any kind, users of modern applications expect to have access to previous versions of the file, and undo work that they don't like or made in error. Implementing a versioning system isn't easy, especially when file systems are generally set up to only allow one "copy" of any given file. It's often up to you, the developer, to handle the versioning and ensure that new versions of uploaded files don't overwrite older ones.

Fortunately, it's exactly this kind of undifferentiated heavy lifting at which AWS excels. Why should you worry about file versioning at the file system level, when you can utilize this as a cheap service? [Versioning is built in to S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html), and comes at no additional cost &mdash; except for all the copies of the files that you store. 

## Enabling File Versioning in CFML

Versioning is enabled on per-bucket basis. There's no way to say "just version this file," or even provide a path prefix that says "only affect these files in this path," as you can do with [lifecycle rules](https://brianklaas.net/aws/coldfusion/2020/06/16/Beyond-Basics-S3-Lifecycle-Rules.html). 

As always, the full code for this and all of my demos can be found in my [AWSPlaybox GitHub repo](https://github.com/brianklaas/awsPlaybox).

Here's how you enable versioning on a bucket in CFML:

{% highlight javascript %}
s3 = application.awsServiceFactory.createServiceObject('s3');
bucketVersioningConfig = CreateObject('java', 'com.amazonaws.services.s3.model.BucketVersioningConfiguration')
    .withStatus('Enabled');
bucketVersioningConfigRequest = CreateObject('java', 'com.amazonaws.services.s3.model.SetBucketVersioningConfigurationRequest')
    .init(bucketName, bucketVersioningConfig);
s3.setBucketVersioningConfiguration(bucketVersioningConfigRequest);
{% endhighlight %}

It's quite simple: you create a bucket versioning config object, set the status to "enabled" and tell the S3 bucket to utilize that new configuration.

Once you have versioning enabled, S3 will store a new version every time you POST a new version of the same file (the same file path/"prefix") in the same bucket. The response object returned from a PutFile() request to S3 [using the AWS Java SDK](https://brianklaas.net/aws/coldfusion/2020/05/08/Beyond-Basics-S3-Upload-File-via-AWS-Java-SDK.html) will give you the version ID of that version of the file which you just uploaded. You'll need to store that ID in your own application along with some kind of metadata about what that version of the file represents. S3 will not tell you the differences in the content between different versions of a file. It's not Git. That's not what it is designed to do. It simply shows all the versions of the file in the bucket. It's up to you to keep track of what each version represents, or a summary of changes between versions.

## Listing Versions of a File in CFML

It's pretty straightforward to retrieve the versions of a file:

{% highlight javascript %}
listVersionsRequest = CreateObject('java', 'com.amazonaws.services.s3.model.ListVersionsRequest')
    .withBucketName(bucketName)
    .withPrefix(pathToFileForVersioning)
    .withMaxResults(5);
versionsResult = s3.listVersions(listVersionsRequest);
summariesArray = versionsResult.getVersionSummaries();
summariesArray.each(function(objectSummary) {
    resultData &= "Key: " & objectSummary.getKey() & ", versionID: " & objectSummary.getVersionId();
});
{% endhighlight %}

The ListVersionsRequest object is what does the work. It has a number of methods for specifying the data you want returned, but the primary required property (beyond the bucket name) is set via the withPrefix() method. It's here that you specify the path to the file in the bucket. You can think of a path to a file in a versioned bucket like a folder that contains all the versions of a file therein. The ListVersionsRequest simply shows you all the files in that "folder." Using the withMaxResults() method limits you to the [n] most recent versions of that file.

Once you tell S3 to list the verisons for that prefix, it returns, within the request result object, an array of summaries about each version of the file. Again, it's not going to tell you what's different in the content between each version. You've got to track that on your own. You can loop through the summaries array to get the IDs of each version of the file (the "key") that you will match to data in your own application about what each version represents. 

## Warning: Cost Increases Ahead!

It's important to remember that while you get all of the versioning functionality in S3 for free, you pay for every version of every file stored. If a file is deleted out of a bucket that has versioning enabled, the file isn't actually deleted. A deletion marker is placed on the file so that it can be un-deleted at a later time. If versioning is disabled on a bucket, all the old versions of each file are retained for safety's sake.

All of these versions of files start to add up over time. It's important to delete older versions of files. There is no way to limit the number of versions kept, or to set a maximum number of versions per file and automatically delete files older than that maximum. 

You're best off building tasks to check for old versions manually, or to [use a lifecycle](https://brianklaas.net/aws/coldfusion/2020/06/02/Beyond-Basics-S3-Storage-Classes.html) rule to have old versions automatically expire after a period of time. The BucketLifecycleConfiguration.Rule. setNoncurrentVersionExpirationInDays(int) method will help you do this transparently.

Versioning is a powerful tool and an easy addition to any S3 bucket. Just watch out for those unintented cost increases. As always, if you have any questions about this, feel free to [message me on the Twitter](https://twitter.com/brian_klaas).