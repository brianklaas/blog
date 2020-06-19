---
layout: post
title:  "Beyond the Basics of Using AWS S3 in CFML: Use Lifecycle Rules to Move Files Into Different Storage Classes"
date:   2020-06-16 13:48:00 -0400
categories: AWS ColdFusion
---

In my last post in this series, we looked at [using different storage classes in S3 via CFML to save money](https://brianklaas.net/aws/coldfusion/2020/06/02/Beyond-Basics-S3-Storage-Classes.html) on long-term storage costs. The use case for that blog post assumed you knew the utilization pattern for the files you uploaded. They would either be very busy files, files that were accessed occasionally, or files that were accessed rarely. What happens, though, when file acess moves over time from frequently access to only occasional access? Wouldn't it be nice if S3 would automatically move those files from a more expensive storage class with the fastest response time to a storage class where response times were a few miliseconds longer, but cost half the price?

S3 provides [lifecycle rules](https://docs.aws.amazon.com/AmazonS3/latest/dev/object-lifecycle-mgmt.html) to do exactly that.

## How Lifecycle Rules Work

You can only apply lifecycle rules to a whole bucket, but you can define filters in these rules so that only specific files in the bucket are affected by the lifecycle rule. There are two types of lifecycle rules that can be applied to a S3 bucket. There are <em>transition actions</em>, which we'll look at now, and <em>expiration actions</em>, which we will look at in a future post.

Transition actions tell S3 to <em>transition</em> files from one storage class to another when certain criteria are met. For example, you can tell S3 to automatically move files to infrequent access, one zone storage 30 days after creation. You can tell S3 to automatically move files to Glacier deep archive after 365 days. This not only can save you money over time (do you really need the drafts of the 2014 annual report to stay in S3 standard storage?), but it also automates determining which files should be moved and when they should be moved.

## Intelligent Tiering vs. Transition Actions

S3 does offer a storage class called "[intelligent tiering](https://aws.amazon.com/s3/storage-classes/#Unknown_or_changing_access)." This tier of storage looks at access patterns for every file in this storage class, and automatically moves files from S3 standard to S3 infrequent access if the file has not been accessed in the last 30 days. The advantage to intelligent tiering is that if a file remains frequently accessed after 30 days, it will stay in the S3 standard class. Transition actions ignore _individual_ file access patterns. Transition actions are apply to all files in the bucket, regardless of individual file access patterns. If your lifecycle rule says to transition all files in the bucket from S3 standard to S3 infrequent access after 30 days, that's going to happen at the 30 day mark even if the file is still frequently accessed.

Intelligent tiering does have an overhead charge of $0.0025 per 1,000 objects monitored. This can add up over time, and may not be the best choice if your general access patterns for files in a bucket are well known. Intelligent tiering only manages files over 128KB in size, so this is not a good approach for small log files. (It's also important to note that S3 standard infrequent access and S3 one zone infrequent access storage both have a minimum billable object size of 128KB.)

## Enabling Lifecycle Rules in CFML

If you decide that a lifecycle rule is the best way to automate changing storage classes for your files in S3, you can manage those rules (and change or delete them) from within your CFML applications. As always, the full code for this and all of my demos can be found in my [AWSPlaybox GitHub repo](https://github.com/brianklaas/awsPlaybox).

There are five steps to enabling lifecycle rules via the AWS Java SDK:

1. Create a BucketLifecycleConfiguration object to apply to the whole bucket
2. Create a BucketLifecycleConfiguration$Rule object that defines a transition rule
3. Create a BucketLifecycleConfiguration$Transition object that specifies the storage class to change to after a specified period
4. Enable the rule
5. Apply the BucketLifecycleConfiguration to the bucket

Let's look at this in code:

{% highlight javascript %}
bucketLifecycleConfig = CreateObject('java', 'com.amazonaws.services.s3.model.BucketLifecycleConfiguration').init();
rule = CreateObject('java', 'com.amazonaws.services.s3.model.BucketLifecycleConfiguration$Rule').init();
// The rule Id just helps you to identify the purpose of the rule in plain text
rule.setId("Move to One Zone Infrequent Access after 30 days");
storageClassObj = CreateObject('java', 'com.amazonaws.services.s3.model.StorageClass');
transition = CreateObject('java', 'com.amazonaws.services.s3.model.BucketLifecycleConfiguration$Transition')
    .withDays(30)
    .withStorageClass(storageClassObj.valueOf('OneZoneInfrequentAccess'));
// setTransitions expects a Java List, which is just a CFML array
rule.setTransitions([transition]);
// Rules must be set to ENABLED to actually function on the bucket
rule.setStatus(bucketLifecycleConfig.ENABLED);
bucketLifecycleConfig.setRules([rule]);
s3.setBucketLifecycleConfiguration(s3BucketName, bucketLifecycleConfig);
{% endhighlight %}

The code follows the list of steps above, but there are a few points that bear explanation:

- If you're wondering what's up with the $ in some of the Java class names, that's how you instantiate nested Java classes in CFML. The Rule and Transition objects are nested classes of the BucketLifecycleConfiguration object in the AWS Java SDK, so you reference them with the $ in the class name. A big thanks to [Ben Nadel for documeting that tip](https://www.bennadel.com/blog/1370-ask-ben-instantiating-nested-java-classes-in-coldfusion.htm)!
- Rules are not automatically enabled when created or applied to a Transition object. They must be explicity ENABLEd to take affect. (This means you can also disable them as needed without deleting them off a bucket lifecycle configuration.)

## Rules and Filters

While it's easy to create a transition action for a whole bucket, you can only have _one_ "all files" transition action per bucket. That is: you can only have one rule that applies to <em>every file</em> in the bucket. You can have many transition actions per bucket, but they must be defined with individual rules that apply <em>filters</em> to the files affected by the transition actions. You can filter by an object key prefix (the "path" to the object in the bucket), known as a [Prefix Predicate](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/s3/model/lifecycle/LifecyclePrefixPredicate.html), or a [tag](https://docs.aws.amazon.com/AmazonS3/latest/dev/object-tagging.html) that's applied to a file, known as a [Tag Predicate](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/s3/model/lifecycle/LifecycleTagPredicate.html). I'll cover applying tags to files when you upload them to S3 in a future post.

Using both prefix predicates and tag predicates, you can have several rules for any given bucket and be specific about when different kinds of files in different parts of a bucket are automatically transitioned from one storage class to another. This is useful when you have long-established buckets with a lot of data and don't want to incur the cost (and overhead) of moving the files into separate buckets to enforce different transition actions.

In the next post in this series, we'll look at expiration actions, which are perfect for handling files with short lifespans, like log files. As always, if you have any questions about this, feel free to [message me on the Twitter](https://twitter.com/brian_klaas).