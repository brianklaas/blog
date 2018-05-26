---
layout: post
title:  "Using Simple Notification Service (SNS) in CFML: Sending Messages"
date:   2018-06-09 13:18:00 -0400
categories: AWS ColdFusion
---

Sending a message to SNS from CMFL via the AWS Java SDK is easy. You need to:

1. Give your message a subject
2. Give your message a body (plain text only)
3. Create a PublishRequest object
4. Tell your SNS client to publish your PublishRequest

> If you haven't already read the entry on [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html), please do so now.

Here's the relevant code from /sns.cfm in the [AWSPlaybox app](https://github.com/brianklaas/awsPlaybox):

{% highlight javascript %}
subject = "AWS Playbox SNS CFML Demo";
message = "Hello there!" & chr(13) & chr(13) & "The current time is " & DateTimeFormat(Now(), "Full") & ".";

sns = application.awsServiceFactory.createServiceObject('sns');
publishRequest = CreateObject('java', 'com.amazonaws.services.sns.model.PublishRequest').init(application.awsResources.snsTopicARN, message,subject);

sns.publish(publishRequest);
{% endhighlight %}

If you actually subscribed to the topic that we created in [the previous post in this series](/aws/coldfusion/2018/06/07/Using-Simple-Notification-Service-In-CFML-Part-3.html) (or in the [AWSPlaybox app](https://github.com/brianklaas/awsPlaybox)), you should get a copy of the message within seconds. Here's what the messages look like when delivered:

Email

<img src="/assets/postImages/sampleSNSEmail.png" align="center" width="400" height="183" border="1" alt="Sample email message from SNS" />

SMS

<img src="/assets/postImages/sampleSNSText.png" align="center" width="400" height="244" border="1" alt="Sample text message from SNS" />

You can see that in the SMS message, the text message was prefixed by the topic name. The subject value was omitted.

That's it! It's really quite simple to send a message to SNS via the AWS Java SDK from within your CFML application. If you are sending SMS messages, there are lots of [options around price and SMS message type that you can set from within the Java SDK](https://docs.aws.amazon.com/sns/latest/dg/sms_publish-to-phone.html). You do not need to use these &mdash; they're simply there to help you build a more robust SMS application (if that's what you are doing).

Those are the basics of working with SNS from CMFL. In the next post, I'll go over some caveats about working with SNS that I've encountered.