---
layout: post
title:  "Using AWS Transcribe in CFML: Checking for Job Completion"
date:   2018-09-28 08:35:00 -0400
categories: AWS ColdFusion
---
We've [started a Transcribe job](https://brianklaas.net/aws/coldfusion/2018/09/18/Using-AWS-Transcribe-in-CFML-Part-2.html), now we need to see if the job is done. Jobs in Transcribe can take seconds, or even hours, depending on the length of the source media. Jobs are never immediately completed. As such, you'll need to approach this in an asyncrhonous manner.

Given that ColdFusion lacks the niceties of Promises in a langugae like JavaScript, and ColdFusion only recently added [support for Java Futures in ColdFusion 2018](https://coldfusion.adobe.com/2018/07/asynchronous-programming-in-coldfusion-2018-release/), we need to think of ColdFusion as the executor and checker of asynchronous jobs in AWS. In [the previous post](https://brianklaas.net/aws/coldfusion/2018/09/18/Using-AWS-Transcribe-in-CFML-Part-2.html), we started a job, and put a reference to that job name in the application scope. (In a real production applicaiton, you'd want to save the job name to a database or other method of persistence.)

Now we can check on the status of the job.

## The Code to Retrieve Job Status for a Transcribe Job

We'll again follow our basic pattern when working with the AWS Java SDK:

1. Get a copy of the client that's making a connection to the service you want to use.
2. Create a "request" object.
3. Fill the "request" object with the parameters (or other objects) you need to supply.
4. Tell the client to make the request.
5. Get back a "response" object.

Here are the steps to checking the status of a Transcribe job via the AWS Java SDK:

1. Get a copy of [the Transcribe client we created in the first part of this series](/aws/coldfusion/2018/07/23/Using-AWS-Transcribe-In-CFML-Part-1.html).
2. Create a "GetTranscriptionJobRequest" object.
3. Set the job name in the GetTranscriptionJobRequest object.
4. Run the GetTranscriptionJobRequest.
5. Get back a GetTranscriptionJobResult object.

Here's how we do this in the [AWSPlaybox app](https://github.com/brianklaas/awsPlaybox):

> If you haven't already read the entry on [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html), please do so now.

As there are three basic actions when working with Transcribe job, I've broken out each of those into three separate code blocks in /transcribe.cfm. The second, containing the code to check on a Transcribe job, starts with:

{% highlight html %}
<cfif structKeyExists(URL, "checkTranscribeJob")>
{% endhighlight %}

Note that you can only pass a URL.checkTranscribeJob value if you've already started a Transcribe job in the AWSPlaybox application.

If you continue on in this code block, you'll see the five steps listed above translated into code:

{% highlight javascript %}
// 1. Get a copy of the Transcribe client
transcribeService = application.awsServiceFactory.createServiceObject('transcribe');
// 2. Create a "GetTranscriptionJobRequest" object.
getTranscriptionJobRequest = CreateObject('java', 'com.amazonaws.services.transcribe.model.GetTranscriptionJobRequest').init();
// 3. Set the job name in the GetTranscriptionJobRequest object.
getTranscriptionJobRequest.setTranscriptionJobName(URL.checkTranscribeJob);
// 4. Run the GetTranscriptionJobRequest and 5. get back a GetTranscriptionJobResult object.
getTranscriptionJobResult = transcribeService.getTranscriptionJob(getTranscriptionJobRequest);
{% endhighlight %}

## Processing the GetTranscriptionJobResult

The GetTranscriptionJobResult contains a property called, confusingly, TranscriptionJob. It's this object that has the actual status information about the job in question. Key properties of this object are the TranscriptioJobName, CreationTime,  MediaFileUri, and Status.

Here's the appropriate code from /transcribe.cfm:

{% highlight javascript %}
transcriptJob = getTranscriptionJobResult.getTranscriptionJob();
transcribeJobResult.status = transcriptJob.getTranscriptionJobStatus();
transcribeJobResult.jobName = transcriptJob.getTranscriptionJobName();
transcribeJobResult.createdOn = transcriptJob.getCreationTime();
transcribeJobResult.sourceMediaUri = transcriptJob.getMedia().getMediaFileUri();
{% endhighlight %}

The getStatus() method of the TranscriptionJob object can return one of three values: COMPLETED, FAILED, and IN_PROGRESS. If the job is currently in progress, there's nothing we can do but wait around and make another GetTranscriptionJobRequest at a later time.

If the status is COMPLETED, we can get more information, including the job completion time, and, most critically, the URI of the transcript itself:

{% highlight javascript %}
if (transcribeJobResult.status IS "COMPLETED") {
  transcribeJobResult.finishedOn = transcriptJob.getCompletionTime();
  // The URI location of the transcription output is in a Transcript object returned from the getTranscript method.
  transcribeJobResult.transcriptUri = transcriptJob.getTranscript().getTranscriptFileUri();
  deleteJob = 1;
{% endhighlight %}

Note the setting of the deleteJob flag. This is only done to remove completed jobs from our list of currently running jobs in the application. It doesn't do anything to the actual job on AWS.

If the status is FAILED, Transcribe does give us a failure reason. If the job has failed, we should also remove it from our list of currently running jobs in the application.

{% highlight javascript %}
else if (transcribeJobResult.status IS "FAILED") {
  transcribeJobResult.failureReason = transcriptJob.getFailureReason();
  deleteJob = 1;
{% endhighlight %}

Finally, we output data on the completed (or failed) job:

{% highlight html %}
<p>Result of Transcribe job check:</p>
<cfoutput>
  <p>Job Name: #transcribeJobResult.jobName#</p>
  <p>Source Media: #transcribeJobResult.sourceMediaUri#</p>
  <p>Created On: #DateTimeFormat(transcribeJobResult.createdOn, "long")#</p>
  <p>Status: <strong>#transcribeJobResult.status#</strong></p>
  <cfif (transcribeJobResult.status IS "COMPLETED")>
    <p>Finished On: #DateTimeFormat(transcribeJobResult.finishedOn, "long")#</p>
    <p>Transcript file location: #transcribeJobResult.transcriptUri#</p>
    <p><a href="#transcribeJobResult.transcriptUri#" download>Download the full transcript job output</a>
    <br/><br/><a href="transcribe.cfm?getTranscriptText=#transcribeJobResult.jobName#" download>Download just the text transcript</a>
    <br/><br/><small>Note: these links are only valid until #DateTimeFormat(DateAdd('n',5,now()), 'short')#</small></p>
  <cfelseif (transcribeJobResult.status IS "FAILED")>
    <p>Reason for failure: #transcribeJobResult.failureReason#</p>
  </cfif>
</cfoutput>
{% endhighlight %}

If the job is completed, we construct a URL pointing directly to the full transcript job output &mdash; and give a warning about the five minute expiry of that URL. Why is there a time limit on that URL? We'll cover in the next blog post. 