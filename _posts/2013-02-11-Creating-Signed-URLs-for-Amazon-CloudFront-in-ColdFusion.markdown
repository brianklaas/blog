---
layout: post
title:  "Creating Signed URLs for Amazon CloudFront in ColdFusion"
date:   2013-02-11 13:40:00 -0500
categories: AWS ColdFusion
---

My team is in the process of shifting all of our online course content from being delivered in a Flash-based format to delivery in a video-based format. At the same time, we're trying to figure out ways of improving delivery of this content to our global audience. We've got students all over the world, and in some fairly low-resource areas (where they have Internet access from 3-5pm on Tuesdays and Fridays, for example). One of the strategies we're going to employ is distribution of our online course content via a CDN. Although we've used Akamai as a CDN in the past, we're going with Amazon S3 and [CloudFront](http://aws.amazon.com/cloudfront/) because of cost and integration benefits.

Some of the content we're going to distribute needs to be protected, however. Although we can claim "fair use" for some copyright-protected content, we have to make sure that only individuals in our online courses have access to this content. We want to take advantage of the global distribution network that Amazon CloudFront provides, but we also need to make sure we limit who can access that content. Enter signed URLs in CloudFront.

[As the CloudFront docs explain](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/PrivateContent.html), signing URLs allows you to specify that the current request really should have permission to access a resource in CloudFront (or, more specifically, a S3 bucket or on some kind of other origin server) even though general requests to that object in CloudFront are blocked. This is how companies like O'Reilly let you download an ebook that you've bought but prevent those that have not bought the book from downloading the book using the same URL.

