---
layout: post
title:  "Start and Stop the ColdFusion Server from Anywhere in the Mac OS Terminal"
date:   2013-10-07 15:54:00 -0400
categories: ColdFusion
---

I've been using ColdFusion for years, but finally got around to setting up my Mac OS Terminal so that I could start and stop the ColdFusion 10 application server from anywhere in my Terminal, not just when I'm in the /ColdFusion10/cfusion/bin directory. This has become a common task with the removal of the simple ColdFusion application manager that existed prior to ColdFusion 10.

On the Mac OS, there's a handy system service that lets you [open any folder in the Terminal by right-clicking on it in the Finder](http://osxdaily.com/2011/12/07/open-a-selected-finder-folder-in-a-new-terminal-window/). That's all well and good, but then you have to open the /Applications/ColdFusion10/[instance name]/ directory in the Finder. It would be a whole lot easier just to go into the Terminal, no matter where you are in the file system, and say "coldfusion restart."

Here's how to do this:

1. Launch Terminal.
2. Type "cd ~" to go to your home folder.
3. Type "touch .bash_profile" to create a new bash (Terminal shell) profile file for your Mac OS X account. If you already have a bash profile, you don't need to do this.
4. Type "open -e .bash_profile" to open your bash profile file in TextEdit.
5. Add the ColdFusion /bin directory to your bash pofile by adding this line to your .bash_profile:

{% highlight bash %}
export PATH=$PATH:/Applications/ColdFusion10/cfusion/bin
{% endhighlight %}

If the path to your ColdFusion /bin directory is different, you'll need to adjust it from the above.

6. Save your edited .bash_profile file.
7. Type ". .bash_profile" to reload your bash profile. (Yes, that's "dot space dot")

Once you do this, you can type any of the following commands from anywhere in the Terminal and it will affect your default ColdFusion 10 instance.

- coldfusion restart
- coldfusion stop
- coldfusion start
- coldfusion status
