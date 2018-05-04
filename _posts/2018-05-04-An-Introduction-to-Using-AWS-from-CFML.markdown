---
layout: post
title:  "An Introduction to Using AWS from CFML"
date:   2018-05-04 13:37:00 -0400
categories: AWS ColdFusion
---

I've spent a lot of time in the past few years speaking at conferences about using [Amazon Web Services](https://aws.amazon.com/) from within CFML applications. I've covered many services and how to take advantage of them using the [AWS Java SDK](https://aws.amazon.com/sdk-for-java/). If you haven't attended one of my conference sessions, you likely haven't seen how this works. This series of blog posts aims to provide a basic understanding of both how the tools in AWS work, and how to utilize them from within CFML.

This series will explain services using my [AWSPlaybox application, available on GitHub](https://github.com/brianklaas/awsplaybox/). The AWSPlaybox app has code examples for the following AWS services:

- [S3 &mdash; Simple Storage Service](https://aws.amazon.com/s3/)
- [SNS &mdash; Simple Notification Service](https://aws.amazon.com/sns/)
- [Lamdba](https://aws.amazon.com/lambda/)
- [CloudWatch](https://aws.amazon.com/cloudwatch/)
- [DynamoDB](https://aws.amazon.com/dynamodb/)
- [Step Functions](https://aws.amazon.com/step-functions/)
- [Rekognition](https://aws.amazon.com/rekognition/)
- [Transcribe](https://aws.amazon.com/transcribe/)
- [Translate](https://aws.amazon.com/translate/)
- [Polly](https://aws.amazon.com/polly/)

I will not be going over running an actual CFML server within AWS. There are many options for running CFML servers in [EC2](https://aws.amazon.com/ec2/), the vast array of virtual servers that Amazon offers to you. You could also run Adobe ColdFusion or Lucee in a Docker container on [Elastic Container Service (ECS)](https://aws.amazon.com/ecs/), [Elastic Container Service for Kubernetes (EKS)](https://aws.amazon.com/eks/), or [Fargate](https://aws.amazon.com/fargate/). However, my focus is on showing how to use the *application services* from AWS within your CFML applications, not running CFML engines on AWS.

Each post in the series will build upon previous posts. As such, it's important to at least start with the intitial post on the basic setup of the AWSPlaybox app, and how AWS credentials are handled.

> Please note that I will be giving only a cursory glance to IAM, the permissions model in AWS. There is extensive (and excellent) [documentation on using IAM in AWS](https://aws.amazon.com/documentation/iam/), and will not cover it in detail in this series beyond saying "put your access key and secret key here."

You'll also note that, as the series progresses, you'll see more and more examples in JavaScript. That's because we'll be building [Step Function](https://aws.amazon.com/step-functions/) workflows that rely on Node.js-based functions in [Lamdba](https://aws.amazon.com/lambda/), a serverless execution environment. Invoking these workflows from CFML is covered, and there will be code examples for that.

My hope is that you will come to see how easy it is to tap into the vast computing power that AWS provides from within your CFML applications. There are whole new classes of functionality that you can add to your apps when utilize the offerings from AWS. I want to show you how easy it is to get started.

First up: the basics of using the AWS Java SDK from within your CFML application. I'll cover how to get set up, how to pass in your IAM credentials, and how to create basic service objects.