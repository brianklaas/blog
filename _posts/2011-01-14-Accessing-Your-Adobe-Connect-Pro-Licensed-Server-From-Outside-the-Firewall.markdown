---
layout: post
title:  "Accessing Your Adobe Connect Pro Licensed Server From Outside the Firewall"
date:   2011-01-14 09:54:00 -0500
categories: AdobeConnect
---

I'm a pretty big fan of [Adobe Connect](http://www.adobe.com/products/adobeconnect.html). We use it in my job and it's a critical part of our learning infrastructure. Adobe even featured us in a case study about using Connect in higher education.

We've been using Connect since it was Breeze Meeting 5 (remember that naming debacle?), and the upgrade process from one version to another is really quite simple and seamless, particularly if you're not building multiple clusters with multiple origin-edge configurations. There's one big, giant gotcha about setting up Connect that has not changed since the days of Breeze Meeting 5: getting people outside your network to actually access Connect meetings via a Web browser.

The default approach of the *licensed* version of Connect is that Connect will be deployed behind your firewall to your company and its employees alone, and that no one from outside your network will need to access your Connect setup. That works well for quite a few of the customers who have a licensed version of Connect. If you're an educational institution that needs to have students from around the world use your Connect setup, that's a bit of a problem.

The solution, which isn't well documented, is this: you need to add a line to the custom.ini file in your Connect installation that tells the Flash Media Server inside of Connect (the server which powers the actual Connect meetings, not the administrative UI) how to resolve itself to your external domain name. The line needs to be all in lowercase, and should look like this:

host.machine name=connectServer.domain.name

So if the machine name of your Connect instance is bigBadConnect1, and your domain is connect.mysupercoolsite.com, the line you need to add to the custom.ini file is:

host.bigBadConnect1=connect.mysupercoolsite.com

This tells the Flash Media Server running inside of Connect to resolve to the *external* domain name of the server. Without it, people from outside your network will be able to access the Connect administrator, but not actually connect to any meeting sessions or watch any recordings. They'll get stuck "Connecting..." to your Connect meetings and never get any farther in the process, even if all the appropriate ports are open on your (and their) firewall. With this addition to the custom.ini file, all works well.

Note that this doesn't affect the hosted version of Connect at all. It's only an issue for licensed (internally hosted) customers.