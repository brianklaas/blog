---
layout: post
title:  "Beyond the Basics of Using AWS S3 in CFML: Restricting Access to Files in S3 by Signing URLs"
date:   2020-05-12 16:48:00 -0400
categories: AWS ColdFusion
---

Our next stop in this series on going beyond the basics of using AWS S3 in CFML is learning how to protect content in a S3 bucket by only allowing access via digitally signed URLs. 

Far too many people make the mistake of making their S3 buckets world-readable. This is quick and easy to do, and makes it so you can just link to files in S3 just like you would link to a file on any web server. The problem with this approach is that it's very insecure. Anyone can access your world-readable S3 buckets and steal that content. 

If you are storing any kind of personal files or personally identifying information, or any data that you don't want the world to see, you need to turn of world read access to your S3 buckets. 

Let me correct myself: 

If you're storing any kind of information in a S3 bucket that's not a static website, <b>turn off world-read right now</b>. You can provide access to the files in buckets without world-read access by providing digitally signed URLs instead. This ensures that your content cannot be hijacked by third parties and is not reachable except when you want it to be reached.

Signed URLs allow access via a direct link to download a file or display content inline in a web page. You can specify the expiration time of the URL, which can be anywhere from one second to one week. If you're protecting content on your website via signed URLs, you probably want to make the expiration time a few seconds. Be careful, though, because if the time it takes to download or completely transfer file data is longer than the expiration time in the signed URL, the transfer will get interrupted before the transfer completes. AWS is pretty smart to kill in-progress requests if the signature expires.

## Making Signing URLs Easiser

Signing URLs involves working with the com.amazonaws.services.s3.model.GeneratePresignedUrlRequest class. It's fairly straightforward to use, but there's often more to accessing files in a web applicaiton than just accessing the file. You may not be able to store a file &mdash; particularly if it's a user-generated file &mdash; using the orignial file name. No one would feel comfortable uploading their "2020taxes.pdf" file to a service that didn't rename the file so a curious employee with access to data storage wouldn't be able to quickly spot a prime target. You also often need to be able to specify if a file should be delivered as an attachment (downloadable) or inline. [At my job](https://courseplus.jhu.edu/), we allow downloads of online course videos in addition to in-browser playback. We want the content to be protected regardless of the method of delivery. Finally, S3 doesn't automatically store file MIME types. It's not file storage. S3 is object storage, and MIME types are metadata that are of no importance to an object storage system. You'll quickly find that you often need to add the correct MIME type to a file in order for the browser (ahem: Firefox) to understand how to handle the file. 

In order to handle these common tasks, I created [my S3SigningUtils component](https://github.com/brianklaas/ctlS3Utils). It's a wrapper around the various options you can set when generating a digitally signed URL to a file on S3. It contains one method, with a straightforward method signature:

{% highlight javascript %}
public string function createSignedURL(
    required string s3BucketName, 
    required string objectKey
) {}
{% endhighlight %}

By default, the URLs generated via this method expire one hour after creation. The full method signature shows the various options you can pass, including specifying:

- the exact date/time the URL will expire, up to one week from now()
- the file name for the request, instead of using the name of the file as stored on S3
- if the file should be delivered as an attachment (download) or inline
- the MIME type header which should be added to the file on return

{% highlight javascript %}
public string function createSignedURL(
    required string s3BucketName, 
    required string objectKey, 
    date expiresOnDate = dateAdd("h",1,Now()), 
    string fileNameToUse = "", 
    boolean isAttachment = false, 
    string mimeType = ""
) {}
{% endhighlight %}

The component code itself is pretty straightforward, but it's a lot easier to use this one method than to create the many little objects which the com.amazonaws.services.s3.model. GeneratePresignedUrlRequest.generatePresignedUrl method uses.

After you generate the signed URL, you use it exactly as you would any other URL. Using it, however, makes your &lt;a href&gt;s a lot more secure!

As always, if you have any questions about this, feel free to [message me on the Twitter](https://twitter.com/brian_klaas).