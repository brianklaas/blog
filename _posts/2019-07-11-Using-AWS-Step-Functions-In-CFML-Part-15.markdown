---
layout: post
title:  "Using AWS Step Functions in CFML: Speaking Translated Text into a MP3 File"
date:   2019-07-11 15:51:00 -0400
categories: AWS ColdFusion
---

We are now at the final step in this parallel execution inside our Step Functions workflow that transcribes a video, takes the transcript and translates it into a different langugage (Spanish, in this parallel execution), and speaks that translated text into a MP3 file.

This final step, titled "makeSpanishAudioFile" in our workflow, uses another AWS service that I haven't yet covered in any of my many posts on using AWS from CFML: [AWS Polly](https://aws.amazon.com/polly/).

### Hello, Polly

AWS Polly is, at heart, the engine that powers Alexa's ability to speak to us. When you ask Alexa to do something, and Alexa responds, it's Polly that is working under the hood to generate the audio output. Polly is an enormously powerful text-to-speech service. [Polly can curently speak to 29 languages and language variants](https://docs.aws.amazon.com/polly/latest/dg/voicelist.html). Most of those languages have one or more male and female voices to choose from. Polly also uses the [SSML markup language](https://developer.amazon.com/docs/custom-skills/speech-synthesis-markup-language-ssml-reference.html) to provide more natural-sounding speech through techniques like emphasis, pauses, whispers, and more. The code for this task in our Step Functions workflow does use different voices in each language, but does not cover SSML.

> Auto-generating SSML markup for translated text from a random source would be a very difficult task.

### Speaking Our Translated Text

The makeSpanishAudioFile state in our parallel execution is defined as follows:

{% highlight json %}
"makeSpanishAudioFile": {
    "Type": "Task",
    "Resource": "arn:aws:lambda:us-east-1:123456789:function:convertTextToSpeech",
    "Retry": [
        {
            "ErrorEquals": [ "States.ALL" ],
            "IntervalSeconds": 60,
            "MaxAttempts": 3,
            "BackoffRate": 10
        }
    ],
    "End": true
}
{% endhighlight %}

Here again we use a Retry block to deal with the [service limits for AWS Polly](https://docs.aws.amazon.com/polly/latest/dg/limits.html). 

More important is the "End":true attribute at the end of this task definition. As mentioned previously in this series, the "End":true attribute in a parralel execution tells the Step Functions execution environment that this task is the last step in a parallel execution. Once the last step is reached in a parallel execution, the Step Functions execution environment considers that parallel branch "done," and waits for all other parallel branches to complete before moving to the next step in the larger Step Functions workflow.

The [code for the convertTextToSpeech function](https://github.com/brianklaas/awsPlaybox/blob/master/nodejs/lambda/transcribeTranslateExample/convertTextToSpeech.js) first makes a MP3 from the incoming text input, and then saves that MP3 (currently only in memory) to an actual file on S3. Here's the code for makeMP3FromText, which is the where the magic of Polly happens:

{% highlight javascript %}
function makeMP3FromText(textToSpeak, languageOfText) {
    return new Promise(function(resolve, reject) {
        var voiceToUse = 'Ivy';
        // Polly has a current maximum character length of 3000 characters
        var maxLength = 2900;
        var trimmedText = textToSpeak.substr(0, maxLength);
        switch(languageOfText) {
            case 'es':
                voiceToUse = (Math.random() >= 0.5) ? "Penelope" : "Miguel";
                break;
            case 'fr':
                voiceToUse = (Math.random() >= 0.5) ? "Celine" : "Mathieu";
                break;
            case 'de':
                voiceToUse = (Math.random() >= 0.5) ? "Vicki" : "Hans";
                break;
        }
        var params = {
            OutputFormat: "mp3",
            SampleRate: "8000",
            Text: trimmedText,
            VoiceId: voiceToUse
        }
        var speakPromise = Polly.synthesizeSpeech(params).promise();
        speakPromise.then(function(data) {
            resolve(data);
        }).catch(function(err) {
            console.log("Error generating MP3 file:\n", err);
            reject(Error(err));
        });
    });
}
{% endhighlight %}

As with AWS Translate, we handle the AWS Polly service limit of speaking no more than 3,000 characters in one API call by truncating the incoming text. Again, this a very simple approach to the service limit. A more robust approach would, of course, split the incoming text into chunks, create MP3 files for each chunk, and stitch those chunks together in the final output. (There's a [WordPress plugin powered by AWS Polly that does just this](https://wordpress.org/plugins/amazon-polly/).)

The next part of the makeMP3FromText function is a switch statement that randomly picks a voice (male or female) to use for each of the supported languages. This gives us some variety in our output, which is useful when we test (or demo) this multiple times.

Finally, we set our service call parameters (8000mhz for voice-only a MP3 file is perfectly fine), and then tell the Polly service to synthesizeSpeech. We wrap this call in a Promise because, again, synthesizing the speech is not an instantaneous process. It's awfully fast, especially with a limit of 3,000 characters per API call, but it does take a few seconds.

If there's an error, the Retry block in our makeSpanishAudioFile step definition will catch the error, and retry it up to three times.

> It's important to note that you very well may run in to the service limits on both Translate and Polly while executing this workflow. There are three branches running in parallel, all using the same input text, and making calls to Translate and Polly at roughly the same time (or, easily within a few seconds of each other). Retry blocks are an excellent way of handling these service limits without writing any of the retry code yourself.

Upon successful synthesis of speech, the in-memory version of the MP3 file is then [written to S3](https://github.com/brianklaas/awsPlaybox/blob/master/nodejs/lambda/transcribeTranslateExample/convertTextToSpeech.js), and the Spanish parallel execution is complete.

Whew! 

That is the end of our Step Functions workflow. We've transcribed a video, translated the transcription into a different language, and turned that translation into a MP3 file. In the final post in this long series, we'll look at capturing the output from this workflow in our CFML application.