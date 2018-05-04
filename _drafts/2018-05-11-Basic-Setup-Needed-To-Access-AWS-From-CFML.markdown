---
layout: post
title:  "Basic Setup Needed to Access AWS from CFML"
date:   2018-05-11 13:17:00 -0400
categories: AWS ColdFusion
---

In order to access [Amazon Web Services](https://aws.amazon.com/) from within CFML applications, you're going to need to use the [AWS Java SDK](https://aws.amazon.com/sdk-for-java/) CFML runs on top of the JRE, so we can tap into the vast array of Java SDKs that companies make available to Java developers.

I understand that for some CFML developers, using Java can feel daunting. After all, we chose CFML as a development language because it makes us highly productive and abstracts away most of Java's idioms and verbose language constructs. CFML can feel a lot like JavaScript in its simple language syntax, and that's a good thing.

Using the Java SDKs from other companies provides us with a fast way to unlock the power of their services, without having to write our own http call wrappers. You can absolutely work with AWS using pure http requests, but it's a lot simpler with [the various SDKs that they offer](https://aws.amazon.com/tools/). 

### Adding the .jar Files to Your CFML Engine

First, make sure you have downloaded the [AWS Java SDK](https://aws.amazon.com/sdk-for-java/).

If your CFML engine is [Adobe ColdFusion](https://www.adobe.com/products/coldfusion-family.html), you add the following .jar files to your cfusion/lib directory.

If your CFML engine is [Lucee](http://lucee.org), add the following to the /WEB-INF/lucee/lib/ directory in the root of your site.

- aws-java-sdk-1.11.xxx.jar
- jackson-annotations-2.6.0.jar
- jackson-core-2.6.7.jar
- jackson-databind-2.6.7.1.jar
- joda-time-2.8.1.jar

All of these .jar files are in the AWS Java SDK download. The AWS SDK .jar is in the /lib directory, and the others are in the /third-party directory.

> For this series, we used the aws-java-sdk-1.11.311.jar. The .311 jar or later is required to work with the examples in this series.

You will need to restart your CFML server to be able to work with these .jars.

### Providing AWS Credentials to Your Application

As I mentioned in the previous post, I won't be going in to the many details of [Identity Access Management (IAM) in AWS](https://aws.amazon.com/documentation/iam/). I *will* say that in order to use AWS services from within the Java SDK, you will need to pass credentials to AWS from your Java SDK-based calls.

Briefly, you will need to create an IAM user that has permissions to access all of the services listed below:

- [S3 &mdash; Simple Storage Service](https://aws.amazon.com/s3/)
- [SNS &mdash; Simple Notification Service](https://aws.amazon.com/sns/)
- [Lamdba](https://aws.amazon.com/lambda/)
- [CloudWatch](https://aws.amazon.com/cloudwatch/)
- [DynamoDB](https://aws.amazon.com/dynamodb/)
- [Step Functions](https://aws.amazon.com/step-functions/)
- [Rekognition](https://aws.amazon.com/rekognition/)
- [Transcribe](https://aws.amazon.com/transcribe/)
- [Translate](https://aws.amazon.com/translate/)
- [Polly](https://aws.amazon.com/polly/)

You can give this user "*" (everything) permissions, either to each service or to AWS in general. AWS works off the principle of least privilege, so you have to explicity grant access to services (and even individual sub-functions of services). You can say "oh give this account permission to everything in AWS," but if someone gets ahold of those credentials, it could wind up [costing you thousands of dollars a day](https://wptavern.com/ryan-hellyers-aws-nightmare-leaked-access-keys-result-in-a-6000-bill-overnight).

When you create an IAM user, you will receive an access key (username) and secret key (password) for that user. You will need this information to successfully use the AWS Java SDK.

In the [AWSPlaybox code](https://github.com/brianklaas/awsplaybox/), you add the access key and secret key to model/awsCredentials.cfc.

When working with credentials in the AWS Java SDK, you have multiple options. To keep everyting encapsulated in code (rather than fiddling with environment configs), we are going to use the Basic AWS Credentials object to connect to AWS. We need to:

1. Create a Basic Credentials object. 
2. Pass the Basic Credentials object to a Credentials Provider object, which makes the actual connection to AWS.

In the AWSPlaybox code, this is done in /model/awsServiceFactory.cfc. This component is responsible for making connections to the various services that we use in the AWSPlaybox app. Here's the relevant code:

{% highlight javascript %}
var awsCredentials = CreateObject('java','com.amazonaws.auth.BasicAWSCredentials').init(yourAccessKey, yourSecretKey);
variables.awsStaticCredentialsProvider = CreateObject('java','com.amazonaws.auth.AWSStaticCredentialsProvider').init(awsCredentials);
{% endhighlight %}

When we want to connect to an individual service in AWS, we have to build a connection to that service using a Client Builder object. For example, if we want to connect to Lambda (a serverless code execution environment), we would do the following:

1. Create a Client Builder object for the service (in this case, AWSLambdaClientBuilder. Other services follow the same pattern with the names of their builers &mdash; ie; AmazonDynamoDBClientBuilder, AmazonRekognitionClientBuilder, AmazonSNSClientBuilder, etc.)
2. Tell the Client Builder what kind of builder object you want to use. It's simplest to use the standard builder.
3. Pass in your credentials via the StaticCredentialsProvider object.
4. Tell the Client Builder which [AWS region](https://docs.aws.amazon.com/general/latest/gr/rande.html) you're working in.
5. Tell the Client Builder to build (make) the connection.

> [AWS Regions](https://aws.amazon.com/about-aws/global-infrastructure/) are globally distributed centers of operation that contain 3 or more individual data centers that are spread a few kilometers apart. Each data center in a region is known as an Availability Zone.

Here's the relevant code:

{% highlight javascript %}
serviceObject = CreateObject('java', 'com.amazonaws.services.lambda.AWSLambdaClientBuilder').standard().withCredentials(variables.awsStaticCredentialsProvider).withRegion('us-east-1').build();
{% endhighlight %}

And that's it! You now have a connection to an individual AWS service. If you want to utilize multiple services, you have to make a unique connection object to each service that you want to use. These objects which represent a connection to a service (Lambda, DynamoDB, SNS, etc) should be [Singletons](https://en.wikipedia.org/wiki/Singleton_pattern) in your application and re-used whenever possible.

In the next installment in this series, we'll be looking at the one AWS service that has native support in CFML: S3, or Simple Storage Service.