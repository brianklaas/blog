---
layout: post
title:  "Slides from My CFObjective Talk: Making High Performance Caching Easy with ColdFusion"
date:   2012-05-20 09:00:00 -0400
categories: ColdFusion Conferences
---

Attached to this post is a PDF of the slides used in the talk I gave yesterday at CFObjective: "Making High Performance Caching Easy with ColdFusion." CFObjective is a great conference (and not only about ColdFusion, by any stretch of the imagination), and one that I felt honored to have been able to speak at. I thought the presentation went well, though I did seem to be talking a bit faster than I usually do (and I'm a fast talker).

[Slides from "Making High Performance Caching Easy with ColdFusion"](/assets/pdf/MakingHighPerfCachingEasyWithCF.pdf)

After the session, Mark Drew from Railo showed me something very cool that they are working on in Railo 4: the ability to cache the results of a function in the function definition itself. Instead of writing code inside the function to check and see if the item exists in the cache, return it if it does, and build the object and put it in the cache if it doesn't, Railo will do that for you if you add a single attribute to your function definition. Less code = a good thing!

Mark also pointed out that Railo supports Ehcache, memcached, and Redis, another very popular key/value store, and has the ability to put various caches (query, template, saved content, etc) in different locations. It's a very flexible setup.