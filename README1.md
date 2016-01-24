# NQ.js

JavaScript library for the `NQ` (read: eNQueue) framework.


## How to use

See the (parent repo)[https://github.com/hilbix/nq.git] for full description.

In your web application page include
```
<script src="PATH-TO-JS/nq.js"></script>
```

This pollutes the main with the global object `window.NQ` ready to use.

- `NQ.setup(PATH, BACKEND, SETTINGS, CALLBACK, ERROR)` same as `new NQ(PATH, BACKEND, SETTINGS_OBJECT)`
- 
NQ.call(REQUEST, OBJECT_TO_SEND, SETTINGS, SUCCESS_CB, ERROR_CB)  <->  NQ(REQUEST, OBJECT_TO_SEND, SETTINGS, SUCCESS_CB, ERROR_CB)
NQ(..).bind(CALLBACK, ERROR, SETTINGS)                            <->  NQ(..)(CALLBACK, ERROR, SETTINGS)
- NQ.setup(PATH, BACKEND, SETTINGS, CB, ERROR)
The prototypes are:

- reconfigure with: `NQ = new NQ('URL-path');`
- reconfigure with: `NQ = new NQ({url:'URL-path',fail:function(fail){}});`
- prepare request:  `prep = NQ('req', {ob:['to','send']});`, this is a "promise", compatible to (RSVP.js)[https://github.com/tildeio/rsvp.js/]
- do the request:   `prep(function(success){}, function(fail){})`

Special configs:

- `url`: The URL path component from `new NQ(url)`
- `fail`: a global `fail` routine
- `success`: a global `success` routine
- `promise`: a wrapper object for function call returns:.  With `NQ = new NQ({promise:RSVP})` each call to `NQ(x)` is like calling `new RSVP(NQ(x))` before.  Note that you cannot chain wrappers.


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

The NQ API can be used with any middleware.  However it is designed to be used with ZeroMQ with the help of some small wrappers implemented in your favorite dynamic web language.

Currently those wrappers (called "web-backend" here) are only available for PHP.  More language might be added in future.

> Help welcome, as I wanto to concentrate on other things than writing a Node wrapper etc.  But please abide to the license.

## License

This Works is placed under the terms of the Copyright Less License,
see file COPYRIGHT.CLL.  USE AT OWN RISK, ABSOLUTELY NO WARRANTY.

Read:  This is free as in free speech, free beer and free born baby.  Copyright is slavery.

