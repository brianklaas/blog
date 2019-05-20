---
layout: post
title:  "Using AWS Step Functions in CFML: Looping in Step Functions Workflows via the Wait State"
date:   2019-06-03 15:51:00 -0400
categories: AWS ColdFusion
---

At this point in the [second example Step Functions workflow](https://github.com/brianklaas/awsPlaybox/blob/master/stateMachines/transcribeTranslateSpeakWorkflow.json), we're waiting for our [AWS Transcribe](https://brianklaas.net/aws/coldfusion/2018/09/14/Using-AWS-Transcribe-in-CFML-Part-1.html) job to complete. AWS Transcribe works faster than real-time, but it's not instantaneous. If you're trying to transcribe a 90 minute podcast or 70 minute video, it's going to take more than a few minutes for Transcribe to do its work. Our workflow of translating the transcription cannot continue until this work is done.

So how do we wait until an asyncrhonous task like this is done? How do we even know when it's done?

### Looping in Step Functions Workflows

In order to wait for something that will complete at an indefinite point in the future, we have to create a loop in our Step Functions workflow. This is a common pattern for handling tasks (not neccessarily task states) that take an indefinite amount of time to complete.

The loop construct in the second example Step Functions workflow is represented as follows:

{% highlight json %}
"WaitForTranscriptionComplete": {
    "Type": "Wait",
    "Seconds": 30,
    "Next": "Get Job Status"
},
"Get Job Status": {
    "Type": "Task",
    "Resource": "arn:aws:lambda:us-east-1:123456789:function:CheckTranscribeJobStatus",
    "Next": "Transcript Complete?"
},
"Transcript Complete?": {
    "Type": "Choice",
    "Choices": [
        {
          "Variable": "$.jobStatus",
          "StringEquals": "FAILED",
          "Next": "Transcript Failed"
        },
        {
          "Variable": "$.jobStatus",
          "StringEquals": "COMPLETED",
          "Next": "Get Transcription File"
        }
    ],
    "Default": "WaitForTranscriptionComplete"
}
{% endhighlight %}

The above code represents three steps:

1. Wait for 30 seconds
2. Run a Lambda function that checks the status of the Transcribe job
3. If the status of the Transcribe job is FAILED or COMPLETED, move on to the appropriate next step. Otherwise, go with the Default state value, and return to the wait state.

It's this last line that allows a loop to happen in our Step Functions workflow. If the Transcribe job isn't FAILED OR COMPLETED, we default to the Default choice, which goes back to the start of our loop (WaitForTranscriptionComplete). 

> It's important to note that loops can be expensive in Step Functions Workflows. You are [charged on the number of state transitions (moving from one state in your worfklow to another) that occur](https://aws.amazon.com/step-functions/pricing/). If a Transcribe job takes ten minutes to complete, we will go through this loop 20 times, resulting in 60 separate state transitions (20 x 3 separate states). That's not too bad. If the job takes a day to complete, however, the loop will execute 2,880 times, resulting in 8,640 state transitions, which is a lot more expensive. You also need to add to that the execution cost (if any) of the task steps inside the loop. You need to balance the wait time in your wait step against the cost of unneccessarily fast loops.

### Wait States

The code above also introduces a new type of state available in Step Functions: wait states. Wait states do exactly that: they wait for a specified amount of time. In this example, we've hard-coded the Seconds value, but you can reference either a numeric variable or a date/time string variable.

Wait states aren't just for looping. If you know that there will be a set pause in your workflow every time you run it, you can account for that with a wait state. For example, if you have a complex set of data transformations that you need to do on an EC2 instance, or a database ETL script that runs nightly, and you know that it takes these jobs, on average, 10 minutes to execute, you can tell Step Functions to wait those 10 minutes before proceeding to the next step in a workflow.

### Checking the Transcribe Job Status

Although [I covered how to check on a Transcribe job status from CFML in an earlier series of posts](/aws/coldfusion/2018/09/28/Using-AWS-Transcribe-in-CFML-Part-3.html), doing that from Node.js is a bit different. The code for this work can be found in [/nodejs/lambda/transcribeTranslateExample/checkTranscribeJobStatus.js](https://github.com/brianklaas/awsPlaybox/blob/master/nodejs/lambda/transcribeTranslateExample/checkTranscribeJobStatus.js), and appears below:

{% highlight javascript %}
const util = require('util');
const AWS = require('aws-sdk');
const transcribe = new AWS.TranscribeService;

exports.handler = (event, context, callback) => {
    var jobName = event.jobName;
    
    var params = {
        TranscriptionJobName: jobName
    }
    
    var request = transcribe.getTranscriptionJob(params, function(err, data) {
        if (err) {  // an error occurred
            callback(err, null);
        } else {
            // successful response, return job status
            var returnData = {
                jobName: jobName,
                jobStatus: data.TranscriptionJob.TranscriptionJobStatus,
                transcriptFileUri: data.TranscriptionJob.Transcript.TranscriptFileUri,
                transcriptFileName: jobName
            };
            callback(null, returnData);
        }
    });
    
};
{% endhighlight %}

This Lambda function always returns a set of data about the job &mdash; including the job status &mdash; as long as it does not error out in the process. As you can see, the job name, which we defined in the first step in the workflow (startTranscribeMP4), is critical. Without the job name as created in that first step, we can't check on the job status. That's why the jobName is always passed in every step in the workflow.

### Retrieving the Transcript File And Moving On

The getTranscriptionJob function also returns the URI of the transcript output file. Before we can start the second half of this second example workflow, we need to grab that file and store it for future use. There's are many potential uses for the full transcription of the video. Rather than getting it from the default, AWS-controlled bucket into which the transcription goes upon completion, we'll put it in a bucket of our own. (If you don't know why the transcription ends up in an AWS-controlled bucket, please see my eariler series on [working with AWS Transcribe](http://brianklaas.net/aws/coldfusion/2018/10/05/Using-AWS-Transcribe-in-CFML-Part-4.html).)

The code in [nodejs/lambda/transcribeTranslateExample/getTranscriptionFile.js](https://github.com/brianklaas/awsPlaybox/blob/master/nodejs/lambda/transcribeTranslateExample/getTranscriptionFile.js) shows how to do this. It's basically just reading the file from the AWS-controlled S3 bucket, reading the transcription file into memory, and then writing that file to our own S3 bucket. 

This completes the first half of this workflow, which was dominated by the wait loop. The second half of the worfklow focuses on parallel tasks, and, obviously, introduces the parallel task state. That's what we'll look at next.