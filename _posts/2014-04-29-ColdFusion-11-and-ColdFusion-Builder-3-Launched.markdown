---
layout: post
title:  "ColdFusion 11 and ColdFusion Builder 3 Launched!"
date:   2014-04-29 08:38:00 -0400
categories: ColdFusion
---

Adobe today released new versions of both the ColdFusion server (v11) and the ColdFusion Builder IDE (v3). There's lots of new features in both, but I wanted to take the time to point out my two favorites:

1 &ndash; New member functions in the Adobe version of the CFML language

The ColdFusion documentation wiki [explains this feature in detail](https://wikidocs.adobe.com/wiki/display/coldfusionen/Using+the+member+functions), but the reasons that I really appreciate this addition to the Adobe CFML runtime are that 

a) it's much more in-line with how you write similar code in other languages, especially JavaScript, and 

b) it's brought support for map() and reduce() functionality to structures, arrays, and lists in Adobe CFML.

This second point is really important because as developers, we should be thinking about operating on collections and not individual items. Map() and reduce() are key to expressive programming in JavaScript and across many "big data" toolsets (you could say that Hadoop is just a giant runtime built to run two functions: map() and reduce()). If you want to program in a functional style, map() and reduce() are key tools in achieving that goal &mdash; at least in my experience, which has largely revolved around JavaScript. 

2 &ndash; The performance of ColdFusion Builder 3

Damn is performance better in Builder 3! The IDE starts up and completes all the Eclipse-based setup in seconds rather than almost a minute under Builder 2. Everything is so much more responsive and rarely am I bitten by waiting for content refreshes as I was in prior versions of Builder. I've already made the switch to using ColdFusion Builder 3 full time as my CF IDE.

[The full list of what's new in ColdFusion 11](https://wikidocs.adobe.com/wiki/display/coldfusionen/New+in+ColdFusion) can be found on the ColdFusion documentation wiki. Explore and enjoy.