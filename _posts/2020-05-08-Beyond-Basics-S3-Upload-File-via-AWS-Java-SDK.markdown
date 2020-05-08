---
layout: post
title:  "Beyond the Basics of Using AWS S3 in CFML: Uploading a File via the AWS Java SDK"
date:   2020-05-08 08:58:00 -0400
categories: AWS ColdFusion
---

Our first stop in this new series on going beyond the basics of using AWS S3 in CFML is learning how to upload a file to S3 via the AWS Java SDK. [CFML runtimes have long had support for uploading files to S3 using built-in CFML functions and tags](https://brianklaas.net/aws/coldfusion/2018/05/24/Using-Simple-Storage-Service-In-CFML-Part-1.html). It's important to know how to do this with the AWS Java SDK, however, because all of the advanced features of using S3 from CMFL require use of Java file objects in conjunction with requests to S3.

Additionally, and perhaps more importantly, you can use [minimal IAM permissions](https://www.youtube.com/watch?v=ucn90XLDlPw) to upload files via the AWS Java SDK. The built-in file tags and functions in Adobe ColdFusion require more than 20 separate S3-specific IAM permissions to work. Some of these permissions include bucket-level permissions. This presents a large potential attack surface should someone get ahold of your IAM credentials. AWS best practice is that you give _absolutely minimal_ permission for an IAM user, group, or role. By using the AWS Java SDK to upload (and manipulate) files in S3, you can significantly reduce your potential attack surface and have truly secure uploading. Your IAM permissions to upload to S3 can be as simple as this:

{% highlight json %}
{
    Effect: "Allow",
    Action: [
        "s3: PutObject"
    ]
}
{% endhighlight %}

That's a whole lot more secure than needing 20 or more permissions for a simple file upload.

## Show Me the Code!

As always, the full code for this and all of my demos can be found in my [AWSPlaybox GitHub repo](https://github.com/brianklaas/awsPlaybox).

In awsplaybox/model/awsServiceFactory.cfc, I generate a reference to the S3 service itself. This is done via the com.amazonaws.services.s3.AmazonS3ClientBuilder class. Then, assuming I have a file input on a form, I can upload a file to S3 as follows:

{% highlight html %}
    s3 = application.awsServiceFactory.createServiceObject('s3');
    uploadedFile = fileUpload(getTempDirectory(), "form.fileToPutOnS3", " ", "makeunique");
    fileLocation = getTempDirectory() & uploadedFile.serverFile;
    // The AWS SDK putFileRequest object requires a Java file object in binary format
    fileContent = fileReadBinary(getTempDirectory() & uploadedFile.serverFile);
    javaFileObject = CreateObject('java', 'java.io.File').init(fileLocation);
    putFileRequest = CreateObject('java', 'com.amazonaws.services.s3.model.PutObjectRequest').init(trim(form.s3BucketName, uploadedFile.serverFile, javaFileObject);
    s3.putObject(putFileRequest);
{% endhighlight %}

The code above is fairly straightforward, but let me point out a couple of key points:

- S3 works with java.io.File objects (and, as we'll see, fileInputStreams). This kind of object expects a binary file as input. That's why we have to read the uploaded file &mdash; no matter the content &mdash; as a binary file.
- The AWS Java SDK utilizes a Request/Response object pattern throughout the majority of services. You pass in some kind of Request object with all the parameters for the API request, and you receive a Response object back with the results. 
- You don't "upload a file," you make a "put object request" in S3. There are no files in S3. There are only objects. 
- We tell our reference to the S3 service to "putObject" using the "putFileRequest," and that's what uploads our file to S3.

This is absolutely more verbose than [uploading a file to S3 in CFML](https://brianklaas.net/aws/coldfusion/2018/05/24/Using-Simple-Storage-Service-In-CFML-Part-1.html). As you'll see in upcoming posts, however, knowing how to do this unlocks the real power of S3 and all the additional features and functionality you have beyond "putObject."