---
layout: post
title:  "Mach-II Modules and Views"
date:   2009-11-30 11:56:00 -0500
categories: ColdFusion
---

It looks like I'm going to spend another Monday morning writing about Mach-II modules. In this entry in the series, I'm going to look at how your CFML-based views are affected by the Mach-II module structure.

Mach-II provides a lot of convenience methods for handling views. In Mach-II 1.8, there's a whole [new set of tag libraries](http://www.mach-ii.com/index.cfm/go/blog:showEntry/entryId/41CE16F0%2DEC26%2DC39B%2D4A9315BA9DA78C19/) which make tying your views to other actions in a Mach-II application even simpler. Of particular interest are the [&lt;form&gt;](https://greatbiztoolsllc-trac.cvsdude.com/mach-ii/wiki/MachII1.8SpecificationFormTagLib) and [&lt;view&gt;](https://greatbiztoolsllc-trac.cvsdude.com/mach-ii/wiki/MachII1.8SpecificationViewTagLib) tag libraries as they make dealing with mundane tasks like binding form fields to event objects and creating links simpler and more flexible under the Mach-II framework. Prior to Mach-II 1.8, there were the [BuildURL() and BuildURLToModule() methods](https://greatbiztoolsllc-trac.cvsdude.com/mach-ii/wiki/WhatsNewInMachII1.5), which were your biggest friends in building links within and between modules in a Mach-II app.

I realize that not everyone likes to have a framework write their URLs for them. There are issues of convention, framework agnosticism, SEO optimization, and individual style that can and often should be factored in. However, if you're using Mach-II modules or Mach-II SES URL rewriting, or the new [routes support in Mach-II 1.8](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/MachII1.8SESImprovements), it would be quite unwise to write URLs on your own. I'd argue that as you've made the investment in the framework to begin with, you should let the framework generate your URLs for you via its various convenience methods in all cases. If you ever switch domains or even move the base URL path of your application or need to swap one module for another, changing a single value in your Mach-II XML config file is going to be a lot faster (and a lot less error prone) than doing a search and replace on every &lt;a href&gt; in your code base.

(As a side note, I've had issues with outputting BuildURL() calls within JavaScript blocks, and did use that as an exception to the above rule. However, the new [buildUnescapedUrl() function introduced in Mach-II 1.8](https://greatbiztoolsllc-trac.cvsdude.com/mach-ii/ticket/178) eliminates that exception.)

**Using BuildURL() and BuildURLToModule()**

By using BuildURL() and, more relevant to this post, BuildURLToModule(), you let Mach-II handle the writing of your &lt;a href&gt; tags. These functions are available in views only, and not inside plugins, filters, or listeners. If you use Mach-II modules, you're going to want to use BuildURLToModule() over BuildURL() in most cases. Here's why:

- BuildURLToModule() ensures that the module is preprended to the event name in the resulting URL. This ensures that the user is routed to the right module when the event is requested.
- BuildURLToModule() allows for an empty module name in the module argument of the function. This lets you write events which point to the main/parent application and forces the request to go to the main/parent application.

If you just use BuildURL(), Mach-II is going to assume that you want an event in the same/current module. This works fine if you are at the top level of your application, in the main/parent portion of the application. This doesn't work so well if you're inside a module. For example, if you have an event called "logout" tied to a "Logout" link and you are inside a module and a user clicks on the "Logout" link, BuildURL() is going generate a  link that looks for an event called "logout" **in the current module**. As the logout event is (probably) defined in the main/parent application, Mach-II won't be able to find the event in the current module, and an exception will be thrown.

As a result, it's a good idea to use BuildURLToModule() instead of BuildURL() in both the parent *and* child (module) application views. This way, you ensure that the user is being routed to the right event in the right module, even if it's the default/parent module. This is my personal preference, and not an official guideline of Team Mach-II.

**Using the &lt;view&gt; Library**

The &lt;view:a&gt; tag is a handy way to let Mach-II write your &lt;a href&gt; links. It does much of what the BuildURL() and BuildURLToModule() functions have done for some time now, only in a simpler and more elegant wrapper. Now that I'm using Mach-II 1.8 to build my applications (and for this particular, module-heavy project), I prefer it to BuildURL() for generating &lt;a href&gt;s. I still use BuildURLToModule() when I'm not generating a &lt;a href&gt; tag.

There are [lots and lots of options when using the &lt;view:a&gt; tag](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/MachII1.8SpecificationViewTagLib#TheaTag), but the basic format of generating a link is as follows:

{% highlight html %}
<view:a module="blog" event="showEntry" p:entryId="456"></view:a>
{% endhighlight %}

You simply specify the event name in the event attribute, the module name in the module attribute, and any key/value pairs you want to add to the link in the p attribute. If you have multiple key/value pairs you want to add, just put multiple p attributes in the tag, as follows:

{% highlight html %}
<view:a module="blog" event="showComments" p:entryId="456" p:commentID="89"></view:a>
{% endhighlight %}

You can put CF variable names in lieu of static values, or use the expression language syntax available in Mach-II to generate the values:

{% highlight html %}
<view:a module="blog" event="showComments" p:entryId="${event.entry.entryID}" p:commentID="89"></view:a>
{% endhighlight %}

Again, you should provide a blank value for the module attribute if you want the link to point to an event in the main/parent application. This ensures that Mach-II looks for an event in the main/parent application and not in the current module.

**Using the &lt;form&gt; Library**

When it comes to generating form action attributes, the &lt;form&gt; tag library works just like the &lt;view:a&gt; tag. An example:

{% highlight html %}
<form:form actionEvent="createEntry" actionModule="blog">
{% endhighlight %}

It's very unlikely you'd be doing a form post from a module back to the parent/main application. As a matter of fact, I can't think of a good use for it. It's still a good idea to specify the module in which the event exists via the actionModule attribute, however, to ensure that the link is always written properly.

There's [lots of documentation for the &lt;form&gt; tag library](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/MachII1.8SpecificationFormTagLib), so please review it to see what it's capable of. Binding, for example, is just plain useful and cool.

**Reducing Most of the Above to a Single Point**

The key point is this: no matter how you write your URLs, always make sure that a module (or an empty string for a module) is specified. This will ensure that event requests are routed to the right place. Start doing this when you begin to build your views, and you'll find it saves you time down the road when you're connecting everything.

