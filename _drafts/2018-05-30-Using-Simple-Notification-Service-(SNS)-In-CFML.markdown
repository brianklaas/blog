---
layout: post
title:  "Using Simple Notification Service (SNS) in CFML"
date:   2018-05-21 13:18:00 -0400
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

SNS is simple (as you'll see in a moment), flexible, and powerful. It is important to note that it is not a classic message queue. If you want a message queue with acknowledgements, message ordering, retries, or dead letter queues, you need to use [SQS (Simple Queue Service)](https://aws.amazon.com/sqs/) instead.

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

### Creating Topics

The [JavaDocs for the AWS Java SDK](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html) are comprehensive and always up-to-date. They are, alas, also just JavaDocs. You're not going to find detailed examples of how to complete full tasks in these docs. If you look at the [documentation for the com.amazonaws.services.sns class](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/sns/AbstractAmazonSNS.html), you can see all the things that you can do with a connection to SNS.

When working with the AWS Java SDK, there's a basic pattern that you'll follow. It goes like this:

1. Get a copy of the client that's making a connection to the service you want to use.
2. Create a "request" object.
3. Fill the "request" object with the parameters you want to add.
4. Tell the client to make the request.
5. Get back a "response" object.

We're working directly with Java here, so everything is an object. CFML fortunately shields us from a lot of the verbose impementation fallout from this reality. The AWS Java SDK is not CFML, so we have to get a little verbose.

Following the pattern above, in order to create a new topic, we:

1. Make a connection to SNS with a SNS client.
2. Create a CreateTopicRequest object.
3. Use the CreateTopicRequest's fluent "with" interface to set the name of the topic.
4. Tell the SNS client to createTopic() using the CreateTopicRequest.
5. Get back a createTopicResult.

The createTopicResult object will contain the ARN of the topic inside AWS. What's an ARN?

> An ARN &mdash; Amazon Resource Name &mdash; is the unique identifier of an object within all of AWS. Almost every object you'll work with in AWS will have an ARN.

Those are the basics. Here's the relevant code from /sns.cfm in the AWSPlaybox app:

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
- We tell the SNS client connection to create the topic, and we get a CreateTopicResult back.
- The CreateTopicResult object contains the unique ARN of our new topic. We need this ARN to make future requests to this topic.

We're storing the new topic ARN in the application scope in the AWSPlaybox app. You'd want to store this in a database in a production app that uses SNS.

Now that we have a topic to work with, we can subscribe to that topic. 

### Subscribing to Topics

Subscribing to topics is a two-step process.

1. Make the subscription request.
2. Confirm that you (the client) actually wants to subscribe.

The AWSPlaybox app contains code that shows you how to make a subscription request programmatically. It follows the same pattern as described above:

1. Make a connection to SNS with a SNS client.
2. Create a SubscribeRequest object.
3. Use the SubscribeRequest's fluent ("with X") interface to set the topic ARN, protocol, and endpoint of the subscription.
4. Tell the SNS client to subscribe() using the SubscribeRequest.

Here's the relevant code from /sns.cfm in the AWSPlaybox app:

{% highlight javascript %}
sns = application.awsServiceFactory.createServiceObject('sns');

subscribeRequest = CreateObject('java', 'com.amazonaws.services.sns.model.SubscribeRequest').withTopicARN(application.awsResources.currentSNSTopicARN).withProtocol("email").withEndpoint(Trim(FORM.emailAddress));

sns.subscribe(subscribeRequest);
{% endhighlight %}

Breaking this down:

- A subscribe request requires that you pass in three parameters:
  1. The ARN of the topic you want to subscribe to.
  2. The Protocol that you will use for the subscription. This can be SMS, email, or http/s, among others. The [full listing of protocol and endpoint options](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/sns/model/SubscribeRequest.html) is in the docs.
  3. The Endpoint for the subscription. This is the email address or phone number (among others) you want to use to subscribe.
- In this case, we tell the SNS client to subcribe but don't ask for a SubscribeResult back. If we wanted to manage this individual subscription programatically in the future, we'd need to get the ARN of the individual subscription and store that in a database:

{% highlight javascript %}
subscribeResult = sns.subscribe(subscribeRequest);
storeInDatabaase(subscribeResult.getSubscriptionArn());
{% endhighlight %}

That's the first part of subscribing to a SNS topic.

The second step of the process is the endpoint (email address, phone number, etc) accepting the subcription. Opting in to a subscription important because malicious applications could otherwise automatically spam SNS or email messages to unwilling (and unhappy) subscribers.

If your SNS topic sends messages via email or SNS, the recipient simply needs to respond to a "Do you want to subscribe to this topic?" message that's automatically generated by SNS when you subscribe to a topic. That part of the process is fully controlled by AWS and isn't something you can customize.  

If you want the example in the AWSPlaybox app to work, you'll need to submit your email address to the provided form, and then respond to the email message sent from AWS to subscribe to the topic that you just created. If you want to subscribe to this topic via SMS, you'll need to [sign in to the AWS Console](https://console.aws.amazon.com/console/home) and add that subscription manually within the SNS management console. You could also add that code to /sns.cfm in the AWSPlaybox app and submit a pull request! :)

If you want your CFML application to act as a subscriber to an SNS topic, you need to follow the steps outlined in the [documentation on sending SNS messages to HTTP/S endpoints](https://docs.aws.amazon.com/sns/latest/dg/SendMessageToHttp.html). This requires that your CFML application be able to confirm that it wants to subscribe to a SNS topic via a GET HTTP request and then process individual SNS messages as they come in.

### Sending Messages

Sending a message to SNS is easy. You need to:

1. Give your message a subject
2. Give your message a body (plain text only)
3. Create a PublishRequest object
4. Tell your SNS client to publish your PublishRequest

Here's the relevant code from AWSPlaybox/sns.cfm:

{% highlight javascript %}
subject = "AWS Playbox SNS CFML Demo";
message = "Hello there!" & chr(13) & chr(13) & "The current time is " & DateTimeFormat(Now(), "Full") & ".";

sns = application.awsServiceFactory.createServiceObject('sns');
publishRequest = CreateObject('java', 'com.amazonaws.services.sns.model.PublishRequest').init(application.awsResources.snsTopicARN, message,subject);

sns.publish(publishRequest);
{% endhighlight %}

If you actually subscribed to the topic that we just created, you should get a copy of the message within seconds. Here's what the messages look like when delivered:

Email

<img src="/assets/postImages/sampleSNSEmail.png" align="center" width="400" height="183" border="1" alt="Sample email message from SNS" />

SMS

<img src="/assets/postImages/sampleSNSText.png" align="center" width="400" height="244" border="1" alt="Sample text message from SNS" />

You can see that in the SMS message, the text message was prefixed by the topic name. The subject value was omitted.

That's it! It's really quite simple to send a message to SNS via the AWS Java SDK from within your CFML application. If you are sending SMS messages, there are lots of [options around price and SMS message type that you can set from within the Java SDK](https://docs.aws.amazon.com/sns/latest/dg/sms_publish-to-phone.html). You do not need to use these &mdash; they're simply there to help you build a more robust SMS application (if that's what you are doing).

### Caveats About Working with SNS

SNS is a great notification service, but it's not a good transactional or marketing email service. You really should not use SNS for that purpose. You can use [SES (Simple Email Service)](https://aws.amazon.com/ses/) or, better yet, one of many third-party services that do an amazing job of sending transactional and marketing emails. SNS email messages would be a good fit for your DevOps team when something goes wrong with your EC2 instances or if you exceed service limits in AWS. It's a bad tool for sending email messages to your CEO. Messages are simple text strings, not HTML, and won't end up looking pretty.

SNS is drop-dead simple for sending SMS (text) messages. SNS is actually designed to support both marketing and transactional SMS messages. However, you can only send 100 SMS messages per month for free to US phone numbers. Everything else (and sending to anywhere outside the US) [incurs a cost](https://aws.amazon.com/sns/sms-pricing/). You can [set spend limits on a SNS topic](https://docs.aws.amazon.com/sns/latest/dg/sms_preferences.html) so that you don't spend more than a set amount in a given month. Even so, sending SMS messages through SNS can add up quickly.

Sending SMS messages through SNS also means that each message needs to conform to standard text messaging limits &mdash; ie; 140 ASCII characters per message. Longer messages are automatically split up into multiple messages for you, and you're charged for each "part" of that split message.

If SNS fails to send a message (of any kind) to a subscriber, it will retry up to three times, waiting 20 seconds between retries by default. That's really handy. However, if you need guaranteed delivery, first-in-first-out queues, or anything slightly more complex than basic retries, you should be using Simple Queue Service (SQS) to send messages that will be processed by remote worker applications.

### Go Do!

It's really easy to send messages through SNS via ColdFusion. Most of this post was explaining how the service worked. The code was short and quite simple to understand.

If you're interested in performing native mobile push notifications via SNS, the always-excellent [AWS Docs cover how to set that up](https://docs.aws.amazon.com/sns/latest/dg/SNSMobilePush.html). It's not something that I'm doing at this time.

As mentioned previously, if you want your CFML application to act as a subscriber to an SNS topic, you need to follow the steps outlined in the [documentation on sending SNS messages to HTTP/S endpoints](https://docs.aws.amazon.com/sns/latest/dg/SendMessageToHttp.html). This requires that your CFML application be able to confirm that it wants to subscribe to a SNS topic via a GET HTTP request and then process individual SNS messages as they come in. This is more than I wanted to show in the AWSPlaybox application (especially as SNS needs to be able to communicate with your server, requiring it have a public DNS or use a service like [ngrok](https://ngrok.com)). This topic would make for a good future post, no?

If you've got questions about this, feel free to <a href="https://twitter.com/brian_klaas">message me on Twitter</a>!