---
layout: post
title:  "Slides and Resources for My ColdFusion Summit 2015 Presentation on Using AWS Services from CF9"
date:   2015-11-12 10:02:00 -0500
categories: AWS ColdFusion Conferences
---

This year's ColdFusion Summit had the biggest attendance of any ColdFusion Summit thus far, and was another good conference in a year of pretty darn good conferences. Although my session on using [Amazon Web Services](https://aws.amazon.com) from ColdFusion was scheduled in the last slot of the last day, I had a great turnout with an enthusiastic response and some great questions.

Below are links to both my slides with and without presenter notes, and to the GitHub repo for the demonstration application that I used in the presentation:

- [Slides](https://dl.dropboxusercontent.com/u/139065/presentationResources/CFSummit2015-CFandAWS.pdf)
- [Slides with presenter notes](https://dl.dropboxusercontent.com/u/139065/presentationResources/CFSummit2015-CFandAWS-presenterNotes.pdf)
- [GitHub repo for the demo application](https://github.com/brianklaas/cfsummit2015)

The demo application shows you how to connect to the following Amazon Web Services from ColdFusion:

- Simple Notification Service
- Lambda
- DynamoDB

The slide deck also talks about CloudFront, but there isn't any demo code for that. The repo for the [CTL CloudFrontUtils](https://github.com/brianklaas/ctlCloudFrontUtils) package is publicly available, and should help you a lot if you need to sign CloudFront URLs in ColdFusion. We also have a similar package for [signing requests to Simple Storage Service (S3) from ColdFusion](https://github.com/brianklaas/ctlS3Utils), as you'll need that if you want to do change file names or set other attributes on the fly when working with S3 from ColdFusion.

The GitHub repo has pretty lengthy instructions on how to get it set up, and includes the list of resources you'll need to set up in AWS in order to get everything to work. It may seem like a lot of setup, but one glance at the code should show you how very simple it is to interact with Amazon Web Services from ColdFusion once you do have the basic resources in place.

Don't forget that there is also the AWS sub channel on [the CFML Slack channel](http://cfml.slack.com). I'm often there to answer questions and, if not, there are other knowledgeable CFMLers who current work with AWS who can answer questions as well.