---
layout: post
title: Real World RxJS (for Beginners)
date: 2020-03-26 06:00:00 -0700
---

RxJS can be overwhelming.  Early in my learning process, I was disappointed that much of the available material:

* is designed for advanced tinkering (rendering it borderline unusable for a novice)
* uses examples so esoteric that a lot of context-mapping opportunities are squandered - similar to the [foo/bar complaint](https://twitter.com/dan_abramov/status/1241285551136935936){:target="_blank"}.

Here are the kind of examples I wish I had when I was getting started:

**switchMap**: You wake up to find your pet dog, Muddles, sitting at the foot of the bed watching you intently.  Now that he knows you're awake, Muddles _switches_ focus and runs to the kitchen to watch his food bowl. [more on switchMap](#more-on-switchmap)

**flatMap/mergeMap**: You and your little cousin both need help getting ready for school. First, your cousin asks your dad to pack her lunch.  Then you ask for breakfast so he puts some bread in the toaster for you. While he's waiting for it to pop, your cousin asks him to comb her hair. Then the toast comes up and your dad gives it to you. Finally the bus arrives, and you and your cousin leave. Your dad collapses _flat_ on the floor in exhaustion. 
[more on flatMap/mergeMap](#more-on-flatmapmergemap)

**concatMap**: You ask your dad for help getting ready for school.  Halfway through tying your shoes, your little cousin comes running up asking for help too! Your dad tells her to _"concat-a-wait your turn."_ Your cousin rooooooolls her eyes, but ok.  Your dad finishes tying your shoes. Then - knowing that your cousin is still waiting - he helps tie her shoes, too.
[more on concatMap](#more-on-concatmap)


**exhaustMap**: Your baby brother is teething, and will NOT stop crying.  Your mom (who is _exhausted_) does not hear anything that you, your little cousin, or your dad ask her to do until she gets your brother to sleep. [more on exhaustMap](#more-on-exhaustmap)

----

## More on switchMap

### When To Use: 
* you need to wait for some event to occur / information to become available, and then take action as a result.
* the event you're waiting for is singular rather than repetitive, e.g. bootstrapping an application

### Other Real-World Examples: 

* __Watching a Relay Race__: 
You watch the current baton-holder until they hand off the baton, at which point you _switch_ focus to the next runner. You were subscribed to Tianna Bartoletta;  now you're subscribed to Allyson Felix.[^1]

* __Roadtrips__:
You keep asking your parents 

    > _"are we there yet?"

    ...until they finally say..
    
    > _"Thank God, **yes**."_  

    Then you _switch_ focus, and start watching Old Faithful. 

### Code:

```javascript
const Rx = require('rxjs/Rx');
const RxOp = require('rxjs/operators');

const winnerOfPrimary = Rx.Observable.of("Biden");
const candidateInGeneral = Rx.Observable.of([
    { 'name': 'Bernie', 'age': 78, 'gender': 'male'},
    { 'name': 'Biden', 'age': 77, 'gender': 'male'},
    { 'name': 'Elizabeth', 'age': 70, 'gender': 'female'}
])

winnerOfPrimary.pipe(
        RxOp.switchMap(name => candidateInGeneral.pipe(
            RxOp.map(usersInfo => usersInfo.find(user => user.name === name)),
        )),
    ).subscribe((candidateInfo) => {
        console.log(`how old is the candidate in the general election?  ${candidateInfo.age}`);
    });

/**
 * output: 
 *  how old is the candidate in the general election? 77
 */

```


## More on flatMap/mergeMap

### When To Use: 
* when maintaining order within the stream is not important, i.e. Event #2 can leapfrog Event #1 without negative consequences
* when all events in the stream matter and should be processed

### Other Real-World Examples:

* __Orders in a Restaurant Kitchen__:
If you've ever ordered lasagna and a baked potato at a restaurant, you've seen flatMap in action: your order might go _into_ the kitchen first, but the table seated after you will still get their soup & salad before you get your meal.
Importantly, their orders don't have to queue up behind yours - they can be prepared and served with no impact on your own eventual dinner (aside from perhaps wishing you'd ordered something else).


### Code

> This example deliberately demonstrates that initial queue order is not maintained - the salad, soup and burger will all overtake the lasagna in the queue, because they all have shorter prepTime values (and we have constructed an example where prepTime influences observable completion time in realistic way).

``` javascript
const Rx = require('rxjs/Rx');
const RxOp = require('rxjs/operators');

const prepTimeByDish = Rx.Observable.of(
    { dish: 'lasagna', prepTime: 100 },
    { dish: 'a salad', prepTime: 20 },
    { dish: 'soup', prepTime: 5 },
    { dish: 'a burger', prepTime: 20 }
)

prepTimeByDish.pipe(
    RxOp.tap((item) => console.log(`${item.dish} has been ordered.`)),
    RxOp.flatMap((item) => Rx.Observable.of(item).pipe( RxOp.delay(100*item.prepTime) )),
    ).subscribe((kitchenOutput) => {
        console.log(`${kitchenOutput.dish} is ready! (took ${kitchenOutput.prepTime}ms to prepare)`);
    });

/**
 * output: 
 *  lasagna has been ordered.
 *  a salad has been ordered.
 *  soup has been ordered.
 *  a burger has been ordered.
 *  soup is ready! (took 5ms to prepare)
 *  a salad is ready! (took 20ms to prepare)
 *  a burger is ready! (took 20ms to prepare)
 *  lasagna is ready! (took 100ms to prepare)
 */

```


## More on concatMap

### When To Use: 
* when maintaining order within the stream _is_ important (i.e. any time when it would be bad to "jump the queue")
* when all events in the stream matter and should be processed

### Other Real-World Examples:

* __Documents in a Print Queue__:
The printer is cold, unthinking technology and it does not care that you have to conduct an interview in 10 minutes and haven't printed the candidate resume yet - it isn't going to start printing your document until those ahead of it in the queue are printed.  

* __Waitstaff Interaction with Customers__:
A waiter's behavior is usually an concatMap - he doesn't leave you in the middle of taking your order just because a new customer has walked in!  But he does then attend to that new customer (by contrast to an exhaustMap, where the waiter would effectively be blind to the existence of any customers who arrived while he was interacting with you).

### Code

> __This__ example demonstrates the contrast between flatMap and concatMap: even though the first document is much longer (and therefore will take longer to print), the shorter documents are not printed first. 


```javascript
const Rx = require('rxjs/Rx');
const RxOp = require('rxjs/operators');

const documentsInQueue = Rx.Observable.of(
    { name: 'my-spreadsheet.xls', numPages: 100 },
    { name: 'Untitled Document', numPages: 20 },
    { name: 'Resume - Jane Gray', numPages: 5 },
)

documentsInQueue.pipe(
        RxOp.concatMap(document => Rx.Observable.of(document).pipe( RxOp.delay(10*document.numPages) )),
    ).subscribe((documentPrinted) => {
        console.log(`printed: ${documentPrinted.name}`);
    });

/**
 * output: 
 *  printed: my-spreadsheet.xls
 *  printed: Untitled Document
 *  printed: Resume - Jane Gray
 */

```

## More on exhaustMap

### When To Use:
* when maintaining order in the stream is important
* when not all events matter/need to be processed
* when you want an exclusive lock on something

### Other Real-World Examples:

* __Clever Websites__:
Most of us have encountered form submissions on a website that warn us not to click the "Submit" button twice - for fear of double entry.  Broadly speaking, an exhaustMap is a great way to programmatically avoid this problem: if the user's credit card charge has not completed, the form should ignore any additional submit-clicks (and unlike concatMap, should explicitly _not_ remember them for processing after the submission _does_ complete).


### Code:

#### Example One
> A fundamental exhaustMap demonstration: because the initial emit takes 300ms to complete, and the other emissions are arriving behind it _with no delay_, they are all ignored/dropped on the floor.

```javascript
const Rx = require('rxjs/Rx');
const RxOp = require('rxjs/operators');

const queueOne = Rx.Observable.of(3,1,1,5)

queueOne.pipe(
    RxOp.exhaustMap( item => Rx.Observable.of(item).pipe( RxOp.delay(1000*item) )),
    ).subscribe((emission) => {
        console.log('exhaust map - emitting: ', emission);
    });

/**
 * output: 
 *  exhaust map - emitting: 3
 */
```

#### Example Two
> Here, we create temporal space between emits arriving at the exhaustMap, for the purposes of more concretely demonstrating what the blocking action looks like.

> In the previous example, all emits were arriving at the exhaustMap at (basically) the same time (to see this, place a `tap`/`console.time` before the exhaustMap on the above example)

> In __this example__, we use concatMap to forcibly space emits, which helps us see that the initial emission blocks for _600ms_ - and any emissions which occur _after_ that window are not blocked.

```javascript
/**
 * 3 arrives to exhaustMap @ 300ms and completes at 600ms.
 * 1 arrives to exhaustMap @ 400ms but is lost (because 3 is still in progress).
 * 1 arrives to exhaustMap @ 500ms but is lost (because 3 is still in progress).
 * 5 arrives to exhaustMap @ 1000ms and completes at 1500ms.
 * 
 */
const Rx = require('rxjs/Rx');
const RxOp = require('rxjs/operators');

const queueOne = Rx.Observable.of(3,1,1,5)

queueOne.pipe(
    RxOp.concatMap( item => Rx.Observable.of(item).pipe( RxOp.delay(1000*item) )),
    RxOp.exhaustMap( item => Rx.Observable.of(item).pipe( RxOp.delay(1000*item) )),
    ).subscribe((emission) => {
        console.log('exhaust map - emitting: ', emission);
    });

/**
 * output: 
 *  exhaust map - emitting: 3
 *  exhaust map - emitting: 5
 */

```

### Additional Resources:

[This video](https://www.youtube.com/watch?v=rUZ9CjcaCEw){:target="_blank}" is a well-known classic.

---

#### Footnotes
[^1]: Conceptualizing this as a subscription to individual runners - rather than the race as a whole - is important.  If you were subscribing to the _race_, each runner's actions would simply be another event emission, and you would never need to switch.
