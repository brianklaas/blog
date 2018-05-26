---
layout: post
title:  "Using Simple Notification Service (SNS) in CFML: Caveats About Working with SNS in CMFL"
date:   2018-06-11 13:18:00 -0400
categories: AWS ColdFusion
---

SNS is a great notification service, but it's not a good transactional or marketing email service. You really should not use SNS for that purpose. You can use [SES (Simple Email Service)](https://aws.amazon.com/ses/) or, better yet, one of many third-party services that do an amazing job of sending transactional and marketing emails. SNS email messages would be a good fit for your DevOps team when something goes wrong with your EC2 instances or if you exceed service limits in AWS. It's a bad tool for sending email messages to your CEO. Messages are simple text strings, not HTML, and won't end up looking pretty.

SNS is drop-dead simple for sending SMS (text) messages. SNS is actually designed to support both marketing and transactional SMS messages. However, you can only send 100 SMS messages per month for free to US phone numbers. Everything else (and sending to anywhere outside the US) [incurs a cost](https://aws.amazon.com/sns/sms-pricing/). You can [set spend limits on a SNS topic](https://docs.aws.amazon.com/sns/latest/dg/sms_preferences.html) so that you don't spend more than a set amount in a given month. Even so, sending SMS messages through SNS can add up quickly.

Sending SMS messages through SNS also means that each message needs to conform to standard text messaging limits &mdash; ie; 140 ASCII characters per message. Longer messages are automatically split up into multiple messages for you, and you're charged for each "part" of that split message.

If SNS fails to send a message (of any kind) to a subscriber, it will retry up to three times, waiting 20 seconds between retries by default. That's really handy. However, if you need guaranteed delivery, first-in-first-out queues, or anything slightly more complex than basic retries, you should be using Simple Queue Service (SQS) to send messages that will be processed by remote worker applications.

### Go Do!

It's really easy to send messages through SNS via ColdFusion. Most of this five-part post was dedicated to explaining how the service worked. The code was short and quite simple to understand.

If you're interested in performing native mobile push notifications via SNS, the always-excellent [AWS Docs cover how to set that up](https://docs.aws.amazon.com/sns/latest/dg/SNSMobilePush.html). It's not something that I'm doing at this time.

As mentioned previously, if you want your CFML application to act as a subscriber to an SNS topic, you need to follow the steps outlined in the [documentation on sending SNS messages to HTTP/S endpoints](https://docs.aws.amazon.com/sns/latest/dg/SendMessageToHttp.html). This requires that your CFML application be able to confirm that it wants to subscribe to a SNS topic via a GET HTTP request and then process individual SNS messages as they come in. This is more than I wanted to show in the [AWSPlaybox application](https://github.com/brianklaas/awsPlaybox) (especially as SNS needs to be able to communicate with your server, requiring it have a public DNS or use a service like [ngrok](https://ngrok.com)). This topic would make for a good future post, no?

If you've got questions about this, feel free to <a href="https://twitter.com/brian_klaas">message me on Twitter</a>!