There are a number of SDKs available from Amazon for CloudFront integration. As you might expect, however, there are no SDKs specifically for ColdFusion. There are some libraries available from the ColdFusion community &mdash; [Simon Free's AWS Wrapper on GitHub](https://github.com/simonfree/cfAWSWrapper) is perhaps the most complete &mdash; but none included CloudFront functionality.

One of the great things about ColdFusion is that it's built on top of the JVM. As a result, we can use a huge number of Java-based SDKs in our ColdFusion apps. It's not always obvious what you need to do, but nearly all Java-based SDKs include auto-generated API documentation that make it pretty darn easy to figure out. So that's what we did.

I've split this post into two major parts: Setting up the CloudFront distribution for protected content, and the ColdFusion code. You have to do everything in the first part before proceeding to the code, or the code won't work.

<h3>Setting Up the CloudFront Distribution for Protected Content</h3>

I'm going to assume you've done the following:

- Created an AWS account
- Have at least one accessKey and secretKey created in Amazon IAM (Identity and Access Management)
- Set up a S3 bucket which will contain the content you want to protect
- Set up a CloudFront download distribution which points to the S3 bucket with your content

If not, Amazon has a ton of getting started materials to do any of the above tasks.

(I'm also only looking at download distributions here. Streaming CloudFront distributions are slightly different, but the code shown below should still work.)

To generate a signed URL for CloudFront, you're going to need both AWS IAM security credentials (the accessKey and secretKey) and a CloudFront keypair. Generating a signed URL will work with any IAM security credentials on the account for which you create a CloudFront keypair. You should have at least one accessKey and secretKey to work with &mdash; if you don't, you should [go do that now](http://aws.amazon.com/console/).

In the [CloudFront docs on creating signed URLs](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/PrivateContent.html), it says that you need to specify one or more trusted signers for your CloudFront distribution. Trusted signers are the AWS accounts that you want to have permission to create signed URLs. Each trusted signer has to have a CloudFront key pair. This is different from an EC2 key pair (if you use Amazon EC2). Amazon explains how the CloudFront key pair works when you sign URLs as follows:

> When you create a signed URL, you use the private key from the trusted signer's key pair to sign a portion of the URL. When someone uses the signed URL to access an object, CloudFront compares the signed portion of the URL with the unsigned portion to verify that the URL hasn't been tampered with.

A trusted signer is linked to a cache behavior in the setup of your CloudFront distribution. Again, [from Amazon](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-trusted-signers.html):

> Download distributions: You add trusted signers to cache behaviors. If your distribution has only one cache behavior, users must use signed URLs to access any object associated with the distribution. If you create multiple cache behaviors and add trusted signers to some cache behaviors and not to others, you can require that users use signed URLs to access some objects and not others.

So if you want to have a mixed CloudFront distribution where some URLs need to be signed (to set a new timeout expiration on a cached object in CloudFront, for example) but some URLs don't need to be signed, you would need to have multiple cache behaviors. If you want *all* content in a CloudFront distribution to be protected and require signed URLs, then you only need one cache behavior with a trusted signer.

Also, IAM users are currently not allowed to create CloudFront key pairs, so you cannot use IAM users as trusted signers. IAM users are all the accounts you set up in Amazon AWS that use the accessKey and secretKey to access Amazon AWS. So...the account that's going to create the CloudFront key pair is the account that you use to sign in to AWS. Fun!

To create a CloudFront key pair:

- Sign into the AWS portal
- Go to Account -> Security Credentials
- Click the "Key Pairs" tab
- Click "Amazon Cloud Front Key Pairs" and then "Create New Key Pair"

This will create a new key pair and automatically downloads the private key to your machine.

**It's up to you to secure this file.** Don't put it in a web-accessible directory when you deploy your CloudFront-signing code to production. Hide it away somewhere on your network and make it accessible only by a special account on the machine(s) running your code.

Once you create the key pair, you now have to alter your CloudFront distribution to authorize the account for the key pair you just created to sign URLs. To add a trusted signer to your CloudFront distribution, do the following:

- Sign into the AWS portal
- Go to the CloudFront console
- Next to the distribution where you want to have trusted signers, click the [i] button
- Click on the "Behaviors" tab
- Click "Create Behavior"
- Set up the cache behavior as needed, including the path pattern to match for this behavior. The default is to match everything (*)
- For "Forward Query Strings" select "Yes" if you are using query strings to do object versioning in CloudFront.
- For "Restrict Viewer Access (Use Signed URLs)" select "Yes"
- The default trusted signer is "self" - the account which manages your AWS distribution and for which you created the CloudFront key pair. (If you want another account to be able to sign URLs for this CloudFront distribution, you need to look up and use the canonical AWS account number.)
- Save the changes to the CloudFront distribution.

As with any CloudFront distribution, making changes will take about 10 minutes to process.

Whew! Now that the AWS setup is done, we can get to some code.

<h3>The ColdFusion Code</h3>

First up, you're going to need to get the [core AWS SDK for Java](http://aws.amazon.com/sdkforjava/). This vastly simplifies the kinds of calls you have to make. The AWS SDK has a ton of files in it, but you only need one: aws-java-sdk-1.3.30.jar

Like any .jar file you want to make available to ColdFusion at server startup, this .jar needs to be added to your cfusion/lib/ directory and you'll need to restart ColdFusion after you add the .jar. Note that I did not use [Mark Mandel's excellent JavaLoader project](http://www.compoundtheory.com/?action=javaloader.index) or the [built in .jar loading functionality in Adobe ColdFusion 10](http://help.adobe.com/en_US/ColdFusion/10.0/Developing/WSe61e35da8d318518-106e125d1353e804331-7ffe.html). I have access to my servers in both development and production, and felt this was the simplest way of handling the issue.

Next, we're going to use the [JetSet libraries](http://jets3t.s3.amazonaws.com/) for dealing with CloudFront. The AWS docs pretty much say "use these" and don't provide any other options for abstracting away CloudFront AWS calls. Fortunately for us, the jets3t-0.8.1.jar that comes bundled with ColdFusion 10 works just fine.

If you are using ColdFusion 9, you can download the latest version of the JetSet library and add it to your cfusion/lib/ directory. However, in our testing, we found that the latest JetS3t library breaks S3 functionality in the &lt;cffile&gt; tag in Adobe ColdFusion 9. Apparently, the method signatures for some functions changed between the jets3t-0.7.3.jar that comes with CF9 and the jets3t-0.8.1.jar that comes with CF10. Signing CloudFront URLs described in this post will work using the jets3t-0.8.1.jar in Adobe ColdFusion 9, but the Amazon S3 integration in the  &lt;cffile&gt; tag will break. You will have to use third-party code to interact with Amazon S3 if you go this route. Additionally, **you must remove jets3t-0.x.0.jar that comes bundled with Adobe ColdFusion 9 that's already in the cfusion/lib/ directory.** If you don't do this, ColdFusion won't be able to find all the methods that are in the jets3t-0.9.0.jar because there are versions of some methods in both .jars and ColdFusion can get confused about which version of which method should be used and gives you a "method cannot be found" error even though the method is in the source Java objects.

Once you've got those two .jars in your cfusion/lib/ directory and you've restarted ColdFusion, let's test to make sure that the AWS and JetSet .jars loaded properly. 

Create a new file with the following:

{% highlight html %}
<cfset accessKey = "YOUR ACCESS KEY" />
<cfset secretKey = "YOUR SECRET KEY" />
{% endhighlight %}

The accessKey and secretKey can be any IAM accessKey/secretKey combination in the AWS account that is your trusted signer for CloudFront URLs.

{% highlight html %}
<cfset awsCredentials = createobject("java", "org.jets3t.service.security.AWSCredentials").init(accessKey, secretKey) />\r\n<cfdump var="#awsCredentials#" />
<cfset cloudFrontService = createobject("java", "org.jets3t.service.CloudFrontService").init(awsCredentials) />
<cfdump var="#cloudFrontService#" />
{% endhighlight %}

If this runs successfully, you will see the simple dump of the JetSet AWSCredentials object followed by the various methods available to the JetSet CloudFront service object.

Add to the current file the following:

{% highlight html %}
<cfset distributions = cloudFrontService.listDistributions() />
<cfloop array="#distributions#" index="thisDistrib">
   <cfoutput><p>#thisDistrib.toString()#</p></cfoutput>
</cfloop>
{% endhighlight %}

This will return a list of all your current CloudFront distributions. If you didn't set any up, none will be listed!

If you want to play with other methods of the CloudFront service, you can see what's offered in the dump of the CloudFrontService object, or, better yet, view the JavaDocs that come with the JetSet download.

> In full disclosure, I have to mention that JetSet provided a long list of code samples that I used as the basis for my code work. Those samples can be found here: (http://jets3t.s3.amazonaws.com/toolkit/code-samples.html#cloudfront-manage)

Moving on, you're also going to need an instance of JetS3t's serviceUtils object for date conversions when signing, so add this:

{% highlight html %}
<cfset jets3tServiceUtils = createObject("java", "org.jets3t.service.utils.ServiceUtils").init() />
{% endhighlight %}

And, finally, let's not forget the JetS3t EncryptionUtil object for handling the conversion of Amazon's .pem files into the Java-readable .der file format:

{% highlight html %}
<cfset jets3tEncryptionUtils = createObject("java", "org.jets3t.service.security.EncryptionUtil").init("dkhcoaj3fh2sk0oay") />
{% endhighlight %}

Note that you have to pass some initial password to the EncryptionUtil object. It's not important what the initial password is as we are not persisting data using this EncryptionUtil object. You just need to pass in something.

We're now at the point where we can create a signed URL! There are two methods in the cloudFrontService object that let us do this: signURL and signURLCanned. signURL lets you sign a URL with a custom policy JSON document stating what the access restrictions on the file in S3 are. signURLCanned lets you sign a URL with a simple statement of when the URL will expire. For this example, we're going to use signURLCanned because we want our customers to only have access to a file in CloudFront for a limited period of time.

If you look at the docs for JetSet's CloudFront service, the method signature for signURLCanned is as follows:

{% highlight java %}
public static java.lang.String signUrlCanned(java.lang.String resourceUrlOrPath, java.lang.String keyPairId, byte[] derPrivateKey, java.util.Date epochDateLessThan) throws CloudFrontServiceException
{% endhighlight %}

So we're going to need the following:

- A string that is the URL of the object we want
- The keyPair ID of the CloudFront keypair from Amazon that you made earlier
- The der-encrypted version of your CloudFront private key
- An ISO8601-formatted date that indicates the date and time when the signed URL will expire

Let's look at each item in order:

*resourceUrlOrPath*- this will be the full URL of the object you want to access on CloudFront. For example: https://gobb7edyg00k.cloudfront.net/somefile.mov

{% highlight html %}
<cfset cloudFrontObjURL = "https://gobb7edyg00k.cloudfront.net/somefile.mov" />
{% endhighlight %}

*keyPairID* - this is the ID of the CloudFront key pair that you made earlier. If you didn't write this down, you have to log in to the AWS portal, select Account -> Security Credentials and click the "Key Pairs" tab. Your CloudFront key pair ID will be listed there.

*derPrivateKey* - When AWS creates key pairs, they are created with .pem encoding. However, Java can't natively read .pem encoded key files. Java works with .der encoded key files. So you've got to convert the .pem encoded files to .der encoding so that Java can properly work with them. This is where the jets3tEncryptionUtils object that we created earlier comes in. It has a method called convertRsaPemToDer() that converts .pem encoded files into .der encoding so that we can create a valid CloudFront signature.

The docs for convertRsaPemToDer() state that it takes a Java FileInputStream. This is simple enough to do:

{% highlight html %}
<cfset privateKeyFilePath = "/path/to/you/keyfile/pk-YOURKEYFILENAME.pem" />
<cfset keyFileInputStream = createObject("java", "java.io.FileInputStream").init(privateKeyFilePath) />
<cfset derPrivateKey = jets3tEncryptionUtils.convertRsaPemToDer(keyFileInputStream) />
{% endhighlight %}

*epochDateLessThan* - This needs to be an ISO8601-formatted date that tells CloudFront the date and time on which the signed URL will expire. How do we get the date into the right format? That's where the JetSet ServiceUtils object we created earlier comes in:

{% highlight html %}
<cfset expiresOnDate = jets3tServiceUtils.parseIso8601Date("2013-02-14T22:30:00.000Z") />
{% endhighlight %}

Note that the date string has to be in the supplied format has to be in UTC time format. You can use time offsets if you need.

Here is the complete code we need to create the signed URL:

{% highlight html %}
<cfset accessKey = "YOUR ACCESS KEY" />
<cfset secretKey = "YOUR SECRET KEY" />
<cfset awsCredentials = createobject("java", "org.jets3t.service.security.AWSCredentials").init(accessKey, secretKey) />
<cfset cloudFrontService = createobject("java", "org.jets3t.service.CloudFrontService").init(awsCredentials) />
<cfset jets3tServiceUtils = createObject("java", "org.jets3t.service.utils.ServiceUtils").init() />
<cfset jets3tEncryptionUtils = createObject("java", "org.jets3t.service.security.EncryptionUtil").init("dkhcoaj3fh2sk0oay") />
<cfset cloudFrontObjURL = "https://gobb7edyg00k.cloudfront.net/somefile.mov" />
<cfset privateKeyFilePath = "/path/to/you/keyfile/pk-YOURKEYFILENAME.pem" />
<cfset keyFileInputStream = createObject("java", "java.io.FileInputStream").init(privateKeyFilePath) />
<cfset derPrivateKey = jets3tEncryptionUtils.convertRsaPemToDer(keyFileInputStream) />
<cfset expiresOnDate = jets3tServiceUtils.parseIso8601Date("2013-02-14T22:30:00.000Z") />
<cfset signedUrl = cloudFrontService.signUrlCanned(cloudFrontObjURL,keyPairId,derPrivateKey,expiresOnDate) />
{% endhighlight %}

You can then use the signed URL value in an &lt;a href&gt; or anywhere you can supply a valid URL.

Hella-long blog post? Yes. Hopefully those of you who need to create signed URLs for CloudFront using ColdFusion will find it a big time-saver!
