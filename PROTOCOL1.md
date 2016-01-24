# NQ protocol version 1

## Basics

NQ is specifically designed to talk to ZeroMQ, but can function independently of ZeroMQ.  In contrast to ZeroMQ you do not need to decide complicated things at the beginning, like pick a special communication type.  Everything is handled the same, and there can be a very complex network behind the scenes to process your messages.  You do not need to think about this when using NQ, because this all is handled by the NQ protocol and NQ wrappers for you.

ZeroMQ has several shortcommings and NQ tries to come over this, automagically, without you thinking of it:

- Reliability: ZeroMQ is not reliably telling you if there were problems encountered while processing a message.  Like a sudden reboot of a hub or the like.

- Starvation: If a process dies before it is able to answer, the sender will starve, as there never will be an answer.  ZeroMQ considers timeouts, but timeouts are very difficult to choose, as some processing takes microseconds, while others may take a century.

- Retransmission: ZeroMQ has no notion of retransmitting things in case something breaks unexpectedly on some layer.  If a request has gone, you have to send another request.

- Integrity: (Future Implementation) ZeroMQ does not protect against unauthorzed altering of packets or keeping the content secure.  You can add encryption yourself, but this needs special code on both sides.  Things like AAA (Authentication Authorization Accounting) shall not need additional code, it should be just a design decision on a case-by-case basis.

NQ compensates for all this with the NQ protocol.  You do not have to worry about anything.  Messages will be reliably be retransmitted in a starvation situation until you receive your proper answer.  This does involve timeouts, but the timeouts are part of the protocol, not part of the message processing.  So regardless if processing takes microseconds or years, everything is handled fine.  Also the protocol supports progress messages, such that you can inform the requester on how far the progress is done if a processing takes a bit longer.

The interesting part is, that you can always shutdown parts of the ZeroMQ network any time.  When you restart it, processing automagically resumes because the clients will retransmit lost requests.  If some requests are still due (and the stop and restart is done properly) then the clients even may get into contact to the processors again.

Note that ZeroMQ is only one way of an intermediate transport.  If it is used, ZeroMQ needs to know little about NQ.  The NQ protocol usually is only needed for the receivers (the web service), however the situation can improve if it is running on the processors (endpoings etc.), too.  In a degenerated case NQ could be run on the endpoints alone, if wrapped in suitable containers which allow to reache those endpoints.  Which should be relatively easy to do on your client side.


## Always 2-way

The protocol is always 2-way.  So not only clients are able to talk to services, services are able to talk to clients as well.  This usually is called "push" technology.

This is not real "push", as clients still decide when they are doing messaging.  So implicitely this is solved by doing "pull"s on regular intervals.  This interval can be freely chosen, it is much like Keepalive-Messages in case there is nothing to transmit.

But with Web-Sockets you will get the real duplex (`push`-type) communication.


## Multi messages channel

The protocol is handling multiple messages over a single channel.

The problem with modern web applications is, that they may need to do may things in parallel.  However web resources are usually limited, like a few concurrent connections to a single web server or over a certain proxy.

NQ solves this problem by channeling all the messages over a single channel.  So only a single connection must be managed, which frees resources on the web service side.  The connection is automatically opened if needed and closed in case everything is idle.

Mobile applications even may want to conserve energy by interrupting communication in idle times until polling again.  NQ is meant to support that if the need arises (it is not yet implemented, but already forseen in the protocol.  Even if it is not builtin, you can already quiesce the transfer module for a while if you like).


## Batching

To better utilize throughput, NQ automatically queues requests and sends them in batches if there is more than one request open.  This way it minimizes the number of connection requests needed.


## simplex and duplex transports

NQ supports both, simplex and duplex transports.  HTTP is simplex, as it first sends and then receives, but not on the same time.  Web-Sockets however can send and receive data any time, so they are duplex.  (Currently there is no Web-Socket support.)


## Symmetric

The NQ protocol is symmetric, so there is no server side or client side.  Both sides can initiate transmissions, both sides can answer messages, and there is only one single message format.


# Protocol

- `DATA`: Any JavaScript object which is automatically serialized and deserialized via JSON.  An error occurs if the object cannot be serialized or contains loops (which are not imported).  The library might support to serialize special objects (like a DOM tree) which usually cannot be converted to JSON in a special way.  If speed is of concern, stick to simple values like strings.

- `ID`: Truthy simple single JSON string.  You can use integers, but they might come back as string.  Also beware that floats or high integer numbers are prone to rounding.  If you leave this alone, an `ID` is generated for you.

- `ERR`: Error, warning or note object.  For asynchronous conditions.  This does not include synchronous errors (like failed call to a library function).  Errors contain following keys:
  - `type`: an error type, chosen by the protocol.
    - `local`: From the `nq` library on your side
    - `remote`: From the `nq` library on the remote side (was received as `.type==='local'`)
    - `app`: From the application (the one who processed the message)
    - `tmp`: Transient error.  This is not really an error, but indicator of possible error condition which might go away.  Transient errors are only generated if `VERBOSE` is truthy.
    - `warn`: Transient error.  This is not really an error, but indicator of possible error
    - `note`: Transient error.  This is not really an error, but indicator of possible error
    - `debug`: Debug messages.  These are normally not transmitted, except verbosity is enabled
    - `sys`: A system message, like answer to a poll.  Usually internally processed by NQ.
     - These type of messages are only meaningful to NQ and subject to change frequently
  - `fatal`: boolean, true if this is a fatal error which means, the message must be assumed dead and this probably is the last notification about this matter
  - `err`: a short error code, which depends on the `type`
    - set for `.type==='local'` or `.type==='remote'`
    - may be present for non-errors, too, if this is in an error context
    - normally missing on transient (`.type==='tmp'`) errors
  - `warn`: a short warning code, mostly for `.type==='warn'`, can be missing
  - `note`: a short note code, mostly for `.type==='note'`, can be missing
  - `text`: The error text, english string, may contain placeholders:
    - '{0}' refers to the err/warn/note code (first present), use '{err}', '{warn}', '{note}' to refer to a special one
    - '{1}' refers to '.args[0]'
    - '{n}' refers to '.args[n-1]'
    - If an argument is missing it is replaced by the empty string
  - `args`: Arguments for the error text, array of arguments of strings, index 0 is referred to by '{1}'
    - use `.format()` to format the message, this is a shortcut to `.format(.text, .args)`, so you can put your translated code as the first argument if you like.

