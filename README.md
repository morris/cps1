# Continuation-Passing Style 1 (CPS1)

__Open Specification Draft__

Continuation-passing style (CPS) is the predominant way of handling asynchronous operations in JavaScript, especially in [Node.js](http://nodejs.org/).
CPS functions expect one or more callbacks, or continuations, that continue the program flow after certain events.
Promises moved in and out of the Node.js core,
and eventually got specified with [Promises/A+](https://promisesaplus.com/),
but even after Node.js went back to CPS, it always lacked a detailed specification.

This specification defines a subset of well-formed CPS functions and callbacks and their behavior, named __CPS1__.
It is being designed to match Node.js which widely uses __single, error-first callbacks__ for CPS.
Otherwise, the specification is portable.


## 1. Terminology

- 1.1. A "function" is any JavaScript function.
- 1.2. A "value" is any JavaScript value, including functions.
- 1.3. "Truthy" is any value that is considered true in JavaScript.
- 1.4. "To ignore (an argument)" means to execute a function regardless of that argument's value.
- 1.5. "CPS1" is short for Continuation-Passing Style 1.


## 2. CPS1 Functions

A CPS1 function expects a (possibly optional) callback as its last argument,
and it must call the callback exactly once,
either with an error or with a constant number of arguments.

- 2.1. A function `f` is a __CPS1 function__ if there is a constant `N >= 0` for which `f` conforms to the following:
- 2.2. When `f` is called, `f` must decide which argument may hold the `callback`, if any.
The decision is arbitrary, but arguments after the callback __must be ignored__.
- 2.3. If `callback` is a function, it must be eventually called __exactly once__.
- 2.4. `callback` must be called with one of the following argument lists:
  - 2.4.1. `( err, arg1, arg2, ..., argN )` with arbitrary values
  - 2.4.2. `( err )` where `err` is truthy
  - 2.4.3. `()` if `N = 0`
- 2.5. `callback` may be called using `callback( ... )`, `callback.apply( ... )`, `callback.call( ... )`, or any other means to execute a function.
- 2.6. `callback` may be called directly or indirectly by `f` or other functions, immediately or at any time.
- 2.7. `callback` should be an optional argument. If omitted, `f` should treat it like a `noop`, i.e. `function() {}`.
- 2.8. `f` should __immediately throw__ programmer errors, e.g. argument errors.
  - 2.8.1. In particular, if `callback` is optional, it should immediately throw an argument error if it is neither a function nor `undefined`.
  - 2.8.2. Otherwise, it should immediately throw an argument error if `callback` is not a function.
- 2.9. Other errors and exceptions should not be thrown, and instead be catched and passed to `callback( err )`.
- 2.10. When `err` is truthy, it should be an object containing error and exception details.

## 3. CPS1 Callbacks

A CPS1 callback must ignore any arguments if an error is passed.

- 3.1. A function `callback` is a __CPS1 callback__ if it conforms to the following:
- 3.2. When `callback` is called, and the first argument `err` is truthy, all remaining arguments __must be ignored__.

## 4. Notes

- 4.1. Point 2.2. supports a variable number of arguments and arbitrary argument mappings,
but forces the callback to be the last argument.
- 4.2. Point 2.4. is designed to ensure the number of callback arguments is constant,
while providing shortcuts for success, errors and passthroughs.
- 4.3. The term "CPS1" was chosen because the defined functions expect a single callback which is called exactly once.


## 5. Examples

```javascript
// 5.1
// setTimeout is not CPS1
// Violates 2.2.: Callback is not last argument
// function setTimeout( callback, t )

// CPS1
function setTimeoutCPS1( t, callback ) {

	return setTimeout( callback, t );

}


// 5.2
// func is not a CPS1 function
// Violates 2.4.: Number of callback arguments is not constant, no such N
function func( one, two, callback ) {

	if ( !two ) callback( null, one );
	else callback( null, one, two );

}

// CPS1, N = 2
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
