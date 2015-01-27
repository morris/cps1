# Continuation-Passing Style 1 (CPS1)

__Open Specification Draft__

Continuation-passing style (CPS) is the predominant way of handling asynchronous operations in JavaScript, especially in [Node.js](http://nodejs.org/).
CPS functions expect one or more callbacks, or continuations, that continue the program flow after certain events.
Promises moved in and out of the Node.js core,
and eventually got specified with [Promises/A+](https://promisesaplus.com/),
but even after Node.js went back, the style used in Node.js has not been specified.

The CPS specified in this document is named __CPS1__ because it was designed to match Node.js
which widely uses a __single callback, error-first&nbsp;CPS__.


## 1. Preliminaries

- 1.1. A __function__ is any JavaScript function.
- 1.2. A __value__ is any JavaScript value, including functions.
- 1.3. A value is __truthy__ if it is considered _true_ in JavaScript.
- 1.4. A value is __falsy__ if it is not truthy.
- 1.5. A __function call__ is the evaluation of a function by any means,
including but not limited to `f(...)`, `call`, and `apply`.
- 1.6. An __argument__ is an entry in the `arguments` list of a function call.
- 1.7. In a function call, an argument is the __semantically last argument__
if all following arguments are ignored,
i.e. neither presence nor value may have effect on the function call.


## 2. CPS1 Operations

A CPS1 operation is a hereby specified sequence of events and actions:

- 2.1. A CPS1 operation must be started by a function call.
- 2.2. The operation must reserve the _semantically last argument_ of the function call for a callback.
- 2.3. If the callback is not a function,
the operation may use a default callback instead,
or should throw an argument error immediately.
- 2.4. The operation should not throw any exceptions besides argument errors in 2.3.
Instead, exceptions should be caught and passed to the callback (see section E).
- 2.5. The operation must eventually complete with one of __Error__, __Success__, or __Uncaught Exception__, defined by sections E, S, and U.
- 2.6. In any case, the callback must be called at most once by the operation.
- 2.7. The operation must not take any actions after completion.

### E. Error

- E1. The callback is called with an error as its first argument, immediately or at any time.
- E2. The error value must be truthy.
- E3. The error value should be an instance of a JavaScript error class, e.g. `Error`.
- E4. Additional arguments may be passed to the callback. They are ignored by CPS1 callbacks.


### S. Success

- S1. The callback is called with a falsy first argument and result arguments, if any, immediately or at any time.
- S2. The result(s) of the operation may be passed to the callback as additional arguments.
- S3. If no results need to be passed, the callback may be called with an empty argument list.
- S4. The number of result arguments must be obvious, and should be constant for equivalent contexts.
- S5. In particular, if the last result value is intended to be `undefined`, it must be explicitly passed to the callback.


### U. Uncaught Exception

- U1. During the operation, an exception is thrown and not caught by the operation,
immediately or at any time.
- U2. Uncaught exceptions should be avoided (see 2.4).


## 3. CPS1 callbacks

A CPS1 callback is a function which

- 3.1. must reserve the first argument for an error.
- 3.2. must ignore any other arguments if the error argument is truthy.
- 3.3. should not throw any exceptions.
- 3.4. should return `undefined`.


## 4. Notes

- 4.1. Because functions may have multiple signatures and program flows,
this specification defines abstract CPS1 operations, _not_ functions.
This way, function calls may or may not execute a CPS1 operation, depending on arguments or context
(that is, anything that may effect a function call, like the current program state and environment).
- 4.2. Functions designed to be CPS1-compliant should always start a CPS1 operation when called.
If not, it must be obvious for which arguments and context a function call starts a CPS1 operation.
This may be achieved through documentation, comments, or function signatures.



## 5. Examples

```javascript
// 5.1
// setTimeout is not CPS1
// Violates 2.2: Callback is not semantically last argument.
// function setTimeout( callback, t )

// CPS1
function setTimeoutCPS1( t, callback ) {

	return setTimeout( callback, t );

}


// 5.2
// func is not CPS1
// Violates S5: Must explicitly pass undefined as a result.
function func( one, two, callback ) {

	if ( !two ) callback( null, one );
	else callback( null, one, two );

}

// CPS1
function funcCPS1( one, two, callback ) {

	if ( !two ) callback( null, one, undefined );
	else callback( null, one, two );

}


// 5.3
// ready is not a CPS1 callback
// Violates 3.2.: Must ignore data on error
load( function ready( err, data ) {

	if ( err ) console.warn( err );
	console.log( data );

} );

// CPS1
load( function readyCPS1( err, data ) {

	if ( err ) console.warn( err );
	else console.log( data );

} );
```


## 6. License

This work is dedicated to the public domain.

https://creativecommons.org/publicdomain/zero/1.0/


## 7. Contributors

- [Morris Brodersen](mailto:mb@morrisbrodersen.de)


## 8. Acknowledgments

- http://thenodeway.io/posts/understanding-error-first-callbacks/
- https://promisesaplus.com/
- https://github.com/creationix/safereturn
