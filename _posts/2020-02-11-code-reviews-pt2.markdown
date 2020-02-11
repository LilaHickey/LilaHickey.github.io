---
layout: post
title: code reviews - do this, not that - pt2
date: 2020-02-11 06:00:00 -0700
---

> Note: This is the second of a three-part series on how to give better code reviews.


1. [Can't Win, And So I Will Not Try]({% post_url 2020-02-04-code-reviews-pt1 %})
2. [Everything is Awful (When You're Part of a Team)](#everything-is-awful-when-youre-part-of-a-team)
3. The Captain Jack Sparrow

## Everything is Awful (When You're Part of a Team)

We've all seen this: a pull request riddled with critical feedback, but no examples of how to fix the problems being highlighted, or even guidance on relative severity/recommended prioritization.  The hallmark of this review is __overwhelming negativity__.  Causes include:

1. Explicit distrust of the author[^1].
3. Incentives created by our industry's tendency to equate criticality with skill.
    > ...I found 10 problems, you only found 1...I'm your 10x dev!

Most of us don't enjoy giving these kinds of reviews - a negative headspace is unpleasant to occupy, it can cloud our professional judgement - and the review is often unhelpful to the author, both acutely and for their skill development.

So how can we do better, even if everything actually _is_ awful?

### Think Positive: Guiding Principles

1. __Assume Good Intent__. Not because it's kind, but because it produces a better review.  If your automatic answer to the question:
    >__"Huh, why did they do it that way?"__ is __"Because they're incompetent!"__

    you risk overlooking additional context that would explain their particular choices.
2. __Do _NOT_ Assume Good Implementation__. Many of us waste time trying to make sense of egregiously wrong code; the resulting frustration makes us more likely to fall into negative review patterns.  Be quick to ask clarifying questions
3. __Don't Shoot the Developer__.  Code is the result of choices made by a long train of stakeholders. Be mindful of whether your criticisms are pointed at the right person/group.
4. __Keep Your Eye on the Prize__.  What's your goal? Is your feedback serving it? 

### Concrete Techniques:
1.  __Use the "Decline" and "Needs Work" Buttons.__ Powering through a review when the code isn't ready places an unfair burden on reviewers _and_ produces the high volume of negative feedback we want to avoid.  
    * _Be sure to include a reason that helps the author understand how to proceed_, e.g.:
    ```
    Declining this because our microservices pattern calls for aggregation
    to occur at the individual service layer; suggest you migrate your code
    to that repository and re-open.
     ```
2. __Programmatically Enforce "Low Level" Standards.__  For example, only allow pull requests to open with a passing run of lint and unit tests. Why?
    *  uniform standards ensure we're not acting with bias - [there's a lot to be said for consistency](https://youtu.be/XXtQMz1RGNw?t=66){:target="_blank"}.
    *  it eliminates opportunities to enter a "distrust" cycle - over things which may _actually be a bad proxy for developer skill[^2]_
3. __Justify Your Comments__: This is a great way to spot distrust or general negativity in your reviewing.  
    * _Empirical concerns_: explain the consequences of the current code.
    * _Subjective concerns_: explain your thought process as a starting point for conversation
4. __Provide Examples.__ Most people are already writing the best code they know how to - telling them they're wrong is infinitely less useful than _showing them what right looks like_. 
5. __Avoid Opinion Poisoning.__ Perform your initial review with other reviewer's comments hidden.
6. __Constrain your feedback to the current changes__.
7. __Compliment good work!__ This reminds the author that you're an ally, which may increase their receptivity to your feedback and _is also just a nice thing to do._


### Related Topics

Although it's not intrinsic to this anti-pattern, "bikeshedding" is often involved; those not familiar with it by name will quickly recognize it [by description](https://en.wikipedia.org/wiki/Law_of_triviality).

---
#### Footnotes

[^1]: Persistent distrust signals a need for managerial intervention, but that isn't your job and a code review is not the venue. As ICs, our responsibility is to communicate the concern to a manager, provide some tangible examples of the problem(s), and then _try to minimize its impact on how we do our own job_[^3].  

[^2]: Even absent enforcement, failure to observe existing standards should make us leery about someone's attention to detail and/or the value they place on consistency, but it should probably be a more nuanced concern than we would feel on seeing significant logic errors.  

[^3]: This is usually easier said than done.
