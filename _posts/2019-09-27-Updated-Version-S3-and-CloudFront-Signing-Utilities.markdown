---
layout: post
title:  "Updated Versions of My AWS S3 and CloudFront Request Signing Utilities"
date:   2019-09-27 16:39:00 -0400
categories: AWS ColdFusion
---

About six years ago, I released two utilities for cryptographically signing requests for objects in btoh AWS S3 and CloudFront. The code in those repos still works just fine, but the code relies on bespoke methods for generating the request signature. Back then, all we really had was bespoke methods, and had to do things in a more roundabout way, utilizing multiple Java libraries. Things have definitely changed for the better in the intervening six years.

I recently rewrote these components to utilize native methods in the [AWS Java SDK](https://aws.amazon.com/sdk-for-java/). Not only are these methods more succinct, they also generate signatures that are compatible with [version 4 of the AWS request signing process](https://docs.aws.amazon.com/general/latest/gr/sigv4_signing.html). All AWS SDKs generate these by default, which is another reason to use the AWS SDKs over creating bespoke methods. Additionally, as I've detailed in a previous blog post, [AWS will soon require v4 request signing on all newly-created S3 buckets](https://brianklaas.net/aws/coldfusion/2019/06/20/End-Of-V2-Signature-Requests-For-S3.html).

The refactored components are shorter, and simpler, and, I think, easier to use. 

You can find the updated components here:

- [S3 Request Signing Utility](https://github.com/brianklaas/ctlS3Utils)
- [CloudFront Request Signing Utility](https://github.com/brianklaas/ctlCloudFrontUtils)

### Why Would I Want to Use These?

These utilities are designed to let you use some of the very cool features of S3 and CloudFront when it comes to requesting files via a URL. You can specify an expiration date and time for a URL. You can change the file name from what's stored in S3. You can specify if the file should be retrieved inline (displayed in browser) or as an attachment (downloaded). You can specify the version of an object in the CloudFront cache.

In order to do any of the above (and more), you have to cryptographically sign your requests (URLs). That's exactly what these utilities are for.