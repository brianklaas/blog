---
layout: post
title:  "Using AWS Lambda in CFML: Debugging, Observability, and More"
date:   2019-03-01 7:51:00 -0500
categories: AWS ColdFusion
---

Developing for AWS Lambda, or Google Cloud Functions, or Azure Functions, is a core part of what is known as "serverless" development. You focus on application logic and workflow, and never really worry about the servers that power the execution of that code. It's a powerful, liberating development model, but not one without issues. It's a maturing space, and like all maturing spaces, has pain points. This post explores a couple of those pain points: debugging, observability, and planning the right execution environment.

### Debugging Lambda Functions Called from CFML

Debugging Lambda function execution is one of the biggest pain points in Lambda development right now. Because you don't have access to the execution environment for your Lambda function, you can't really see what happens when something goes wrong. You'll just be told that something went wrong. 

There are a couple strategies for dealing with this limited debugging ability:

First, you can console.log() information during function execution to help you see what's happening. When you log from within a Lambda execution environment, regardless of the language used, that information goes to [AWS CloudWatch](https://aws.amazon.com/cloudwatch/), AWS' core logging and metrics reporting service. The pain here is that you have to switch from your local development environment (if you're developing locally) or the Lambda console in AWS over to CloudWatch, and then search for your output in CloudWatch. The CloudWatch console provides some rudimentary tools for searching for log output, but a it's often like searching for a needle in a haystack.

Second, you can develop your Lambda function locally. After all, all officially supported Lambda languages are available to run locally. There are two problems with this:

1. You can't really call other AWS services from the local environment.
2. Your function has to be adjusted and rewritten for the Lambda execution context.

You can mitigate the first issue by using [aws-sdk-mock](https://www.npmjs.com/package/aws-sdk-mock), a very handy NodeJS package for mocking out AWS services in your tests. There are similar mocking and local execution libraries for other languages. In Python, for example, you'd likely use [moto](https://github.com/spulec/moto). The Serverless framework offers the [Serverless offline plugin](https://github.com/dherault/serverless-offline) to develop your Serverless framework apps locally. AWS SAM offers the [SAM CLI to test Lambda functions locally](https://github.com/awslabs/aws-sam-cli).

The second issue, where your function ultimately has to expect the standard event, context, and callback objects, is somewhat more problematic. Again, if you're not using the Serverless framework or AWS SAM, you're going to have to figure this one out, and either mock things locally or write locally, upload to Lambda, run your function, test the results, and repeat the process over and over and over. That's a lot of time wasted.

Both of these problems can be mitigated, in part, by developing in the Lambda console itself. The Lambda console includes the excellent [AWS Cloud 9 editor](https://aws.amazon.com/cloud9/), and offers [integrated debugging for NodeJS Lambda functions](https://docs.aws.amazon.com/cloud9/latest/user-guide/tutorial-lambda.html). You get breakpoints, watch expressions, and many other tools you'd see in the browser while developing JavaScript locally. This is a **very** handy way to develop and debug your NodeJS Lambda applications.

As you're calling your Lambda functions from CFML, I would strongly recommend the following workflow:

1. Build, test, and debug your Lambda application separately from your CFML application.
2. Build, test, and debug your CFML code using a *mocked* call to Lambda.
3. Build, test, and debug the code which invokes the AWS Java SDK to call Lambda.

Error messages from the AWS Java SDK aren't always very helpful &mdash; particularly in the CFML execution context. You'll often get "can't find this method" messages even though the method is in the AWS Java SDK. This is most often the result of [overloaded Java method signatures](https://www.geeksforgeeks.org/overloading-in-java/), and you just have to provide the right kinds of arguments to the method call. 

### Observability When Calling Lambda Functions from CFML

As with debugging, observability of your Lambda function execution is pretty limited. You do get a little bit of information in the *context* object that's in every Lambda function invocation. You also get basic metrics in the AWS Lambda console itself. A number of companies offer wrappers for your Lambda functions that tie in to their logging and observability platforms. DataDog, Epsagon, IOPipe, and more offer such toolkits to bring broader observability to your Lambda function executions.

AWS also offers [X-Ray](https://aws.amazon.com/xray/), an integrated tracing package that can show you the time it takes for a call to run across all AWS services that are requested. X-Ray is hugely helpful if you are making calls from your Lambda function to othe AWS services. The visualizations ([service map)](https://docs.aws.amazon.com/xray/latest/devguide/xray-console.html#xray-console-servicemap) provided by X-Ray are excellent for discovering bottlenecks in your serverless application workflows.

On the CFML side, Lambda is a black box. You make a request, you get a response. That's pretty much it. There's little that you can really see about the Lambda function execution from within your CFML application. You can run a getTickCount() before and after your Lambda function invocation to see how long things take, but that's about the limit of observability from within your CFML application.

### Lambda Execution Environment Configuration and Speed

One of the few things you can actually configure about your Lambda execution environment is the amount of memory available to the execution environment. *This has a direct and measurable impact on the speed of execution of your Lambda function*, regardless of language. Much has been written about this issue, but here's an example of how [increasing memory allocated to your Lambda execution environment affects performance](https://epsagon.com/blog/how-to-make-lambda-faster-memory-performance-benchmark/). The basic principle is this: the more memory you allocate (and therefore pay for), the faster your Lambda function will run. There are limits to this, of course, as most Lambda functions stop seeing performance gains around the 2GB mark. 

This does not mean that you should allocate 2GB of memory for all of your functions. If you have simple functions that are processing small JSON objects, you can likely get great performance with 128MB of RAM. If you're processing 5MB CSV files on every Lambda function invocation, you probably want more memory to increase performance. Either way you have to test your Lambda function, and see the results of execution time in the Lambda console or whatever observabilty platform you choose to monitor your Lambda executions. 

### Where Lambda and CFML Intersect

As I mentioned in the first post, the power of Lambda and related serverless development environments is enormous. It's a hugely freeing development model. You can do pretty much anything you want in a serverless model, but it's not the right tool for *every* job.

Most of us have existing CFML codebases. Lambda, and the whole serverless development model as a whole, gives us many options to add in entire classes of functionality that would otherwise be impossible or impractical from within the CFML runtime. Here are some examples:

- **Data Science**: Python, with packages like Pandas and NumPy, is the king of data science right now. You could try to build out a base level of statistical analysis from within your CFML applications, but you'd be reinventing the wheel. There are many .jar libraries which handle statistical analysis, but few match the capability and elegance of their Python counterparts.

- **Machine Learning**: Do you really want to set up a machine learning environment? Not just the servers, but the training and model execution runtimes? That's undifferentiated heavy lifting which sucks resources away from your core business. By making remote calls to platforms which offer managed machine learning runtimes, you can add this functionality in to your CFML applications without having to have the expertise in setting this up yourself. Your call to Lambda from CFML opens up a whole host of possibilities for using ML on your current application data. I've already covered [computer vision](https://brianklaas.net/aws/coldfusion/2018/07/23/Using-AWS-Rekognition-In-CFML-Part-1.html), [translation](https://brianklaas.net/aws/coldfusion/2018/10/21/Using-AWS-Translate-in-CFML-Part-1.html), and [audio-to-text](https://brianklaas.net/aws/coldfusion/2018/09/14/Using-AWS-Transcribe-in-CFML-Part-1.html) on this blog.

- **Event Processing**: As Lamdba is hugely scalable, it can serve as an ingest point for a large number of events very quickly. If you want to do click-tracking in your CFML application, you have to be very careful about having super-performant CMFL code to do this work and keep up with a large number of events in a short amount of time. This may not be possible without building out a separate CFML environment for this work. With Lambda, you simply make invocations as needed, and let your Lambda functions do all the processing while your CFML application continues to hum along. You could use [AWS Kinesis](https://aws.amazon.com/kinesis/) to do this event ingestion work, but Lambda is much simpler and likely perfectly appropriate for many CFML applications.  
  
- **ETL**: Although the default for invoking Lambda functions from your CFML applications is a synchronous call, you don't have to return lots of data in that synchronous call. You can instead just say "here's some data to kick off some other complex task" and return a "received" message to your calling CFML. Your Lambda function (or serverless infrastrucure) can then fire off a whole series of complex, longer-running tasks in the background. [ETL](https://www.webopedia.com/TERM/E/ETL.html) is a great example of work that can be done in the background, and doesn't need to be done from within your CFML application. Lambda functions can run for up to 15 minutes. You can get a lot of work done in that time without burdening your CFML runtime with very, very long-running requests.

- **Media Transcoding**: While CFML does have image manipulation capabilities built in, they can be [very slow](http://www.carehart.org/blog/client/index.cfm/2012/5/28/cf_image_resizing_problem_and_solution). One of the very first demonstrations for Lambda was using it as an image resizing tool, as the Lambda execution environment has [ImageMagick](https://en.wikipedia.org/wiki/ImageMagick) built in. It will be a lot faster to make 100 Lambda invocations to resize 100 images than it will be to do the same work in CFML. Beyond images, AWS and other providers offer an array of audio and video [transcoding](https://aws.amazon.com/mediaconvert/) [services](https://aws.amazon.com/elastictranscoder/) that are cheap and provide high-quality output in a variety of formats. This is much more performant than calling cfexecute on a local copy FFMPEG and hoping that doesn't bring down your server in the process.

- **Building a Hybrid Cloud Application**: As mentioned in many of the examples above, you don't always want to build out entire systems to handle new pieces of functionality in your applications. You may not want to run your own data warehouse. You may not want to manage a video encoding farm. You may not have in-house machine learning expertise. Lambda can act as a gateway between applications in your data center and the cloud. You can run core parts of your application in CFML and then others in the cloud. Lambda is the highly performant hub through which data passes to both of these environments.

I'm big on Lambda and other serverless execution environments. Hopefully this series has shown you how easy it is to get started with serverless functions, and inspired you to try building hybrid CFML-Lambda workflows on your own. As always, if you have any questions about any of the posts in this series, feel free to <a href="https://twitter.com/brian_klaas">message me on the Twitter</a>!