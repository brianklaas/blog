---
layout: post
title:  "Using AWS Step Functions in CFML: Getting Data Into and Out of Step Functions States"
date:   2019-04-27 11:31:00 -0400
categories: AWS ColdFusion
---

In the [last post in this series](/aws/coldfusion/2019/04/25/Using-AWS-Step-Functions-In-CFML-Part-2.html), we looked at invoking task states to do actual work in our image analysis Step Functions workflow. As with any workflow, passing data into a task and getting result data out of a task invocation is a common requirement. You can, of course, persist data in DynamoDB or another storage mechanism to maintain state between steps in a Step Functions workflow, but that's often overkill (and expensive) when the Step Functions execution environment can maintain that state data for you.

In the "generateRandomNumber" task definition, we get the result of the random number generator function in this line of code:

{% highlight json %}
"ResultPath": "$.randomNumber",
{% endhighlight %}

The ResultPath is one of four fields that we can use to control how data flows into and out of any step in a Step Functions workflow. [Amazon has pretty good documentation on this](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-input-output-filtering.html), but it's still sometimes confusing.

### How Data Flows Into and Out of Step Functions States

Individual states in a Step Functions workflow receive JSON as input and usually pass JSON as output to the next state.

The following four fields control the flow of JSON input/output between states:

- InputPath
- OutputPath
- ResultPath
- Parameters

InputPath, Parameters, ResultPath, and OutputPath each manipulate JSON as it moves through each state in the workflow.

Each of the above four steps can use [JsonPath](https://github.com/json-path/JsonPath) to select portions of the JSON that's going in to the step as input or coming out of the step as output. JsonPath prefixes the appropriate node in the JSON with a $. 

**InputPath** selects which of the incoming JSON to pass to a task state.

**Parameters** get combined with the InputPath and get passed into a task state as one complete JSON structure.

**ResultPath** selects what combination of the state input and task *result* to pass to the output.

**OutputPath** can filter the *ResultPath* to limit what is passed out of the state and on to the next state in your workflow.

Here's an image from the AWS documentation which shows where each of the above four fields comes in to play during task execution:

<img src="/assets/postImages/stepFuncInputOutputPaths.png" align="center" width="600" height="394" border="0" alt="Diagram showing how state data flows in to and out of a state execution" />

You can see from the above that *InputPath* and *Parameters* combine to go into a task execution, and *ResultPath* and *OutputPath* filter the data that comes out of the task execution. You do not have to use all four of these fields in every step. You can use any combination (or none at all) to get the result you want.

The AWS docs do a good job of giving examples of how you can [dynamically assign values to input parameters for a task state](https://docs.aws.amazon.com/step-functions/latest/dg/input-output-inputpath-params.html). There are also examples of using *ResultPath* to replace the task input with the result, and, more usefully, [including the result of the task with the original input to the task](https://docs.aws.amazon.com/step-functions/latest/dg/input-output-resultpath.html) or [updating a node in the input with the result of the task](https://docs.aws.amazon.com/step-functions/latest/dg/input-output-resultpath.html).

Understanding how data flows into and out of task steps is critical in developing any kind of reasonably advanced Step Functions workflow. Learn how it works. 

The image analysis Step Functions workflow uses input/output in a relatively simple way. The second workflow that I'll cover in this series &mdash; transcribing, translating, and speaking the translated content of a video &mdash; will have more complex forms of task state input/ouput.