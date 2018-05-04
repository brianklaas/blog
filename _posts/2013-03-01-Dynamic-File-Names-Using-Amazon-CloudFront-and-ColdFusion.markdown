---
layout: post
title:  "Dynamic File Names Using Amazon CloudFront and ColdFusion"
date:   2013-03-01 13:11:00 -0500
categories: AWS ColdFusion
---

There are a lot of use cases which require you to store customer-uploaded files with a system-generated file name instead of the original file name that the file had when the customer uploaded the file. You may want to ensure that every customer-uploaded file has a unique file name. You may want to provide a random file name so that someone poking through a server directory or Amazon S3 bucket doesn't see file names like "income_taxes_jill_smith.pdf" You may have a specific file-naming convention that's used for storing files in multiple hierarchies in your application. Whatever the reason, you also often need to change the name of the file when it's requested by the customer after the initial upload. Jill Smith will want her file with the original name, and not "ks8pqz0hs7.pdf"

This is simple enough to achieve by using the [content-disposition HTTP response header](http://en.wikipedia.org/wiki/List_of_HTTP_header_fields). You can set this really easily in ColdFusion:

{% highlight javascript %}
<cfheader name="Content-Disposition" value="filename=someFileName.pdf">
{% endhighlight %}

We recently started a major project wherein we will be delivering tons of lecture content in MP4/WebM format through Amazon CloudFront. This will help speed delivery of the video content to our truly global audience and reduce bandwidth consumption on our network. However, one of the major hurdles we ran into when setting up our CloudFront distribution was figuring out how to dynamically rename files when our learners requested them. We store the video files with very generic names in our S3 buckets (or, previously, on our server file systems). This lets us avoid having to rename the video files every time the course instructors rename their lectures, or change lecture numbers, or reorder content in their course or even within a specific lecture. Having to manually update thousands of files every quarter is a tedious, time-consuming, and error-prone process. We have all the metadata about a lecture stored in a database, so we can use that to dynamically rename the video files when they are requested.

Out of the box, CloudFront doesn't allow you to pass along response headers like content-disposition. If you try to append HTTP headers to your URL string that points at CloudFront, they'll simply be ignored. Amazon requires that you pass along specifically-named query parameters as the HTTP headers you want sent back from CloudFront/S3. The supported HTTP headers you can use are listed in the [documentation for GET requests to S3](http://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html). (Remember, we're using S3 as our origin for all CloudFront requests. If you are hosting your own origin server, you have to handle query strings with HTTP parameters on your own.) All supported HTTP headers that can be sent back in the response are prefixed with response.

However, you'll get a S3 access error message if you try to simply append ?response-content-disposition=attachment;filename=somefile.pdf to a CloudFront URL. (This assumes you are using S3 as your origin server. If you're using a custom origin, it's up to you to determine how you handle query strings and response headers.) As the docs say, you must sign the request, either using an Authorization header or a pre-signed URL, when using these parameters. They can not be used with an unsigned (anonymous) request. So what does this mean? What do you need to do in order to pass along response headers to a CloudFront request?

In order to dynamically name files on a per-request basis in CloudFront, you have to do the following:

- Sign your CloudFront request URLs using a valid CloudFront key pair
- Set up your CloudFront distribution with an origin access identity

If you don't have URL signing for CloudFront requests working, please see my post on [http://www.iterateme.com/blog/index.cfm/2013/2/11/Creating-Signed-URLs-for-Amazon-CloudFront-in-ColdFusion](creating signed CloudFront URLs with ColdFusion). You must have everything working with basic/"canned" signed URLs before moving forward.

You also need to add an "Origin Access Identity" to the CloudFront distribution so that S3 sees a named user making a request when CloudFront passes the request to S3. Only named, authorized users can pass query strings which include response-disposition headers to S3. More information on [Origin Access Identities](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html) can be found in Amazon's CloudFront documentation.

To add an origin access identity to an existing CloudFront distribution in the CloudFront Management Console, do the following:

- Next to the distribution where you need to add an origin access identity, click the [i] button
- Click the "Origins" tab
- Click the checkbox next to your S3 origin bucket, and select "Edit"
- For "Restrict Bucket Access" select "Yes"
- For the "Origin Access Identity", select "Create a New Identity" if you have none. Otherwise, use an existing origin access identity.
- For "Grant Read Permissions on Bucket," you have to decide the following:
  - If you want people &mdash; including developers on your own team &mdash; to be able to directly access your objects in S3 for any reason, you cannot let CloudFront update the bucket policy. You must also have correct permissions set up in your S3 bucket to allow anyone to read your objects in that bucket.
  - If you don't want people to be able to directly access your objects in S3 and only want them to go through CloudFront, then you want CloudFront to update the bucket policy.
- Save your changes.

As with all CloudFront distribution updates, it takes about 10 minutes for your distribution to be properly updated.

Now you can add response- HTTP headers to the URLs that you pass into your CloudFront URL signing method. If you use the code from my post on [creating signed CloudFront URLs with ColdFusion](http://www.iterateme.com/blog/index.cfm/2013/2/11/Creating-Signed-URLs-for-Amazon-CloudFront-in-ColdFusion), here's what you would need to add:

{% highlight javascript %}
<cfset cloudFrontObjURL = "https://" & distributionDomain & "/" & s3ObjectKey & "?response-content-disposition=attachment%3Bfilename%3DsomeReadableFileName.pdf" />
{% endhighlight %}

Amazon AWS is very, very picky about proper URL encoding, so make sure you have properly escaped non-alphanumeric characters in your response-content-disposition header (or any custom header you may use as allowed in the Amazon S3 docs). Note, though, that the = sign after response-content-disposition is not escaped, because S3 won't recognize it as a valid response header parameter if it is.
