---
layout: post
title:  "Using AWS Step Functions in CFML: Translating Text into Multiple Languages"
date:   2019-06-14 15:51:00 -0400
categories: AWS ColdFusion
---

In the last post in this series on using Step Functions from CFML, I introduced the last of the different Step Functions states: the parallel state. We're now tracing the execution of a single parallel state in our workflow &mdash; the translation and speaking of the audio from the source video into Spanish.

The first state in this parallel workflow is makeSpanishStart. Here's the definition of this state:

{% highlight json %}
"makeSpanishStart": {
    "Type": "Pass",
    "Result": "es",
    "ResultPath": "$.languageToUse",
    "Next": "makeSpanishText"
}
{% endhighlight %}

All that's happening in this pass state is setting the $.languageToUse variable to "es," so that the rest of the tasks in this parallel execution will know what language they are targeting.

### The Value of Generic Functions

This state is critical, because without it, we would have to have unique Lambda functions for each target language into which we want to translate and then speak the original transcript. We could have built a "translate into Spanish" Lambda function, and a "translate into French" Lambda function, and a "translate into German" Lambda function, and so on, but that would be wasteful.

As always, it's important to keep your Lambda functions flexible. Instead of making a language-specific translate function, make a generic translate function, and then compose that function into other services (or steps in a Step Functions workflow). That's the strategy employed in this workflow, and you'll see it again when speaking the translated text.

### Translating Text Using AWS Translate

Now that we've set the language to use, the next step in this parallel workflow is the makeSpanishText step. This is where we use AWS Translate to take the original, English-language text from our source video and translate that text into another language (Spanish). Here is the state definition for this step:

{% highlight json %}
"makeSpanishText": {
    "Type": "Task",
    "Resource": "arn:aws:lambda:us-east-1:123456789:function:translateText",
    "Next": "prepSpanishVoiceOutput",
    "Retry": [
        {
            "ErrorEquals": [ "States.ALL" ],
            "IntervalSeconds": 60,
            "MaxAttempts": 3,
            "BackoffRate": 10
        }
    ]
}
{% endhighlight %}

I already covered [AWS Translate in some depth in a previous series](https://brianklaas.net/aws/coldfusion/2018/10/21/Using-AWS-Translate-in-CFML-Part-1.html). It's an easy to use service, but you have to be aware of its [service limits](https://docs.aws.amazon.com/translate/latest/dg/limits-guidelines.html), especially when trying to translate large amounts of text.

Those service limits are handled by the Retry block in the state definition, above. It's a whole lot easier to let the Step Functions execution environment to handle retries and backoff than trying to write all of that in my Lambda function code!

The [code for the translateText Lambda function](https://github.com/brianklaas/awsPlaybox/blob/master/nodejs/lambda/transcribeTranslateExample/translateText.js) starts with retrieval of the translated, English text file which we stored in our own S3 bucket earlier in this series. Once the Promise for that function is resolved, we then go on to the translateText function, which is shown below:

{% highlight javascript %}
function translateText(textToTranslate, languageToUse) {
    return new Promise(function(resolve, reject) {
        // Translate has a current maximum length for translation of 5000 bytes
        var maxLength = 4500;
        var trimmedString = textToTranslate.substr(0, maxLength);
        // We don't want to pass in words that are cut off in the middle
        trimmedString = trimmedString.substr(0, Math.min(trimmedString.length, trimmedString.lastIndexOf(" ")));
        console.log("We're going to translate:\n", trimmedString);
        var params = {
            SourceLanguageCode: 'en',
            TargetLanguageCode: languageToUse,
            Text: trimmedString
        };
        Translate.translateText(params, function(err, data) {
            if (err) {
                console.log("Error calling Translate");
                reject(Error(err));
            } else {
                console.log("Successful translation:\n");
                console.log(data);
                resolve(data);
            }
        });
    });
}
{% endhighlight %}

This function begins by wrapping execution in a Promise, which is always a good idea given the asynchronous nature of both Node.js and calling other services in AWS. It might take 2 miliseconds for Translate to do its job, or it might take 3200 miliseconds. We never quite know.

As noted in the inline comment, Translate has a service limit of 5,000 bytes per translate call. A simple implementation of code that handles this limit can be seen above: just cut off the input text at the last possible word break in the input text that won't go over the 5,000 bytes limit. A more robust implementation of this function would break the incoming text into a series of chunks, translating each, and then sending the complete translation back to the caller.

> It's important to note that AWS Translate does look at the context of each word when generating a translation. Arbitrarty chunking of text strings based solely on string length could result in completely incorrect translations, especially at the start of any chunk after the first chunk. String splitting removes some of the semantic context which makes sentences understandable to us as humans, and to the AWS Translate service. 

We then tell the Translate service object to translate the text using the specified input parameters. If there's an error, it will bubble up back to our Step Functions execution environment, which will then retry the translateText function up to three times. If there are three failed executions, the Step Functions execution itself will error out. Otherwise, the Translate.translateText() result object is returned to our handler function.

The result of translateText() is added to the data being returned to the Step Functions execution environment in our main handler:

{% highlight javascript %}
var returnData = {
    translatedText: translatedTextObj.TranslatedText,
    languageOfText: languageToUse,
    sourceTranscriptFileName: sourceTranscriptFileName
}
callback(null, returnData);
{% endhighlight %}

We now have text that is translated into our target language (Spanish, in this parallel execution). We're ready to speak that text into a MP3 file for our audience to consume. There's a catch, though. If we pass the data in the current execution to our "convert text to speech" Lambda function right now, it won't work. We'll find out why in the next post.