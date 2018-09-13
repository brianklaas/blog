---
layout: post
title:  "Using AWS Transcribe in CFML: Overview and Creating the Transcribe Service Object"
date:   2018-09-14 10:51:00 -0400
categories: AWS ColdFusion
---

As detailed in my previous series, [AWS offers a host of machine learning services](https://aws.amazon.com/machine-learning/) that you can tap into from within CFML. A CFML engine isn't neccessarily the best fit for doing machine learning, so I like to leave the heavy, undifferentiated lifting of setting up and training a machine learning infrastructure to people who really know what they are doing.

In my last series of posts, I detailed [using AWS Rekognition from within CFML](aws/coldfusion/2018/07/23/Using-AWS-Rekognition-In-CFML-Part-1.html). We're going to move on from using Rekognition, a computer vision service, to using another ML-powered service: [Transcribe](https://aws.amazon.com/transcribe/).

### What Can Transcribe Do for You?

Transcribe is, as its name implies, a transcription service. Transcribe takes [a media file stored in S3](https://aws.amazon.com/s3/), pulls out the audio track of that file, and translates the spoken words into plain text. You can achieve a number of business goals using a tool like this, including:

- Transcription of audio or video files on your site to help meet globally-enforced accessibility requirements.
- Automation of subtitle creation for videos.
- Create a searchable database of the audio and video in your site (or company).
- Analyze interview data in legal depositions, government work, or call center activity.

In my work, we use Transcribe for the first two goals. Accessibility is a human right, not just a nice feature to have in your web applications. We are also building out a multi-language translation workflow for our ESL students, and Transcribe is a key component of that workflow. I'll detail that workflow later in the larger series on using AWS from CFML.

As with other AWS services, Transcribe is also dead simple to use from within CFML applications via the AWS Java SDK.

### Working with Transcribe from CFML

I'll once again use [my AWSPlaybox application](https://github.com/brianklaas/awsPlaybox) for all the example code.

> If you haven't already read the entry on [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html), please do so now.

As with all AWS services, you need to first create a client for the service that you want to use. This was covered in [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html) post, but here are the steps involved:

1. Create a Client Builder object for the service &mdash; AmazonTranscribeClientBuilder.
2. Tell the Client Builder what kind of builder object you want to use. It's simplest to use the standard builder.
3. Pass in your credentials via the StaticCredentialsProvider object (created upon instantiation of awsPlaybox/model/awsServiceFactory.cfc).
4. Tell the Client Builder which [AWS region](https://docs.aws.amazon.com/general/latest/gr/rande.html) you're working in. (I generally use 'us-east-1'.)
5. Tell the Client Builder to build (make) the connection to the service.

Here's the relevant code from AWSPlaybox/model/awsServiceFactory.cfc:

{% highlight javascript %}
serviceObject = CreateObject('java', 'com.amazonaws.services.transcribe.AmazonTranscribeClientBuilder').standard().withCredentials(variables.awsStaticCredentialsProvider).withRegion('us-east-1').build();
{% endhighlight %}

Now we can work with Transcribe from within our CFML application. 

In the rest of this series of posts on working with Transcribe from CFML, I'll cover starting a Transcribe job and the myriad options available to you in a transcrbe job, checking on a job's status, and processing the results of a completed job including parsing out the full text transcript from a Transcribe result object.