- ES6 introduces a new type of function that does not behave with the run-to-completion behavior but rather a cooperative concurrency (a special kind of function that can suspend: can be paused in mid-completion (entirely preserving its state), and it can later be resumed from where it left off). This new type of function is called a "generator" with the following syntax: `*functionName(){...}`

- The pause/resume interchange is cooperative rather than preemptive, which means that the *generator* has the sole capability to *pause itself*, using the `yield` keyword, and the _iterator_ that controls the generator has the sole capability (via `next(..)` method) to *resume the generator*.
   Each time you construct an _iterator_, you are implicitly constructing an *instance* of the generator which *that iterator* will control (i.e. you can have multiple instances of the same generator running at the same time):

```js
function *foo() {
	var x = yield 2;
	z++;
	var y = yield (x * z);
	console.log( x, y, z );
}

var z = 1;

// The `it1 = foo()` operation does NOT execute the `*foo()` generator yet,
// it only CONSTRUCTS an iterator that will control its execution:
var it1 = foo();
var it2 = foo();

// A generator can be treated as a PRODUCER of values that we can extract one at a time through an iterator interface's `next()` calls.
// A `next(..)` call RESUMES the generator from its respective `yield`(i.e. if we pass in a value as its argument, it'll REPLACE the `yield`ed expression where an execution was paused with that value). 
// The FIRST `next()` call basically RESUMES a non-existing `yield`-paused execution, therefore, you must ALWAYS start a generator with an argument-free `next()`.
var val1 = it1.next().value;			// 2 <-- yield 2
var val2 = it2.next().value;			// 2 <-- yield 2

val1 = it1.next( val2 * 10 ).value;		// 40  <-- x:20,  z:2
val2 = it2.next( val1 * 5 ).value;		// 600 <-- x:200, z:3

it1.next( val2 / 2 );					// y:300
										// x:20 y:300 z:3
										
it2.next( val1 / 4 );					// y:10
										// x:200 y:10 z:3
```

- The `yield` / `next(..)` duality is not just a control mechanism, it's actually a two-way message passing mechanism. A `yield ..` expression essentially pauses waiting for a value, and the next `next(..)` call passes a value (or implicit `undefined`) back to that paused `yield` expression.

- Generators preserve a sequential, synchronous, blocking code pattern for async code, which lets our brains reason about the code much more naturally by hiding potential asynchrony behind the `yield` keyword: moving the asynchrony to the code where the generator's _iterator_ is controlled.

- Because the generator pauses at each `yield`, its state (scope) is kept around, meaning there's no need for the closure boilerplate to preserve variable state across calls.
---

### Iterables
- An array (which is an *object* that **contains** an _iterator_ that can iterate over its values) is called an ***iterable*** and has the `next()` method on its interface. The way to retrieve an _iterator_ from an _iterable_ is that the _iterable_ must have a function on it, whose name being the special ES6 symbol value `Symbol.iterator`. When this function is called, it returns an _iterator_:

```js
var a = [1,3,5,7,9];

// This loop:
for (var v of a) {
	console.log( v );
}
// 1 3 5 7 9


// Is equivalent to this:
var it = a[Symbol.iterator]();

console.log(it.next().value);	// 1
console.log(it.next().value);	// 3
console.log(it.next().value);	// 5
..
```

