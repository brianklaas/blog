---
layout: post
title:  "Start and Stop the ColdFusion 2018 Performance Monitoring Toolset from Anywhere in the Mac OS Terminal"
date:   2019-08-23 15:54:00 -0400
categories: ColdFusion
---

Once of the really nice additions to ColdFusion 2018 is the [Performance Monitoring Toolset](https://helpx.adobe.com/coldfusion/performance-monitoring-toolset/overview-coldfusion-performance-monitoring-toolset.html). This is a very robust tool which helps you see directly inside the black box that the Adobe ColdFusion runtime can sometimes be. 

I develop on the MacOS, and after a restart of my machine, I have to also restart the Performance Monitoring Toolset. The toolset executable on the MacOS is normally located at /Applications/ColdFusion2018PerformanceMonitoringToolset/bin/. Rather than having to change directories to this directory each and every time I need to start or stop the Performance Monitoring Toolset, I'd like to simply say "Start the Performance Monitoring Toolset" no matter where I am in the terminal.

[I wrote about how to do this for your ColdFusion application server](https://brianklaas.net/coldfusion/2013/10/07/Start-and-Stop-the-ColdFusion-Server-from-Anywhere-in-the-Mac-OS-Terminal.html) a couple of years ago, and the same principle applies for the Performance Monitoring Toolset.

Here's how to do this:

1. Launch Terminal.
2. Type "cd ~" to go to your home folder.
3. Type "touch .bash_profile" to create a new bash (Terminal shell) profile file for your Mac OS X account. If you already have a bash profile, you don't need to do this.
4. Type "open -e .bash_profile" to open your bash profile file in TextEdit.
5. Add the Performance Monitoring Toolset /bin directory to your bash pofile by adding this line to your .bash_profile:

{% highlight bash %}
export PATH=$PATH:/Applications/ColdFusion2018PerformanceMonitoringToolset/bin
{% endhighlight %}

If you're already including the path to your ColdFusion application server in your .bash_profile, then you'd put a colon (:)
 character between paths, like this:

{% highlight bash %}
export PATH=$PATH:/Applications/ColdFusion2018/cfusion/bin:/Applications/ColdFusion2018PerformanceMonitoringToolset/bin
{% endhighlight %}

6. Save your edited .bash_profile file.
7. Type ". .bash_profile" to reload your bash profile. (Yes, that's "dot space dot")

Once you do this, you can type any of the following commands from anywhere in the Terminal and it will affect the Performance Monitoring Toolset:

- perfmontoolset start
- perfmontoolset stop
- perfmontoolset restart
- perfmontoolset status
