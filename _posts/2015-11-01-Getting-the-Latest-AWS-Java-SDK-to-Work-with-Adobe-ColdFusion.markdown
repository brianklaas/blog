---
layout: post
title:  "Getting the Latest AWS Java SDK to Work with Adobe ColdFusion"
date:   2015-11-01 14:44:00 -0500
categories: AWS ColdFusion
---

I'm really happy to speak again at this year's [Adobe ColdFusion Summit](https://cfsummit.adobeevents.com/). My talk next Tuesday is titled "[Expand Your ColdFusion App Power with Amazon Web Services](https://cfsummit.adobeevents.com/schedule/sessions/expand_your_coldfusion_app_power_with_amazon_web_services.html)." People who know me know that I'm all about [AWS](http://aws.amazon.com/) and the awesome power that their ever-expanding suite of services provides. I'm also about ColdFusion and how incredibly easy it makes app building on top of the Java Virtual Machine, and how easy it is to tap in to the hundreds of thousands of Java libraries that are out there to make your ColdFusion apps more powerful.

One of those libraries is the AWS Java SDK, and you're going to need that library if you want to do anything in AWS that goes beyond the simple, easy, kinda powerful support that ColdFusion has for [Amazon S3 (Simple Storage Service)](https://aws.amazon.com/s3/). [ColdFusion makes it drop-dead easy to utilize S3](http://help.adobe.com/en_US/ColdFusion/9.0/Developing/WSd160b5fdf5100e8f-4439fdac128193edfd6-7f08.html) because it provides a series of tag and function-based wrappers for the underlying AWS Java SDK to access services within S3.

If you want to do more with AWS in Adobe ColdFusion than just utilize S3, however, you're going to need to create objects and call methods on the objects in the AWS Java SDK. The problem you'll quickly run into is that the version of the AWS Java SDK bundled with Adobe ColdFusion &mdash; even ColdFusion 11 &mdash; is pretty old. If you want to use any of the services introduced in the last two years, like [Lambda](https://aws.amazon.com/lambda/) or the API Gateway, you're going to need to use an updated version of the AWS Java SDK.

You cannot, however, just drop the latest AWS Java SDK into your cfInstallRoot/lib directory and be done with it. You also can't use [the new Java class loader introduced in Adobe ColdFusion 11](https://helpx.adobe.com/coldfusion/developing-applications/using-web-elements-and-external-objects/integrating-jee-and-java-elements-in-cfml-applications/enhanced-java-integration-in-coldfusion.html) to load up the AWS Java SDK .jar files. That's because a version of the core AWS Java SDK .jar file is already in cfInstallRoot/lib and trying to load a new version on top of that .jar using Java loader in Adobe ColdFusion 11 and beyond will conflict with the one that's loaded during app server startup.

I'll be covering this briefly in my talk, so I thought it would be best to post instructions on how to use the latest AWS Java SDK in Adobe ColdFusion for both those in attendance at the talk and for future general reference.

To get the latest AWS Java SDK working in Adobe ColdFusion 10 or later, do the following:

1. Download the latest SDK from (https://aws.amazon.com/sdk-for-java)

2. Move the current aws-java-sdk-x.x.xx.jar file from your cfInstallRoot/lib elsewhere for backup.

3. Move the aws-java-sdk.jar from the download into your cfInstallRoot/lib directory.

4. Inside the "third-party" folder in the download, move the following .jar files into your cfInstallRoot/lib directory:
    - jackson-annotations
    - jackson-core
    - jackson-databind
    - joda-time
    
5. Restart ColdFusion.

It used to be that the AWS Java SDK was one monolithic .jar which included all third-party dependencies. Amazon changed things a while back to make it easier to manage dependencies using tools like [Maven](https://maven.apache.org/) by breaking out the SDK's dependencies. This is why you can't just drop the  aws-java-sdk-x.x.xx.jar file into the cfInstallRoot/lib directory and have things work.

The AWS Java SDK actually requires quite a few more third-party dependencies but fortunately most of those are already bundled with Adobe ColdFusion (version 10 or later) and have only patch version differences between what's in bundled with Adobe ColdFusion and what's bundled with the AWS Java SDK. I've not experienced any problems at all on that front.