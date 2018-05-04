---
layout: post
title:  "Mach-II Call Method Command, How Do I Love Thee?"
date:   2010-02-19 10:34:00 -0500
categories: ColdFusion
---

Now that [Mach-II 1.8 is out](http://www.mach-ii.com/index.cfm/go/blog:showEntry/entryId/DA42ECE6%2DA834%2DA779%2DC42C698CA7FBF691/), I figured I'd write a little bit about one of my favorite new features of the release, the &lt;call-method&gt; command.

Here's the problem that this new feature is designed to solve:

In a well-designed Mach-II application, you don't want to have a lot of business logic in your [http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/IntroToListeners](listeners). As a result, a lot of the methods in Mach-II listeners wound up being simple pass-throughs to a method in the service layer. This was repetitive, and tedious. Here's an example:

{% highlight javascript %}
<cffunction name="getAllUsers" access="public" output="false" returntype="query">
  <cfargument name="event" type="MachII.framework.Event" required="yes" />
  <cfreturn variables.userService.getAllUsers() />
</cffunction>
{% endhighlight %}

We had to write a function that simply called a service layer method and returned the results. It's not hard to do. It is, however, clutter.

The &lt;call-method&gt; command eliminates the need for these kinds of placeholder method calls in your listener. Now you can do this in your Mach-II XML configuration file:

{% highlight xml %}
<call-method bean="userService" method="getAllUsers" resultArg="qryAllUsers" />
{% endhighlight %}

The "bean" referenced in this tag is a [ColdSpring](http://www.coldspringframework.org/)-managed bean that has already been auto-wired by the [ColdSpring property in Mach-II](https://greatbiztoolsllc-trac.cvsdude.com/mach-ii/wiki/UsingColdSpringWithMach-II). So this feature only works if you are also using ColdSpring (and &mdash; I've said it many times before &mdash; if you're not using ColdSpring or some other IoC container in your ColdFusion applications, you're costing yourself time and money.)

By using the &lt;call-method&gt; command, you've turned four lines of code into one. Not bad!

What if your service layer methods require arguments to be passed to them? That's easy, and Mach-II handles this.

{% highlight xml %}
<call-method bean="userService" method="getUser" args="${event.userID}" resultArg="user" />
{% endhighlight %}

The args attribute of the &lt;call-method&gt; command can take single or multiple, comma-separated values, or even an argument collection structure.

But what's the ${ } syntax? That's the expression language syntax that's built in to Mach-II. This feature arrived in [Mach-II 1.5](https://greatbiztoolsllc-trac.cvsdude.com/mach-ii/wiki/WhatsNewInMachII1.5) under the name "Bindable Property Placeholders," and has evolved since then and is even more useful. It allows you to provide variables evaluated at runtime within the Mach-II configuration file. You can even provide default values in case the variable you're looking for in the expression doesn't exist (ie; ${event.userID:0}). So in the example above, it's going to to pass the result of evaluating event.userID (getting the value of the userID variable in the current event) to the userService.getUser() method.

I'm using the &lt;call-method&gt; command in my latest project using 1.8. It's saving me time, makes the event flow as defined in the Mach-II XML config file clearer, and eliminates a duplicate layer of abstraction in the app. 

The official [documentation for the &lt;call-method&gt; command](https://greatbiztoolsllc-trac.cvsdude.com/mach-ii/wiki/MachII1.8CallMethodCommand) can be found in the Mach-II wiki.

