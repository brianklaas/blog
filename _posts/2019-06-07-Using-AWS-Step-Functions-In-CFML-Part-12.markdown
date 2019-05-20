---
layout: post
title:  "Using AWS Step Functions in CFML: Speeding Workflows with Parallel States"
date:   2019-06-07 15:51:00 -0400
categories: AWS ColdFusion
---

We've now worked through the first half of this [second example Step Functions workflow](https://github.com/brianklaas/awsPlaybox/blob/master/stateMachines/transcribeTranslateSpeakWorkflow.json), which transcribes, translates, and speaks the content of a video file. The first half was dominated by a wait loop, while we waited for transcription of the video to complete. Now that we have the transcription of the content of the video, we can move on to translating that information into multiple lanaguages and speaking it into those translated languages.

### Introducing the Parallel State

The final Step Function state type that we'll use in this workflow is the parallel state. A parallel state is simply an array of individual states that are executed in parallel. There's no good reason that all work must be done serially, and it's often vastly faster to have a series of similar tasks executed in parallel. 

Parallel states contain an array of individual state flows that will execute simultaneously. Each parallel flow is defined as a separate member of the the "Branches" array. Each parallel flow in the array of parallel executions begins with a "StartAt" property &mdash; just like a regular Step Functions workflow. The last state in each parallel flow in the array ends with an "End" property being set to true. This tells the Step Functions Execution environment that the flow of that parallel branch is at an end. Finally, the parallel state itself has a single "Next" property, which tells the Step Functions execution environment where to go next once *all* parallel exections complete.

The [full code for the workflow can be found in the AWSPlaybox repository](https://github.com/brianklaas/awsPlaybox/blob/master/stateMachines/transcribeTranslateSpeakWorkflow.json). Below in an except of the Step Functions JSON so you can see how the parallel state is defined.

{% highlight json %}
"Make Versions": {
    "Type": "Parallel",
    "Next": "Workflow Complete",
    "Branches": [
        {
          "StartAt": "makeSpanishStart",
          "States": {
            "makeSpanishStart": { },
            "makeSpanishText": { },
            "prepSpanishVoiceOutput": { },
            "makeSpanishAudioFile": { 
                "End": true
            }
          }
        },
        {
          "StartAt": "makeFrenchStart",
          "States": {
            "makeFrenchStart": { },
            "makeFrenchText": { },
            "prepFrenchVoiceOutput": { },
            "makeFrenchAudioFile": {
              "End": true
            }
          }
        },
        {
          "StartAt": "makeGermanStart",
          "States": {
            "makeGermanStart": { },
            "makeGermanText": { },
            "prepGermanVoiceOutput": { },
            "makeGermanAudioFile": {
              "End": true
            }
          }
        }
    ]
}
{% endhighlight %}

In this workflow, the steps for each parallel flow are essentially the same. This does not always need to be the case. The steps in each parallel flow can do completely different work. In this workflow, however, each parallel flow represents a different language for which we'll execute the same tasks to yield similar output across languages.

### Limitations of Parallel States

Parallel states are great because you can speed up the execution of similar tasks that need to all execute before continuing on in the workflow. They have one major limitation, however: you cannot dynamically generate the array of parallel flows. Every step in every array must be created manually in your Step Functions JSON definition. This is probably the biggest customer request for improvments to Step Functions. It would be great to pass in a list or array and have the Step Functions execution environment iterate over that list or array and run the contained workflow on each item. Alas, for now, we have to write out everything manually.

You can see from the above workflow that all we are doing is performing the same tasks on the source transcription file in Spanish, French, and German. We easily could have added additional languages to the list, should we want to do so. However, for each additional language, the Step Functions JSON would have to list out all the individual steps in the parallel flow.

In the next post, we'll look at the first step in each of these parallel flows: taking the transcript of the video and translating it into another language.