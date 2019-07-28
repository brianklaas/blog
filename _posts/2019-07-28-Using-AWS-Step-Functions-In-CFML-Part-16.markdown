---
layout: post
title:  "Using AWS Step Functions in CFML: Wrapping Up the Transcribe, Translate, and Speak Workflow"
date:   2019-07-28 15:45:00 -0400
categories: AWS ColdFusion
---

After fifteen posts, this is the final post in this series on using Step Functions (and a whole bunch of other AWS services) from CFML applications. The [first workflow](https://brianklaas.net/aws/coldfusion/2019/04/25/Using-AWS-Step-Functions-In-CFML-Part-2.html) picked a random image and described the content of the image via Rekognition, a computer vision service. The [second workflow](https://brianklaas.net/aws/coldfusion/2019/05/24/Using-AWS-Step-Functions-In-CFML-Part-9.html), which we finish now, transcribed a video, translated the transcription into a different language, and turned that translation into a MP3 file.

We now need to get the output of the transcribe, translate, and speak workflow.

## Invoking and Tracking a Step Functions Workflow from CFML

This was covered when we looked at [getting the output of the first example workflow](https://brianklaas.net/aws/coldfusion/2019/05/20/Using-AWS-Step-Functions-In-CFML-Part-8.html), but the process of kicking off a Step Functions workflow and getting the output of that workflow is the same for this, more complicated, workflow as well.

In order to kick off a Step Functions workflow execution, we follow the typical request/reponse object pattern found throughout the [AWS Java SDK](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html): you create a request object of some type, send it to the service, and get a response object back from the service. The relevant code can be found in [stepFunctions.cfm](https://github.com/brianklaas/awsPlaybox/blob/master/stepFunctions.cfm) in the AWSPlaybox demo app. 

Here again, the Step Functions job name must be unique per invocation. I highly recommend haivng some unique identifier in the job name which also is a unique ID in your CFML application. This makes correlating between state data in your CFML applcation and your Step Functions workflows much easier. It also significantly speeds forensic analysis when Step Functions workflows fail for some random reason in production. Knowing that the PDF request for userID 12345 failed and being able to search for userID 12345 in your Step Functions job names is a huge time saver when debugging failures.

Tracking the progress of a Step Functions workflow is the same as before as well. Your “start execution” invocation returns immediately with an object containing information you can use to check on the status of that particular Step Functions execution in the future. The ARN (Amazon Resource Name &mdash; the unique identifier of that *thing* in all of AWS) of the Step Function execution is what you need to periodically check for completion. Again, the relevant code can be found in [stepFunctions.cfm](https://github.com/brianklaas/awsPlaybox/blob/master/stepFunctions.cfm) in the AWSPlaybox demo app. 

You can check on the status of any individual Step Functions execution by running a DescribeExecutionRequest request against the ARN you've stored in memory (or, in production, in a database/persistent data store). If the status property of the DescribeExecutionRequest object is "SUCCEEDED," you can look for the output of the workflow. If the status property is "IN PROGRESS," you have to wait and check on the status again in the future.

All of this was covered in detail in the post on [getting the output of the first example workflow](https://brianklaas.net/aws/coldfusion/2019/05/20/Using-AWS-Step-Functions-In-CFML-Part-8.html) if you need to see this described in detail.

## Getting the Final MP3 Output

Unlike the first example workflow, this workflow doesn't return any data to the invoker. Although there is data being passed from state to state in the workflow execution, that data doesn't really matter to our calling CFML applicaiton. All that matters is the MP3 output of the spoken, translated content.

The last Lambda function executed in our workflow &mdash; [convertTextToSpeech](https://github.com/brianklaas/awsPlaybox/blob/master/nodejs/lambda/transcribeTranslateExample/convertTextToSpeech.js) &mdash; writes the final MP3 file to a destination that's hard-coded. In this case, we write the output to the exact same folder where our input MP4 file exists. Each of the three workflow output files &mdash; Spanish, French, German &mdash; goes into the same folder, and the file name of each file is the job name with -es, -fr, or -de appended, as appropriate. You can download a sample [MP3 file of the Spanish output of this workflow](https://awsplayboxbucket.s3.us-east-1.amazonaws.com/audio/confDemo-1554925168467-es.mp3) if you'd like to hear what the output sounds like.

Our CFML application may want to persist the fact that the workflow is complete for the given source file, and make the individual MP3 files available to users. The CFML application could even move the final MP3 files into a different bucket for public consumption or for global replication via AWS' content delivery network, [CloudFront](https://aws.amazon.com/cloudfront/). Our CFML application could even take the JSON output of this workflow (available in the Output property of the DescribeActivityResult property of the DescribeExecutionRequest object), and store information locally about when the workflow completed, or more.

## Use Step Functions!

I hope that this in-depth series has shown you just how powerful Step Functions workflows can be. They can add entirely new classes of functionality to your CFML applications (or any application written in any language), and can do so with powerful branching and retry flow controls built in. Step Functions have transformed the way we think about asynchronous workflows in our application development process. It's likely they'll do the same for you.