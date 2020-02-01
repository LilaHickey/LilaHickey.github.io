---
layout: post
title: code reviews - do this, not that
date: 2020-02-04 06:00:00 -0700
---


> Note: This is the first of a three-part series on how to give better code reviews. 


Code reviews are hard.  You have limited time to match the author's knowledge of a topic. You may be the last defense against A Bug in Production.  And sadly, many organizations count dunks but not alley-oops, which means you're incentivized to hurry back to your own code.

So it's no surprise how often we see one of these code review anti-patterns:

1. [Can't Win, Won't Try](#cant-win-wont-try)
2. Everything is Awful (When You're Part of a Team)
3. The Captain Jack Sparrow

This post is focused on spotting these patterns in ourselves, and doing something better, instead!

## Can't Win, Won't Try

"Can't Win" typically occurs when reviewers are overwhelmed by other commitments or by the size/content of a pull request.  They react by ignoring the request entirely or granting a rubberstamp approval.  

This style is particularly likely to let bugs into the trunk, but also causes frustration for individual contributors.  Those looking to polish their skills will feel deprived of precious peer feedback, and those who feel socially disconnected from the larger group (e.g. new hires, remote workers) will feel they have no mechanism by which to get their code merged, if a reviewer is not timely. 

So what should we do instead?

#### Start With The Right Questions:

1. will your existing workload allow you to review the code in a timely fashion?
2. does your expertise make you a reasonable reviewer?

    If the answer to #1 or #2 is "no," it's time to tell the author you won't be reviewing their code.  But you can still help, by suggesting an alternate reviewer.  In addition to aiding the author, you're facilitating visibility/opportunity for less-senior or under-utilized reviewers, which will ultimately reduce your future review-workload!

    > Clear communication is a mark of professionalism; an inability to say "no" is not. 

3. is the PR a reasonable size[^1]?  
4. do you understand the goal of the PR?

    Again, if the answer to either question is "no," your powering through a review isn't good for anyone.  Contact the author with your concerns - can the PR be broken into multiple, smaller PRs? Can additional code comments, PR annotations, or descriptive information be added, to help guide reviewers?

Ok, so: you're the right person, you have the time, and the PR is as good as it can be.  But it's still _a beast_.  Now what?

#### Have a Deliberate Review Strategy:

1. begin by asking yourself if the change invalidates any existing assumptions.  this will inform #2
2. _before diving into the code diff_, imagine how __you__ would implement the change.  think specifically about:
    1. what code you would need to touch
    2. which changes would be core to the effort
    3. which changes would be ancillary (e.g. changing import locations, updating function calls)
3. now validate your mental model against the actual diff, investigating and reconciling significant discrepancies[^2]
4. assuming the diff is reasonable, it's time to get detailed:
    1. start with the core changes (while you're mentally fresh). errors here are likely to produce the most dangerous bugs. 
    2. now review secondary changes. you'll likely get a speed boost here for already being familiar with the core changes (e.g. it will be easier to validate a series of refactored function calls if you've already familiarized with how the function signature itself was refactored)
5. don't forget testing: does this change warrant removing, updating or adding tests? check that those changes are included! 
6. if you still have concerns, a real-time meeting with the author and other reviewers may be warranted.  this is _sometimes_ an inefficient use of time, but has the advantage of higher bandwidth communication.  (bonus: people may raise concerns that they didn't think warranted a comment in the code review system.)

Hopefully some or all of these steps will come in handy the next time an overwhelming PR crosses your desk. 


#### Footnotes
[^1]: Keep in mind that how to bound a pull request is always a judgement call, and some diffs look deceptively large.  It might take a few minutes to assess/answer that question.
[^2]:  Ideally this will come in the form of improving your mental model but occasionally it means you've caught a significant error.  Congrats - you did exactly your job!