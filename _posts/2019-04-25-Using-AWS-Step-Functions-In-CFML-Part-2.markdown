---
layout: post
title:  "Using AWS Step Functions in CFML: The First Example Workflow and an Exploration of Task States"
date:   2019-04-25 16:51:00 -0400
categories: AWS ColdFusion
---

In the first post in this series, I introduced Step Functions and described the critical role they play in building serverless *applications*, not just functions. The rest of the series will examine two different Step Functions workflows. The first of these workflows is a relatively simple workflow that analyzes one of two randomly chosen images using AWS Rekognition.

> Remember that the full code for everything I talk about in this series is in [my AWSPlaybox application](https://github.com/brianklaas/awsPlaybox).

I [covered AWS Rekognition](/aws/coldfusion/2018/07/23/Using-AWS-Rekognition-In-CFML-Part-1.html) in a previous blog series, so if you're not familiar with what Rekognition is, please review that series first. Rekognition is a machine vision service that allows you to perform all sorts of analysis on the content of an image or video.

This first example Step Function workflow is pretty simple. The steps are:

1. Start the workflow.
2. Generate a random number between 1 and 100
3. If the random number generated in step 2 is less than or equal to 50, choose image 1.
4. If the random number generated in step 2 is above 50, choose image 2.
5. Analyze the chosen image.
6. End the workflow and return the results of the analysis to the calling environment.

Here's what this workflow looks like in the [Amazon States Language](https://states-language.net/spec.html), which is how you declare (write) a Step Functions workflow:

{% highlight json %}
{
  "Comment": "A simple example of making choices in Step Functions.",
  "StartAt": "generateRandomNumber",
  "States": {
    "generateRandomNumber": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789:function:confDemoRandomNumber",
      "ResultPath": "$.randomNumber",
      "Next": "ChoiceState"
    },
    "ChoiceState": {
      "Type" : "Choice",
      "Choices": [
        {
          "Variable": "$.randomNumber",
          "NumericLessThanEquals": 50,
          "Next": "setDataForImageOne"
        },
        {
          "Variable": "$.randomNumber",
          "NumericGreaterThan": 50,
          "Next": "setDataForImageTwo"
        }
      ],
      "Default": "DefaultState"
    },

	"setDataForImageOne": {
      "Type" : "Pass",
      "Result": { "s3Bucket": "awsplayboxbucket", "s3Key": "images/familyFaces.jpg" },
      "Next": "getImageLabels"
    },

  "setDataForImageTwo": {
      "Type" : "Pass",
      "Result": { "s3Bucket": "awsplayboxbucket", "s3Key": "images/dogForLabels.jpg" },
      "Next": "getImageLabels"
    },

	"getImageLabels": {
      "Type" : "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789:function:detectLabelsForImage",
      "Next": "Finish"
    },

    "DefaultState": {
      "Type": "Fail",
      "Cause": "We really should not have ended up here from a numeric value decision."
    },

    "Finish": {
      "Type": "Succeed"
    }
  }
}
{% endhighlight %}

