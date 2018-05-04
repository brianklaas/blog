---
layout: post
title:  "Creating A CF Administrator View in CFBuilder 2 via Extensions"
date:   2011-03-07 12:20:00 -0500
categories: ColdFusion
---

One of the really great new features of ColdFusion Builder 2 is the ability to create brand-new views in Eclipse from a CFBuilder 2 extension. This opens up a whole host of possibilities for developing with ColdFusion.

Here's a quick example of how this can be really useful: say you want to have a way to quickly access the ColdFusion server administrator on your localhost from within CFBuilder. Sure, you can ALT-Tab to switch to Firefox/Safari/Chrome and pull up the administrator, but wouldn't it be nice to not have to leave your IDE? With extensions in CFBuilder 2, it's really easy.

Attached to this post is a .zip file of a simple ColdFusion Builder 2 extension that does just this. I'm not going to go over the basics of extensions &mdash; you can read the docs or just Google that. Instead, I'm going to focus on the parts of the extension directly related to creating a new view in Eclipse.

In the ide_config.xml file for the extension (which sets up the basic configuration for the extension), I tell CFBuilder that I want a new menu item in the project view. Any time I right-click in the project view, "Open CFAdmin on Localhost" will appear as a menu item. I also tell CFBuilder to run openview.cfm when this menu item is selected.

openview.cfm is really simple. Here's the entirety of the file and the new CFBuilder 2 magic:

{% highlight javascript %}
<cfheader name="Content-Type" value="text/xml">
<cfoutput>
  <response showresponse="true">
    <ide url="http://localhost/cfide/administrator/">
      <view id="cfAdmin" title="CF Admin on Localhost" />
    </ide>
  </response>
</cfoutput>
{% endhighlight %}

All that happens here is we set the content type to text/xml so that CFBuilder can parse the response, and then in the &lt;response&gt; tag, I tell the IDE to open a new view with an ID of "cfAdmin," label it "CF Admin on Localhost" and then give it the URL of the CF administrator on my local server. The &lt;ide&gt; &lt;view&gt; &lt;ide&gt; block is what makes this happen. 3 lines of code is all it takes to create a new view in Eclipse using CFBuilder 2. Try that with native Java code.

There are other ways of generating views in Eclipse using CFBuilder 2, and you should read the docs to find out the what those other methods bring to the table. In the meantime, you now know how simple it is to create a new view in Eclipse using a CFBuilder extension. You can bring your Web-based bug tracking system into Eclipse this way, keep an eye on your GMail, or handle any number of other Web-based tasks right inside CFBuilder. Just change the URL specified in the &lt;ide&gt; tag and you're good to go.

One important caveat: if the URL you bring up in the Eclipse view has buttons or links that open new windows, they won't appear. There's also no support for Flash or other plug-ins. However, if the site you're bringing up requires basic HTTP authentication, CFBuilder will use your Keychain on Mac OS X to supply the required username and password. Sweet!

Finally, as the CFBuilder 2 docs note, any views that you create via extensions do not persist between restarts of CFBuilder. You'll need to open those views every time you launch the application, or as needed. Perhaps this is something Adobe can address for CFBuilder 3.

[Download the CF Administrator View extension](/assets/code/OpenAdminWindow.zip) (Requires ColdFusion Builder 2)