---
layout: post
title:  "Beyond the Basics of Using AWS S3 in CFML: Use Different Storage Classes to Save Money"
date:   2020-06-02 12:48:00 -0400
categories: AWS ColdFusion
---

S3 offers a rediculous level of durability for the objects stored in the service. On top of [11 9's of durability](https://wasabi.com/blog/11-nines-durability/) in physically distributed storage, Amazon has developed formal proof-of-correctness algorithms that map across all the systems with which S3 interacts to ensure that data is always correct when requested. They create checksums across loosely-coupled systems (including checking for bitslips in RAM) to ensure transfers never become corrupt, and use complex actuarial models that anticipate when the hard drives on which data is actually stored will fail. AWS has automated “durability auditors” that repeatedly crawl every byte of S3 data to verify that when you retrieve your stuff, it will be correct. This is incredibly powerful, _difficult_ stuff to achieve. There's a [fascinating video from re:Invent 2018](youtube.com/watch?v=nLyppihvhpQ) if you want to learn more about the way in which S3 is built.

The upshot of all this work is that you simply won't lose files stored in S3. The default way in which files are stored in S3 causes them to replicated across at least three availability zones in a single geographic AWS region. Even if one data center physically blows up, you won't lose your data. As this is S3's storage default, this is the way in which all files using CFML's built-in file functions will be stored in S3. This is also the most expensive way to store files in S3.

## Storage Classes and Saving Money

S3 isn't expensive, but there are ways to save money on storage when you put files in S3. There are a number of different storage classes, and putting files into lower-cost storage classes is a great way to save a lot of money when your total S3 storage starts moving into the terrabyte range. 

[Storage classes and pricing can be found on the AWS site](https://aws.amazon.com/s3/pricing/). Here's a brief rundown of the different storage classes, with prices current as of June, 2020:

- Standard (starting at $0.023 per GB): sub-10ms response time, replicated across three availability zones in a region
- Infrequent Access ($0.0125 per GB): sub-second response time, replicated across three availability zones in a region
- Intelligent Tiering (starting at $0.023 per GB, also $0.0025 per 1,000 objects for monitoring): automatically moves objects greater than 128KB in size from standard to infrequent access when they aren't heavily requested in the past 30 days
- S3 One Zone - Infrequent Access ($0.01 per GB): sub-second response time, objects exist in only one availability zone
- S3 Glacier ($0.004 per GB) - 1-12 hour retrieval time, for long-term archiving only
- S3 Glacier Deep Archive ($0.00099 per GB) - 1-12 hour retrieval time, a file can only be requested twice per year

As you can see, you can save almost 50% per month on storage if you put files in the Infrequent Access storage class. You can't do this with the built-in CFML functions, though. You have to use the AWS Java SDK. This may not be the best option for highly accessed files, but how often do customer files get accessed in your web apps if you're not Facebook or TikTok?

It's important to note that there is no savings for storing files smaller than 128KB in Infrequent Access or S3 One Zone - Infrequent Access. At scale, GET requests to Infrequent Access storage are more expensive than GET requests for standard storage. You also cannot make a synchronous request to get a file in Glacier. You'll get a response token back when you want to pull a file from Glacier. At some point in the next 12 hours, S3 will grab the file from Glacier and put it in temporary storage for return to you. These are all important considerations when you look at cost differences across storage classes.

## Storing Files with Different Storage Classes in CFML

So how do you store files with a non-standard storage class when you put them in S3? As always, the full code for this and all of my demos can be found in my [AWSPlaybox GitHub repo](https://github.com/brianklaas/awsPlaybox). Let's take a look:

{% highlight javascript %}
    s3 = application.awsServiceFactory.createServiceObject('s3');
    fileContent = fileReadBinary(getTempDirectory() & uploadedFile.serverFile);
    javaFileObject = CreateObject('java', 'java.io.File').init(fileLocation);
    putFileRequest = CreateObject('java', 'com.amazonaws.services.s3.model.PutObjectRequest')
        .init(s3BucketName, fileName, javaFileObject);
    // The storage class property of the PutObjectRequest tells S3 to use a non-standard storage class
    storageClassObj = CreateObject('java', 'com.amazonaws.services.s3.model.StorageClass');
    putFileRequest.setStorageClass(storageClassObj.valueOf("StandardInfrequentAccess"));
    s3.putObject(putFileRequest);
{% endhighlight %}

As mentioned previously, when you want to do something other than a standard object upload to S3 using the AWS Java SDK, you'll add property objects to the putFileRequest which represent those additional features. When we want to use a storage class other than standard, we create a StorageClass object with the appropriate storage class string and add that to our PutObjectRequest. It's very simple.

## A Warning About Data Egress

Storing files in S3 is cheap. A lot of companies use S3 as the file storage for all their publicly accessible files because it's also incredibly fast. It seems just about perfect &mdash; until the monthly bill arrives and you see that you were charged a ton of money for the data flowing _out_ of your S3 buckets to the Internet.

[NASA found this out the hard way](https://www.theregister.co.uk/2020/03/19/nasa_cloud_data_migration_mess/). They currently store 32 petabytes of data in S3. This data is shared with and used by scientists around the world, and is expected to grow to 247 petabytes by 2025. Their "data out" bills for S3 at that point will be about $30 million each year. Ouch.

Data out to the Internet from S3 currently starts at $.09/GB. That's $90/TB. If you have a popular site, or serve up a lot of data from S3, you're going to be paying a lot, and quickly. You're likely better off putting all file requests behind a CloudFront distribution, which gives you a global CDN with 216 (currently) points of presence. Data transfer costs are generally lower than the costs of egress from S3 &mdash; certainly at scale &mdash; and you get a whole host of additional security and data transfer features built-in to the base CloudFront charges. I wrote a utility for [creating signed requests to CloudFront distributions from CFML](http://brianklaas.net/aws/coldfusion/2019/09/27/Updated-Version-S3-and-CloudFront-Signing-Utilities.html) to make it easier to work with CloudFront from CFML.

It's easy to save money with S3 by not using the standard storage class if you don't have to. This is probably the most compelling reason to stop using built-in CFML functions (prior to Adobe ColdFusion 2020) and instead use the AWS Java SDK to manage your file transfers to S3. As always, if you have any questions about this, feel free to [message me on the Twitter](https://twitter.com/brian_klaas).