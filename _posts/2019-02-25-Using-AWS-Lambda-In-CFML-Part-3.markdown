---
layout: post
title:  "Using AWS Lambda in CFML: Invoking Your Lambda Function From CFML"
date:   2019-02-25 7:51:00 -0500
categories: AWS ColdFusion
---

Once you have [set up and tested your Lambda function](/aws/coldfusion/2019/02/20/Using-AWS-Lambda-In-CFML-Part-2.html), calling the function from CFML is a breeze thanks to the [AWS Java SDK](https://aws.amazon.com/sdk-for-java/).

### CFML Code to Invoke a Lambda Function

In [my AWSPlaybox application](https://github.com/brianklaas/awsPlaybox), the code for invoking a Lambda function from CFML can be found on [/lambda.cfm](https://github.com/brianklaas/awsPlaybox/blob/master/lambda.cfm).

While you can pass in arguments of any type to your Lamdba function, the example function we set up in the previous post expects arguments in JSON format. That's the first part of the code:

{% highlight json %}
payload = {
  "firstName": "Brian",
  "lastName": "Klaas",
  "email": "brian.klaas@gmail.com",
  "classes": [
    {
      "courseNumber": "260.710.81",
      "role": "Faculty"
    },
    {
      "courseNumber": "120.641.01",
      "role": "Student"
    }
  ]
}
{% endhighlight %}

I'm using CFML's implict struct and array notation here, rather than explicity defining each structure and each sub-array in CFML. Note that if you're using ColdFusion 10 or lower, ColdFusion upper cases serialization of keys in a structure unless you use array notation to define the keys in the struct. That will cause you problems in case-sensitive JavaScript on the NodeJS backend function.

The heart of Lambda function invocation is the typical request/reponse object pattern found throughout the AWS Java SDK. You create a request object of some type, send it to the service, and get a response object back from the service. In this case, we're going to create an [InvokeRequest](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/lambda/model/InvokeRequest.html) object, set the required properties, send that InvokeRequest object off to the Lambda service, and get an [InvokeResult](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/lambda/model/InvokeResult.html) object back.

Here's the code in CFML, which is pretty straightforward:

{% highlight javascript %}
lambda = application.awsServiceFactory.createServiceObject('lambda');
invokeRequest = CreateObject('java', 'com.amazonaws.services.lambda.model.InvokeRequest').init();
invokeRequest.setFunctionName(application.awsResources.lambdaFunctionARN);
invokeRequest.setPayload(jsonPayload);
result = variables.lambda.invoke(invokeRequest);
{% endhighlight %}

The functionName property of the invokeRequest object is the ARN &mdash; [Amazon Resource Name](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) &mdash; of your Lambda function running in AWS. The ARN is the unique identifier, across all of AWS, for your Lambda function. It can be found in the AWS Lambda console.

The payload (the data you're sending to the Lambda function) has to be serialized into JSON before setting via the setPayload() method, because that is what is expected by the NodeJS-based function that is being called. If you're not sending JSON, then you can put the raw data, in whatever format it might exist, into the payload. The payload size cannot exceed a maximum of 6MB in a *synchronous* function invocation, which is what we are doing here.

> Note that the AWS Java SDK has asynchronous client builders along with the synchronous client builders that are used in the AWSPlaybox application. The async clients return [Futures](https://www.baeldung.com/java-future) on each function invocation. Given that [ColdFuson 2018 now supports Java Futures for asynchronous programming workflows](https://coldfusion.adobe.com/2018/07/asynchronous-programming-in-coldfusion-2018-release/), I can foresee myself starting to use the async AWS Java SDK clients. If a Lambda function is invoked asynchronously, the payload size [cannot exceed 256KB in size](https://docs.aws.amazon.com/lambda/latest/dg/limits.html).

We've made our request to Lambda. Lambda receives that request, fires up a tiny little execution environment (if one with our code is not already running), executes our code, and then returns a result to the caller. Our caller (our CFML code) waits around for execution completion because this is a synchronous call.

### Getting Data Back from a Lambda Function Invocation

Since we're using the standard Lambda client from the AWS Java SDK, and not the async client, we're going to wait for a response from Lambda. Here's the code to read in the response:

{% highlight javascript %}
result = variables.lambda.invoke(invokeRequest);

charset = CreateObject('java', 'java.nio.charset.Charset').forName("UTF-8");
charsetDecoder = charset.newDecoder();
lambdaFunctionResult = charsetDecoder.decode(sourcePayload).toString();
{% endhighlight %}

What's up with this charset decoder? Although we're sending a plain text string back to the caller from our Lambda function, the payload returned from a Lambda function invocation in the Java SDK is always a Java binary stream. As such, it needs to be decoded into a string of characters to be displayed as a string in our CFML output. Java makes this pretty simple with the decoder's toString() method.

If you were returning JSON from your Lambda function, you'd still want to decode it via toString() and then serializeJSON() the result. If you were returning a binary object (a PDF, for example), you would need to use the appropriate decoder to decode the stream into an appropriate file format.

That's it for invoking a Lambda function and receiving a result back from within CFML. In the next post, we'll look step back and look at some big picture issues with using Lambda from within your CFML apps, including debugging, observability, and broader use cases for mixing Lambda into your CFML application architecure.