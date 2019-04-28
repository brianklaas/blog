---
layout: post
title:  "Using AWS Step Functions in CFML: Looping in Step Functions Workflows via the Wait State"
date:   2019-05-28 15:51:00 -0400
categories: AWS ColdFusion
---

At this point in the [second example Step Functions workflow](https://github.com/brianklaas/awsPlaybox/blob/master/stateMachines/transcribeTranslateSpeakWorkflow.json), we're waiting for our [AWS Transcribe](https://brianklaas.net/aws/coldfusion/2018/09/14/Using-AWS-Transcribe-in-CFML-Part-1.html) job to complete. AWS Transcribe works faster than real-time, but it's not instantaneous. If you're trying to transcribe a 90 minute podcast or 70 minute video, it's going to take more than a few minutes for Transcribe to do its work. Our workflow of translating the transcription cannot continue until this work is done.

So how do we wait until an asyncrhonous task like this is done? How do we even know when it's done?

### Looping in Step Functions Workflows

In order to 