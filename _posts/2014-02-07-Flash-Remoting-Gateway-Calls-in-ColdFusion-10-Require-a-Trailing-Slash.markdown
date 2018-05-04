---
layout: post
title:  "Flash Remoting Gateway Calls in ColdFusion 10 Require a Trailing Slash"
date:   2014-02-07 08:42:00 -0500
categories: ColdFusion
---

With the rise of ECMAScript 6, tools like WebRTC, the Web Audio API, and HTML5 and CSS3 in general, we're seeing fewer and fewer Flash applications in the wild. My team, like many, still have a few out there, mostly revolving around real-time communications because WebRTC is limited to newer browsers (no Mobile Safari, sigh), and polyfills aren't always the real thing.

We ran into an issue this week where a customer reported that one small Flash app wasn't logging activity like it used to. Logging was done via a call from the Flash app to a Flash Communication Server app instance which would then pass the data to be logged to a ColdFusion cluster. (Flash Communication/Interactive Server was required for the realtime portion of the application.) To give you an idea of how small and unused this app is, the app had stopped logging properly in May, 2013, but the problem wasn't reported to us until just this week. &gt;.&lt;

After a lot of head scratching, we found the source of the failed Flash Remoting calls from the Flash Media Server:

In ColdFusion 9, you could refer to the Flash Remoting Gateway in your main.asc file as:

{% highlight javascript %}
NetServices.setDefaultGatewayUrl("http://yourwebsite.com/flashservices/gateway");
{% endhighlight %}

In ColdFusion 10, a trailing slash is required when you create the Flash Remoting Gateway reference:

{% highlight javascript %}
NetServices.setDefaultGatewayUrl("http://yourwebsite.com/flashservices/gateway/");
{% endhighlight %}

If you do not include that trailing slash, the call to NetServices.setDefaultGatewayUrl will fail.