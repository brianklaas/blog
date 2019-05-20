---
layout: post
title:  "Using AWS Step Functions in CFML: Error Handling and Retries in Step Functions Workflows"
date:   2019-05-31 15:51:00 -0400
categories: AWS ColdFusion
---

Our [second example Step Functions workflow](https://github.com/brianklaas/awsPlaybox/blob/master/stateMachines/transcribeTranslateSpeakWorkflow.json) kicks off with a task that uses AWS Transcribe to "listen" to the audio content of a movie and transcribe that content to a text file. If you're not familiar with how Transcribe works, please review [my brief series on the Transcribe service](https://brianklaas.net/aws/coldfusion/2018/09/14/Using-AWS-Transcribe-in-CFML-Part-1.html).

Here's the definition of the task state:

{% highlight json %}
"startTranscribeMP4": {
    "Type": "Task",
    "Resource": "arn:aws:lambda:us-east-1:123456789:function:startTranscribeJob",
    "Next": "WaitForTranscriptionComplete",
    "Retry": [
        {
          "ErrorEquals": [ "States.ALL" ],
          "IntervalSeconds": 30,
          "MaxAttempts": 3,
          "BackoffRate": 10
        }
    ]
}
{% endhighlight %}

Task states have been explained in detail [earlier in this series](https://brianklaas.net/aws/coldfusion/2019/04/25/Using-AWS-Step-Functions-In-CFML-Part-2.html). The startTranscribeJob Lambda function is written for Node.js, and the code for this function is below:

{% highlight javascript %}
const util = require('util');
const AWS = require('aws-sdk');
const transcribe = new AWS.TranscribeService;

exports.handler = (event, context, callback) => {
    var jobName = 'translateTranscribeSpeakWorkflow-' + Date.now();
    var srcFile = event.urlOfFileOnS3;
    var mediaType = event.mediaType;
    var returnData = {
        jobName: jobName
    }
    
    var params = {
        LanguageCode: 'en-US',
        Media: {
            MediaFileUri: srcFile
        },
        MediaFormat: mediaType,
        TranscriptionJobName: jobName
    }
    
    transcribe.startTranscriptionJob(params, function(err, data) {
        if (err) {
            console.log(err, err.stack);
            callback(err, null)
        } else {
            console.log(data);           // successful response
            callback(null, returnData);
        }
    });
    
};
{% endhighlight %}

The code should be fairly straightforward, but there are a few details that bear explanation:

- The job name is critical when working with Transcribe. It must be unique across all Transcribe service reuqests in your AWS account. That's why we append the current date and time when creating the job name.

- The urlOfFileOnS3 and mediaType values are passed in to the Step Functions workflow when the workflow is invoked. They form the initial set of input variables (on the [InputPath](https://brianklaas.net/aws/coldfusion/2019/04/27/Using-AWS-Step-Functions-In-CFML-Part-3.html)) to the first state in the workflow. Later in the series, I will show you how to pass these values into your Step Functions workflow invocation.

- I know that the soruce language of all of my videos is US English, so I hardcoded that here. If you have source videos in multiple languages, you'd need to pass this value in as a variable.

- I'm not using Promises when telling the Transcribe service to startTranscriptionJob(). Transcription is a fully asynchonous process, and I don't know how long it will take for any given transcription job to complete. As such, the Transcribe service immediately returns some data about the job now in its queue, and nothing more. I grab that data and pass it back to the Step Functions execution envrionment to continue through the workflow.

### Error Handling and Retries in Step Functions Workflows

The new and interesting part of the startTranscribeMP4 task definition code is the Retry block:

{% highlight json %}
"Retry": [
    {
        "ErrorEquals": [ "States.ALL" ],
        "IntervalSeconds": 30,
        "MaxAttempts": 3,
        "BackoffRate": 10
    }
]
{% endhighlight %}

Retries are a powerful construct in Step Functions workflows, and should be included in almost every task state definition that you write. But why is that?

Nearly all AWS services have limits on one or more aspects of the service. Lambda, for example, has a [limit of 1,000 concurrent Lambda executions](https://docs.aws.amazon.com/lambda/latest/dg/limits.html) in any AWS account. Any single invocation also cannot run for longer than 15 minutes. Transcribe has a [limit of 10 invocations per second on the StartTranscriptionJob function](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#limits-amazon-transcribe) and an overall limit of 100 concurrent transcription jobs per account. Translate will not translate requests that are [longer than 5,000 bytes in size](https://docs.aws.amazon.com/translate/latest/dg/limits-guidelines.html). Polly's SynthesizeSpeech operation has a [limit of 3000 characters per operation](https://docs.aws.amazon.com/polly/latest/dg/limits.html).

All of these limits must be handled in any code we write that uses these servcies. This means keeping track of simultaneous executions, or chunking text into separate blocks for translation or speech. If you do happen to reach a service limit, or your requests get throttled by AWS because of internal load on the service, you probably don't want your whole workflow to error out. You want to be able to try things again, in the hopes that service capacity has eased or limits have been reset due to the passage of time.

You could write all of the if-then-else, switching, branching, and tracking code to handle all of this yourself. You could store state in DynamoDB every step of the way to see what specific task failed and where, how many retries you have attempted, how many retries you have left, and so on. You could add hundreds (or thousands) of lines of code to your workflow to handle this.

Or you could just use the Retry feature of task states to do all of this work for you.

The Retry feature takes an array of retry approaches, each uniquely identified by an error string. In the example above (and throughout this example Step Functions workflow), we simply catch the "States.ALL" error (a generic error for any error in the execution of a state), and tell our Step Functions workflow to try running the state again.

Not having to write all that error handling code ourselves is handy enough. Step Functions takes it a "step" further by adding in a simple way to include delay, multiple retries, and backoff in our retries:

- *IntervalSeconds* defines the number of seconds between attempts that Step Functions will wait before retrying the task. In our case, if the Step Functions execution encounters an error when invoking the startTranscribeJob Lambda function, it will wait 30 seconds before trying again.

- *MaxAttempts* defines the number of attempts Step Functions will make retrying the startTranscribeJob Lambda function before it (and the entire workflow) errors out.

- *BackoffRate* is the number of seconds Step Functions will add to each IntervalSeconds on each subsequent retry beyond the first. This means that Step Functions will wait 30 seconds before the first retry, 40 seconds before the second, and 50 seconds before the third. Including service call backoff (and, specifically, [exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff)) is a best practice when designing resilient systems.

Instead of you writing hundreds of lines of codes to handle all of the above, Step Functions allows for all of this functionality in three short lines of JSON. 

You should almost always include a Retry block in every task definition in your Step Functions workflow. You never know when a single Lambda function execution may hang, or when other AWS services (or third-party APIs) may throttle your invocations. There are certainly situations where a Retry block isn't neccessary, but they are not very common. Including a Retry block in every task definition makes your workflow more resilient and more likely to end at a success state. If you have workflows that take hours to complete, Retry blocks are hugely helpful in not wasting time, compute power, generated data.

Our second example Step Functions workflow has now started the Transcribe job, and we've got to wait for that job to finish before we can proceed through the rest of the workflow. How do we do that, though? How do we handle waiting around for tasks that may take minutes, hours, or days to complete? In the next post, we'll look at the solution to this problem.