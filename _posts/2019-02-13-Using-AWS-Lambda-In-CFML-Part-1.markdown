---
layout: post
title:  "Using AWS Lambda in CFML: Overview and Creating the Lambda Service Object"
date:   2019-02-13 15:51:00 -0500
categories: AWS ColdFusion
---

What would your developer workflow be like if you could just write and deploy code, without having to worry about the execution environment of that code? No servers to set up, patch, and maintain. No 2AM panics that a server went down, or 3AM panics that a core library in the server has a huge security flaw. Just write, deploy, and build business value.

Functions as a Service (or FaaS) platforms have been promising this vision for the past four years. I was at [AWS re:Invent](https://reinvent.awsevents.com) (the annual AWS developer conference) in November 2014 when [Lambda](https://aws.amazon.com/lambda/) &mdash; a FaaS runtime &mdash; was introduced. The next morning, I had my first Lambda function deployed to production. I didn't have to stand up servers. I didn't have to worry about scale. I just wrote code and deployed. That first Lambda function is still running in production and has cost my team a whopping $0 in four years.

Most people refer to Lambda a "serverless" programming model. Serverless doesn't mean that there are no servers involved. There are always servers involved. You just don't see them, don't manage them, and can't touch them. You write code for a specific runtime and deploy it to the serverless environment. The environment &mdash; in this case, Lambda &mdash; takes care of everything else, including scalability and core security. This is a powerful, and seductive, model for developers.

This series of posts is about CFML, though, and there is no FaaS runtime that directly supports CFML out of the box. [Pete Frietag built the amazing FuseLess project](https://fuseless.org), which lets you run CFML in Lambda with a bundled Lucee server. It's super impressive work, and I strongly encourage you to check it out if you're interested in writing Lambda functions in CFML.

Instead of discussing running CFML inside of Lambda, this series is going to be about tapping into the potential and power of Lambda from within your CFML apps.

### What Can Lambda Do for You?

Lambda is a gateway to all of the services in AWS. Lambda is a radical shift in our understanding of what it costs to run code, and, therefore, applications. Lambda is an opportunity to introduce other languages into your deployment stack without having to have expertise in running those language runtimes in production. Lambda is a new way to manage servers running in AWS. Lambda is a data transformation engine. Lambda is a backend to millions of API requests per second.

Lambda can be pretty much whatever you want it to be.

At its core, Lambda is just a runtime for code. As of February, 2019, the following languages are supported natively by Lambda:

- Java (which is how FuseLess runs)
- NodeJS
- Python
- C#
- Go
- Ruby
- Powershell

Lambda also has an [API to create additional, custom runtimes](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-custom.html). PHP is already a popular custom runtime. There's a [list of current runtimes over on GitHub](https://github.com/mthenw/awesome-layers). One day, there might even be a CFML custom runtime so that you won't have to bundle a customized CFML runtime .jar to deploy CFML to Lambda.

But if Lambda can't run CFML natively, why bother?

As a manager (and as someone who develops software), I'm deeply concerned with my team members growing their skills beyond what's new in CFML. Runtimes like Node, Python, and Go provide an incredibly rich set of modules on top of great runtimes that can significantly speed time to market for new features. CFML has some of this &mdash; and the [Ortus](https://www.ortussolutions.com) team is doing great work building out a CFML module ecosystem with [ForgeBox](https://www.forgebox.io). However, CFML is not always the best tool for the job. Sometimes other languages are plainly superior. I'd go with Python over CFML for data analysis any day.

I do not want to set up servers to run my Python code. The setup and maintenance of servers &mdash; especially for non-consistent workloads &mdash; is a gross misuse of my time. Lambda avoid this problem, gives me flexibility, and makes it easy for me to grow my developer toolbox.

Working with Lambda isn't all unicorns and rainbows. There are some significant pain points, especially around tooling and observability, but those issues are mitigated bit by bit every day. I'll point out these pain points as we go along, and if things hold up with Lambda as they do with most AWS services, my complaints will be largely obviated a year after this post is published.

### Working with Lambda from CFML

I'll once again use [my AWSPlaybox application](https://github.com/brianklaas/awsPlaybox) for all the example code.

> Remember: these code examples assume that you have the AWS SDK .jar in your CFML runtime's /lib directory. If you need guidance on this basic setup, please see [this post](/aws/coldfusion/2018/12/10/Update-On-Using-AWS-Java-SDK-With-ColdFusion-2018.html).

As with all AWS services, you need to first create a client for the service that you want to use. This was covered in [the basic setup needed to access AWS from CFML](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html) post, but here are the steps involved:

1. Create a Client Builder object for the service &mdash; AWSLambdaClientBuilder.
2. Tell the Client Builder what kind of builder object you want to use. It's simplest to use the standard builder.
3. Pass in your credentials via the StaticCredentialsProvider object (created upon instantiation of awsPlaybox/model/awsServiceFactory.cfc).
4. Tell the Client Builder which [AWS region](https://docs.aws.amazon.com/general/latest/gr/rande.html) you're working in. (I generally use 'us-east-1'.)
5. Tell the Client Builder to build (make) the connection to the service.

Here's the relevant code from AWSPlaybox/model/awsServiceFactory.cfc:

{% highlight javascript %}
case 'lambda':
  javaObjectName = "com.amazonaws.services.lambda.AWSLambdaClientBuilder";

serviceObject = CreateObject('java', '#javaObjectName#').standard().withCredentials(variables.awsStaticCredentialsProvider).withRegion(#variables.awsRegion#).build();
{% endhighlight %}

Now we can work with Lambda from within our CFML application. 

In the rest of this series of posts on working with Lambda from CFML, I'll cover writing and deploying a function in Lambda, using that function from within your CFML app, issues of debugging and observability in Lambda, and discuss more use cases for calling Lambda from your CFML applications.