This code can be found in [my AWSPlaybox application](https://github.com/brianklaas/awsPlaybox) in the sateMachines/choiceDemoStateMachine.json file.

I'll break down all of the above for you, and introduce you to a couple of key state types in Step Functions as we go along.

### Starting a Step Functions Workflow

Every Step Functions workflow has to begin with a starting point. This starting point, or starting state, is defined with the "StartAt" property at the beginning of a Step Functions workflow:

{% highlight json %}
"StartAt": "generateRandomNumber"
{% endhighlight %}

This tells our Step Functions workflow to start at the "generateRandomNumber" state.

States are defined in a Step Functions document as a structure of individual states that the workflow may go in to or out of at any time in the workflow. The "generateRandomNumber" state will be the first step in our workflow, and represents the most important type of state in any Step Functions workflow: a task.

### Task States in Step Functions Workflows

Task states are the key to getting anything done in a Step Functions workflow. In a task state, you perform a task. Task states either point to Lambda functions which you have written, or to a predefined set of other AWS services that you can invoke directly from your Step Functions workflow, without any intermediary Lambda function. Let's look at calling a Lambda function first.

In our workflow, the first step in our workflow is the "generateRandomNumber" state, which looks like this:

{% highlight json %}
"generateRandomNumber": {
    "Type": "Task",
    "Resource": "arn:aws:lambda:us-east-1:123456789:function:confDemoRandomNumber",
    "ResultPath": "$.randomNumber",
    "Next": "ChoiceState"
}
{% endhighlight %}

As you can see from the code, the type of this state is a *task* state. The task points to a *resource* to do work. In this case, the resource is a Lamdba function in our AWS account titled "confDemoRandomNumber." The string that defines this resource ("arn:aws:lambda...") is an ARN &mdash; an Amazon Resource Name &mdash; for the confDemoRandomNumber Lambda function. The ARN is the unique identifier for a resource across all of the services and all of the data centers in *all* of AWS. You can find the ARN of your Lambda functions in the Lambda console.

The code for the confDemoRandomNumber Lambda function is quite simple, and is written for Node.js:

{% highlight javascript %}
exports.handler = (event, context, callback) => {
    var min = event.min ? event.min : 1;
    var max = event.max ? event.max : 100;
    var result = Math.floor(Math.random() * (max - min)) + min;
    callback(null, result);
};
{% endhighlight %}

All this function does is generate a random number between 1 and 100, and returns that value to the calling environment &mdash; in this case, it's our Step Functions workflow.

How do we get the result from our random number generator function back into our Step Functions workflow? Not only do we need to make sure that a value is returned from our Lambda function, we also need to capture that value in our Step Functions workflow definition. We do that with this line of code:

{% highlight json %}
"ResultPath": "$.randomNumber"
{% endhighlight %}

I'll go into more depth about variable passing in Step Functions workflows in the next post, but, for now, just understand that the result from our confDemoRandomNumber Lambda function is going into a *variable* titled "$.randomNumber" in our Step Functions workflow.

Finally, this task state tells the workflow where to go next via the "Next" property of the task state. In this case, the generateRandomNumber state tells the Step Functions execution environment to go to the ChoiceState state next:

{% highlight json %}
"Next": "ChoiceState"
{% endhighlight %}

That's the essence of a task state. You invoke a Lambda function, get results back (if needed), and tell the workflow where to go next. You don't, however, always have to invoke a Lambda function in a task state. There are other options.

### Invoking AWS Services Directly in a Task State

When the Step Functions service initially launched, you could only invoke Lambda functions from within a task state. After a year of use, AWS looked at common patterns of function invocation and saw that a small set of core AWS services were being invoked repeatedly from Lambda functions in Step Functions workflows. To make it so that developers had to write less code, AWS updated the Step Functions execution environment so that those key services could be invoked directly from within the Step Functions workflow definition code, wihtout an intermediary Lambda function doing the work.

The following services are directly integrated into the Step Functions execution environment:

- AWS Batch
- DynamoDB (insert and get item only)
- Elastic Container Service (task execution)
- Simple Notification Service (publish to a topic)
- Simple Queue Service (add a message to a queue)
- AWS Glue
- AWS SageMaker

For example, if you wanted a state in a Step Functions workflow to publish a message to a Simple Notifiction Service (SNS) topic, you'd write your task state like this:

{% highlight json %}
"Publish to SNS": {
  "Type": "Task",
  "Resource": "arn:aws:states:::sns:publish",
  "Parameters": {
    "Message": "Your request to unsubscribe has been processed.",
    "PhoneNumber": "+12065550123"
  },
  "Next": "Next State in the Workflow"
}
{% endhighlight %}

Instead of having to write a simple Lambda function to publish a message to SNS, you can do that directly from your Step Functions workflow code. This opens up some powerful options in Step Functions workflows, particularly when, for example, you are running a workflow over a large batch of data files that should be processed by a machine learning toolset on SageMaker, or when you have a Docker image running inside of Elastic Container Service that will perform complex tax calculations on a set of passed-in data. 

The examples in the AWS Playbox application don't invoke AWS services directly. If you'd like to learn more about this feature of Step Functions, please refer to the [Step Functions documentation](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-connectors.html).

In the next part of this series, we'll look at passing data into and out of individual task states. This is important to understand because we're about to go into a state that chooses one of two options based on the result of our first task.