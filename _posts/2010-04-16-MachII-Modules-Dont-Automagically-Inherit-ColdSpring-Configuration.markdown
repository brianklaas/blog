---
layout: post
title:  "Mach-II Modules Don't Automagically Inherit ColdSpring Configuration"
date:   2010-04-16 13:01:00 -0400
categories: ColdFusion
---

This is probably obvious to anyone who's spent time with Mach-II and ColdSpring, but I ran in to the issue the other day, spent about an hour missing the obvious, and thought I'd post.

If you have a Mach-II module, are using ColdSpring to manage dependencies in the parent application, the Mach-II module won't automatically inherit the ColdSpring setup. I had begun work on a simple module that didn't have any of its own objects to be managed by ColdSpring but instead would exclusively use objects managed at the parent application level. So I went about setting up a simple "home" view which needed some data from an object managed by ColdSpring in the parent app, and made a &lt;call-method&gt; command to it. I received an error stating that the bean I was looking for couldn't be found.

"Well that's not right," I thought. "I know the bean is in the parent app and other parts of the application are calling it just fine." After an hour or so of debugging, thinking that someone had broken the object in the latest build of the application, and getting a little annoyed, it dawned on me that I had never set up the ColdSpring property in this child module. Without the ColdSpring property being defined in the module, Mach-II had no idea how to call the bean I had specified in the  &lt;call-method&gt; command.

So if you are creating a module that has no ColdSpring-managed objects of its own, but do want to use ColdSpring-managed objects in the parent application, you still have to set up a basic ColdSpring Mach-II property in your module, and point it to an (essentially) empty ColdSpring config file. Your ColdSpring config file can look like this:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans>
</beans>
{% endhighlight %}

Once that is in place (and as long as you also define the parentBeanFactoryScope in your module's ColdSpring property CFC), your module will have access to all the ColdSpring managed objects in the parent application.