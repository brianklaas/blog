---
layout: post
title:  "A Few Thoughts on Google's Course Builder"
date:   2012-09-12 10:06:00 -0400
categories: EducationalTechnology
---

As my primary work revolves around online education and using technology to create an excellent learner experience, part of my job is to keep on top of many of the learning platforms available. [Coursera](http://coursera.org/) has been making a big splash of late, and I've taken a number of courses on the platform. [Lore](http://lore.com/) has been gathering a large amount of buzz, especially as a replacement for more traditional learning management systems (and not a system for teaching at massive scale, like Coursera).

Today, Google [released the first version of their Course Builder tool](http://google-opensource.blogspot.co.uk/2012/09/helping-world-to-teach.html). It's not a web-based tool that you visit in your browser and build. It's actually a downloadable code base that you run inside Google App Engine. So that's pretty cool: download the GAE runtime, make your changes for your course and your material, and then upload to GAE for deployment. No hosting, no database setup, none of the system admin work that's the hallmark of more traditional learning management systems like Blackboard or Moodle, but also giving you the flexibility to build the course as you please.

And that's the problem.

Of the major issues with adopting any new technology &mdash; specifically in higher education &mdash; is a lack of time on the part of the content creators. Rarely do faculty have the time or inclination to learn new technologies to incorporate into their teaching. New technologies have to be very easy to learn, with a minimum of amount of friction, and allow content experts to be content experts, not technologists. While there absolutely is a percentage of any faculty body that will gladly pick up a new tool or a new programming language to get the job done, they're in the minority.

Google's Course Builder requires that you know JavaScript, and HTML. Stupid easy for us web developers, no doubt, but a major barrier for entry for educators from the primary level through graduate school. They don't know JavaScript, or HTML, and they don't have the time (amidst the crush of actually teaching and communicating with students and doing all their required administrative work) to learn, as "easy" as we may perceive it. The frequent response to this is "Well, there's got to be a TA or staff member in IT who can help them with this." The problem there is that the TA or staff member can't generate the content. The subject matter expert has to do that. If the subject matter expert has to generate the content and then sit down and spend time with the person who knows how to translate that into JS/HTML or whatever the toolkit requires, it's a time loss on both sides. More problematic, these kinds of positions aren't often funded, or the funding is minimal. 

[Not everyone should know how to code](http://www.codinghorror.com/blog/2012/05/please-dont-learn-to-code.html). I know that this is an initial release of Course Builder from Google, but it will appeal largely to people who already know how to code and probably have built their own online course materials using their own website or an existing LMS.

A couple other notes about this project:

- Those promoting educational technology systems &mdash; specifically learning management systems &mdash; need to get over the idea that course content means video. There are so many other ways of building interesting, interactive learning experiences, but example after example promotes video as the way in which all content should be delivered. Google does the same in their [guide for Course Builder](https://code.google.com/p/course-builder/wiki/CreateLessons).
- Putting quiz/assessment questions and answers in JavaScript is a bad idea. Easy to implement, yes, but any student who knows how to view source will see the answers. Not acceptable for real assessment.
