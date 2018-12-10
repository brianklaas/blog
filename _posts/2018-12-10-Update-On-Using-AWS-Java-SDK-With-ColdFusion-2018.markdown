---
layout: post
title:  "Update on the Basic Setup for Using the AWS Java SDK with ColdFusion 2018"
date:   2018-12-10 9:24:00 -0500
categories: AWS ColdFusion
---
In most of my posts on using AWS services from within CFML, I've pointed to [my original post on how to use the AWS Java SDK from within your CFML engine](/aws/coldfusion/2018/05/21/Basic-Setup-Needed-To-Access-AWS-From-CFML.html). However, the information on that page about the required .jar files for using the AWS Java SDK no longer applies to the [Adobe ColdFusion 2018 release](https://www.adobe.com/products/coldfusion-family.html).

If you are running Adobe ColdFusion 2018, **you do not need to add anything other than the AWS Java SDK .jar file itself to your cfusion/lib directory** to have access to the full power of the AWS Java SDK. ColdFusion 2018 includes the jackson-annotations, jackson-core, jackson-databind, and joda-time .jar files out of the box. The versions included with Adobe ColdFusion 2018 Update 1 are actually more recent than those included with the AWS Java SDK 1.11.x download!

If you are running Adobe ColdFusion 2016 or ColdFusion 11, however, you will still need to add the following to your cfusion/lib directory:

- aws-java-sdk-1.11.xxx.jar
- jackson-annotations-2.6.0.jar
- jackson-core-2.6.7.jar
- jackson-databind-2.6.7.1.jar
- joda-time-2.8.1.jar

All of these .jar files are in the AWS Java SDK download. The AWS SDK .jar is in the /lib directory, and the others are in the /third-party directory.

If your CFML engine is [Lucee](http://lucee.org/), add those files to the /WEB-INF/lucee/lib/ directory in the root of your site. If any of those .jar files exist and are newer than the ones listed above, you don't need to replace them.

So tapping in to the awesome power of AWS from the Java SDK just got a lot easier with the Adobe ColdFusion 2018 release. It's just one .jar you have to add (or get your operations team to add), and you're done!

If you have any questions about any of the posts in this series, feel free to [message me on the Twitter](https://twitter.com/brian_klaas)!