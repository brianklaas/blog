---
layout: post
title:  "CloudFront URLs Do Not Include the S3 Bucket Name"
date:   2013-04-22 12:29:00 -0400
categories: AWS
---

This is probably obvious to anyone who has worked with Amazon CloudFront for some time, but I spent the last day or so banging my head against the proverbial wall when trying to make dynamic requests to the CloudFront CDN. 

S3 is the origin service for my CloudFront distributions, and I could see the files sitting in S3. I could even request them directly from S3 using https. However, every time I tried to access the files via CloudFront, I received the following error:

{% highlight html %}
{
  <Code>NoSuchKey</Code>
  <Message>The specified key does not exist.</Message>
}
{% endhighlight %}

As it turns out (thanks to the ongoing genius that is Stack Overflow), [CloudFront URLs must not include the S3 bucket name](http://stackoverflow.com/questions/14206072/what-url-should-i-use-for-amazon-cloudfront-content). This makes sense when you think about it because a CloudFront distribution gets its own domain name, and each CloudFront distribution points to a specific bucket in S3. The bucket name is already "included" in the distribution, so there's no point in including it in your object key/URL.