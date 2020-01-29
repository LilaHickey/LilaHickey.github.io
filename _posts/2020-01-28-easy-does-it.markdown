---
layout: post
title: easy (to read) does it
date: 2020-01-28 06:00:00 -0700
---

_Oh, just optimize it! What can it hurt?_

Our Eyes.

We rarely talk explicit about the trade-off between performance optimization, readability, and simplicity. Just as we ask if code is optimized, we should ask if it is simple and easy to read:

1.  how many distinct operations must the reader mentally perform to understand the code?
2.  does its execution path match the way we would describe the workflow to a human?
3.  does the structure of the code guide the reader towards _what is important_?
4.  how can this code fail, and what will happen if it does?
5.  what order of magnitude performance benefit are we achieving?

I recently participated in a bugfix that nicely illustrated the tension between (perceived) performance and readability/simplicity.

## The Bug

The bug itself was garden variety: an API was reporting successful operations but returning a malformed response. Specifically, we were missing a ‘filepath’ attribute which would be used to set a download link in the UI.

A successful response from the API looks thusly:

``` json
{   
    response: "Successful",
    file: "requested-log.json",
    filepath: "http://currentNodeIp/logfile-folder/requested-log.json"
}
```

But the malformed responses consistently lacked the ‘filepath’ attribute:

``` json
{ 
    response: "Successful",
    file: "requested-log.json"
}
```
Now, unlike Private Hudson, I love a good bughunt, and believe that exploring unfamiliar parts of the stack will make you better in your own. But our timeline - and mandate - was clear: be efficient, stick to our expertise, and _ensure the UI responded gracefully in all scenarios._

## The Fix

Our minimal goal was notify the user when the download link was undefined; after inspecting the response structure and chatting with the API developers, I determined we could _derive_ the filepath, by concatenating:

*   the (known) IP address of the host node
*   the (fixed) relative path to the file
*   the name of the file, which _was_ reliably available in the API response

## The Confusion

Pleased with such a simple solution, I created a branch demonstrating the fix and sent it to the primary developer.

I was perplexed to receive a message thanking me for the branch, and noting that he would add a conditional to handle the malformed response scenario.

> “But…there’s no need for a conditional. This code will generate the download link correctly for all API responses,” I explained.
> 
> ” Yes, but shouldn’t we prefer the ‘filepath’ attribute?” he asked.
> 
> I was again, perplexed. Whether it was concatenated or pulled directly from the response, the resulting download link would be the same.
> 
> “Of course,” he agreed. “But if the path already exists, we can avoid building it!”

Now I understood our miscommunication: he was prioritizing performance, and I (having determined there was no relevant performance difference[^1] ) was prioritizing simplicity of code.

## Readability & Simplicity

My prototype looked like this:

```javascript
request()
.success((res) => {
    downloadLink = `${currentNodeIp}/logfile-folder/${res.file}`;   
});
```


The primary developer’s would look like this:

```javascript
request()
.success((res) => {
    if(!!res.filepath) {
        downloadLink = res.filepath;
    } else {
        downloadLink = `${currentNodeIp}/logfile-folder/${res.file}`; 
    }  
});
```

So why is option #1 better?

### Readability

*   Understanding option #1 requires that the reader:
    1.  parse a string interpolation and variable assignment.
*   Understanding option #2 requires that the reader:
    1.  evaluate the conditional to understand _how_ execution will fork.
    2.  parse the code inside each conditional to understand _what_ will be executed.
    3.  remember which code will be executed in which condition.
*   In addition, the _very existence of an execution fork_ in option #2 creates the impression that we are doing two explicitly different things (either from a programmatic perspective, from a user experience perspective, or both). In reality, the user experience and the end state of the UI will be will be identical. This is arguably a violation of [the Principle of Least Astonishment](https://en.wikipedia.org/wiki/Principle_of_least_astonishment).

### Simplicity

Just by existing, the conditional in option #2 adds complexity, which in turn increases exposure to errors and performance problems.

*   What if the conditional logic is wrong, and references the ‘filepath’ attribute when it isn’t available? Now we have failed at our fundamental task: to protect the user from a broken download link.
*   Worse: _really_ bad logic could cause an explicit javascript error (e.g. referencing an undefined nested slice in the response object). In the absence of robust error-handling, we may render _the entire page_ unusable.
*   A poorly-written conditional (e.g. using ‘hasOwnProperty’) could lead to [overall slower execution](https://jsperf.com/easy-to-read-does-it), chewing through any benefit of the optimization - and then some.

## In Conclusion...

Readable code is maintainable code; simple code is robust code. Performance-optimized code may be neither, and that's OK.  But software development is a balancing act, and as with any balancing act, we'll do our best work with our eyes wide open. 

---

#### Footnotes

[^1]: The internet is already full-to-bursting with information about (premature) optimization, but the crux of why it was not useful here lies in the issue of proportion: the primary task (of generating the logfile) might take several minutes; setting the download link at the UI level is an operation that takes microseconds, and is simply not worth optimizing, by comparison.

    It is also sometimes useful to consider performance optimization from a horizontally-sliced perspective (e.g. how many times is *this UI instance* going to perform this operation, potentially concurrently?). However in this scenario there is a 1:1 relationship between UIs & APIs, and the API generates an exclusive lock while performing the operation, i.e. the UI can never have more than a single such operation in flight, and repeated calls to the operation are (effectively) rate-limited by the server-side time required to perform it.


    