- A generator itself is ***not*** technically an _iterable_ (though it's very similar): when you *execute* the generator, you get an _iterator_ back. ***After*** that, the generator's ==_iterator_ also== has a `Symbol.iterator` function on it, which basically does a `return this`, which makes it ==an _iterable_==

```js
function *something() {
	var nextVal;

	while (true) {
		if (nextVal === undefined) {
			nextVal = 1;
		}
		else {
			nextVal = (3 * nextVal) + 6;
		}

		yield nextVal;
	}
}

for (var v of something()) {
	// `something` is now an iterable!
	console.log( v );

	// don't let the loop run forever!
	if (v > 500) {
		break;
	}
}
// 1 9 33 105 321 969

//-------------- Stopping the generator ---------------
// In the above example, it SEEMS that the iterator instance for the `*something()` generator was basically left in a suspended state forever after the `break` in the loop was called.
// But there's the "Abnormal completion" (i.e., "early termination") of the `for..of` loop -- generally caused by a `break`, `return`, or an uncaught exception -- sends a TERMINATE signal to the generator's iterator.
// If you want to MANUALLY terminate the iterator, you can call its `return()` method:

for (var v of something()) {
	// `something` is now an iterable!
	console.log( v );

	if (v > 500) {
		console.log(
			// terminate the generator's iterator
			// set the returned `value` to whatever you passed in to `return(..)`:
			it.return( "We're done." ).value
		);
		// the `done` property of `next()`is set to `true`
		// no `break` needed here
	}
}
// 1 9 33 105 321 969
// We're done.

//----------------------------
// If you specify a `try..finally` clause inside the generator, it will always be run even when the generator is externally completed. This is useful if you need to clean up resources (database connections, etc..):

function *something() {
	try {
		var nextVal;

		while (true) {
			if (nextVal === undefined) {
				nextVal = 1;
			}
			else {
				nextVal = (3 * nextVal) + 6;
			}

			yield nextVal;
		}
	}
	// cleanup clause
	finally {
		console.log( "cleaning up!" );
	}
}

for (var v of something()) {
	// `something` is now an iterable!
	console.log( v );

	// don't let the loop run forever!
	if (v > 500) {
		break;
	}
}
// 1 9 33 105 321 969
// cleaning up!

//--------------
// If we call `it.return(..)`, it immediately terminates the generator, which of course runs the `finally` clause
for (var v of something()) {
	// `something` is now an iterable!
	console.log( v );

	if (v > 500) {
		console.log(
			// terminate the generator's iterator
			// set the returned `value` to whatever you passed in to `return(..)`:
			it.return( "We're done." ).value
		);
		// the `done` property of `next()`is set to `true`
		// no `break` needed here
	}
}
// 1 9 33 105 321 969
// cleaning up!
// We're done.
```
---

### Async with Generators 
- Replacing callback-driven pattern (by abstracting the asynchrony away as an *implementation detail*):

```js
// "Callback" approach:
function foo(x,y,cb) {
	ajax(
		"http://some.url.1/?x=" + x + "&y=" + y,
		cb
	);
}

foo( 11, 31, function(err,text) {
	if (err) {
		console.error( err );
	}
	else {
		console.log( text );
	}
} );

//-----------------
// Express the same flow control with a generator. 
// One of the main benefits of this approach is having the `yield`-pause nature of generators means that not only do we get synchronous-looking `return` values from async function calls
// but we can also synchronously `catch` errors from those async function calls:
function foo(x,y) {
	ajax(
		"http://some.url.1/?x=" + x + "&y=" + y,
		function(err,data){
			if (err) {
			    // `yield` pausing ALSO allows the generator to `catch` an error.
				// throw an error INTO `*main()`:
				it.throw( err );
			}
			else {
				// resume `*main()` with received `data`
				it.next( data );
			}
		}
	);
}

// Abstracting away the async Promise flow by using a generator:
function *main() {
	try {
		// `yield` allows us to have what appears to be blocking, synchronous code, but it doesn't actually block the whole program;
		// it only pauses/blocks the code in the generator itself:
		var text = yield foo( 11, 31 );  // actually `yield undefined`, THEN it gets defined AFTER `it.next( data )` is executed
		
		// `text` received its value from the completed response of `it.next( data )`
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}

var it = main();

// start it all up!
it.next();

//-------------------- Catch errors FROM a generator --------------------
function *main() {
	var x = yield "Hello World";

	yield x.toLowerCase();	// cause an exception!
}

var it = main();

it.next().value;			// Hello World

try {
	it.next( 42 );
}
catch (err) {
	console.error( err );	// TypeError
}
```
---

### Generator Delegation
- If you want to call one generator from another generator:

```js
// The purpose of `yield`-delegation is mostly code organization, and in that way is symmetrical with normal function calling.
// `yield *` is a syntactic shortcut for manually iterating over the steps of `*foo()` while inside of `*bar()`:
function *foo() {
	console.log( "`*foo()` starting" );
	yield 3;
	yield 4;
	console.log( "`*foo()` finished" );
}

function *bar() {
	yield 1;
	yield 2;
	yield *foo();	// `yield`-delegation: calling `foo()` creates an iterator
	yield 5;
}

var it = bar();

// As soon as the `it` control finishes the entire `*foo()` iterator, it automatically returns to controlling `*bar()`:
it.next().value;	// 1
it.next().value;	// 2
it.next().value;	// `*foo()` starting
					// 3
it.next().value;	// 4
it.next().value;	// `*foo()` finished
					// 5
```

- Errors/exceptions also pass in both directions:

```js
function *foo() {
	try {
		yield "B";
	}
	catch (err) {
		console.log( "error caught inside `*foo()`:", err );
	}

	yield "C";

	throw "D";
}

function *bar() {
	yield "A";

	try {
		yield *foo();
	}
	catch (err) {
		console.log( "error caught inside `*bar()`:", err );
	}

	yield "E";

	yield *baz();

	// note: can't get here!
	yield "G";
}

function *baz() {
	throw "F";
}

var it = bar();

console.log( "outside:", it.next().value );
// outside: A

console.log( "outside:", it.next( 1 ).value );
// outside: B

// This sends the error message `2` into `*bar()`, which delegates that to `*foo()`, which then `catch`es it and handles it gracefully.
// Then, the `yield "C"` sends `"C"` back out as the return `value` from the `it.throw(2)` call:
console.log( "outside:", it.throw( 2 ).value );
// error caught inside `*foo()`: 2
// outside: C

// The `"D"` value that's next `throw`n from inside `*foo()` propagates out to `*bar()`, // which `catch`es it and handles it gracefully. 
// Then the `yield "E"` sends `"E"` back out as the return `value` from the `it.next(3)` call.
console.log( "outside:", it.next( 3 ).value );
// error caught inside `*bar()`: D
// outside: E

// the exception `throw`n from `*baz()` isn't caught in `*bar()`, but caught outside instead. Therefore, `it.next( 4 )` didn't return "G":
try {
	console.log( "outside:", it.next( 4 ).value );
}
catch (err) {
	console.log( "error caught outside:", err );
}
// error caught outside: F
```
---

### Generator Concurrency
```js
// two instances of `*reqData(..)` run truly concurrently:

var res = [];

function *reqData(url) {
	var data = yield request( url );

	// transfer control
	yield;

	res.push( data );
}

var it1 = reqData( "http://some.url.1" );
var it2 = reqData( "http://some.url.2" );

var p1 = it1.next().value;
var p2 = it2.next().value;

// both instances receive their `data` as soon as their respective responses come back
p1.then( function(data){
	it1.next( data );
} );

p2.then( function(data){
	it2.next( data );
} );
// then each instance does another `yield` for control transfer purposes (the second `yield` in `*reqData(..)`)

// We choose what order to resume them in the `Promise.all([ .. ])` handler:
Promise.all( [p1,p2] )
.then( function(){
	it1.next();
	it2.next();
} );
```
---
