---
layout: post
title:  "Using AWS Step Functions in CFML: Performing the Image Analysis Task"
date:   2019-05-10 12:51:00 -0400
categories: AWS ColdFusion
---

It's taken us a few posts to get to this point, but we've now randomly chosen an image on which we'll perform analysis using [AWS Rekognition](https://brianklaas.net/aws/coldfusion/2018/07/23/Using-AWS-Rekognition-In-CFML-Part-1.html), a machine vision service. This post introduces no new state types, and focuses on the Lambda code used to perform the image analysis.

The last state in the execution of our workflow was one of two pass states: either setDataForImageOne or setDataForImageTwo. Both of these pass states specified "getImageLabels" as the next step in the workflow. Here's the definition of the getImageLabels state:

{% highlight json %}
"getImageLabels": {
    "Type" : "Task",
    "Resource": "arn:aws:lambda:us-east-1:123456789:function:detectLabelsForImage",
    "Next": "Finish"
}
{% endhighlight %}

This is a simple task state that finishes up our workflow. [The code for the detectLabelsForImage Lambda function](https://github.com/brianklaas/awsPlaybox/blob/master/nodejs/lambda/detectLabelsForImage.js) is as follows:

{% highlight javascript %}
const util = require('util');
const AWS = require('aws-sdk');
const rekognition = new AWS.Rekognition();

exports.handler = (event, context, callback) => {

    const srcBucket = event.s3Bucket;
    // Object key may have spaces or unicode non-ASCII characters.
    const srcKey = decodeURIComponent(event.s3Key.replace(/\+/g, " "));

    var params = {
        Image: {
            S3Object: {
                Bucket: srcBucket,
                Name: srcKey
            }
        },
        MaxLabels: 10,
        MinConfidence: 70
    };
    
    rekognition.detectLabels(params).promise().then(function (data) {
        data.s3Bucket = srcBucket;
        data.s3Key = srcKey;
        callback(null, data);
    }).catch(function (err) {
        callback(err);
    });

};
{% endhighlight %}

Note that the event.s3Bucket and event.s3Key values came from the setDataForImageOne or setDataForImageOne pass states.

If you'd like to know what the MaxLabels and MinConfidence values represent in the params structure, please read [the post on detecting labels in an image](https://brianklaas.net/aws/coldfusion/2018/07/29/Using-AWS-Rekognition-In-CFML-Part-2.html) from my [series on working with the Rekognition service from CFML](https://brianklaas.net/aws/coldfusion/2018/07/23/Using-AWS-Rekognition-In-CFML-Part-1.html).

The important code in this Lambda function comes at the end, where we invoke the detectLabels() function on the Rekognition service object. This function returns a set of data that we will later return to the caller of our Step Functions workflow. We add the s3Bucket and s3Key values to that data object so our caller will know which image was randomly chosen. That data is then sent back to our caller &mdash; the Step Functions workflow.

This task state is the final practical step in this example Step Functions workflow. The question then is: how does a Step Functions workflow know when to finish? How does the result of this Step Functions workflow get back to the thing that invoked this Step Functions workflow execution in the first place? 

That's the subject of the next post.