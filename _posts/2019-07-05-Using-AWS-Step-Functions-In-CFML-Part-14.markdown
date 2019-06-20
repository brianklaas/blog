---
layout: post
title:  "Using AWS Step Functions in CFML: The Frustration of Variable Reassignment in Step Functions"
date:   2019-07-05 15:51:00 -0400
categories: AWS ColdFusion
---

As mentioned at the end of the last post in this series, if we try to take our translated text and pass it directly into the next (and final) step in this parallel execution, it won't quite work. We would get an error that the textToSpeak property wasn't defined in the event object. Why?

I'm not going to show the code for the convertTextToSpeech.js function quite yet, but it expects three properties to exist in the event argument passed to that Lambda function:

- textToSpeak
- languageToSpeak
- transcriptFileName

However, at this point in our parallel Step Function execution, we have the following variables being passed from state to state:

- translatedText
- languageOfText
- sourceTranscriptFileName

It's pretty clear how these different variable names match up, so why can't we just say that textToSpeak = translatedText in our Step Functions workflow?

Because we can't.

### The Lack of Variable Reassignment in Step Functions Workflows

There are, to me, two major pain points in Step Functions workflows as of the writing of this post:

1. Inability to dynamically loop over arrays (or any iterable object)
2. Inability to reassign variable names in workflows

You simply can't say foo = bar in your Step Function workflow definitions, not even in Pass states. You have to create a separate Lambda function (an [adapter, as some people have called it](https://medium.com/ki-labs-engineering/chaining-serverless-functions-for-stateful-workflows-aws-step-functions-using-adapter-pattern-9edb0db7b35e)) to reassign the variable names. That's what the prepSpanishVoiceOutput step is for:

{% highlight json %}
"prepSpanishVoiceOutput": {
    "Type": "Task",
    "Resource": "arn:aws:lambda:us-east-1:123456789:function:prepTranslatedTextForSpeech",
    "Next": "makeSpanishAudioFile"
}
{% endhighlight %}

The Node.js code for the prepTranslatedTextForSpeech Lambda function is below:

{% highlight javascript %}
exports.handler = (event, context, callback) => {
    
    var textToSpeak = event.translatedText;
    var languageOfText = event.languageOfText;
    var transcriptFileName = event.sourceTranscriptFileName;
    
    var returnData = {
        textToSpeak: textToSpeak,
        languageOfText: languageOfText,
        transcriptFileName: transcriptFileName
    }
    
    callback(null, returnData);
};
{% endhighlight %}

Again, all we are doing in this function is reassigning variable names because we can't do that in our Step Functions workflow JSON.

You may be asking: "why not just name the translatedText property of the translateText function textToSpeak? You could have avoided this whole intermediate step." I hope you can see from that very question the issue with naming the property textToSpeak. We're not *speaking* text in that function, we're *translating* text. I might want to use the translateText function for any number of other purposes. Having a property of the translateText output object named "textToSpeak" simply wouldn't make logical sense to other developers using that function. By keeping the input variables appropriate to the purpose of the function, I create something that is re-usable by many Step Functions workflows or other applications.

So now we have our translated text, properly assigned to the expected variable names for the next, and final step in our workflow. The next post will cover the process of taking that text and "speaking" it into an MP3 file.