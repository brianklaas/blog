---
layout: post
title:  "Mach-II Modules and ColdSpring"
date:   2009-11-23 11:32:00 -0500
categories: ColdFusion
---

Ah, [ColdSpring](http://coldspringframework.org/). In my opinion, you're the bees' knees. You're what makes managing object dependencies a breeze. You simplify and enable the development of object-oriented ColdFusion apps so much that you're core to most large-scale ColdFusion applications written in the last couple of years. The teams behind Mach-II, Model-Glue and ColdBox like you so much that they made you a key player in their framework stacks. I'm not going to sell you on the virtues of ColdSpring any longer, so if you're not using it already, [start please](http://coldspringframework.org/coldspring/examples/quickstart/)<a href=""></a>.

The key to using ColdSpring in Mach-II modules is the super-fantastic [ColdSpring property](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/WhatsNewInMachII1.6#ColdSpringProperty) that comes bundled with Mach-II 1.6 (and later). You need to be using the ColdSpring property and *not* the old ColdSpring plug-in which was available prior to Mach-II 1.6. I know that updating core infrastructure components like frameworks can be scary, but Team Mach-II has done a great job with making Mach-II backwards-compatible to the 1.1 version, so upgrading to take advantage of newer framework features shouldn't be too scary.

If you need to learn how to get the ColdSpring property working in your Mach-II application, please [review the documentation](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/UsingColdSpringWithMach-II).

The key point about using ColdSpring in Mach-II modules is that you can both set up individual ColdSpring factories for each module in your application *and* have access to a main, or parent, factory defined at the base level of your application. This is quite powerful for two reasons:

- ColdSpring managed objects become specific to the module in which they are needed, and don't clutter up the rest of the application, and,
- Commonly used objects can be created at the main/parent application level and be re-used throughout the entirety of the application, as needed.

In order to use ColdSpring in a module, you need to create a ColdSpring property and define it in your Mach-II XML configuration file for the module:

{% highlight xml %}
<mach-ii version="1.8">
<!-- INCLUDES -->
<includes>
  <include file="/myApp/modules/sampleMod/config/sampleMod-coldSpringProperty.xml" />
</includes>

...listeners, events, plugins, etc...

</mach-ii>
{% endhighlight %}

The basic information you need to provide in your coldSpringProperty.xml file to use ColdSpring in a module is as follows:

{% highlight xml %}
<mach-ii version="1.0">
  <properties>
    <property name="coldSpringProperty" type="coldspring.machii.ColdspringProperty">
    <parameters>
      <parameter name="beanFactoryPropertyName" value="serviceFactory"/>
      <parameter name="configFile" value="/path/to/my/coldSpringServicesFile.xml"/>
      <parameter name="configFilePathIsRelative" value="false"/>
      <parameter name="resolveMachIIDependencies" value="true"/>
      <parameter name="parentBeanFactoryScope" value="application"/>
      <parameter name="parentBeanFactoryKey" value="serviceFactory"/>
      </parameters>
    </property>
  </properties>
</mach-ii>
{% endhighlight %}

Here's what each parameter does:

- **beanFactoryPropertyName** &mdash; This one isn't required, but you really should give your ColdSpring bean factory a name. You *must* do this in the ColdSpring property XML file for your base application, for reasons I'll describe below.
- **configFile** &mdash; This is the path to the ColdSpring XML file for your module, in which ColdSpring-managed beans are defined. This one's required.
- **configFilePathIsRelative** &mdash; This lets Mach-II know if it should be using an absolute or relative path to find the ColdSpring XML file defined above. Defaults to false.
- **resolveMachIIDependencies** &mdash; This defaults to false, but if you plan on using the super-convenient [Mach-II autowriring](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/UsingColdSpringWithMach-II#UsingAutowirebyDependsAttribute) introduced in Mach-II 1.6, then this must be set to true, or nothing gets autowired in to your plugins, listeners, or filters.
- **parentBeanFactoryScope** &mdash; Here's where you tie in the bean factory from your base/parent Mach-II application to your module. By setting this to true, you complete one half of the steps needed to pull in beans defined in the base application but that are used across your whole application. This merely defines the scope in which your parent bean factory is stored. That's most often going to be the application scope.
- **beanFactoryPropertyName** &mdash; The value provided here *must* match the beanFactoryPropertyName value defined in your parent/base app. If it does, you've completed the second half of the steps needed to pull in beans defined in the base application but that are used across your whole application.

So how does all of this work in practice?

I'm not going to go in to how you define ColdSpring-managed beans in the ColdSpring XML file. That's all in the [ColdSpring documentation](http://coldspringframework.org/index.cfm/go/documentation). I'll show you how you can use the beans defined in your base app in your module.

Let's say you have a userService that is defined in your base ColdSpring XML file. This is a pretty common scenario, as you'll most likely need that userService (for things like getting user information) throughout your application, and in all of your modules. Here's the userService defined in your base ColdSpring XML file:

{% highlight xml %}
<beans>
  <bean id="userService" class="path.to.userService">
    <constructor-arg name="dsn">
      <value>${dsn}</value>
    </constructor-arg>
  </bean>
  ...more beans defined here...
</beans>
{% endhighlight %}

In your module's ColdSpring XML file, you can then utilize the userService like this:

{% highlight xml %}
<beans>
  <bean id="reportingService" class="path.to.reportingService">
    <property name="userService">
      <ref bean="userService"/>
    </property>
  </bean>
  ...more beans defined here...
</beans>
{% endhighlight %}

**You don't define the userService in your module's ColdSpring XML file.** Because you've defined it in the parent/base app's ColdSpring XML file, and you set the parentBeanFactoryScope property to "application" (because that's where the parent bean factory is stored) and you set the beanFactoryPropertyName property to the same value as defined in the parent/base app's ColdSpring XML file, Mach-II's ColdSpring property is smart enough to figure out how to grab the userService and put it in the right place.

This is just one example. You can use as many beans from your parent/base app in your modules as you need. By defining commonly used beans in your parent/base app, you reduce your application's overall memory footprint and encourage good object reuse.

**One Big Caveat**

There's one thing you really need to watch out for when using ColdSpring and Mach-II modules: bean name collisions. If you define a bean in your parent/base app called "page" and you also define a bean in your module ColdSpring XML called "page," guess which one is going to get used in your module? I'd recommend that you always using module-specific bean names for the beans defined in your module's ColdSpring XML file. This way, you can ensure that you're not going to run in to naming collisions.

**A Few Other Options**

You can put the bean factory from your module back in to the parent scope if you need to utilize the module bean factory in the parent/base app. You may have bits of functionality encapsulated in a module that need to be utilized elsewhere in your application. For example, if you have a threaded discussion module, you may want to display the 10 most recent posts on your application's home page. As the functionality for doing this is already defined in a bean that is managed by the threaded discussion module's ColdSpring bean factory, you may want to use that bean in the parent/base application rather than defining it again in the parent/base application's ColdSpring XML file. 

To accomplish this, you need to add this one line to your module's ColdSpring property file:

{% highlight xml %}
<parameter name="placeFactoryInApplicationScope" value="true"/>
{% endhighlight %}

This places the module's ColdSpring bean factory in the parent application's application scope. As explained in the [Mach-II documentation for the ColdSpring property](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/UsingColdSpringWithMach-II#ParentChildBeanFactoriesConfigurationforUsewithModules), Mach-II appends the *name of the module* to the beanFactoryPropertyName value when the module's ColdSpring bean factory is added to the parent's bean factory to avoid beans in the module overwriting those defined in the parent. Pretty smart stuff!

Finally, you can use ColdSpring in your module, but not worry about utilizing beans defined in the parent/base application at all. You lose some power and code-reuse, but you may have no good need to use the parent bean factory. In that case, just omit the parentBeanFactoryScope and parentBeanFactoryKey parameters from your module's ColdSpring property XML file, and you're good to go.

Whew! These entries are turning out to be longer than I expected. But there are more to come!

