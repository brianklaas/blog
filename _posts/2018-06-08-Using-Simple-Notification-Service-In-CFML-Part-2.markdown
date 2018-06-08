---
layout: post
title:  "Using Simple Notification Service (SNS) in CFML: Creating Topics"
date:   2018-06-08 10:47:00 -0400
categories: AWS ColdFusion
---

The process of creating a topic in SNS via CFML is simple, as is working with SNS in general.

> If you haven't already read the entry on [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html), please do so now.

The [JavaDocs for the AWS Java SDK](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html) are comprehensive and always up-to-date. They are, alas, also just JavaDocs. You're not going to find detailed examples of how to complete full tasks in these docs. If you look at the [documentation for the com.amazonaws.services.sns class](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/sns/AbstractAmazonSNS.html), you can see all the things that you can do with a connection to SNS.

When working with the AWS Java SDK, there's a basic pattern that you'll follow. It goes like this:

1. Get a copy of the client that's making a connection to the service you want to use.
2. Create a "request" object.
3. Fill the "request" object with the parameters you want to add.
4. Tell the client to make the request.
5. Get back a "response" object.

We're working directly with Java here, so everything is an object. CFML fortunately shields us from a lot of the verbose impementation fallout from this reality. The AWS Java SDK is not CFML, so we have to get a little verbose.

Following the pattern above, in order to create a new topic, we:

1. Make a connection to SNS with a [SNS client](/aws/coldfusion/2018/06/03/Using-Simple-Notification-Service-In-CFML-Part-1.html).
2. Create a CreateTopicRequest object. (This is, literally, the request to create a new topic.)
3. Use the CreateTopicRequest's fluent "with" interface to set the name of the topic.
4. Tell the SNS client to createTopic() using the CreateTopicRequest.
5. Get back a createTopicResult.

The createTopicResult object will contain the ARN of the topic inside AWS. What's an ARN?

> An ARN &mdash; Amazon Resource Name &mdash; is the unique identifier of an object within all of AWS. Almost every object you'll work with in AWS will have an ARN.

Those are the basics. Here's the relevant code from /sns.cfm in the [AWSPlaybox app](https://github.com/brianklaas/awsPlaybox):

{% highlight javascript %}
sns = application.awsServiceFactory.createServiceObject('sns');

topicName = "AWSPlayboxDemoTopic-" & dateTimeFormat(Now(), "yyyy-mm-dd-HH-nn-ss");
createTopicRequest = CreateObject('java', 'com.amazonaws.services.sns.model.CreateTopicRequest').withName(topicName);

createTopicResult = sns.createTopic(createTopicRequest);

application.awsResources.currentSNSTopicARN = createTopicResult.getTopicArn();
{% endhighlight %}

Breaking this down:

- Remember that we use the AWSPlaybox/model/awsServiceFactory.cfc to make the actual client connection to SNS, passing in our credentials during instantitation of the awsServiceFactory.cfc object.
- Every topic has to have a name that's unique to your AWS account. Here we append the current date/time to make the topic name unqique.
- Most request/result objects in the AWS Java SDK have a fluent interface. That is, you can create an object and chain together the setting of the properties of that object. Here we set the name of our new topic with the withName() function.
- We tell the SNS client connection to create the topic, and we get a CreateTopicResult object back.
- The CreateTopicResult object contains the unique ARN of our new topic. We need this ARN to make future requests to this topic.

We're storing the new topic ARN in the application scope in the AWSPlaybox app. You'd want to store this in a database in a production app that uses SNS.

Now that we have a topic to work with, we can subscribe to that topic. That's the subject of the next post.