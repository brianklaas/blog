---
layout: post
title:  "A Brief Series on Mach-II Modules"
date:   2009-11-10 10:40:00 -0500
categories: ColdFusion
---

I've been doing a *lot* of work with modules in [Mach-II](http://www.mach-ii.com/) in the past couple months and thought I'd blog about some of the stumbling blocks I've encountered.

First, a little background:

A Mach-II module is an application which runs inside of a parent Mach-II application. A module usually groups together common functionality in to a related package, encapsulates that package nicely within your Mach-II app, and inherits a lot of the base configuration from the parent Mach-II application. Some people build Mach-II modules that are meant to be reused across multiple applications, for multiple sites, while others build open-source modules which are meant to be redistributable and used by anyone, anywhere. My situation is a little different.

I'm in the process of re-architecting one of our major applications, getting years of old cruff and rebuilding everything on top of Mach-II 1.8. A large part of this project entails taking some existing Mach-II apps (build under 1.1, 1.5 and 1.6) and integrating them in to the new, main Mach-II 1.8 app. A good amount of the work involves throwing out what's handled by the parent app, adding in new parent-child integration points where needed, and refactoring views to get them to work inside of modules.

So that's the context for my work. Your experience may be different, of course, especially if you are looking to build a standalone module that can be integrated in to anyone's Mach-II app, like the super-fantastic Mach-II Dashboard application or Mach Blog. (BTW: If you develop with Mach-II and you're not using the Mach-II Dashboard application, you're wasting time and effort. Single-listener, plugin, filter or config reload is a *huge* timesaver &mdash; and that's only part of what you can do with the Dashboard.)

The [full introduction to Mach-II modules](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki/IncludesAndModules) can be found on the Mach-II wiki. I'm going to assume that you're quite familiar with how Mach-II works and know your way around a Mach-II configuration file.

I also want to say that I've not used each and every feature supported by Mach-II modules. I've not had need (yet) for things like the runParent attribute of the &lt;plugin&gt; tag. If you're interested in that topic, please see [this thread on the always-useful Mach-II Google group](http://groups.google.com/group/mach-ii-for-coldfusion/browse_thread/thread/fe36fb241e5a1609/f7e5a32ad9a93fa4?lnk=gst&q=runparent#f7e5a32ad9a93fa4).

There's a lot of ground to cover, so I thought I'd break things apart in to a series of posts:

- Modules and Mach-II config files
- Modules and ColdSpring
- Modules and Views
- Converting an Existing App to a Module

I'm going to *try* to get one of these out a week. In the end, if Team Mach-II finds it interesting, useful, and remotely accurate, I'll consolidate some of this to post on the Mach-II [wiki](http://greatbiztoolsllc.trac.cvsdude.com/mach-ii/wiki).
