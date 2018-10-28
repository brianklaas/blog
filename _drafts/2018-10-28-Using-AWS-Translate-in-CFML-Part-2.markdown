---
layout: post
title:  "Using AWS Translate in CFML: Making a Translation Request and Dealing with Service Limits"
date:   2018-10-28 07:25:00 -0400
categories: AWS ColdFusion
---
[AWS Translate](https://aws.amazon.com/translate/) is a simple service to use from within your CFML or Java application. The service only does one thing: translate text from one language to another.

As always when working with an AWS service via the AWS Java SDK, you follow this pattern:

1. Get a copy of the client that's making a connection to the service you want to use.
2. Create a "request" object.
3. Fill the "request" object with the parameters (or other objects) you need to supply.
4. Tell the client to make the request.
5. Get back a "response" object.

The [JavaDocs for the AWS Java SDK](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html) make it very clear how simple this service is. If you look at the [documentation for the com.amazonaws.services.transcribe.AmazonTranslateClient](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/translate/AmazonTranslateClient.html), you can see all the things that you can do with Translate via the Java SDK.

Here are the steps to translating text via the AWS Java SDK:

1. Get a copy of [the Translate client we created in the first part of this series](/aws/coldfusion/2018/10/21/Using-AWS-Translate-In-CFML-Part-1.html).
2. Create a "TranslateTextRequest" object.
3. Set the source and target language codes and the text you want translated in the TranslateTextRequest object.
4. Tell the Translate service to run the TranslateTextRequest.
5. Get back a translateTextResult object.

## The Code to Run a Translate Job

Here's how we do this in the [AWSPlaybox app](https://github.com/brianklaas/awsPlaybox):

> If you haven't already read the entry on [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html), please do so now.

The code to perform the five steps above, like using the service itself, is straightforward.

The service needs both a target language for translation and some text to translate. We start with a check for both in the code:

{% highlight javascript %}
if (not(len(trim(FORM.targetLanguage))) or not(len(trim(FORM.textToTranslate)))) {
  writeOutput('You need to supply both a target language and text to translate.');
  abort;
}
{% endhighlight %}

Creating the TranslateTextRequest and setting the source and target languages is also simple:

{% highlight javascript %}
translateService = application.awsServiceFactory.createServiceObject('translate');
translateJobRequest = CreateObject('java', 'com.amazonaws.services.translate.model.TranslateTextRequest').init();
// In the AWSPlaybox demo, we assume that the input language is always English.
translateJobRequest.setSourceLanguageCode('en');
translateJobRequest.setTargetLanguageCode(trim(FORM.targetLanguage));
{% endhighlight %}

Making the translation request is also really simple:

{% highlight javascript %}
translateJobRequest.setText(chunkToTranslate);
translateTextResult = translateService.translateText(translateJobRequest);
{% endhighlight %}

If you look at the code in translate.cfm in the [AWSPlaybox app](https://github.com/brianklaas/awsPlaybox), you'll notice that there is a looping block and some math to determine the number of loops to use. Why are we looping here?

## Dealing with AWS Translate Service Limits

Most AWS service have limits. These are put in place so that no single client of AWS can overwhelm the services with their work. Translate has some very specific [service limits](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#limits_amazon_translate). Specifically, these are:

- 5,000 *bytes* of UTF-8 text in a single request
- 10,000 *bytes* per 10 seconds per language pair (ie; en-fr, es-en, en-ch, etc.)
- 20 transactions per second per language pair

5,000 bytes of UTF-8 text is a pretty good amount of text if you're building a chat app. If you're trying to translate [Moby Dick](https://www.gutenberg.org/files/2701/2701-h/2701-h.htm) or your company's annual report, it's not going to be nearly enough data to perform all your work in a single request.

As such, you're going to need to make sure that you're sending less than 5,000 *bytes* (not characters) of text in any single TranslateTextRequest. Additionally, you have to make sure that you're not sending more than 10,000 bytes per 10 seconds. If you violate either of these rules, your TranslateTextRequest will fail with a service limit error.

There are many ways to handle this, and translate.cfm shows a simple way to deal with this.

First, we calculate the number of chunks we need to break the text into in order to stay under the 5,000 bytes per request service limit. Then we need to calculate the number of pauses we'll need to take in order to not go over the 10,000 bytes per 10 seconds limit:

{% highlight javascript %}
trimmedSourceText = trim(FORM.textToTranslate);
totalChunks = ceiling(len(trimmedSourceText) / 4900);
totalPauses = ceiling(totalChunks / 2);
{% endhighlight %}

We then build an outer loop that goes through all the chunks (4,900 character slices of the text we want to translate), and another check inside of that loop that pauses the loop using Java's sleep() method. This way, we avoid exceeding service limits (provided, of course, that we aren't also going over the 20 transactions per second across all requests from our AWS account limit as well).

{% highlight javascript %}
for (currentChunkCounter = 1; currentChunkCounter <= totalChunks; currentChunkCounter++) {
  chunkToTranslate = mid(trimmedSourceText, currentEndPosition, 4900);
  currentEndPosition += 4900;

  ...

  if ((currentChunkCounter mod 2) eq 0) {
    if (currentPauseCounter LTE totalPauses) {
      sleep(10000);
      currentPauseCounter++;
    }
  }
} // End totalChunks loop
{% endhighlight %}

## Better Handling of Word Truncation

If we slice our text into equal chunks of 4,900 characters each, we will no doubt start truncating words in the process. That won't be good for making our final translated text read properly. Translate has a basic understanding of language semantics, so if you cut off the word "elephant" in the middle of the word, Translate won't be able to properly translate "ele" and "phant" as separate words. You'll end up with garbage.

As such, you should make sure you're not cutting words off in the middle. In translate.cfm, we do that as follows:

{% highlight javascript %}
if (len(chunkToTranslate) GTE 4900) {
  lastWord = ListLast(chunkToTranslate, " ");
  chunkToTranslate = left(chunkToTranslate, (len(chunkToTranslate) - len(lastWord)));
  currentEndPosition -= len(lastWord);
}
{% endhighlight %}

There are probably more elegant, complex solutions to this challenge that better handle non-English (and non-Western) source languauge text, but since Translate only translates to and from English, this is a fairly good solution.

## Outputting the Translation Result

Again, the Translate service is quite simple to work with. The translateTextResult object that's returned from a TranslateTextRequest is little more than a POJO (plain old Java object) with a few simple properties. The property we care about is the translatedText property.

Because we're looping and adding one chunk of text to another, we simply concatenate any existing translated text with the translated text that just came back from the TranslateTextRequest:

{% highlight javascript %}
finalTranslation &= translateTextResult.getTranslatedText();
{% endhighlight %}

When we're done, we simply output the finalTranslation string. In a production application, you'd probably want to store the finalTranslation string in a database for further processing or re-use. 

That's it for working with AWS Translate from within CFML. As I mentioned, it's really simple and most of the code in the example was for dealing with service limitations. If you have any questions about any of the posts in this series, feel free to <a href="https://twitter.com/brian_klaas">message me on the Twitter</a>!