---
layout: post
title:  "Using AWS Step Functions in CFML: Making Choices and Branching in a Step Functions Workflow"
date:   2019-04-25 15:51:00 -0400
categories: AWS ColdFusion
---

Thus far in this first [example Step Functions workflow that performs image analysis on a randomly selected image](https://github.com/brianklaas/awsPlaybox/blob/master/stateMachines/choiceDemoStateMachine.json), our workflow has progressed in a purely linear fashion. We started at the first step in our workflow, and moved to the second step. We could continue to move linearly from one step to another, but how often do you build applications (or even individual functions) that are purely linear in execution? We often have to make choices based on the state of the application at any given time. One of the great powers of the Step Functions environment is that we can make choices and take different paths of execution based off those choices.

Now that we've generated a random number betwen 1 and 100 in the generateRandomNumber task, and put the result of that task into a variable titled $.randomNumber, we pass that $.randomNumber value into our next state, "ChoiceState," the definition of which is this:

{% highlight json %}
"ChoiceState": {
    "Type" : "Choice",
    "Choices": [
    {
        "Variable": "$.randomNumber",
        "NumericLessThanEquals": 50,
        "Next": "setDataForImageOne"
    },
    {
        "Variable": "$.randomNumber",
        "NumericGreaterThan": 50,
        "Next": "setDataForImageTwo"
    }
    ],
    "Default": "DefaultState"
}
{% endhighlight %}

"ChoiceState" is, well, a state where you make a choice and execution continues based on the choice that is evaluated in the state definition.

### How Choice States Work

[Choice states](https://docs.aws.amazon.com/step-functions/latest/dg/amazon-states-language-choice-state.html) are, effectively, our "if-then-else" flow constructs in our Step Functions workflow. Choice states are made up of an array of choices. Each choice has a comparator that lets the Step Functions execution environment which choice should be made. Additionally, each choice state has a default state which will automatically be chosen if there is no match to any of the comparators in the array of choices. There many comparators to chose from, including:

- StringEquals
- StringLessThan
- StringGreaterThan
- StringLessThanEquals
- StringGreaterThanEquals
- NumericEquals
- NumericLessThan
- NumericGreaterThan
- NumericLessThanEquals
- NumericGreaterThanEquals
- BooleanEquals
- TimestampEquals
- TimestampLessThan
- TimestampGreaterThan
- TimestampLessThanEquals
- TimestampGreaterThanEquals
- And
- Or
- Not

In our case, we have a random number as our input, so our comparators will be numeric comparators. The first choice in the array of chocies says "If $.randomNumber is less than or equal to 50, the next step in this execution will be setDataForImageOne."

{% highlight json %}
"Variable": "$.randomNumber",
"NumericLessThanEquals": 50,
"Next": "setDataForImageOne"
{% endhighlight %}

The next (and final) choice in our array of choices says "If $.randomNumber is greater than 50, the next step in this execution will be setDataForImageTwo."

{% highlight json %}
"Variable": "$.randomNumber",
"NumericGreaterThan": 50,
"Next": "setDataForImageTwo"
{% endhighlight %}

Finally, we have our default option, which exists outside of the array of choices and will automatically be selected if there is no matching comparator in the array of choices. We should never arrive at the default choice because our randomly generated number will always be between 1 and 100. We still have to specify a default state, however, and if we reach it, there's an error in our execution.

Next up, we'll look at the setDataForImageOne and setDataForImageTwo states, which demonstrate how you can inject key-value pairs into the current set of data being passed through your Step Function workflow.