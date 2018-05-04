---
layout: post
title:  "cfcache Early, cfcache Often, 'Cause It's Just So Darn Easy"
date:   2010-09-02 09:21:00 -0400
categories: ColdFusion
---

Each new version of ColdFusion (be it from Adobe, Railo or OpenBD) brings with it a raft of new features. As a busy developer just trying to get things done, it's easy to skip over most of the new features and use what you've always used. I'd like to share a brief story about the &lt;cfcache&gt; tag in Adobe ColdFusion 9 and why it's the one tag in CF9 that you should start using now.

Improving performance is always an issue, regardless of the technology you're using to build and deploy Web sites. I ran into an issue yesterday where a tool that had worked just fine suddenly began to experience significant slowdowns. The tool is for managing groups of students in classes. Normally, a class will have 3-7 groups of students. In two cases, relatively large classes (100+ students) needed to divide students into groups in multiple ways. In one class, they ended up with 42 different groups of students. In the other, they ended up with almost 100 groupings. With this many groups, the main listing page of all groups in these courses started to run quite slowly. Response times dropped from a couple of seconds to 40+ seconds.

After a brief investigation, the issue was that core information about the class was being retrieved for each and every group on the page. The core information about the class included quite a bit of information, usually returned in approximately 420ms, and the increase in repeat calls to get this information was really slowing things down.

>I'm not going in to the discussion of using queries or structs instead of business objects for performance reasons at this point. Refactoring the course data retrieval service to return a simple struct or query would definitely improve performance, but a) that's not what this post is about and b) would involve mucking with core business objects and that's not something I wanted to do given the need to quickly address the situation.

Repeatedly retrieving the same data from the data source is costly and foolish. A quick solution to the problem was to cache the required data and use that cached information instead. As the core information about a course doesn't change very often, I could cache this information for 5 minutes with no problems and get a nice performance boost.

Until CF9, you couldn't cache objects without rolling your own struct-based caching code. In CF9, though, the [&lt;cfcache&gt; tag](http://help.adobe.com/en_US/ColdFusion/9.0/CFMLRef/WSc3ff6d0ea77859461172e0811cbec22c24-7d5a.html) makes it super easy to cache objects, page fragments, query results, or whatever you need.

As this call to get the core course information was being made a number of times in the same CFC, I refactored this call in to a private function. Here's the code:

{% highlight javascript %}
<cffunction name="getCourseInfo" access="private" output="false" returntype="course">
  <cfargument name="courseID" type="numeric" required="yes" />
  <cfset var courseNameInCache = "courseID" & arguments.courseID />
  <cfset var cachedCourseInfo = cacheGet(courseNameInCache) />
  <cfset var course = 0 />
  <cfif isNull(cachedCourseInfo)>
    <cfset course = variables.courseService.getCourse(arguments.courseID) />
    <cfset cachePut(courseNameInCache, course, CreateTimeSpan(0,0,5,0)) />
  <cfelse>
    <cfset course = cachedCourseInfo />
  </cfif>
  <cfreturn course />
</cffunction>
{% endhighlight %}

The key to this function are the cacheGet, isNull, and cachePut functions. You use cacheGet(itemName) to put something in to the cache, and cacheGet(itemName) to get something out of the cache. Super simple, right?

This function shows off some of the issues you need to be aware of when using CF9's built-in instance of [Ehcache](http://ehcache.org/).

- I've been asked "How do you make sure you have a unique name for the item you want to put in the cache and how do you remember that unique name later when you need to get the item from the cache?" The first cfset line takes care of that. It's convenient to generate the name for the item in the cache using a key value associated with that item, like courseID. You just have to make sure that no other item anywhere in the cache is going to have the same name.
- You need to check to see if an item in the cache exists, otherwise how will you know to use the cached item or put the item in the cache? That's what the  cacheGet(courseNameInCache) and the isNull(cachedCourseInfo) calls do. You first try to get the item out of the cache. If it exists, great! If not, CF will return a null value for the item in the cache. If you get a null value for the item in the cache, you need to put the item in the cache.
- If the item isn't in the cache, you need to generate the item and then put it in the cache via cachePut(itemName). Again, be sure to make your item names unique!
- If the item *is* in the cache, return the cached version of the item.

One concern that's often raised about using the default &lt;cfcache&gt; setup in ColdFusion 9 is that items put in the cache are put in the JVM along with everything else that ColdFusion puts there. As such, the items you cache will take up some memory, along with the CF runtime, your application code, items in persistent scopes (server, application, session), and objects generated on a per-request basis. However, in many cases, you're going to be caching items that take up 1K of memory or less. That means it'll take about 1000 items in the cache before you use 1MB of memory. Unless you are caching really big data sets using &lt;cfcache&gt; or expect that you'll have hundreds of thousands of items cached on a server with limited memory, this generally won't be a problem and you shouldn't let caching items in memory scare you. (You can [configure CF9 to use a separate, standalone instance of Ehcache](http://www.brooks-bilson.com/blogs/rob/index.cfm/2009/11/17/Caching-Enhancements-in-ColdFusion-9--Part-6-Configuring-Caches-and-Working-with-ehcachexml), should you so desire.)

To wrap up my little story, the end result of taking 15 minutes to write up this simple caching function, run my unit tests, and deploy the change was a reduction in page load times from 40+ seconds to under two (2) seconds. That's an excellent improvement for so little work.