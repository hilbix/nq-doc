*This is in an early pre-alpha state*

# NQ.js

NQ (read: "eNQueue") is a higly scaleable communication framework for web based applications to provide all the dynamic content, push notifications and upload/download you need.

NQ is not meant to provide static content nor script nor module loading.  Usually you do not want that to be included into a protocol like NQ for design efficiency.

NQ is meant to provide a solution to the [C100k problem](http://en.wikipedia.org/wiki/C10k_problem) and above on an Arduino.  Just kidding.  About Arduino, not C100k.

> Help is welcome:
> - NQ currently only supports browsers.  This is mainly because I cannot test how to properly create a Node/AMD/require/whatever binding.
> - NQ currently has no CI/Travis/whatever autobuild/test/whatever thingie, simply because I never took the time to grok how to setup and include those.
> - There are trainloads of missing documentation, missing examples and missing wrappers today.
> - If you want to help:  Beware!  First look at the license please!  If you cannot abide by the license, you cannot help me!

## Demonstration

T.B.D.

## How to use

- On your development tree:
```
git submodule add https://github.com/hilbix/nq.git PATH-TO-MODULES/nq
cp PATH-TO-MODULES/nq/nq.js PATH-TO-WEB/PATH-TO-JS/
```
Pick a web backend (like `php`) to use and include that example in your service:
```
cp PATH-TO-MODULES/nq/BACKEND/*.php PATH-TO-WEB/SUBPATH/
```
In your web application page include
```
<script src="PATH-TO-JS/nq.js"></script>
```

This pollutes the main with the global object `window.NQ` ready to use.

### Short teaser

The NQ communication uses JSON to encapsulate/decapsulate data.

In your applications script do:

```
NQ = new NQ('SUBPATH/');

NQ('request', {arg1:true})(function(success){ /* as usual */ }, function(fail){ /* as usual */));
```
As this is a bit of uncommon syntax, for convenience, you can write this as:

```
NQ = new NQ('SUBPATH/');

NQ('request', {args}).bind(function(data){ /* as usual */ });
```
NQ also can be used in the context of a promise via something like (RSVP.js)[https://github.com/tildeio/rsvp.js/]:

```
new RSVP(NQ('req', {arg1:true})).then(function(data) { /* the usual */ });
```
And by passing the object providing Promises to the initialization, you can make NQ directly return Promises, too:
```
NQ = new NQ('SUBPATH/', {promise:RSVP});
NQ('request').then(..);
```

If you looked carefully at the above, you will recognize, that we have replaces the `window.NQ` object with the `new NQ(..)`.  This is right and you do not loose anything.  All you do this way is to replace the original NQ object with a differently configured one.

An alternative to use `new` is to use `setup` on `window.NQ` to reconfigure it.  The general syntax is:

```
NQ.setup(PATH, BACKEND, SETTINGS, CALLBACK, ERROR)                <->  new NQ(PATH, BACKEND, SETTING_SOBJECT)
NQ.call(REQUEST, OBJECT_TO_SEND, SETTINGS, SUCCESS_CB, ERROR_CB)  <->  NQ(REQUEST, OBJECT_TO_SEND, SETTINGS, SUCCESS_CB, ERROR_CB)
NQ(..).bind(CALLBACK, ERROR, SETTINGS)                            <->  NQ(..)(CALLBACK, ERROR, SETTINGS)
```
The reverse of `NQ.bind` is `NQ.unbind`.  And you can use `.bind` and `.unbind` on the global object to register/unregister default callbacks.

> Passing `this` context:
> 
> A major task is passing the `this` context when you call your callbacks.  There are trainloads of variants possible.  But the probably most easy way is to pass it as the first argument to the functions.
>
> If NQ detects an object as the first argument instead of some non-object like a string/number or function, it assumes, it is the `this` context, and adjusts accordingly.  Easy and simple as that.  I wish other frameworks would do it the same.  This also is naturally, you just type `this,` instead of the more common `this.`.
>
> An important note is that this does not work well with classes, which happen to be functions.  Just abstain from that concept in your application.  Else you must do this the "old" way, or pass the `this` as `{context:this}` with the settings object, which I consider somewhat clumsy way to do.

### The deal

This all does not seem to be a big deal yet.  But the interesting part comes, when you open many requests in parallel.  NQ multiplexes those requests, per default, over max 2 parallel connections.  You can reduce this number to 1 or increase it as you like.  So the catch is, that, regardless how many things your app does in parallel, your web service will not be overwhelmed by all those parallel requests.

There is another catch, which happens behind the scenes.  All requests are short-lived.  There is, at maximum, a single "send-waiting_for_processing-response` cycles seen on your web service per client (this is the "poll for update" if there are outstanding requests).  The request itself is delivered to your web services and there always is an immediate response that the request was received.  The backend now can take all the time it needs to process things.  When the result is ready, the result is handed back to the client.  The client picks it up at the next round, usually within the fraction of a second.

> In future, support for partial requests and partial results will be implemented.  This, however, needs a little tweak to the API.  For ease of use, the above examples always only send full requests and get full responses.

Everything is queued in a handy fashon within your browser and can be processed serially/parallely by your backend as it seems fit.  The bookkeeping comes handy, as you can easily present those numbers to your customer (there are 8 requests outstanding, now 5, now all processed).

> Note that the request tracking is browser-side.  It is up to your backend how starved responses are handled.  Usually they just sit there and wait until the client comes back.  Timeout with Garbage Collection is your friend.  Taking up starved responses is WiP (Work in Progress) currently, we will see how this evolves in the future as the need arises.
>
> Would be a nice if your app is able to say "well, last time you were logged in from another browser with some things outstanding, here are the results of your operation".  But that's not done today, as the example web backend is entirely session based.  If you tear down (timeout) the session, you tear down the respnses as well.

And as last important step, the roles of request/response can be reversed.  So your backend can request things from your application.  This usually is referred to as **Push notifications**.  This can be used for things like telling your users things like "We have a maintainance window in 30 minutes, please save your work." or to offload calculations to your web clients, to (ab)use them as some cloud service to pay for using your service (which, correctly implemented in a fair way, is far better than placing annoying ads!).

### ZeroMQ

The NQ API can be used with and without middleware.  However it is designed to be used with ZeroMQ with the help of some small wrappers implemented in your favorite dynamic web language.

Currently those wrappers (called "web-backend" here) are only available for PHP.  More language might be added in future.

> Help welcome, as I wanto to concentrate on other things than writing a Node wrapper etc.  But please abide to the license.

## License

This Works is placed under the terms of the Copyright Less License,
see file COPYRIGHT.CLL.  USE AT OWN RISK, ABSOLUTELY NO WARRANTY.

Read:  This is free as in free speech, free beer and free born baby.  Copyright is slavery.

