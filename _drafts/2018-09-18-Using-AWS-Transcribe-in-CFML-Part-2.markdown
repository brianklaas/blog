---
layout: post
title:  "Using AWS Transcribe in CFML: Starting a Transcribe Job
date:   2018-09-18 13:35:00 -0400
categories: AWS ColdFusion
---
There are three distinct phases in working with [AWS Transcribe](https://aws.amazon.com/transcribe/): starting a job, waiting for the job to complete, and parsing the results of the job. This is because Transcribe is a fully asynchronous service. Transcribe cannot transcribe an hour long video in miliseconds. It takes time to do the work.

In this post, I'll show you how to start a Transcribe job and explore some of the options available to you.

As always when working with an AWS service via the AWS Java SDK, there's a basic pattern that you follow:

1. Get a copy of the client that's making a connection to the service you want to use.
2. Create a "request" object.
3. Fill the "request" object with the parameters (or other objects) you need to supply.
4. Tell the client to make the request.
5. Get back a "response" object.

As I wrote in the series on using Simple Notification Service (SNS) from CFML, the [JavaDocs for the AWS Java SDK](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html) are comprehensive and always up-to-date. They are, alas, also just JavaDocs. You're not going to find detailed examples of how to complete full tasks in these docs. If you look at the [documentation for the com.amazonaws.services.rekognition.AmazonRekognitionClient](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/rekognition/AmazonRekognitionClient.html), you can see all the things that you can do with Rekognition via the Java SDK.

Here are the steps to starting a Transcribe job via the AWS Java SDK:

1. Get a copy of [the Transcribe client we created in the first part of this series](/aws/coldfusion/2018/07/23/Using-AWS-Transcribe-In-CFML-Part-1.html).
2. Create a "S3OMediabject" object that specifies the path to the media file on S3 that you want to transcribe.
3. Create a Transcribe job Settings object that details any custom settings that you want to use.
4. Create a "StartTranscriptionJobRequest" object.
5. Give the job a unique name.
6. Set the S3OMediabject into the StartTranscriptionJobRequest object.
7. Set the Settings object and all other required settings into the StartTranscriptionJobRequest.
8. Run the StartTranscriptionJobRequest.
9. Get back a startTranscriptionJobResult object.

If you've read the other posts in this larger series on using AWS from CFML, you'll notice step 7 in the list above is a bit vague. I'll explain why below.

## The Code to Start a Transcribe Job

Here's how we do this in the [AWSPlaybox app](https://github.com/brianklaas/awsPlaybox):

> If you haven't already read the entry on [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html), please do so now.

As there are three basic actions when working with Transcribe job, I've broken out each of those into three separate code blocks in /transcribe.cfm. The first, containing the code to start a Transcribe job, starts with:

{% highlight html %}
<cfif structKeyExists(FORM, "pathToFileOnS3")>
{% endhighlight %}

If you fill out the form that is rendered by default when /transcribe.cfm loads, and enter a valid path, this code block will execute. The code does a basic check to make sure that the path points to either a MP3 or MP4 file:

{% highlight javascript %}
fileExtension = listLast(Trim(FORM.pathToFileOnS3), ".");
  if (NOT listFindNoCase("MP3,MP4", fileExtension)) {
    writeOutput('Unsupported fileExtension: ' & fileExtension);
    abort;
}
{% endhighlight %}

> Transcribe supports the following file formats for processing: MP3, MP4, WAV, and FLAC (see [com.amazonaws.services.transcribe.model.MediaFormat](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/transcribe/model/MediaFormat.html)). 

If you continue on in this code block, you'll see the nine steps listed above translated into code, with a few detours:

{% highlight javascript %}
// 1. Get a copy of the Transcribe client
transcribeService = application.awsServiceFactory.createServiceObject('transcribe');
// 2. Create a "S3OMediabject" object that specifies the path to the media file on S3 that you want to transcribe.
// Wouldn't it be nice if there were greater internal consistency in the AWS Java SDK around objects in S3?
s3MediaObject = CreateObject('java', 'com.amazonaws.services.transcribe.model.Media').init();
s3MediaObject.setMediaFileUri(Trim(FORM.pathToFileOnS3));
// 3. Create a Transcribe job Settings object that details any custom settings that you want to use.
// Read on below for more about these settings.
transcribeJobSettings = CreateObject('java', 'com.amazonaws.services.transcribe.model.Settings').init();
transcribeJobSettings.setShowSpeakerLabels(true);
transcribeJobSettings.setMaxSpeakerLabels(5);
// 4. Create a "StartTranscriptionJobRequest" object.
startTranscriptionJobRequest = CreateObject('java', 'com.amazonaws.services.transcribe.model.StartTranscriptionJobRequest').init();
// 5. The unique identifier for any job in Transcribe is a per-account unique string
jobName = "AWSPlayboxTranscribeJob" & DateDiff("s", DateConvert("utc2Local", "January 1 1970 00:00"), now());
startTranscriptionJobRequest.setTranscriptionJobName(jobName);
// 6. Set the S3OMediabject into the StartTranscriptionJobRequest object.
startTranscriptionJobRequest.setMedia(s3MediaObject);
// 7. Set the Settings object and all other required settings into the StartTranscriptionJobRequest.
startTranscriptionJobRequest.setMediaFormat(fileExtension);
startTranscriptionJobRequest.setSettings(transcribeJobSettings);
startTranscriptionJobRequest.setLanguageCode('en-US');
// 8. Run the StartTranscriptionJobRequest, and 9. Get back a startTranscriptionJobResult object.
startTranscriptionJobResult = transcribeService.startTranscriptionJob(startTranscriptionJobRequest);
transcriptionJob = startTranscriptionJobResult.getTranscriptionJob();
{% endhighlight %}

The code above should be fairly self-explanatory with the comments, but I do want to talk about some of the options available to you when you start a Transcribe job.

First, you need to set the language of the source media file. Currently, Transcribe supports US English (en-us) and US Spanish (es-US). Given that Transcribe uses the same natural language AI used by Alexa, I expect we'll see other languages supported in the future.

Next, the Transcribe Settings object contains a lot of options which you can enable depending on your specific transcription needs. In this case, I have enabled both the speaker labels setting and set the maximum number of identified speakers to five:

{% highlight javascript %}
transcribeJobSettings.setShowSpeakerLabels(true);
transcribeJobSettings.setMaxSpeakerLabels(5);
{% endhighlight %}

Enabling "show speaker labels" has the Transcribe engine do its best to identify every unique voice in the source media object. Transcribe obviously doesn't know that the first person speaking is Jill, the second Tom, and so on. Instead, it identifies each speaker with a number. It's up to you to know what to do with that information. Enabling speaker labels is optional. I enable it here simply to help you understand how to work with the Settings object in Transcribe.

Setting the maximum number of speaker labels is only useful if you enable speaker labels in the first place. You'd want to set this to a number that's appropriate for your source media. You could use this to focus on the two or three people who are the primary speakers in your source media.

Additional options in the Settings object involve identifying the channel in your multi-channel audio where your speaker's audio came from, and using custom vocabularies. Custom vocabularies can be very powerful in helping the Transcribe engine recognize highly domain-specific words, such as medical terminology.

## The Destination of Your Transcribe Job

One option that I did not set in this example is the very important OutputBucketName. This is a setting added to the service in June, 2018.

By default, Transcribe will put the output of the Transcribe job into a bucket owned by AWS itself. You have no direct access to this bucket, except through a special, time-sensitive URL given to you when a Transcribe job completes. I'll explore this more in the next post.

If you want Transcribe to put the output of the Transcribe job into a S3 bucket that you own, you can do that. You need to call setOutputBucketName() on the startTranscriptionJobRequest object and pass in the name of the S3 bucket in your account that you want to use. Note that if you choose to go this route, the IAM user that is making the request to Transcribe to start the transcription job must also have permission to access this S3 bucket, and Transcribe itself needs to have permission to access this S3 bucket.

## The Importance of the Job Name

Although it seems like a throwaway requirement in starting a Transcribe job, the job name is very important. Once you start a Transcribe job, the only way you can get information about the job or the result of the job is by using the job name that you created. As such, it's important that job names are unique (unique per AWS account, not in all of AWS), and meaningful in some way to you.

When you start a Transcribe job, you will get a [StartTranscriptionJobResult object](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/transcribe/model/StartTranscriptionJobResult.html) back. Even so, that job name is what you need to do any future work with your Transcribe job.

In the AWSPlaybox app, I take the job name and the start time and add those to an application-scoped array of current Transcribe jobs:

{% highlight javascript %}
jobInfo = { 
	jobName=jobName, 
	timeStarted=Now() 
};
arrayAppend(application.currentTranscribeJobs, jobInfo);
{% endhighlight %}

In a real, production application, you would want to store this information in a database or some other persistent storage mechanism. It can take minutes or hours for a Transcribe job to complete, and you're going to need to check for the job to see if it's done on a regular basis. That's exactly what we'll do in the next post.