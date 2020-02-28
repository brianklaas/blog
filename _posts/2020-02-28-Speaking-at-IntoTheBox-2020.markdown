---
layout: post
title:  "Speaking at IntoTheBox 2020 on CFML and AWS S3"
date:   2019-09-27 16:39:00 -0400
categories: AWS ColdFusion Conferences
---

I'm privileged to once again speak at the superb [IntoTheBox conference](https://www.intothebox.org) in early May, 2020, in Houston, TX. While skewed toward development in [CMFL](https://en.wikipedia.org/wiki/ColdFusion_Markup_Language), the conference focuses on intermediate to advanced level sessions on modern web application development and cloud deployment. In addition to two days of workshops on Box-specific tools and technologies, there are sessions on migrating legacy applications to modern dev stacks, Java streams, container deployment on AWS Fargate, database performance, and more.

My presentation is "I Didn't Know S3 Could Do That!" It's a look beyond using S3 as simple file storage, which is how most developers work with the service. [S3 can do so much](https://aws.amazon.com/s3/features/), and it would be impossible to cover everything it can do in less than an hour, but I plan to cover:

- Cut storage costs by using different S3 storage classes
- Secure your files in S3 with time-expiring URLs
- Increase security over what's built into the CFML engines by accessing S3 via the Java SDK
- Automatically archive unused files after a set period of time
- Encrypt objects at rest in S3
- Use the rock-solid object versioning available in S3
- Set file metadata so files get served from S3 as users expect them
- Build detailed reports on usage via tags 

If you're a CFML developer and looking to go to one conference this year, I highly recommend that you consider [IntoTheBox](https://www.intothebox.org). I always learn a lot at this conference. You will too.