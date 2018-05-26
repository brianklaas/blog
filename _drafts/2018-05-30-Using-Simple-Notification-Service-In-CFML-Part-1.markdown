---
layout: post
title:  "Using Simple Notification Service (SNS) in CFML: Overview and Connecting to SNS"
date:   2018-05-30 13:18:00 -0400
categories: AWS ColdFusion
---

We're going to move on from [the single AWS service that's natively supported in CFML](/aws/coldfusion/2018/05/21/Using-Simple-Storage-Service-In-CFML.html) to the first service example that uses the AWS Java SDK from within CFML: Simple Notification Service, or SNS.

If you haven't already read the entry on [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html), please do so now.

### What Can Simple Notification Service (SNS) Do for You?

SNS is a simple [publish/subscribe](https://en.wikipedia.org/wiki/Publish–subscribe_pattern) service. Clients subscribe to a SNS topic, and when a message is published to that topic, all subscribers are notififed with a copy of the message. Publishers have no idea who is receiving the message, as clients of different types can subscribe to the topic. You can send emails, text messages, or native push notifications via SNS. 

Some examples of using SNS are:

- Send an email to a subscriber when files are uploaded into a S3 bucket that exceed your file size limits.
- Send a verification code text message as part of two-factor authentication during login to your application.
- Publishing notifications to one or more [SQS (Simple Queue Service)](https://aws.amazon.com/sqs/) queues, from which individual messages would be picked up and processed by connected worker clients/applications.
- Pushing a message to a [server endpoint (URL)](https://docs.aws.amazon.com/sns/latest/dg/SendMessageToHttp.html) in a CFML application to trigger or process work based on the content of the message.
- Pushing a message to a subscriber [Lambda](https://aws.amazon.com/lambda/) function for processing the message's contents.
- Send a message to SNS when one of your [EC2](https://aws.amazon.com/ec2/) instances gets shut down unexpectedly.
- Send a message to SNS when a [CloudWatch alarm](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) is triggered because you've exceeded more than a specified amount of storage in S3.
- Send a [native iOS/Android notification](https://docs.aws.amazon.com/sns/latest/dg/mobile-push-send.html) to a customer when there is a new notification in your mobile app.

SNS is simple (as you'll see), flexible, and powerful. It is important to note that SNS is not a classic message queue. If you want a message queue with acknowledgements, message ordering, retries, or dead letter queues, you need to use [SQS (Simple Queue Service)](https://aws.amazon.com/sqs/) instead.

### Working with SNS from CFML

I'll once again use [my AWSPlaybox application](https://github.com/brianklaas/awsPlaybox) for all the example code.

As with all AWS services, you need to first create a client for the service that you want to use. This was covered in [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html) post, but here are the basic steps:

1. Create a Client Builder object for the service &mdash; AmazonSNSClientBuilder.
2. Tell the Client Builder what kind of builder object you want to use. It's simplest to use the standard builder.
3. Pass in your credentials via the StaticCredentialsProvider object (created upon instantiation of awsPlaybox/model/awsServiceFactory.cfc).
4. Tell the Client Builder which [AWS region](https://docs.aws.amazon.com/general/latest/gr/rande.html) you're working in.
5. Tell the Client Builder to build (make) the connection.

Here's the relevant code from AWSPlaybox/model/awsServiceFactory.cfc:

{% highlight javascript %}
serviceObject = CreateObject('java', 'com.amazonaws.services.sns.AmazonSNSClientBuilder').standard().withCredentials(variables.awsStaticCredentialsProvider).withRegion('us-east-1').build();
{% endhighlight %}

Now we can work with SNS from within our CFML application. 

In the rest of this series of posts on working with SNS from CFML, I'll cover creating topics, subscribing to topics, and sending messages.
