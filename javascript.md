# NQ.js interface


## `NQ.setup`

- `NQ.setup([context], url, settings, success, error)`
- `new NQ([context], url, settings, success, error)`
- supports method chaining

Both are similar.  `setup` reconfigures the NQ , while `new` creates a new object.

List of settings.  There might be more used by extensions:

- `context`: Optional context for the callback functions.  See `settings.context`.  Must be an object!
- `url`: The URL or `function(nq)`.  See `settings.url`.  Normally a relative path to the middleware.
- `settings`: An object or `function(nq)` returning the settings object.  (`this` always is the current context.)
- `success`: A global (fallback) `success` routine
- `fail`: A global (fallback) `fail` routine
- `promise`: A wrapper object for `call`.  See `settings.promise`

### `settings.context`

The `this` object for calls to (all!) callback or evaluation function invoked by NQ.  If not set, then `this` will be set to NULL.

### `settings.url`

See also `NQ.url`.  Normally just a path relatively to the current URL to the middleware.

If `function(nq)`, then this function evaluates the URL.  `nq` is the `NQ` object.

### `settings.success`

`function(success,req,nq)`

- `success` is the passed object.
- `req` is the current `NQ.Req` type object.
- `nq` is the current `NQ` object.

It must return XXX TODO XXX

### `settings.fail`

`function(fail,req,nq)`

- `fail` is the failure object.
- `req` is the current `NQ.Req` type object.
- `nq` is the current `NQ` object.

It must return XXX TODO XXX

### `settings.promise`

These wrappers are not chained, only the most specific one is called.

Example:

If `promise` is set to `RSVP`, then each call to `NQ(x)` translates to `new RSVP(NQ(x))`.


## `NQ.req`

Create a function/object with the prototype of NQ.Req (which has the NQ object as prototype).  If a `success` or `fail` function is added here, a call to `bind` is implicite.

If `settings.promise` is set, then the created object `o` is wrapped into `new settings.promise(o)` which then is returned.

- `NQ.req([context], req, send, settings, success, fail)`
- `NQ([context], req, send, settings, success, error)`
- Returns NQ.Req object.  The prototype chain is NQ.Req then NQ.NQ, so you have all methods of NQ, too.
- allows method chaining

Parameters:

- `context`: Optional context for the callback functions.  See `settings.context`.  Must be an object!
- `req`: A string or `function(nq)` returning the string.  The request to send, which selects the service on the middleware to talk to.
- `send`: The object to send or `function(nq)` returning the object.  The object will be transferred JSON encoded.
- `settings`: see `setup`
- `success`: function called, when the answer arrives.  See `settings.success`
- `fail`: function called, when something fails.  See `settings.fail`


## `NQ.Req.start`

Starts the communication to the backend.  When an answer arrives, this is a `success` (this includes the backend sending some error condition).  A `fail` only happens when the request is aborted by the middleware (like timed out session).

- `NQ(..)([context], success, error, settings)`
- `NQ(..).start([context], success, error, settings)`

See `req` for the parameters.  Note that this function only is present, if `req` returned a NQ.Req object, not when it is wrapped into `setting.promise`.

