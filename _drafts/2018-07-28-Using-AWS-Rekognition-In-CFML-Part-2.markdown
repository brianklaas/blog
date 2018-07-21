---
layout: post
title:  "Using AWS Rekognition in CFML: Detecting and Processing the Content of an Image"
date:   2018-07-28 09:01:00 -0400
categories: AWS ColdFusion
---
The most obvious use case for Rekognition is detecting the objects, locations, or activities of an image. This functionality returns a list of "labels." Labels can be things like "beach" or "car" or "dog." This allows you to build tools like image search.

As I wrote in the series on using Simple Notification Service (SNS) from CFML, the [JavaDocs for the AWS Java SDK](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html) are comprehensive and always up-to-date. They are, alas, also just JavaDocs. You're not going to find detailed examples of how to complete full tasks in these docs. If you look at the [documentation for the com.amazonaws.services.rekognition.AmazonRekognitionClient](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/rekognition/AmazonRekognitionClient.html), you can see all the things that you can do with Rekognition via the Java SDK.

When working with the AWS Java SDK, there's a basic pattern that you'll follow. It goes like this:

1. Get a copy of the client that's making a connection to the service you want to use.
2. Create a "request" object.
3. Fill the "request" object with the parameters (or other objects) you need to supply.
4. Tell the client to make the request.
5. Get back a "response" object.

This was the case when working with SNS. This is the case when working with Rekognition.

Here are the steps to detecting the labels (objects, locations, activities) in an image:

1. Get a copy of [the Rekognition client we created in the first part of this series](/aws/coldfusion/2018/07/23/Using-AWS-Rekognition-In-CFML-Part-1.html).
2. Create a "DetectLabelsRequest" object.
3. Create an "Image" object in the Rekognition service to work with.
4. Create a "S3Object" object that specifies the path to the image on S3 that you want to treat as the image for the Image object created in step 3.
5. Set the S3Object into the Image object.
6. Set the Image object into the DetectLabelsRequest.
7. Run the DetectLabelsRequest.
8. Get back a DetectLabelsResult object.

Remember: this is Java! Everything is an object and the code is going to be a bit verbose.

## The Code to Detect Labels in an Image

Here's how we do this in the [AWSPlaybox app](https://github.com/brianklaas/awsPlaybox):

> If you haven't already read the entry on [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html), please do so now.

Because there are a number of steps to this process, and I wanted to re-use some of the code in different service calls, I encapsulated common Rekognition functionality in /model/rekognitionLib.cfc. We start our detect labels process in /rekognition.cfm by providing a list of paths to images already in S3 that we can randomly choose from:

{% highlight javascript %}
<cfset s3images.imagesForLabels = [ "images/beachForLabels.jpg", "images/dogForLabels.jpg", "images/dogForLabels2.jpg", "images/familyFaces.jpg", "images/giraffePark.jpg", "images/waterfallKids.jpg", "images/tableSetting.jpg","images/dervishFace.jpg","images/lions.jpg","images/temple.jpg"] />
{% endhighlight %}

This is just an array of paths to files on S3, nothing more. The files just need to already be on S3. The code in the AWSPlaybox app does not load them up there for you.

Later in /rekognition.cfm, we randomly select one of those images from the array and run getImageLabels() on that image. We pass in the name of the S3 bucket where the images can be found, and the string value of the path to the randomly selected image.

{% highlight javascript %}
case 'detectLabels':
	sourceImage = s3images.imagesForLabels[randRange(1, arrayLen(s3images.imagesForLabels))];
	getImageLabelsResult = rekognitionLib.getImageLabels(s3images.awsBucketName, sourceImage);
break;
{% endhighlight %}

So now we hop over to rekognitionLib.cfc, where the real work happens. If you read through getImageLabels(), you'll see the eight steps listed above translated into code:

{% highlight javascript %}
// 1. Get a copy of the Rekognition client
variables.rekognitionService = application.awsServiceFactory.createServiceObject('rekognition');
// 2. Create a "DetectLabelsRequest" object.
var labelsRequest = CreateObject('java', 'com.amazonaws.services.rekognition.model.DetectLabelsRequest').init();
// 3. Create an "Image" object in the Rekognition service to work with.
var imageToLabel = CreateObject('java', 'com.amazonaws.services.rekognition.model.Image').init();
// 4. Create a "S3Object" object that specifies the path to the image on S3 that you want to pass into the Image object.
var imageS3Object = CreateObject('java', 'com.amazonaws.services.rekognition.model.S3Object').init();
imageS3Object.setBucket(arguments.awsBucketName);
imageS3Object.setName(arguments.pathToImage);
// 5. Set the S3Object into the Image object.
imageToLabel.setS3Object(imageS3Object);
// 6. Set the Image object into the DetectLabelsRequest.
labelsRequest.setImage(imageToLabel);
// 7. Run the DetectLabelsRequest, and 8. Get back a DetectLabelsResult object.
var labelsRequestResult = variables.rekognitionService.detectLabels(labelsRequest);
{% endhighlight %}