- `VERBOSE`: Message verbosity level
  - `false` `.type==='debug'` messages are suppressed and transient errors are not generated
  - `true` means, they are transported but not generated by NQ, transient errors are generated
  - any truthy string (like `'debug'`) means NQ also generates `.type==='debug'` errors

- `NAME`: This is a routing string.  For an application this is just an extended alphanumeric string which may include `-` or `_`, but without blanks and no special characters inside.  Note that `NAME`+`ID` (read: `.t`+'#'+`.r`) must be unique, so if you send `NAME`+`ID` again, this is a retransmission.  Behavior is undefined if you repeat a message with a previous `NAME`+`ID` and altered parameters.  (You might get back a cached answer, trigger a reprocessing, see nothing, so the message just vanishes, or get back an error.  Applications should be constructed such, that such a condition is handled gracefully and no real bad things happen, like the destruction of an universe.)


## Error conditions

NQ is for distributed computing.  This means things like network splits can happen.  Hence there are never real final states on anything.  So it is only final iff you declare it final.

NQ also does not guarantee, that a message is only processed once.  Messages can be processed any way, so they can become duplicated, retransmitted or delayed over a long period of time.

NQ also does not guarantee, that a message is processed or that an error condition reaches you in time.  Retransmission is implicite, but this is handled internally in the library, normally your application leaves that fully to the library.

Hence, if you do error processing, you need to apply good heuristics on when to do what.  To simplify this, there is the `fatal` property on errors.  If it is true, this means, the NQ library has given up on the particular message and declares it dead.  However this still means, the message might come back like a zombie later on.

Additionally, messages might come backe as a duplicate or even an evil clone when there was no error at all.  So you might receive two answers instead of one.

Usually this all is not a problem and handled by the library.  If multiple answers are received, the first one is delivered to your callbacks, and this makes your callbacks go away.  Hence the duplicate is no more received.

Likewise with errors.  If you are not interested in transient errors, you can look at the `fatal` property.  If it is `false`, just ignore that message.  Or you use the `fatal` callback, which receives all errors whith `.fatal===true` errors.

If you are interested in `warn`ings or `note`s, there are such convenience callbacks, too.  Please note that the `tmp` callback is called `transient`, which does only receive transient errors (this is `.fatal !== true && .type === 'tmp'`).


## Message format

The message format is a JSON object or, if multiple messages are send at once, an array of these JSON objects.  In these objects, all nonnumeric single ASCII values in the range 32 to 127 (including) are reserved keys for the protocol.  All other keys are free to use.  To explain the edge cases:  Underscore ('`_`'), SPC ('` `') and `DEL` are reserved keys, the empty key (''), numeric strings like '`0`' and other control characters like `NUL` are not reserved keys.  This reservation is not enforced, so wrong keys are tolerated, but might misbehave in future versions of this standard if you used them.

The JSON object has following keys:

- a: `ID`: answer
- b: -
- c: `NR*`: Compatibility (or communication) version.  If left away protocol 1 (this here) is assumed.
- d: `DATA`: your payload, message or answer
- e: `ERR`: error object.  Usually `.d` is missing
- f: `NR*`: positive number of messages which follow.  This field is missing if unknown.
- g: -
- h: -
- i: `ID`: assocciated ID or array of IDs
- j: -
- k: `KEY*`: encryption key used
- l: -
- m: -
- n: `NR`: Positive number of message number in stream.  If `d` is present this defaults to `0`.  The total number of messages is `n+f` in case `f` is set, too.
- o: `NR` or `[NR]`: positive number of next message expected or list of missing message numbers.
- p: `ID`: like `.r` for the polling (retransmission) case.  Expects a `sys`-type error message
- q: -
- r: `ID` request ID.  `.t` and `.r` together are considered to always be unique
- s: `NAME`: source (applications usually cannot set this value, as it will be overridden)
- t: `NAME`: target
- u: -
- v: `VERBOSE`: Considered `false` if missing
- w: -
- x: -
- y: -
- z: -

Note: `*` means: Future use

## Request (query, question, message) and answer

The presence of `r` indicates a request.  `t` and `d` are optional.

- out `{ t:"request", r:1234, d:jsonobject }`: send the `jsonobject` to `request` with ID `1234`.
- in `{ a:5678, i:1234 }`: immediate answer that the ID `1234` is accepted.  The backchannel has ID `5678`.
- in `{ a:5678, i:1234, d:"data1" }`: Partial answer
- out `{ i:5678 }`

## poll/answer

- `` xxx

# Unfinished things

## Base128

In case the payload is not UTF-8, like binary encrypted, then a base91 encoding is used.

- There are 95 ASCII-characters which can be encoded with 1 byte in UTF-8.
- In JSON-strings two of them need to be escaped (quote and backslash).

base  64:  3 characters ->  4 characters
base  85:  4 characters ->  5 characters
base  94:  9 characters -> 11 characters
base 101:  5 characters ->  6 characters
base  94: 10 characters -> 11 characters
base  94:  8 characters ->  9 characters

Using 93 bytes:
Using 92 bytes:
Using 91 bytes:

