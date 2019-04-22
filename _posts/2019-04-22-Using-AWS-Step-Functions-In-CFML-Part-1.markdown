---
layout: post
title:  "Using AWS Step Functions in CFML: What They Are and How To Connect to the Service"
date:   2019-04-22 17:21:00 -0400
categories: AWS ColdFusion
---

In my last series on using AWS services from CFML applications, I talked about [using AWS Lambda to write serverless application functionality](https://brianklaas.net/aws/coldfusion/2019/02/13/Using-AWS-Lambda-In-CFML-Part-1.html). Lambda is really powerful, but most developers who get started with Lambda quickly run into a significant architectural stumbling block. Developers (generally) develop applications, not single, standalone functions.

There's a common response to this stumbling block: the developer groups a whole bunch of related functionality into a single Lamdba function, and, depending on the arguments that get passed in, the Lamdba function takes a different path through the code. It works, yes, but the developer has rebuilt a mini monolith (a microlith, if you will) inside of Lambda. Testing these microliths is a lot more complicated that testing a single function with a single responsibility. Debugging becomes harder because there are more and more paths for the code to take. The benefit of the simplicity of using Lambda for code execution starts to fade away.

Along with needing a single function to do more than one thing, developers starting out with serverless code execution tools like Lambda often find they need to store information about the execution state. If the function is supposed to process five files, how do we know when all five files are done? What if the processing of one of those five files fails? Does the whole Lambda function error out? Is our application OK with some of the data being processed and other data not being processed? Code must be written to handle all of this, which, again, complicates our Lambda function, which, again, leads to greater complexity in testing, debugging, and observability.

There are also [strict time limits for Lambda function execution](https://docs.aws.amazon.com/lambda/latest/dg/limits.html). If you're trying to use Lambda in an ETL pipeline that's dealing with 300GB of data per day, you might run over those limits if you try to handle all your ETL in a single Lambda function execution. What if you have 5,000 radiology images that all need to get analyzed in the same way, as part of the same batch? Are you going to write other functions which make sure that all the individual images get processed? What if the execution of that controller function times out?

I think you can see that while Lambda is great for a lot of things, it's not a complete serverless application solution in all situations. If you need to worry about state, or orchestration across large file sets, or don't want to clutter up your Lambda code with error and retry logic, there's a better way of doing things: [Step Functions](https://aws.amazon.com/step-functions/).

### What Can Step Functions Do for You?

Step Functions are a powerful way to orchestrate complicated or long-running serverless workflows. Instead of adding in a lot of state, flow, and retry logic into your individual Lambda functions, you let Step Functions handle this.

Step Functions are, simply, [state machines](https://en.wikipedia.org/wiki/Finite-state_machine). Using a [simple, declarative language](https://states-language.net/spec.html) (which looks just like JSON), you write out, in code, what steps your workflow can take. You move from one state to the next in the state machine, depending on what direction you write that the workflow should go. It's a bit like writing the flow of your application logic in JSON, and then calling individual functions, as needed, as you travel through the workflow.

There are a number of different state types:

- Tasks (call code which does something)
- Choice (branching)
- Wait (pause in execution)
- Pass (don't call code, but maybe add in a variable to the workflow)
- Parallel (execute multiple actions in parallel)
- Success
- Fail

In this series, we'll look at all of these state types and how they work together to form powerful serverless workflows.

You can pass data into any state, and get data out of any state. Data is expressed in JSON format, so it's very much like passing arguments to a function call in a regular, server-based application or API.

Step Functions also has the ability to handle errors and retires, including backoff rates, from within the state machine JSON itself. You don't have to write code which handles errors. You don't have to write code which handles retries and deals with backoff to reduce the possibility of cascading failures. Step Functions does this for you. It's yet another way that Step Functions helps to keep your Lambda functions isolated, simple, and re-usable across workflows.

### Working with Lambda from CFML

I'll once again use [my AWSPlaybox application](https://github.com/brianklaas/awsPlaybox) for all the example code.

> Remember: these code examples assume that you have the AWS SDK .jar in your CFML runtime's /lib directory. If you need guidance on how to do this, please see [this post](/aws/coldfusion/2018/12/10/Update-On-Using-AWS-Java-SDK-With-ColdFusion-2018.html).

As with all AWS services, you need to first create a client for the service that you want to use. This was covered in [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html) post, but here are the steps involved:

1. Create a Client Builder object for the service &mdash; AWSStepFunctionsClientBuilder.
2. Tell the Client Builder what kind of builder object you want to use. It's simplest to use the standard builder.
3. Pass in your credentials via the StaticCredentialsProvider object (created upon instantiation of awsPlaybox/model/awsServiceFactory.cfc).
4. Tell the Client Builder which [AWS region](https://docs.aws.amazon.com/general/latest/gr/rande.html) you're working in. (I generally use 'us-east-1'.)
5. Tell the Client Builder to build (make) the connection to the service.

Here's the relevant code from AWSPlaybox/model/awsServiceFactory.cfc:

{% highlight javascript %}
case 'stepFunctions':
    javaObjectName = "com.amazonaws.services.stepfunctions.AWSStepFunctionsClientBuilder";

serviceObject = CreateObject('java', '#javaObjectName#').standard().withCredentials(variables.awsStaticCredentialsProvider).withRegion(#variables.awsRegion#).build();
{% endhighlight %}

Now we can work with Step Functions from within our CFML application. 

The rest of this series will cover two example workflows that illuminate how Step Fucntions works:

1. Analyzing a randomly chosen image with [AWS Rekognition](/aws/coldfusion/2018/07/23/Using-AWS-Rekognition-In-CFML-Part-1.html).
2. Transcribe, translate, and speak the content of a video into three different languages using [AWS Transcribe](/aws/coldfusion/2018/09/14/Using-AWS-Transcribe-in-CFML-Part-1.html), [AWS Translate](/aws/coldfusion/2018/10/21/Using-AWS-Translate-in-CFML-Part-1.html), and AWS Polly.

I covered [Rekognition](/aws/coldfusion/2018/07/23/Using-AWS-Rekognition-In-CFML-Part-1.html), [Transcribe](aws/coldfusion/2018/09/14/Using-AWS-Transcribe-in-CFML-Part-1.html), and [Translate](/aws/coldfusion/2018/10/21/Using-AWS-Translate-in-CFML-Part-1.html) in previous series on this blog. Please check out each of those series if you'd like to know more about each of the services. 

This is going to be a fairly long series, as I have lots of ground to cover in the two examples. By the end, though, I hope you'll have a better understanding of the power of Step Functions and how they can create pretty amazing serverless *applications*, not just functions. You'll also see how to control and monitor Step Function workflow executions from within CFML.