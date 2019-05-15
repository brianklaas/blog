---
layout: post
title:  "Using AWS Step Functions in CFML: Finishing the Workflow Execution and Returning Data to the Caller"
date:   2019-05-15 16:31:00 -0400
categories: AWS ColdFusion
---

We're finally at the end of this first example Step Functions workflow. We've randomly selected an image from a set of choices and performed machine vision anlysis on that image. How then, does Step Functions know when a workflow is at an end?

There are two state types that tell the Step Functions execution environment that a workflow is at an end: the "succeed" state and the "fail" state. The succeed state obviously indicates that the execution completed successfully, and the fail state indicates that the execution failed at some point. You can have multiple succeed and fail states in your workflow, though typically most workflows only have a single succeed state. 

In this first example workflow of performing analysis on a randomly selected image, the succeed and fail states are represented as follows:

{% highlight json %}
"DefaultState": {
    "Type": "Fail",
    "Cause": "We really should not have ended up here from a numeric value decision."
},

"Finish": {
    "Type": "Succeed"
}
{% endhighlight %}

The "DefaultState" is a fail state, and would only have been called if the random number we generated in the workflow was neither less than or equal to or greater than 50. You can specify a "Cause" value in a fail state, and that will be reported back in the overall Step Functions execution as the reason for failure. If you have multiple fail states in your Step Functions execution, you'd want to use different causes for each different fail state.

The "Finish" state is a succeed state. There isn't anything else you add to this state. 

When a Step Functions execution reaches the succeed state, it takes whatever data is currently in the set of state data in the execution, and returns it to the thing that kicked off the whole Step Functions workflow in the first place. In our case, that is our CFML application.

In the next post, we'll look at how CFML works with Step Functions to both kick off workflows and receive data back from workflows.