---
layout: post
title:  "Mach-II and Module Config Files"
date:   2009-11-16 08:55:00 -0500
categories: ColdFusion
---

In this installment of my mini-series on modules in [Mach-II](http://www.mach-ii.com/), I'm going to talk about the Mach-II XML config file for your module, how it relates to the parent (main application) Mach-II XML config file, and try to offer some tips on what should and should not go in your module's Mach-II XML config file.

Before I go any further, it's important that you understand the context of this particular series on Mach-II modules and the fact that, for my specific purposes, I am not talking about building modules which are intended for drop-in deployment into any Mach-II app anywhere, like the fantastic [Mach-II Dashboard](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/Dashboard). I'm focusing on building modules that represent mini-applications operating under the auspices of a larger application. Examples of these modules would be a threaded discussion system in a larger community site, or an exam system in an online course site.

**A Sample Module XML Config File**

Below is a really simple Mach-II XML config file for a module.

{% highlight xml %}
<mach-ii version="1.8">
  <!-- INCLUDES -->
  <includes>
    <include file="/myApp/modules/sampleMod/config/sampleMod-coldSpringProperty.xml" />
  </includes>
  <!-- PROPERTIES -->
  <properties>
    <property name="applicationRoot" value="/myApp/modules/sampleMod/" />
    <property name="defaultEvent" value="sampleModHome" />
  </properties>
  <!-- LISTENERS -->
  <listeners>
    <listener name="simpleListener" type="myApp.modules.sampleMod.model.simpleListener" />
  </listeners>
  <!-- EVENT-FILTERS -->
  <!-- EVENT-HANDLERS -->
  <event-handlers>
    <event-handler event="sampleModHome" access="public">
      <notify listener="simpleListener" method="getRandomNumber" resultArg="randomNumber" />
      <notify listener="simpleListener" method="getHTTPString" resultArg="httpAddress" />
      <view-page name="sampleModHome" />
    </event-handler>
    <event-handler event="sampleProtectedEvent" access="public">
      <filter name="adminsOnlyFilter" />
      <view-page name="sampleProtectedView" />
    </event-handler>
  </event-handlers>
  <!-- SUBROUTINES -->
  <!-- PAGE-VIEWS -->
  <page-views>
    <!-- View Loaders -->
    <!-- This loads all pages which match /views/**/*.cfm, so a page-view called aboutUs.index would translate to /views/aboutUs/index.cfm -->
    <view-loader type="MachII.framework.viewLoaders.PatternViewLoader" />
    <!-- Define all other page-views here -->
  </page-views>
  <!-- PLUGINS -->
</mach-ii>
{% endhighlight %}

**&lt;includes&gt;**

You can include other configuration XML files in your module XML configuration file with no problem. In this example, I've included a Mach-II property for ColdSpring because there are certain objects that I want managed by ColdSpring and that are only pertinent to this module. I'll talk more about Mach-II modules and ColdSpring in an upcoming post, but I did want to show you how you'd configure the module to utilize ColdSpring, should you need to (and you most likely will).

**&lt;properties&gt;**

At a minmum, you need to include two properties in your module's Mach-II config file:

- applicationRoot &mdash; The absolute path to your module under Webroot
- defaultEvent &mdash; The default event to call should no event be specified by a request

Notice that the following Mach-II properties you would have in a Mach-II XML configuration are *not* listed here:

- eventParameter
- parameterPrecedence
- maxEvents
- MACHII_CONFIG_MODE
- urlBase or any of the URL rewriting properties

All of these properties have to be defined at the main application level and cannot be overridden at the module level.

You can include the following properties in your module's Mach-II XML config to override what's being used in the main application:

- exceptionEvent
- Module-specific caching property
- Module-specific logging property

While you *can* include these here, I would recommend that you handle exceptions, caching and logging at the main application level. This way you consolidate your exception handling, caching and logging setup in to a single location and don't wonder why you've set up logging in the main application but it's not logging as expected in the module due to a configuration override there. There are perfectly legitimate reasons to handle exceptions at the individual module level. I just prefer to do it centrally.

**&lt;listeners&gt;**

You define listeners here exactly as you would in a main Mach-II XML configuration file. Nothing special here. Move on.

**&lt;filters&gt;**

Any filters that are needed for this module and this module only are defined here. You'll notice in my configuration file that I call a filter in the sampleProtectedEvent event, but I don't define any such filter in this XML config file. How can this possibly work?

Your module inherits all of the properties, event filters, and plugins defined in your parent (main) Mach-II configuration file. So if you have a "admins only" filter which you use to only let administrators access certain events in your application, define it in your parent (main) Mach-II configuration file. You can then refer to that filter in any of the module Mach-II configuration files that operate under the parent app. This is much cleaner, simpler, and more encapsulated than defining a "admins only" filter in each of your modules. You're free to write a special "admins only" filter for a specific module that has special requirements, of course, and you'd define it in this block if that is what you need to do.

**&lt;event-handlers&gt;**

Event handlers are defined exactly as they are in the main Mach-II XML configuration file. Nothing special here. Move on. 

**&lt;page-views&gt;**

As with listeners and event-handlers, page views are defined exactly as they are in the main Mach-II XML configuration file. The [&lt;view-loader&gt; introduced in Mach-II 1.8](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/MachII1.8SpecificationViewLoaders) is a godsend and helps to remove all of the endless page-view declarations you'd normally find in a Mach-II XML configuration file, and if you're not using it, you should.

**&lt;plugins&gt;**

As previously mentioned, your module inherits all of the properties, event filters, and plugins defined in your parent (main) Mach-II configuration file. As such, you'd find common plugins like "is the user logged in plugin" and "put properties from a property CFC in to the current event plugin" defined at the main application level. Every event in your module would be subject to the plugins in your main application running on each and every event in the module, just as they do in the main application.

If you have a plugin which needs to run on every event in your module, define it here and it will work as expected.

There is one additional configuration option you can provide for plugins in a module, and that's the runParent attribute. This attribute tells Mach-II when your module plugins should be run in relation to the plugins in the parent/main app. As I mentioned in my first post in this series, I haven't used this myself because I keep to the default behavior in this regard. I'll simply quote from the [Mach-II docs](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/ConfigFileExplained)<a href=""></a> and you can take it from there:

> Defines if or when parent (base application) plugins should be run. This attribute only applies to Mach-II module XML configuration files and has no effect if defined in a base application. A value of after will run the defined module plugins first then the parent plugins, a value of before will run the parent plugins first then the module plugins and a value of none will run only the module plugins with no parent plugins whatsoever. The default value is after if not defined and the XML configuration file is a module.

**Including Your Module's Config File in the Main App Config File**

The developers behind Mach-II made it super easy to add a module to any existing Mach-II application. To add a module to an existing Mach-II application, you simply need a &lt;modules&gt; block which tells Mach-II where to find each module you want to include.

{% highlight xml %}
...all the rest of your main Mach-II config file, including plugins, listeners, events, etc...
<!-- MODULES -->
<modules>
  <module name="sampleMod" file="/myApp/modules/sampleMod/sampleModConfigFile.xml" />
</modules>
{% endhighlight %}

That's it. It takes nothing more than a single line of XML to include your super-complex module in your main application.

If you want to pass startup parameters to your module, you can do that too. Here's the example from the Mach-II Dashboard module:

{% highlight xml %}
<module name="dashboard" file="/MachIIDashboard/config/mach-ii_dashboard.xml">
  <mach-ii>
    <properties>
      <property name="password" value="superSecretPassword" />
    </properties>
  </mach-ii>
</module>
{% endhighlight %}

The name that you give the module here (in the name="" attribute) is really important. The name gets prepended (that is, put before) every event name generated by Mach-II functions such as BuildURL(), BuildURLToModule(), and the &lt;view:a&gt; tag. You also have to include that module name in any links you hand-build to the other events in your module, or other modules in the larger application. It's what tells the framework to look for an event within a specific module, and not in the main Mach-II app itself. So an event name like "dashboard:main" would look for an event called "main" inside of the "dashboard" module. To use the sample module XML config file above, an event with the name "sampleMod:sampleProtectedEvent" would look for the event "sampleProtectedEvent" inside the module named "sampleMod."

If you fail to prepend the module name (or identifier) to the event name, Mach-II looks for an event with the given event name in the the main application. You'll most likely end up with a "missingEvent" exception as a result.

I'm going to cover modules and Mach-II views in more detail in an upcoming post. In the meantime, if you have any questions, comments, or corrections (I'm looking at you, Peter!), please post away!
