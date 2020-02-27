---
layout: post
title: code reviews - do this, not that - pt3
date: 2020-02-26 06:00:00 -0700
---

> Note: This is the third of a three-part series on how to give better code reviews.


1. [Can't Win, And So I Will Not Try]({% post_url 2020-02-04-code-reviews-pt1 %})
2. [Everything is Awful (When You're Part of a Team)]({% post_url 2020-02-11-code-reviews-pt2 %})
3. [The Captain Jack Sparrow](#the-captain-jack-sparrow)

## The Captain Jack Sparrow

{% include youtube-embed.html id="5U4KQ0mqbew" %}  

So how does it manifest?

1. Comments so packed with qualifiers their point is unintelligible:
> "I think it might be the case that the user could potentially encounter an error here, unless you caught it somewhere else that I missed, but I don't think there is anywhere else?"
2. Failure to clearly state how to secure reviewer's Approval.
3. Feedback that simply re-states known facts, refusing to extrapolate to ramifications, severity, etc.
4. Similar to #3 - feedback that never _explicitly states the problem_.


### The Cost of Self-Confidence

Lack of professional confidence is a frequent culprit in unhelpful reviews.  We may worry that we're unqualified to review certain code, that the process will expose our own ignorance, or feel anxiety about whehter we have "the right" to block someone else's code from merging.

> Keep in mind:  we are _peer reviewers_.  No on expects us to swing in from the ceiling and save the day - our role is to provide another pair of eyes and perspective. 

While self-doubt is understandable and common, we have a professional responsibility to eliminate this behavior when we see it in ourselves. Here are some practical techniques for doing so:

1. __Notice - and remove - inaccurate qualifiers in your writing__.  Qualifiers should help the author understand _your_ grasp of the situation, and testing that you've performed.  _They should not be generically tacked onto every remark just in case you missed a gotcha[^1]_. 
    
2. __Notice - and remove -  self-effacing/minimizing language__. This diminishes the value of your professional experience - and leaves authors playing stressful guessing games. 
    > ~~"Maybe it's just me, but I think this ternary operator is kind of hard to read? LOL"~~

    (It's not just you.  It's never just you.)

    > I think this ternary operator is hard to read.

3. __DO use qualifiers to convey what you know and what you've tested__.  
    > ~~I think we might be showing that error message before it's defined.~~

    > I didn't check to see what flag enables this template, but unless it's default-hidden, we are showing that error message before it's defined.


4. __Justify your remarks.__ This not only helps the author, it _helps you_ build confidence in your professional opinions - because you just backed them up!  Inversely, opinions that you cannot justify should be re-examined. 

    > I think this ternary operator is hard to read, because the truth value includes two evals.  Could we make it more parallel by extracting that into a separate variable?

5. __Be willing to take a stance.__ This dovetails with #4 - stances become more comfortable when you're confident that you're on solid ground. But from your peers' perspective, they also make you a much more powerful and useful reviewer.  Who wants a code review from someone who will do nothing more than state the facts? 

6. __Use a consistent set of semantic tags to indicate feedback prioritization.__  Here are mine:


```json
{'must-fix', 'suggestion', 'polish', 'comment', 'question'}
```

Semantic tags won't solve _every_ problem - after all, just because you've tagged something as ```must-fix``` doesn't mean the author will actually do so - but they're a powerful way to communicate clearly. 


### Connect (ALL) the Dots

Another common cause of unhelpful reviews is _insufficient information_ - scenarios where we never actually identify the problem being discussed. How can we talk about a problem without stating it?  Very easily, it turns out, because we tend to give feedback that would help _us_. I recently encountered a perfect example of this:

> You checked compiled code into version control.

Many developers would immediately understand this comment - __but it's exactly the same group who would never receive that comment in the first place__. Assuming good intent, authors do bad things not because they're malicious, _but because they don't know it's bad_. Simply re-stating what they did gives them no new information. Be explicit[^2]:

1. Identify the offending code (attach your comment to a file/line-number if possible)
2. Explain what's wrong - and why:
> "The file you added is a compiled file; it's generally bad practice to version compiled code, because we have no ability to modify it in the future.  If we instead version the _source code_ used to generate the binary, we can produce the binary whenever we want, but can also modify the source and regenerate a new, different binary as well. "
3. Explain what should be done instead:
> "Please delete the compiled file from your PR, and add the file that was used to generate it, instead."
4. [Leave room for them to still need help](https://xkcd.com/1053/){:target="_blank"}:
> "Let me know if you have any additional questions!"

#### A Positive Example

By comparison, here's a piece of feedback I recently received:

> "This should be using `command X` instead of `command Y.` `command Y` doesn't mean 'wait for this event,' - it just returns truthy/falsy based on whether the event has occurred at this time."

I was quickly able to understand the problem, fix it, and now have a much better heuristic for when to use commands X & Y - which will ultimately reduce that reviewer's future burden!


### The Uncomfortable Bits

A final noteworthy cause of unhelpful reviews is interpersonal concerns: reviewers may fear creating bad blood with a coworker, or simply hurting someone's feelings. 

You're unlikely to start a grudge with a single code review, but if you have continued clashes with someone, it's worth asking why, whether the conflict is warranted, and whether you can simply recuse yourself from reviewing their code.

Fear of being "mean" is usually of greater concern.  It's easy to conclude that if negative feedback makes _us_ feel bad, others will feel the same. A challenge for the reader: is it the "pointing out" that makes you feel sad? Or the fact that you wrote bad code in the first place?  Reframing our discomfort as ultimately self-inflicted (and self-centered) can help us build a more accurately empathetic sense of what most of us want as authors - clear feedback, without beating around the bush!

That said: honest feedback is not intrinsically mean, it certainly _can_ be.  But it also can (and should) be kind.  

> And God damn it, you've got to be kind.

----

#### Footnotes
[^1]: It is ok to miss a 'gotcha!' However, this is a great reminder to _never be a jerk when you don't have to_ because missing a gotcha _while_ being a jerk is a great way to look stupid.
[^2]: Many of us are also concerned about not appearing to insult someone's intelligence.  Unless I'm certain the author doesn't already know what I'm about to explain, I preface my feedback accordingly: "You may already know this, but..." or "Apologies if I'm explaining something you already know...". 
