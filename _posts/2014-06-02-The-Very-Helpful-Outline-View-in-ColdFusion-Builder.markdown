---
layout: post
title:  "The Very Helpful Outline View in ColdFusion Builder"
date:   2014-06-02 15:26:00 -0400
categories: ColdFusion
---

Now that ColdFusion Builder 3 is out, I thought it would be a good time to remind people of the very useful Outline view. The Outline view shows you the structure of your .cfc or .cfm file. [Ray Camden has a video from the first version of ColdFusion Builder](http://www.raymondcamden.com/index.cfm/2009/7/17/ColdFusion-Builder-and-Outline-Mode) which explains the view in more depth. I really only use it on CFCs, as there are simply too many tags &mdash; and a mixture of CF tags and HTML tags (sorry, that's how it is with CF) &mdash; in .cfm files.

By default the Outline view will show you all of the tags (in tag-based CFCs) and functions (in script-based CFCs) in your CFCs. You can customize this, however, so that it *only* shows cffunction tags or function declarations, rather than any tag or function in the CFC. I find this to be extremely useful for quick navigation within a CFC. Just click on the function name and &mdash; voil&agrave;! &mdash; he editor jumps to that function. (Yes, this means taking one hand off the keyboard, but I'm very old-school like that.)

To customize the Outline view, go to Preferences &rarr; ColdFusion &rarr; Profiles &rarr; Editor &rarr; Outline, and then choose "Show selected tags." You can then decide which tags you want Builder to show in the Outline view.

<p align="center"><img src="/assets/postImages/builderOutlinePrefs.png" width="500" height="292" border="1" alt="ColdFusion Builder Outline View preferences"></p>

I'd suggest setting it so that it only shows cffunction tags.

It's a small change, but one that I find super-useful on a daily basis.