---
layout: post
title:  "Beyond the Basics of Using AWS S3 in CFML: Encrypting Objects at Rest in S3"
date:   2020-05-27 14:34:00 -0400
categories: AWS ColdFusion
---

AWS Identity Access Management ([IAM](https://aws.amazon.com/iam/)) provides very strong security to all of your data in AWS, and S3 in particular, if you use it properly. Proper IAM policies make it so that only the right people have access to the right resources in AWS. (If you want to learn more about using IAM in CFML, watch [my presentation from Adobe ColdFusion Summit 2019](https://www.youtube.com/watch?v=ucn90XLDlPw).) What if someone steals one of your AWS logins? How can you protect your data in S3 so that bad actors can't see what you've stored in your S3 buckets in case they steal your credentials? What if you have compliance obligations which require you to encrypt every piece of data you store?

Encrypting objects (and data) at rest in AWS is one way to prevent prying eyes from accessing data that they should not see. By default, objects stored in S3 aren't encrypted at rest. The files you put into S3 via CFML tags and functions can't be encrypted at rest because the built-in CFML tags and functions don't support that. Fortuntely, the AWS Java SDK makes it easy to encrypt objects at rest in S3.

## S3 Encryption Options

S3 supports three kinds of encryption for objects at rest in S3:

1. <b>Server-Side Encryption with Amazon S3-Managed Keys</b> (SSE-S3): this is the fully AWS-managed option for encrypting objects at rest. Amazon S3 encrypts each object using AES-256 encryption with a unique key. As an additional safeguard, it encrypts the key for each object with a master key that it rotates regularly. This whole process is transparent to you. Unless you have very specifc regulatory requirements that do not allow for AWS-managed keys, you should choose this option.

2. <b>Server-Side Encryption with Customer Master Keys (CMKs) Stored in AWS Key Management Service</b> (SSE-KMS): this option allows you to bring your own keys to encrypt objects at rest and store them in [Key Management Service](https://aws.amazon.com/kms/) (KMS). It's up to you to rotate keys.

3. <b>Server-Side Encryption with Customer-Provided Keys</b> (SSE-C): this option allows you to bring your own keys and your own method for storing them (ie; [Hashicorp Vault](https://www.vaultproject.io)).

Managing keys is no fun, and most likely not part of your core business. Again, unless you have very specific legal requirements which prevent you from using SSE-S3 (which is compliant with many, [many security standards](https://d0.awsstatic.com/whitepapers/compliance/AWS_HIPAA_Compliance_Whitepaper.pdf)), you should choose SSE-S3.

## Encrypting S3 Objects Using the AWS Java SDK

So how do you encrypt objects when you transfer them to S3? As always, the full code for this and all of my demos can be found in my [AWSPlaybox GitHub repo](https://github.com/brianklaas/awsPlaybox). Let's take a look:

{% highlight javascript %}
    s3 = application.awsServiceFactory.createServiceObject('s3');
    fileContent = fileReadBinary(getTempDirectory() & uploadedFile.serverFile);
    // The ObjectMetadata object is where we set the flag to use SSE-S3
    objectMetadata = CreateObject('java', 'com.amazonaws.services.s3.model.ObjectMetadata')
        .init();
    // As with all encryption protocols, the length of the object is very important!
    objectMetadata.setContentLength(ArrayLen(fileContent));
    // AES_256_SERVER_SIDE_ENCRYPTION is the constant representing SSE-S3
    objectMetadata.setSSEAlgorithm(ObjectMetadata.AES_256_SERVER_SIDE_ENCRYPTION);
    // Storing a file with Server-Side Encryption requires a byte stream, not a standard Java file object
    fileInputStream = CreateObject('java', 'java.io.ByteArrayInputStream')
        .init(fileContent);
    putFileRequest = CreateObject('java', 'com.amazonaws.services.s3.model.PutObjectRequest')
        .init(s3BucketName, fileName, fileInputStream, objectMetadata);
    s3.putObject(putFileRequest);
{% endhighlight %}

As you'll see in the rest of this series, when you want to do something other than a standard object upload to S3 using the AWS Java SDK, you'll add property objects to the putFileRequest which represent those additional features. In order to encrypt objects at rest in S3, we pass a ObjectMetadata object to the putFileRequest. In the ObjectMetadata object, we specify that the SSE (server-side encryption) algorithm should be AES_256_SERVER_SIDE_ENCRYPTION &mdash; the constant representing SSE-S3. We also have to pass the file object in as a fileInputStream instead of a simple Java File object, because Java's encryption methods work with streams, not objects.

As with all things Java, it's a bit verbose, but it's very simple to add at-rest encryption to the files you store in S3 when you use the AWS Java SDK. As always, if you have any questions about this, feel free to [message me on the Twitter](https://twitter.com/brian_klaas).