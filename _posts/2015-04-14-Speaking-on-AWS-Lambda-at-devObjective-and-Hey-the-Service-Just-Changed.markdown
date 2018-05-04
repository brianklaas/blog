---
layout: post
title:  "Speaking on AWS Lambda at dev.Objective() and, Hey, the Service Just Changed"
date:   2015-04-14 09:21:00 -0400
categories: AWS ColdFusion Conferences
---

I'm very lucky to be speaking at [dev.Objective()](http://www.devobjective.com) again this year. It's an awesome conference, especially as it has evolved from cf.Objective() into something more broadly appealing to Web application development professionals while still retaining a lot of its ColdFusion roots.

This year, my talk is "[Node Without Servers: Event-Driven Computing with AWS Lambda](http://www.devobjective.com/sessions/node-without-servers-event-driven-computing-with-aws-lambda)." [AWS Lambda](http://aws.amazon.com/lambda/) is a different way of looking at running "server"-side code: you run code, but you don't run servers. You're not really running web applications as well. You're only running a single function at a time (although that function is more like a main() in Java or C or Go). Even more interestingly, these functions respond to events from other systems, not a request from a Web browser.

The specific topics I plan on covering in this session are:

1. How Lambda works and what makes it different from traditional Node hosting
2. How Lambda listens to events from services like Amazon S3
3. Building functions which respond to your own custom events
4. Using languages other than Node in Lambda
5. Best practices for using Lambda in your Web application stack

I've been working on this presentation a lot in the past few months, and then something fun happened last Thursday (April 9): [Lambda left preview status and is now a full production service at AWS](https://aws.amazon.com/blogs/aws/aws-lambda-update-production-status-and-a-focus-on-mobile-apps/). That's awesome. They made a bunch of very nice improvements to the service (like the addition of [SNS messages as an event source for invoking Lambda functions](http://docs.aws.amazon.com/sns/latest/dg/sns-lambda.html)). They even changed some of the basic workflow to make life simpler for developers &mdash; except that they completely changed the way custom events work.

This has led to some scrambling on my part to get a revised presentation done by the dev.Objective() steering committee deadlines, but I feel like I have to show how things are done now and not rely on tested code which uses methods that were completely deprecated of last Thursday.

This is one of the fun challenges as a developer both using and presenting on services and technologies which are still in "preview" or otherwise under active development in a less than 1.0 release branch. Node itself still isn't at 1.0 (far from it!) but the development team has made a concerted effort not to make breaking changes with each release. There may be some small breaking changes with each release (and, more rarely, large ones), but that's the price you pay for working with newer technologies. Delivering increased customer value and developer happiness is half the reason you have a preview period for your new technology or service, and for those of us willing to make the leap into using services during preview, it's a simple fact we have to weigh in making our technology choice decisions.