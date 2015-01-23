# Continuation-Passing Style 1 (CPS1)

__Specification Draft__

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

- 2.1. A function `f` is a __CPS1 function__ if there is a constant `N >= 0` for which `f` conforms to the following:
- 2.2. When `f` is called, `f` must decide which argument may hold the `callback`, if any.
The decision is arbitrary, but arguments after the callback __must be ignored__.
- 2.3. If `callback` is not a function, it __must be ignored__.
- 2.4. If `callback` is a function, it must be eventually called __exactly once__.
- 2.5. `callback` must be called with one of the following argument lists:
  - 2.5.1. `( err, arg1, arg2, ..., argN )` with arbitrary values
  - 2.5.2. `( err )` only if `err` is truthy
  - 2.5.3. `()` only if `N = 0`
- 2.6. `callback` may be called using `callback( ... )`, `callback.apply( ... )`, `callback.call( ... )`, or any other means to execute a function.
- 2.7. `callback` may be called directly or indirectly by `f` or other functions, immediately or at any time.


## 3. CPS1 Callbacks

- 3.1. A function `callback` is a __CPS1 callback__ if it conforms to the following:
- 3.2. When `callback` is called, and the first argument `err` is truthy, all remaining arguments __must be ignored__.


## 4. Recommendations

- 4.1. When `err` is truthy, it should be an object containing error and exception details.
- 4.2. A CPS1 function should __immediately throw__ programmer errors, e.g. argument errors.
  - 4.2.1. In particular, it should immediately throw an argument error in case that the `callback` argument is neither a function nor `undefined`.
- 4.3. Other errors and exceptions should not be thrown, and instead be catched and passed to `callback( err )`.


## 5. Notes

- 5.1. Point 2.2. supports a variable number of arguments and complex argument mappings,
but forces the callback to be the last argument.
- 5.2. Point 2.5. is designed to fix the number of result arguments while providing shortcuts for success, errors and passthroughs.
- 5.3. The term "CPS1" was chosen because the defined functions expect a single callback which is called exactly once.


## 6. Examples

```javascript
// 6.1
// setTimeout is not CPS1
// Violates 2.2.: Callback is not last argument
// Violates 2.3.: Callback cannot be omitted
// function setTimeout( callback, t )

// CPS1
function setTimeoutCPS1( t, callback ) {

	callback = callback || function() {};
	return setTimeout( callback, t );

}


// 6.2
// Not a CPS1 function
// Violates 2.5.: Number of arguments is not constant, no such N
// Violates 2.3.: Callback cannot be omitted
function func( one, two, callback ) {

	callback( null, one );
	callback( null, one, two );

}

// CPS1, N = 2
function funcCPS1( one, two, callback ) {

	callback = callback || function() {};
	callback( null, one, null );
	callback( null, one, two );

}


// 6.3
// Not a CPS1 callback
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

## 7. License

This work is dedicated to the public domain.

https://creativecommons.org/publicdomain/zero/1.0/


## 8. Contributors

- [Morris Brodersen](mailto:mb@morrisbrodersen.de)


## 9. Acknowledgments

- http://thenodeway.io/posts/understanding-error-first-callbacks/
- https://promisesaplus.com/
- https://github.com/creationix/safereturn