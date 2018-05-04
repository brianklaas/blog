---
layout: post
title:  "Duff's Device in ColdFusion"
date:   2010-12-06 13:09:00 -0500
categories: ColdFusion
---

I'm currently reading "[Even Faster Web Sites: Performance Best Practices for Web Developers](http://www.amazon.com/Even-Faster-Web-Sites-Performance/dp/0596522304)" and one of the topics that is covered in the book is JavaScript performance. There are a *lot* of performance tips and tricks that can be applied when building JavaScript applications. As most developers know, loops are one area where performance tends to slow down and that are usually ripe for optimization.

One loop optimization technique is *[unrolling](http://en.wikipedia.org/wiki/Loop_unwinding)*, or unwinding, a loop</a>. Instead of looping over [n] iterations inside a loop, and incurring the overhead of doing the usual checks to see if you should stop looping, you just write (somewhat) repetitive code and avoid the loop all together. This works well when you have just a couple of iterations to unwind, but not when you have loops of variable or significant length.

[Duff's Device](http://en.wikipedia.org/wiki/Duff%27s_device) is a loop unrolling technique that can handle loops of variable or significant length. Basically, you loop as you normally would, but instead of looping over each iteration, you break up the loop into blocks of 8. (Breaking into blocks of 8 calls in each loop was determined to be the optimal size for the block.) This reduces the cost of variable lookup and evaluation in each loop by reducing the total number of loops via unrolling and also handles loops of variable or significant length.

I wanted to see if there was a ColdFusion-specific implementation of Duff's Device, but couldn't find one. Below is the code I used to implement Duff's Device in CF.

{% highlight javascript %}
<cfscript>
/* Setup */
values = ArrayNew(1);
arraySize = RandRange(400000, 500000);
for (x=1; x < arraySize; x++) {
  values[x] = RandRange(100, 1000);
}

function process(numToProcess) {
  var result = arguments.numToProcess ^ 4;
}

/* The old-fashioned way */
oldwayBegin = GetTickCount();
for (x=1; x < arraySize; x++) {
  process(values[x]);
}
oldwayEnd = GetTickCount();
oldLoopTime = oldwayEnd - oldwayBegin;
WriteOutput("Old Way Total Time: " & oldLoopTime);

duffsBegin = GetTickCount();
/* Duff's Device */
iterations = Ceiling(arrayLen(values) / 8);
startAt = arrayLen(values) % 8;
i = 1; // We start at 1 because all CF arrays start with an index of 1, not zero
do {
  switch(startAt) {
    case 0: process(values[i++]);
    case 7: process(values[i++]);
    case 6: process(values[i++]);
    case 5: process(values[i++]);
    case 4: process(values[i++]);
    case 3: process(values[i++]);
    case 2: process(values[i++]);
    case 1: process(values[i++]);
  }
  startAt = 0;
} while (--iterations > 0);
duffsEnd = GetTickCount();
duffsLoopTime = duffsEnd - duffsBegin;
WriteOutput("Duff's Total Time: " & duffsLoopTime);
</cfscript>
{% endhighlight %}

We begin by creating an array which we'll loop through in both the traditional way (one iteration for each item in the loop) and then via the Duff's Device method. The Duff's Device method takes the array length, divides it by 8, and makes sure that any remainder is properly handled. You then loop through the array and process each item in the array in blocks of 8 calls.

This example is totally contrived, and the process() function does nothing more than force the CF runtime to do some basic math.

So what's the performance gain? Here are the results from 10 iterations on my 3.06Ghz Intel Core Duo 2 iMac running Mac OS X 10.6.4 with 4GB of RAM (and 2GB given to the CF 9.0.1 standalone installation that I used):

|Run|Old Way Total Time:|Duff's Total Time: |
|--- |--- |--- |
|1.|2903|1384|
|2.|1786|1399|
|3.|2866|1315|
|4.|2547|1412|
|5.|2085|1152|
|6.|2619|1231|
|7.|1280|2369|
|8.|2033|3593|
|9.|1323|2994|
|10.|1412|3510|
|11.|1278|1765|
|12.|2326|1563|
|13.|2871|1224|
|14.|1307|1339|
|15.|1232|2857|

Unfortunately, there doesn't seem to be a consistent pattern or benefit here. Most of the time, the Duff's Device method runs faster, but not always. It's a variable array length that I'm looping over, so that may be part of the issue. I'm also not familiar with how the native Java code underlying ColdFusion handles looping and arrays and what optimization is going on there.

The whole point of the exercise was to use Duff's Device in CF. Now that I know about it, I may use it as an optimization technique on future looping code &mdash; if the pre- and post-optimization code shows that there's a consistent benefit.