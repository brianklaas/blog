---
layout: post
title:  "Using AWS Step Functions in CFML: Injecting Key-Value Pairs Into Your Step Function Execution with Pass States"
date:   2019-05-08 10:30:00 -0400
categories: AWS ColdFusion
---

As we work our way through this [example workflow of performing analysis on a randomly selected image](https://github.com/brianklaas/awsPlaybox/blob/master/stateMachines/choiceDemoStateMachine.json), we've looked at task states, task input and output, and making choices in a workflow. In this post, we'll look at a simple, but critical, option in the execution of a Step Functions workflow: adding additional data to the workflow execution.

Our workflow exection has made a choice as to where it goes next in the process. It will go either to the setDataForImageOne state if the random number we generated was less than or equal to 50, or it will go to the setDataForImageTwo state if the random number we generated is greater than 50. Both the setDataForImageOne state and the setDataForImageTwo state are known as "pass" states. 

### Pass States: What Are They Good For?

Not much happens in a pass state. Pass states exist solely to inject new key-value pairs into the set of data being passed into future steps in the Step Functions workflow. In our example, the setDataForImageOne and setDataForImageTwo states are represented as follows:

{% highlight json %}
"setDataForImageOne": {
    "Type" : "Pass",
    "Result": { "s3Bucket": "awsplayboxbucket", "s3Key": "images/familyFaces.jpg" },
    "Next": "getImageLabels"
},

"setDataForImageTwo": {
    "Type" : "Pass",
    "Result": { "s3Bucket": "awsplayboxbucket", "s3Key": "images/dogForLabels.jpg" },
    "Next": "getImageLabels"
},
{% endhighlight %}

It's the Result field that is of interest here. The data in the Result field gets added to the current set of state data being passed into the Pass state. You can pass in a single key-value pair, or multiple pairs. Either way, the content of the Result field needs to be expressed as JSON. 

In these two Pass states, we add two key-value pairs to the current set of state data being passed through our Step Functions workflow. First, we define the s3Bucket key. In both cases, the value of the key is "awsplayboxbucket." This value is the name of the S3 bucket where the image we will analyze resides. The second key-value pair represents the s3Key, or the logical path to the image file in the "awsplayboxbucket" bucket that we want to analyze. Both key-value pairs will be passed in to the next state, getImageLabels, as input.

You can use the ResultPath field in a Pass state to re-assign the data in the Result field to another variable name, or even add it to an existing structure in the set of state data being passed through the Step Functions workflow. I have not done that here, so the s3Bucket and s3Key keys just get added, as new variables, to the set of state data being passed through the Step Functions workflow.

To recap: we have randomly chosen one of two images for analysis, and added the location of that image to our current set of state data. Next up, we will perform analysis on that image using [AWS Rekognition](https://brianklaas.net/aws/coldfusion/2018/07/23/Using-AWS-Rekognition-In-CFML-Part-1.html).