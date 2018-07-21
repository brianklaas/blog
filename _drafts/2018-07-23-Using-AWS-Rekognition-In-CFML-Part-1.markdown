---
layout: post
title:  "Using AWS Rekognition in CFML: Overview and Creating the Rekognition Service Object"
date:   2018-07-23 09:01:00 -0400
categories: AWS ColdFusion
---

CFML is a flexible and powerful language, encapsulating a lot of complexity in some fairly straightforward functions and tags. From creating PDFs to tackling the verbosity of file manipulation in Java, CFML makes it easy to do a lot in a few lines of code. 

Current CFML engines aren't so good at machine learning tasks. Running a machine learning (ML) infrastructure requires a lot of expertise in a lot of areas, not to mention the basic task of keeping the servers which power the ML activities running. Fortunately for developers, many companies now offer on-demand machine learning services. [AWS offers a whole host of machine learning services](https://aws.amazon.com/machine-learning/), of which [Rekognition](https://aws.amazon.com/rekognition/) is one.

### What Can Rekognition Do for You?

Rekognition is a machine vision service. When Facebook knows that you're in a photo that your friend just posted to their timeline, that's machine vision at work. 

> Note: AWS actually offers two services called Rekognition. One is (simply) Rekognition, which we use here, and which focuses on the analysis of static images. The other is [Rekognition Video](https://aws.amazon.com/rekognition/video-features/), which I will not cover, and which focuses on the analysis of video streams. It's Rekognition Video that has [gotten AWS in hot water with privacy advocates](https://www.nytimes.com/2018/05/22/technology/amazon-facial-recognition.html). One of the primary use cases for Rekognition Video is, to be blunt, facial surveillance in security camera feeds.

Rekognition takes [an image stored in S3](https://aws.amazon.com/s3/) and performs a number of different analysis tasks on that image. The available analysis tasks are:

- Detect labels: the service tries to determine the primary elements of the image, including objects, locations, activities, and weather.
- Detect faces: the service detects all the faces of individuals in the image.
- Match faces: the service takes two images and compares them to see if they are images of the same person.
- Detect sentiment: the service analyzes faces to see if people are happy, sad, smiling, frowning, or a whole host of other expressions.
- Unsafe content detection: the service can flag images that it has learned contain "unsafe" content (ie; nudity).
- Celebrity recognition: the service can detect if one of thousands of celebrities it recognizes is in the image.
- Text in images: the service can analyze an image and pull out all the individual words/letter sequences in the image.

Rekognition can be used for simple and fast cataloging of images in a product library, or even your own personal photo library. As you'll see, it provides many labels for images that go way beyond the identification of a single person or people. Rekognition can be used to match the face of an employee standing at a security gate with an image on file. A student waiting to take a test in an online class could have the face on their webcam matched with their photo ID in a university information system, and can only proceed with the exam once that match is made. You can quickly scan photos to make sure that users aren't uploading photos to your application that contain nudity. You can even build a system that reads the text in an image and translates it to other languages (using [AWS Translate](https://aws.amazon.com/translate/), a service that I will cover later in the series).

As a developer, I found Rekognition to be a lot of fun to play with. It's also dead simple to use from within CFML applications, because of the power of the AWS Java SDK.

### Working with Rekognition from CFML

I'll once again use [my AWSPlaybox application](https://github.com/brianklaas/awsPlaybox) for all the example code.

> If you haven't already read the entry on [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html), please do so now.

As with all AWS services, you need to first create a client for the service that you want to use. This was covered in [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html) post, but here are the steps involved:

1. Create a Client Builder object for the service &mdash; AmazonRekognitionClientBuilder.
2. Tell the Client Builder what kind of builder object you want to use. It's simplest to use the standard builder.
3. Pass in your credentials via the StaticCredentialsProvider object (created upon instantiation of awsPlaybox/model/awsServiceFactory.cfc).
4. Tell the Client Builder which [AWS region](https://docs.aws.amazon.com/general/latest/gr/rande.html) you're working in.
5. Tell the Client Builder to build (make) the connection to the service.

Here's the relevant code from AWSPlaybox/model/awsServiceFactory.cfc:

{% highlight javascript %}
serviceObject = CreateObject('java', 'com.amazonaws.services.rekognition.AmazonRekognitionClientBuilder').standard().withCredentials(variables.awsStaticCredentialsProvider).withRegion('us-east-1').build();
{% endhighlight %}

Now we can work with Rekognition from within our CFML application. 

In the rest of this series of posts on working with Rekognition from CFML, I'll cover detecting and processing labels, comparing faces to one another to look for matches, and detecting text in an image.
