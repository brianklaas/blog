---
layout: post
title:  "Using Mach-II Property CFCs and ColdSpring"
date:   2009-09-17 08:44:00 -0400
categories: ColdFusion
---

One of the things that I really like about using [Mach-II](http://www.mach-ii.com/), my MVC framework of choice for developing ColdFusion applications, is the level of support and interaction with the community that the core team behind the framework provides. I know it's a lot of work and a lot of time for them (and time is the one thing that none of us ever have enough of), and I'm always impressed with how they go out of their way to help boneheads like me in our hour of need.

One of the nifty features introduced in Mach-II 1.5 are [property CFCs](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/NewPropertyDatatypes). In addition to supporting structs and arrays of properties in your Mach-II XML configuration file, you can also use CFCs if you want to do more advanced property setup. A common usage scenario for property CFCs is to store application globals. Prior to Mach-II 1.5, you'd create and store application globals in the configure() method of a plug-in, but that's really not what plug-ins in Mach-II are for. Plug-ins intercept execution points on each request, and weren't originally intended for application global functionality, but that's the hack that the community developed to work around a limitation of Mach-II.

When I first used property CFCs to set up application globals, I ran into a bit of a problem. My existing application globals were defined in an "appConstants" plug-in. The plug-in utilized objects managed via [ColdSpring](http://www.coldspringframework.org/), because that's how I manage my object creation and dependency injection in CF (and, really, if you're not using ColdSpring, you really should). I needed to pull in a geographyService object from ColdSpring so I could put a query of states and query of countries in to my app globals. Pretty standard stuff.

Here's what I learned about doing this using Mach-II property CFCs:

1. Property CFCs are managed by Mach-II and **not** by ColdSpring. This is super-important and what really tripped me up. 

Oftentimes in ColdSpring-managed objects where we're doing property injection (rather than constructor argument injection), we'll have something that looks like this (truncated for brevity):

{% highlight javascript %}
<cffunction name="init"...>
  <cfset variables.geographyService = 0 />
</cffunction>
<cffunction name="setGeographyService">
  <cfargument name="geographyService">
  <cfset variables.geographyService = arguments.geographyService />
</cffunction>
{% endhighlight %}

This is pretty standard stuff for ColdSpring object injection. The trouble is that this doesn't really work for property CFCs in Mach-II 1.5 or later. Mach-II needs to handle the injection, not ColdSpring, so if you rely on ColdSpring to do this, you'll always have a variables.geographyService that = 0.

The solution? Use Mach-II 1.6's very handy "depends" autowiring enabled via Mach-II 1.6's [ColdSpring property](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/UsingColdSpringWithMach-II). If you're using Mach-II 1.6 or later you should be using the MachII.properties.ColdspringProperty instead of the older, and deprecated, ColdSpring plugin. The ColdSpringProperty in Mach-II 1.6 includes the ability to add the attribute "depends" to your &lt;cfcomponent&gt; declaration and it will autowire whatever objects you specify in to that object, without any additional calls or references to the ColdSpring framework. It's really sweet and very handy.

So for my appGloabls property CFC, my &lt;cfcomponen&gt; tag looks like this:

{% highlight javascript %}
<cfcomponent displayname="appGlobals" extends="MachII.framework.Property" depends="geographyService" output="false" hint="I set up some app constants used throughout the life of the app.">
{% endhighlight %}

The depends="geographyService" attribute autowires the geographyService (managed by ColdSpring) in to this property CFC. Now I can call it and use it just as I did back in the days of my appConstants plug-in. For example:

{% highlight javascript %}
<cfset globalVars.qryStateProvinceList = variables.geographyService.getStateList() />
<cfset globalVars.qryCountryList = variables.geographyService.getFilteredCountryList() />
{% endhighlight %}

2. If your property CFC depends on ColdSpring in any way, you need to declare that property CFC <b>after</b> the ColdSpring property declaration in your main Mach-II XML config file.

The always-helpful [Peter J. Farrell](http://blog.maestropublishing.com/) (from the Mach-II team) explained to me that if you use &lt;includes&gt; in your Mach-II XML config file, as is the default example and method for utilizing the MachII.properties.ColdspringProperty, properties defined in the &lt;includes&gt; are parsed **after** the properties defined in your main Mach-II XML config file. So if you declare a property CFC that utilizes ColdSpring in your main Mach-II XML config file, it's going to get parsed and processed before the ColdSpring property gets parsed and set up, so the "depends" autowiring just won't work.

The solution? Make a really simple include for your property CFC or CFCs that use ColdSpring and declare that after your MachII.properties.ColdspringProperty include, as follows:

&lt;includes&gt;
&lt;include file="config/coldspringProperty.xml" /&gt;
&lt;include file="config/appGlobalsConfig.xml" /&gt;
&lt;/includes&gt;

Utilizing this setup, I'm now able to utilize ColdSpring dependency injection and properly move my application globals setup in to a property CFC where it belongs.
