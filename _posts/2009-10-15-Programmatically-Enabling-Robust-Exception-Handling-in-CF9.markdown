---
layout: post
title:  "Programmatically Enabling Robust Exception Handling in CF9"
date:   2009-10-15 15:28:00 -0400
categories: ColdFusion
---

I've been going through Daniel Short's excellent new training module, &quot;[ColdFusion 9 New Features](http://www.lynda.com/home/DisplayCourseN.aspx?lpk2=56299)&quot; over on [Lynda.com](http://www.lynda.com/). I really like the depth and breadth of content that's available on Lynda.com (and it's not expensive), and given that Adobe's on in-person training for CF9 won't be available for a while yet, it's great to see them rolling out CF9 training at the same time as the CF9 release.

I've been playing with CF9 for quite a while now, and Daniel does a great job at covering all the major new features, and some minor ones. He covers one cool, very useful thing that I wasn't aware of in CF9: the ability to programmatically enable robust exception handling.

Let's say that you have a production application that's throwing errors. You probably are (should be) already logging most of the activity in the application, but if you need to see what specific variables are being sent to a specific view page that's generating an error, sometimes dumping things out via CF's robust exception handling is the fastest way to go. You may not be able to set up a remote debugging session via CFBuilder on to a production server to get this information. You also certainly don't want robust exception handling enabled by default for anyone who accesses your application, as that's a big security hole. You just need to be able to see this information temporarily so you can figure out exactly what's going wrong.

In CF9, you can enable robust exception handling for a specific IP address by including the following in your application.cfc's constructor (eg; the area where you put all the &lt;cfset this.someVariable=someValue /&gt; statements):

{% highlight javascript %}
<cfset this.debuggingIPAddresses = "list,of,IP,addresses" />
<cfset this.enablerobustexception = true />
{% endhighlight %}

The this.debuggingIPAddresses value is just a list of the IP address(es) that are allowed to see the debugging output and robust exception information. Add your IP address to this list, and you can see the full set of debugging and exception information in your production application.

Now you probably want to go one step further and wrap the above in a &lt;cfif&gt; statement that checks to make sure you actually *should* be showing this, or some random URL variable that your dev team knows about to turn this on on a per-request basis. This way you can enable robust exception handling on a per-request basis, without having to reload the entire application.