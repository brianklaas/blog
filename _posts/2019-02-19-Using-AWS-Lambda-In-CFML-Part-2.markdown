---
layout: post
title:  "Using AWS Lambda in CFML: Creating a NodeJS Lambda Function that You Can Call From CFML"
date:   2019-02-19 10:51:00 -0500
categories: AWS ColdFusion
---

While [the FusionLess project](https://fuseless.org) is awesome in its potential to write and deploy CFML to AWS [Lambda](https://aws.amazon.com/lambda/), this post will focus on calling a non-CFML Lambda function from within your CFML application.

To that end, I'll show you how to create a very basic [NodeJS](https://nodejs.org/) function and get it running inside Lambda. 

### The AWSPlaybox NodeJS Lambda Function Example

[My AWSPlaybox application](https://github.com/brianklaas/awsPlaybox) has the code for a simple, NodeJS-based function which we can call, and get a response from, in CFML. All of the code can be found in the [/nodejs/lambda/returnDataToCaller.js file](https://github.com/brianklaas/awsPlaybox/blob/master/nodejs/lambda/returnDataToCaller.js):

{% highlight javascript %}
exports.handler = (event, context, callback) => {
    var resultString = "";
    var timestamp = new Date();
    
    console.log('firstName =', event.firstName);
    console.log('lastName =', event.lastName);
    console.log('email =', event.email);
    
    for (var i=0; i < event.classes.length; i++) {
        console.log("In " + event.classes[i].courseNumber + ", your role is " + event.classes[i].role);
    }
    
    resultString = "Hello " + event.firstName + " " + event.lastName + 
        ". As of " + timestamp + ", you are currently enrolled in " + 
        event.classes.length + " courses.";
    callback(null, resultString);
};
{% endhighlight %}

Let me break this code down a bit:

Lambda expects a single function to be made available from your code. That's the function it will run when the Lambda function is invoked by a call from your CFML application (or from any number of event sources in AWS). The de facto standard name for this function is "handler," as evidenced in the first line of code above. You can name it whatever you want, but most people go with "handler."

The handler function takes three arguments: the event object, the context object, and a callback function.

- The *event* object contains all the data that gets passed to the function during invocation, provided in JSON format.
- The *context* object contains information about the current Lambda runtime, and is best left alone.
- The *callback* is, essentially, the invoker of the function. In this case, it's your CFML application.

In this case, the event object contains:

- firstName  [ string ]
- lastName  [ string ]
- email  [ string ]
- classes [ array ]

We pass in all of this data via our CFML Lambda function invocation, which we'll see later in the series.

The rest of the function simply loops through the array of classes in the *event* object, and logs each class to the console. In a real application, you could write each of these records to a database. 

Finally, we return a string built from the data found in the *event* object to our calling CFML.

> Note that Lambda does not restrict you to one JavaScript function per .js file. You can have as many functions as you want, and they can be in multiple .js files. You can only have one "handler" function per Lambda function definition, however.

This is a really simple example of a Lambda function in NodeJS. My team has built functions with multiple classes and lots of JS code. While it's best to keep your functions as simple as possible, simple does not mean you can't rely on other functions in the same file or other JavaScript classes when appropriate.

### Running the AWSPlaybox NodeJS Example in Lambda

As the AWS Console is constantly changing, I'm not going to provide extensive instructions on getting this function running inside of Lambda. [AWS has a very clear tutorial on how to do this](https://docs.aws.amazon.com/lambda/latest/dg/getting-started-create-function.html). Although the AWS example uses Python, you'll set your runtime to NodeJS. The Handler function is "index.handler". You don't need to add any triggers and you don't need to set any other configuration options beyond the defaults.

To enter the actual code, use the [AWS Cloud 9 Editor](https://aws.amazon.com/cloud9/) that's built into the Lambda console. You can literally copy and paste the code from the [/nodejs/lambda/returnDataToCaller.js file](https://github.com/brianklaas/awsPlaybox/blob/master/nodejs/lambda/returnDataToCaller.js) file.

> As a side note, Cloud 9 is a pretty darn good cloud-based editor. I do almost all my Lambda development work in Cloud 9, as it has integrated debugging for NodeJS Lambda functions. It's quite capable, robust, and easy to use.

### Testing the AWSPlaybox NodeJS Example in Lambda

One of the nice things about developing functions in the Lambda console is that you can provide test data for your Lambda function from within the console itself, without having to call that event from a real, production source.

If you want to set up a test event, copy and paste the JSON below into the test event setup window:

{% highlight json %}
{
  "firstName": "Brian",
  "lastName": "Klaas",
  "email": "brian.klaas@gmail.com",
  "classes": [
      {
          "courseNumber": "260.710.81",
          "role": "Faculty"
      }
  ]
}
{% endhighlight %}

Click the "Test" button with this sample data selected, and you'll see the results of the Lambda function execution.

### Alternatives to Developing Your Lambda Functions in the Console

Developing in the AWS console is a great way to get started, and to learn about the details of the service. In the long run, though, you're going to want to abstract away a lot of this manual work and, ultimately, move toward an automated build and deployment pipeline. Fortunately, Lambda has two mature frameworks for exactly this:

- [Serverless](https://serverless.com)
- [SAM (Serverless Application Model)](https://aws.amazon.com/serverless/sam/)

Covering either of these frameworks is beyond the scope of this post, but once you understand how to write and deploy a function in the AWS Lambda console, it's absolutely worth your while to learn about &mdash; and use &mdash; these frameworks.

The Serverless framework has a robust ecosystem of plugins that can make complex serverless functions a lot simpler to build and deploy. SAM is maintained by AWS, has a CLI, and has the ability to test Lambda functions locally. A Google search for "Serverless framework vs SAM" will return a lot of discussion on the topic.

Both frameworks ultimately focus on accessing your Lambda functions via http calls to the [AWS API Gateway](https://aws.amazon.com/api-gateway/), rather than direct invocation, which is what we're doing in this series. As such, direct invocation is what we'll cover next.