Note that Rekognition currently only works with objects in S3 or blobs of image bytes up to 5MB in size. Working with files in S3 is, in my experience, simpler because you don't have to check the size of the image. You can always upload, run the Rekognition function, and delete the image after getting the labels back if you don't want to store the image in S3 for the long term.

So now we have a DetectLabelsResult object (called labelsRequestResult) that contains our labels. What do we do with that?

## Processing the Image Labels Array

The detectLabels function in Rekognition returns an array of objects (because this is Java) of the type [com.amazonaws.services.rekognition.model.Label](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/rekognition/model/Label.html). This object  only contains two properties that are of use to us: the label name and the level of confidence that Rekognition has that this is an accurate, valid label for this image.

> The DetectLabelsRequest object has a property named minConfidence. You could run DetectLabelsRequest.setMinConfidence() and pass in a number to say to Rekognition "Don't bother to return any labels where you have less than X% confidence that the label is correct for this image." For example, if you run DetectLabelsRequest.setMinConfidence(70), Rekognition will only return labels where it has 70% confidence that the label is correct for the image. This is a really useful way to limit the results from the DetectLabelsRequest. If Rekognition is only 30% confident that a label is correct for a given image, why would you want that label in your result set?

In order to process the image labels array, we simply loop over the array and pull out the label and level of confidence for each item in the array. Here's the code in rekognitionLib.getImageLabels():

{% highlight javascript %}
var labelsArray = labelsRequestResult.getLabels();

labelsArray.each(function(thisLabelObj, index) {
	counter++;
	returnArray[counter] = structNew();
	returnArray[counter]['label'] = thisLabelObj.getName();
	returnArray[counter]['confidence'] = Int(thisLabelObj.getConfidence());
});
{% endhighlight %}

Note that we're using the 'each' member function of arrays in CFML instead of doing a traditional CFML for loop. The code is just transforming the Label object into a simple structure for display back on the /rekognition.cfm view.

## Displaying the Results

Back on /rekognition.cfm, we can take our CFML array of structures and loop through it to display the results on screen:

{% highlight javascript %}
<cfloop array="#getImageLabelsResult#" index="idxThisLabel">
	<li>#idxThisLabel.label# &mdash; #idxThisLabel.confidence#%</li>
</cfloop>
{% endhighlight %}

Here's an example result set:

<img src="/assets/postImages/rekogResultsGiraffe.png" align="center" width="600" height="358" border="1" alt="Sample result set from Rekognition's detect labels function" />

That's me feeding a giraffe at [the San Diego Zoo Safari Park](http://www.sdzsafaripark.org). As you can see, Rekognition gets the basics correct. There is an animal, a giraffe, which is a mammal. There's wildlife, people, a human, a person. There is Arecaceae (the palm family), and palm trees. I am outdoors, and there is sand, even though those received much lower confidence levels than some of the other labels. There is no bench, though. Rekognition thought the wood fence around the truck bed was a bench, which kind of makes sense when you realize that benches often have parallel pieces of wood as the back. There were many more results returned from the DetectLabelsRequest. I just cut them off in the screenshot.

In this example, we displayed labels on screen. In an image search application, you'd store the labels in a database and then build a front end to search against those labels. You could run Rekognition searches against a set of images in real-time, but that's going to be a whole lot slower than querying existing label results in your database. 

You could also use the Async version of the AWS Rekognition client (com.amazonaws.services.rekognition.AmazonRekognitionAsyncClientBuilder), which returns [Java Futures](https://dzone.com/articles/javautilconcurrentfuture) for each method. Now that [Adobe ColdFusion 2018 has support for aynschronous programming via Java Futures](https://coldfusion.adobe.com/2018/07/asynchronous-programming-in-coldfusion-2018-release/), you could do all of your Rekognition work in an asynchronous fashion &mdash; if you're using Adobe ColdFusion 2018.

So that's how you detect the objects, locations, or activities of an image using Rekognition from CFML. In the next post, I'll cover how to do sentiment analysis of faces in an image. That feature has some interesting &mdash; [and Orwellian](https://gizmodo.com/chinese-school-piloting-face-recognition-software-to-ma-1826142540) &mdash; implications.