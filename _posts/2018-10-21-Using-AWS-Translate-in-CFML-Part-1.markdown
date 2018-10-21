---
layout: post
title:  "Using AWS Translate in CFML: Overview and Creating the Translate Service Object"
date:   2018-10-21 08:41:00 -0400
categories: AWS ColdFusion
---

Our next stop in our trip through working with [the various machine learning services offered by AWS](https://aws.amazon.com/machine-learning/) from within CFML applications takes us to the land of multi-language translation with AWS [Translate](https://aws.amazon.com/translate/).

### What Can Translate Do for You?

Translate is, obviously, a translation service. As of October, 2018, it can translate text input to or from English and any of the following languages:

- Arabic (ar)
- Chinese (Simplified) (zh)
- Chinese (Traditional) (zh-TW)
- Czech (cs)
- French (fr)
- German (de)
- Italian (it)
- Japanese (ja)
- Portuguese (pt)
- Russian (ru)
- Spanish (es)
- Turkish (tr)

Not only can you translate from Simplified Chinese to English, you can translate from English to Simplified Chinese, or any other language combination. You could, quite easily, build out a real-time translation engine for a chat or customer service application using AWS Translate. English will be your constant intermediary language, but a customer typing in German could communicate with a customer service representative who speaks Czech using Translate.

Translate is [pretty cheap](https://aws.amazon.com/translate/pricing/): $15 per million characters, with 2 million characters free per month.

Translate is also one of the easiest services to use from within CFML applications via the AWS Java SDK. There are some interesting service limits for which you have to compensate in your calling CFML code, which I'll cover in the next post.

### Working with Transcribe from CFML

I'll once again use [my AWSPlaybox application](https://github.com/brianklaas/awsPlaybox) for all the example code.

> If you haven't already read the entry on [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html), please do so now.

As with all AWS services, you need to first create a client for the service that you want to use. This was covered in [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html) post, but here are the steps involved:

1. Create a Client Builder object for the service &mdash; AmazonTranslateClientBuilder.
2. Tell the Client Builder what kind of builder object you want to use. It's simplest to use the standard builder.
3. Pass in your credentials via the StaticCredentialsProvider object (created upon instantiation of awsPlaybox/model/awsServiceFactory.cfc).
4. Tell the Client Builder which [AWS region](https://docs.aws.amazon.com/general/latest/gr/rande.html) you're working in. (I generally use 'us-east-1'.)
5. Tell the Client Builder to build (make) the connection to the service.

Here's the relevant code from AWSPlaybox/model/awsServiceFactory.cfc:

{% highlight javascript %}
serviceObject = CreateObject('java', 'com.amazonaws.services.translate.AmazonTranslateClientBuilder').standard().withCredentials(variables.awsStaticCredentialsProvider).withRegion(#variables.awsRegion#).build();
{% endhighlight %}

Now we can work with Translate from within our CFML application. 

In the next post (the only other post on Translate, because the service is so simple), we'll look at how you translate text, and, more importantly, how you compensate for the service limits that AWS Translate puts on your requests.