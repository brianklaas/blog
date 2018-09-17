---
layout: post
title:  "Using AWS Transcribe in CFML: Retrieving a Transcript"
date:   2018-09-26 13:35:00 -0400
categories: AWS ColdFusion
---
Now that our Transcribe job is done, we can retrieve the output of the job. This isn't quite as straightfoward as it seems, and this post will show you how to both grab the full transcript result and just the text transcript itself.

At the end of the last blog post, we saw the output of a transcript job if the status was COMPLETED. I've removed everything else that isn're relevant to outputing information about a completed job:

{% highlight html %}
  <cfif (transcribeJobResult.status IS "COMPLETED")>
    <p>Finished On: #DateTimeFormat(transcribeJobResult.finishedOn, "long")#</p>
    <p>Transcript file location: #transcribeJobResult.transcriptUri#</p>
    <p><a href="#transcribeJobResult.transcriptUri#" download>Download the full transcript job output</a>
    <br/><br/><a href="transcribe.cfm?getTranscriptText=#transcribeJobResult.jobName#" download>Download just the text transcript</a>
    <br/><br/><small>Note: these links are only valid until #DateTimeFormat(DateAdd('n',5,now()), 'short')#</small></p>
{% endhighlight %}

Note the warning at the end: the links that are output are only valid for five minutes from the time the output is created. Why is that?

## Time-Expiring URLs and Transcribe Output

You may recall from [my posts on S3](https://brianklaas.net/aws/coldfusion/2018/05/26/Using-Simple-Storage-Service-in-CFML-Part-2.html) that one of the things you can do with the AWS Java SDK and S3 is create time-expiring URLs. This prevents a user from having unlimited access to a resource on S3. AWS does exactly this with transcription results that go into the default bucket for all of Transcribe. 

As explained previously, unless you specify the OutputBucketName in the StartTranscriptionJobRequest, the results of your Transcribe job go into the default bucket for all of Transcribe. In the AWSPlaybox app, I do not specify the OutputBucketName in the StartTranscriptionJobRequest. Hence the time-expiring URLs on the result.

Your Transcribe job does not get removed from the default Transcribe bucket in S3 once these time-expiring URLs expire. In order to get another time-expiring URL, you need to run another GetTranscriptionJobRequest object using the original job name for the job. Parse the results just as you did in the last blog post, and you'll get your time-expiring URL to the full transcript job output.

## The Full Transcribe Output Object

If you click on the time-expiring link to your Transcribe job, you'll find that you get a JSON object as the result. This is pretty handy for parsing and data retrieval.

At the top level, the full Transcribe job output JSON object contains four properties: accountID, jobName, results, and status. It's the results property that we are interested in.

The results property contains three arrays: transcripts, speaker_labels, and items.

- speaker_labels will only exist if you enable speaker labels in your original StartTranscriptionJobRequest, which we did.
- items (which are individual words in the file) contains an array of structures which include the start and end time of the item (word), the start and end time at which the item (word) occurs, and an array of "alternatives," which are guesses that Transcribe made as to the exact word used and the confidence it has in that guess.
- transcripts contains the full, continuous text transcription of the media file you originally provided.

You have a lot of data to work with in the full Transcribe job output JSON object. You can use the items array to create word-by-word captions for a video, or parse and dump that data into a database and allow people to search for specific words and jump directly to where that word appears in the original media file.

## Getting Just the Full Text Transcription

It's great that we have all this data from Transcribe. What if all you want is the full, continuous text transcription of the media file you originally provided? That's what the third of the three basic actions in /transcribe.cfm in the [AWSPlaybox app](https://github.com/brianklaas/awsPlaybox) handles. The code block we're looking at now begins with:

{% highlight html %}
<cfif structKeyExists(URL, "getTranscriptText")>
{% endhighlight %}

As before, we'll run a GetTranscriptionJobRequest on this job using the original job name, and get the results. We have to do this so that our CFML application can then pull down and parse the results using a time-expiring, secured URL:

{% highlight javascript %}
transcribeService = application.awsServiceFactory.createServiceObject('transcribe');
getTranscriptionJobRequest = CreateObject('java', 'com.amazonaws.services.transcribe.model.GetTranscriptionJobRequest').init();
getTranscriptionJobRequest.setTranscriptionJobName(URL.getTranscriptText);
getTranscriptionJobResult = transcribeService.getTranscriptionJob(getTranscriptionJobRequest);
transcriptJob = getTranscriptionJobResult.getTranscriptionJob();
transcriptUri = transcriptJob.getTranscript().getTranscriptFileUri();
{% endhighlight %}

The previous blog post covered this in detail, so if you can't follow what's going on above, please refer to that blog post.

Now we have the transcriptUri value, which points to the full Transcribe job result object on AWS. We'll use CFML to grab that JSON object:

{% highlight javascript %}
cfhttp(method="GET", charset="utf-8", url="#transcriptUri#", result="transcriptFile");
transcriptFileAsJSON = deserializeJSON(transcriptFile.fileContent, false);
{% endhighlight %}

As mentioned above, at the top level is a results property. Inside that results property is a transcripts property. That's what we want. The transcripts property is an array of properties with the key of "transcript." There's only one transcript in the transcript array, and that contains the full, continuous text transcription of the media file you originally provided.

{% highlight javascript %}
transcriptData = transcriptFileAsJSON.results;
transcriptText = transcriptData.transcripts[1].transcript;
{% endhighlight %}

Finally, we output this as a simple text file to the browser:

{% highlight javascript %}
cfheader(name="Content-Disposition", value="inline; fileName=#URL.getTranscriptText#.txt");
writeDump(transcriptText);
abort;
{% endhighlight %}

Any code that ends in abort isn't pretty, but it does the job. A production application would save the full, continuous text transcription to a database or a file for future use.

That's it for my tour of using Transcribe from within CFML. If you have any questions about any of the posts in this series, feel free to <a href="https://twitter.com/brian_klaas">message me on the Twitter</a>!