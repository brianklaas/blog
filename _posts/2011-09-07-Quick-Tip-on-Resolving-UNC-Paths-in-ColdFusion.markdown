---
layout: post
title:  "Quick Tip on Resolving UNC Paths in ColdFusion"
date:   2011-09-07 09:59:00 -0400
categories: ColdFusion
---

A few months back, we moved all of our user-generated content to a file share, and had to rewrite some of our code to use the new UNC path to the files. As with most things ColdFusion, it's best to let the application server do the hard work for you, so instead of writing out the full UNC path to files in a &lt;cffile&gt; tag, you should use the expandPath() function instead. This is a really good idea if, at some imaginary point in the future, the path to your file share changes. (File shares never change locations in a network, right?)

One thing that tripped me up on more than one occasion was the need to run expandPath() on the directory that points to the file share, and not another directory like webroot or wherever you might be storing files on the actual ColdFusion server. So instead of doing this:

{% highlight javascript %}
<cfset pathToFiles = expandPath("/") & "fileShareDirectory/someOtherDirectory" />
{% endhighlight %}

you have to do this:

{% highlight javascript %}
<cfset pathToFiles = expandPath("/fileShareDirectory/someOtherDirectory" />
{% endhighlight %}

Otherwise, ColdFusion won't be able to resolve the actual path to the file share directory via the appropriate UNC path. It will look for a directory called "fileShareDirectory" in the webroot, even if there's an alias set up at the operating system level to point to the actual file share on another server.