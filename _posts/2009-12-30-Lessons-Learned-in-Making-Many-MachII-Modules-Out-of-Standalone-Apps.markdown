---
layout: post
title:  "Lessons Learned in Making Many Mach-II Modules Out of Standalone Apps"
date:   2009-12-30 09:39:00 -0500
categories: ColdFusion
---

This entry is about a month overdue, but given the holidays and Getting Stuff Done (SM) at work, it's been delayed.

As I mentioned in my first posting in the series, I'm in the process of converting a bunch of standalone Mach-II apps in to modules inside of  a new, monolithic application. It's been a surprisingly smooth process, thanks to the tools that the Mach-II framework gives us to work with. Most of the bumps along the way have been related to bad software design practices in some of the oldest apps that I've created, and continuing, after many years building Web applications, to do dumb things like hardcoding URLs and other strings which will change when converting the app to a module (or moving the app to a different server).

Here are a few key lessons I've learned:

- **Use BuildURL and the View Tag Library generously.** One of the major time sinks I've had to endure in this process is changing all of the links and constructed URLs in the apps-to-modules. If you're using Mach-II and still writing &lt;a href&gt;s in your views, you're throwing money out the window. Sooner or later, those URLs are going to change, and you're going to have to rewrite them. [BuildURL](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/URLManagementFeatures) and the [View Tag Library](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/MachII1.8SpecificationViewTagLib) encapsulate the creation of URLs/links in your views so that when you change servers (or need to roll out a new version of a module in a different directory), you do very little (to no) work to make the URLs/links all update nicely. And, yes, you can use them to create links in your inline JavaScript (if you use inline JavaScript).

- **Avoid redundancies in your ColdSpring configuration files.** If you're going to share functionality across modules and you manage your object dependencies with ColdSpring, you're going to run in to instances where you define a bean multiple times in multiple ColdSpring configuration files. This is a really common situation. If you have a userService, for example, you're probably going to refer to this userService throughout the modules in your application. It's easy to define that userService in each of the ColdSpring configuration files in each of the modules that uses the userService, but don't. Simply define the userService bean in the ColdSpring configuration file at the top level of the application, and then if you inherit beans defined in the parent (top level) app, you have access to that userService in your module. You don't need to define it again, and doing so is wasteful, and may lead to subtle (or not so subtle) issues when the a new version of the service is rolled out in the parent but your child is using an old version of the service because it's been define that way in the child ColdSpring config file. If your module needs a specific version of the service (eg; it needs to keep an old version for compatibility with something else), you can define it in the child ColdSpring config file as needed.

- **Avoid per-module session objects.** This is a really easy trap to fall in to if you are porting existing Mach-II applications to modules inside a new, larger application. Most of your existing apps will have information stored in the session scope. It's tempting to keep your perfectly fine code as-is and just create new session variables as needed in each of your apps-to-modules. The problem here is that if you're creating new session variables in 7 or 15 or 22 modules, you're going to start to eat up memory and, more importantly, you're going to have to try to track down why things are going wrong in 7 or 15 or 22 session variables rather than in a single location. It's just bad form to *not* encapsulate everything you need in the session scope in a single variable created and managed at the top layer of your application. In all likelihood, all those redundant session variables generated in your modules largely contain the same information &mdash; userIDs, user objects, security or role-permission information, and the like. Managing this in one place makes a heck of a lot more sense than spreading it out in to each of your modules.

- **Handle exceptions and errors at the main app level.** As with redundant bean definitions or per-module session objects, per-module exception and error handling is most often redundant. While there may be cases where you want to handle an exception in a special way in a module, there's no good reason to have different exception or error views in each module. Provide a consistent way of handling errors and exceptions for the developers behind the app and a consistent experience for the people using the app. If you want to do some module-specific handling of errors (to try to recover, to log something specific to the module, etc.), you can do this in your module's Mach-II XML config file:

{% highlight xml %}
<event-handler event="exception">
  <filter name="tryToRecoverFromExceptionFilter" />
  <-- OR -->
  <notify listener="someListener" method="doSomeErrorProcessing" />
  <-- FINALLY -->
  <announce module="" event="exception" copyEventArgs="true"/>
</event-handler>
{% endhighlight %}

- **Get rid of the extra junk.** Not using all the plugin points you stubbed out in your plugins? (Or, in my case, left in there because you copied from the samplePlugin.cfc that came with the framework?) Get rid of them. No longer using that sessionFacade.cfc because you're managing your session objects centrally? Get rid of it. Get rid of everything you're not using. It's good practice and will make things cleaner and easier to understand for the poor sap who has to work on the app your absence.

So that's it for this series. I have to give major props to the Mach-II team for making it so easy to convert existing Mach-II applications to Mach-II modules. It's made this big project so much easier.


