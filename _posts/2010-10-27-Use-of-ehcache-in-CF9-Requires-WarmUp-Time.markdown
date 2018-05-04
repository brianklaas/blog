---
layout: post
title:  "Use of ehcache in CF9 Requires Warm-Up Time"
date:   2010-10-27 19:15:00 -0400
categories: ColdFusion
---

One of the great parts of the annual [Adobe MAX](http://max.adobe.com/) conference has been the Unconferences at MAX, and the CF Unconference, organized by the ever-dedicated Ray Camden. One of today's sessions at the CF Unconference was given by [Mike Brunt](http://cfwhisperer.com/) (and one of his colleagues, whose name I completely missed) on scaling ColdFusion 9 applications and, more specifically, how to set up [ehcache](http://ehcache.org/) for scaling. ehcache is built in to ColdFusion 9 and forms the heart of the new, fast, super-easy caching in ColdFusion 9.

During the session, Mike spoke for a bit about one really important fact of the ehcache&lt;-&gt;ColdFusion relationship: ehcache isn't launched by default when ColdFusion 9 starts up. If you don't use the &lt;cfcache&gt; tag in ColdFusion 9, you won't get any of the memory consumption or overhead of ehcache (which, admittedly, isn't a whole lot). However, the very first time you call &lt;cfcache&gt; in your application, ColdFusion makes that initial connection to ehcache and ehcache, in turn, has to spin up and work its magic. This initial spin up time, Mike said, can take around 30 seconds.

That's a pretty long time. If a customer makes a request and ehcache hasn't warmed up yet, they are most likely going to think that their request has timed out. As a result, Mike said, it's important to make a basic call to &lt;cfcache&gt; when the server starts up, perhaps in an onServerStart.cfc (which you can define in the CF administrator) or in the onApplicationStart() function in an application.cfc. This way, if the  ehcache&lt;-&gt;ColdFusion connection needs to be made, it can be done before customers start making requests. If your ehcache servers are separate from your ColdFusion servers (which you can certainly do), they should be up and running before ColdFusion starts up.

I haven't done any testing on this myself, but given Mike's well-known prowess for server performance tuning in the ColdFusion world, I'd take him at his word.