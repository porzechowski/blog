---
title: "Code metrics"
date: 2022-09-10
categories:
  - blog
  - software
tags:
  - code metrics
  - cyclomatic complexity
  - max depth
  - cognitive complexity
toc: true
toc_label: "My Table of Contents"
toc_icon: "cog"
---
# You can't control what you can't measure. Short overview of code metrics.

## What are code metrics?
There are several topics of discussion that might lead to violence: politics, religion, which distro is the best, code reviews... just to name few of them.
Fortunately for code, there are some aspects that can be quantified and thus it allows us to assess its quality. At least to a certain degree.

## Why do we need them?
Because we suck…
Being chased by a lion on savannah didn’t prepare us well to analyze triple nested loop that modifies 3-D array, while having in mind 2 logic conditions. We are quite limited on this field, so we must **keep things simple**.
I get that not everyone cares about metrics and even some people find them useless. I’d argue that code metrics can be very useful, can lead to a better code, fewer bugs and we might identify where potential problems might arise in the future.

## How do they look?

### Lines of code

The first, the simplest and the most useless one is the lines of code.
We can count lines of code in a file, module, project or any other code unit. But is it helpful at all?
I’d say not. It is hard to pinpoint any real benefits from using this metrics.
For example, I can imagine having a module with many utility functions, where each function is simple and have meaningful comments. Because of comments and the nature of this module it would have far more lines of code than a file with only one complex function without a single comment.
So basically, this metric can tell us that one file if bigger then another… We don’t really need code metrics for that.

### Cyclomatic complexity
Ok, so this one is quite useful, it’s not perfect but still useful.
It tells us how many independent paths are there in a piece of code.
The more paths there are, the more complicated the piece of code is and the harder to understand this code.
So low complexity means, the code is better, easier to understand, easier to test, easier to maintain.
And why is it useful?
Well, on the function level it tells us, if a function is simple or complex. If it turns out to be complex, maybe it’s worth considering rewriting it, changing approach to a problem or dividing it into smaller pieces. 
On a module level we can measure average complexity, this gives us information which modules are most complex, so we can consider some additional attention like tests or refactors.

Let’s look at an example:

```cpp
bool CheckLogic (bool a, bool b) {
    bool result = false;
    if ((true == a) && (true == b)) {
        result = true;
    }

    return result;
}
```
This is very simple function that checks some logic conditions, its complexity is equal to 2.
Now let’s look at function that sums elements of 2D array:

```cpp
int sum2DArr (int* arrPtr, size_t heigth, size_t width) {
    int sum = 0;
    for (int i = 0; i < heigth; ++i) {
        for (int j = 0; j < width ; ++j) {
                sum += arr[i][j];
        }
    }

    return sum;
}
```
Its complexity is also 2… however most people would say that second one is a bit harder to grasp.
Even two simple nested loops require more cognitive effort than two if conditions.
Therefore, we can introduce another metric, called **max. depth**. It tells us how nested code is.
The more nested the code the harder it is to grasp. So each if inside if or will increase depth by 1.
This gives us 2 useful metrics, and the idea why do we have them.

But what are the number we are looking for? Is complexity 7 low or high? Is max. depth 11 a lot?
And there is no good answer. This will vary from developer to developer, project to project depending on the skills of a team, internal coding rules or simple developer preferences… but if I were forced to provide some number they would look probably like this:

| Complexity number | Max. Depth | Interpretation     |
| ---               | ---       | ---|
| 1-10              | 1-5       | Easy to understand |
| 10-15             | 5-7       | Some effort is needed to understand |
| 15-20             | 7-10      | Hard to understand |
| 20+               | 10+       | You died |

So far so good, but as always there is more!
Consider the following 2 examples:

```cpp
if((!a && b) || (c && ~d)) {
    //do stuff
}
```

```cpp
if((!a && b) || (c && ~d)) {
    //do stuff
}
```

Both examples have same depth and complexity, however the first one is harder to understand.
This is why smart people invented even better metric:

### Cognitive complexity

This metrics tells us how hard (or easy) certain part of code is to understand. 
It will increment every time something is trying to challenge our limited cognitive capacity.
There are 3 basic rules:

> 1. Ignore structures that allow multiple statements to be readably shorthanded into one
> 2. Increment (add one) for each break in the linear flow of the code 
> 3. Increment when flow-breaking structures are nested
> Additionally, a complexity score is made up of four different types of increments: 
> - Nesting - assessed for nesting control flow structures inside each other 
> - Structural - assessed on control flow structures that are subject to a nesting increment, and that increase the nesting count 
> - Fundamental - assessed on statements not subject to a nesting increment 
> - Hybrid - assessed on control flow structures that are not subject to a nesting increment, but which do increase the nesting count

Sadly, there are only few tools that calculates cognitive complexity, however keeping in mind how this works, might result in simpler code.


# Resources

Few words from Thomas J. McCabe - the author of cyclomatic complexity [McCabe](http://www.mccabe.com/ppt/SoftwareQualityMetricsToIdentifyRisk.ppt)

Detailed description of cognitive complexity [SonarSource](https://www.sonarsource.com/docs/CognitiveComplexity.pdf)

Freeware code metrics tool [Source monitor](https://www.campwoodsw.com/sourcemonitor.html)
