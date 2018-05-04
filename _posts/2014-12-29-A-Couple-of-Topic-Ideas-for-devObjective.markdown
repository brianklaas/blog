---
layout: post
title:  "A Couple of Topic Ideas for dev.Objective()"
date:   2014-12-29 08:49:00 -0500
categories: AWS ColdFusion Conferences
---

cf.Objective() has long been a great ColdFusion conference, and one of my personal favorite speaking engagements. The organizers of cf.Objective() changed the conference name to [dev.Objective()](http://www.devobjective.com) this year to better match the nature of talks given at the conference and to target a wider audience for the great content presented there.

[The call for speakers for dev.Objective() is currently open](http://engage.devobjective.com), and I'm putting in a couple of topic proposals this year. I normally only put in a single, CFML-related topic proposal for the conference, but given the expanded nature of the conference, I thought it would be cool to talk about some of the non-ColdFusion work I'm involved in.

First up, I'm always happy to talk about Amazon Web Services and how ColdFusion fits in with the pretty awesome power of AWS. So my first proposed topic will be an update to my talk on using CF and AWS: "Unlock the Power of Amazon Web Services in Your ColdFusion Apps." There is **so** much to cover in this topic. From automatic integration with Amazon S3, to using the official ColdFusion AMI in AWS, to the myriad other AWS services that you can use with CF, there's a ton of opportunities for mixing CF and AWS.

The second proposal is about something new to both AWS and to me: [AWS Lambda](http://aws.amazon.com/lambda/). It's a different way of looking at running \"server\"-side code: you run code, but you don't run servers. You're not really running web applications as well. You're only running a single function at a time (although that function is more like a main() in Java or C or Go). Even more interestingly, these functions respond to events from other systems, not a request from a Web browser.

The full topic proposal is below, and if you have any feedback about it, I'd love to hear it!

<hr/>

Node Without Servers: Event-Driven Computing with AWS Lambda

All the buzz in modern Web development seems to be about the client side, yet many tasks still require servers and server infrastructure. Many people use Node.js for their server-side code, but this still requires setting up and maintaining servers. What if you could run your JavaScript in the cloud, utilizing the full range of packages available to Node.js, without running any servers at all? What if your cloud-based code could automatically respond to events that happen in other services and entirely separate systems? In this session, we'll take a look at how you can use Lambda, a new part of Amazon Web Services (AWS), to run your Node.js code in an on-demand, event-driven fashion. In this session, you'll learn:

1. How Lambda works and what makes it different from traditional Node hosting
2. How Lambda listens to events from services like Amazon S3
3. Building functions which respond to your own custom events
4. Using languages other than Node in Lambda
5. The trade offs between running your own Node-based infrastructure and using Lambda
