---
layout: post
title: code reviews - do this, not that (pt2)
date: 2020-02-04 06:00:00 -0700
---

> Note: This is the second of a three-part series on how to give better code reviews.


1. [Can't Win, And So I Will Not Try]({% post_url 2020-02-04-code-reviews-pt1 %})
2. [Everything is Awful (When You're Part of a Team)](#everything-is-awful-when-youre-part-of-a-team)
3. The Captain Jack Sparrow

## Everything is Awful (When You're Part of a Team)

This style manifests as a high volume of primarily or exclusively negative feedback, without context about relative severity and recommended prioritization.  It has several causes:

1. explicit distrust of the author[^1]
2. the tendency to equate finding bugs with general skill, leading to:
    a. bughunting to demonstrate dominance over other deveopers
    b. bughunting as a defensive mechanism against criticisms of being unskilled
3. fear of _not_ catching a bug

**Junior engineers** are particularly vulnerable to "Everything is Awful." They may struggle to prioritize feedback, and are likely to interpret the sheer _volume_ as a reflection of their skill (rather than a dysfunctional environment).

So what can we - as organizations and individuals - do instead?

### Organizational Strategies:

These strategies are about providing clear expectations.  Since the specificty of a rule and the scope in which it can be applied tend to be inversely proportional, clear expectations are easiest to provide for "lower level" concerns - linting[^3]/code formatting, unit tests, happy path integration tests, etc.


1. Implement linters and autoformatters.  
2. Make automated testing highly visible, easy-to-use and easy to extend.
3. Tests from #1 and #2 should be automatically executed by your build pipeline; passing runs should be a prerequisite to all PRs.
4. Create contributing guidelines, with particular attention to non-functional requirements and other standards which cannot be quickly inferred from tests. 

> In a nutshell: a first-time author should encounter as few __unpleasant surprises__ as possible when they begin receiving PR feedback.

### Strategies for Individuals:


#### High-Level: 
1. Ask yourself if your actions are good for the code.
1. Make a good-faith effort to know the context under which the author is operating.  If Product is asking for status updates on the hour, they are _probably_ not doing their best work.  Lambasting them for betraying their Fellow Dev for the Product Team might feel good now, but it won't refactor the code that made you feel betrayed in the first place.
2. Assume good *intent*. Except in extreme cases, distrusting an author reduces our ability to perform a good review.  If your answer to the question:
    > "ugh! why did they do it this way?" - is -  "well, obviously they live to make me suffer,"

    ...you risk overlooking additional context which would justify their implementation. Now _you_ seem untrustworthy, for letting feelings impare your professional skills.

3. Do __not__ assume good implementation. Many of us fall into the trap of spending too much time trying to make sense of bad code; the lost time/irritation will make you more prone to slip into "Everything Is Awful" patterns.  Ask clarifying questions sooner rather than later. ---
4. Know Your Audience: A senior engineer can tolerate - and probably enjoys - 'polish' feedback.  A junior engineer may not have the ba                                                         ndwith for that - especially if you've already identified multiple 'must fix' feedback items.  As Charity Majors [fantastically put it](https://charity.wtf/2017/05/11/the-engineer-manager-pendulum/)'...[more senior engineers] need to be able to size up a junior engineer’s ability and craft a meaningful assignment, one that pushes their boundaries without crushing them...'
5. For example: if the pull request introduces tech debt, but is being done deliberately to meet some particular timeline, tie your Approval to the author being able to show you the tech debt tracking ticket they've made to address this in future code. 

#### Tactical:

1. Any feedback item you provide should have:
    1. a indication of seriousness, to help the author prioritize rework - I like an enum, for consistency: 
    ```json
        {'must-fix', 'suggestion', 'polish', 'comment', 'question'})
    ```
    2. an explanation of likely consequences of not heeding feedback (e.g. users will see an uncaught error, test suites will run 10% slower)  
        * NOTE: feedback that isn't intrinsically empirical, e.g. "this code is harder to read" also requires clear justification.
    3. suggested alternatives/solutions if you have them - or an acknowledgement of that, if you don't
2. If possible, perform an initial review with other reviewers' feedback hidden, to avoid perspective poisoning.
3. keep context in mind - are you asking the reviewer to fix something that isn't in the scope of their change?

    1. A specific tactic I sometimes use for this: draft, but don't save/send comments as I move through a review.  When I'm finished I assess the total number of comments and use that to decide the minimum level of seriousness to actually save/send to the author.

### Related Topics

Although it's not intrinsic to this anti-pattern, "bikeshedding" is often involved; those not familiar with it by name will quickly recognize it [by description](https://en.wikipedia.org/wiki/Law_of_triviality).

---

#### Footnotes

[^1]: This likely warrants managerial intervention, but that isn't your job, and a code review is, again, not the venue. As ICs, our responsibility here is to communicate the concern to a manager, and then _try to minimize its impact on how we do our own job_[^2].  


[^2]: This is usually easier said than done.

[^3]: This is not to suggest that code style decisions are trivial and can always be automated (although they often are, and can), but rather that discussing them on a code review is almost always a hijacking of the PR's focus and goals.
