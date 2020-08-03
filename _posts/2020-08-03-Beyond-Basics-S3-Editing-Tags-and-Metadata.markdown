---
layout: post
title:  "Beyond the Basics of Using AWS S3 in CFML: Editing Tags and Using S3 Object Metadata"
date:   2020-08-03 9:55:00 -0400
categories: AWS ColdFusion
---

In [the last post in this series](https://brianklaas.net/aws/coldfusion/2020/07/27/Beyond-Basics-S3-Tags.html), we looked at how to add tags to an object uploaded to S3 via the AWS Java SDK. What if you need to change those tags? Are tags the _only_ way to add metadata to objects in S3? Let's file out.

## Updating Object Tags in S3

It's possible to update the tags assigned to an object in S3. This is particularly important if you already have files in S3 without tags. To do this, you'll tell the S3 client in your application to run a setObjectTagging() request:

{% highlight javascript %}
tag = CreateObject('java', 'com.amazonaws.services.s3.model.Tag').init(tagKey, tagValue);
fileTagging = CreateObject('java', 'com.amazonaws.services.s3.model.ObjectTagging').init([tag]);
// Remember: "key name" is the path to the file (incuding the file name) in the bucket
objectTaggingRequest = CreateObject('java', 'com.amazonaws.services.s3.model.SetObjectTaggingRequest')
    .init(bucketName, keyName, fileTagging);
s3.setObjectTagging(objectTaggingRequest);
{% endhighlight %}

It's important to note that when you run a setObjectTagging() request, it _replaces_ the existing tags on the object. There's no way to just "update" the tags on an object.

## Tags vs. Metadata

When you put a file in S3, [S3 automatically adds quite a bit of metadata](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html), including:

- Content-Length
- Content-Type
- Last-Modified
- x-amz-server-side-encryption
- x-amz-storage-class

and more. If some of these look like regular, plain http response headers, that's because they are.

If you want to add your own custom metadata to an object in S3, you can add metadata instead of tags. Tags are not the same thing as object metadata in S3. Metadata applies only to that object in S3 and cannot be searched on, as you can search with tags. 

To set an object's metadata, you would set the objectMetadata object on your putFileRequest, as follows:

{% highlight javascript %}
objectMetadata = CreateObject('java', 'com.amazonaws.services.s3.model.ObjectMetadata').init();
objectMetadata.setUserMetadata(plainOldCFStructOfMetaData);
putFileRequest.setObjectMetadata(objectMetadata);
{% endhighlight %}

You can read the metadata for an object in S3 when you run a getObject() or [GetObjectMetadataRequest()](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/s3/model/GetObjectMetadataRequest.html) request. The metadata is also sent in the response headers of every plain http request to an object in S3 (ie; https://somebucket.s3.amazonaws.com/somePhoto.jpg).

It's important to note that you cannot edit or change the metadata for an object once that object is created. If you need to do this, you have copy the object into the same place (effectively overwriting the file) with a [CopyObjectRequest](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/s3/model/CopyObjectRequest.html) object with the new metadata. It's not convenient, but if you have metadata that you want to come down with every plain http request to an object in S3, metadata is the only way to go. Tags are not included in plain http requests to objects in S3. 

That's it for object tags and metadata in S3. As always, if you have any questions about this, feel free to [message me on the Twitter](https://twitter.com/brian_klaas).