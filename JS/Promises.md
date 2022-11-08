- Promises are an easily repeatable mechanism for encapsulating and composing _future values_. They create a pattern of expressing async flow in sequential fashion, which helps our brains plan and maintain async JS code better.
 
 They solve the _inversion of control_ issues that plague us with callbacks-only code:

```js
// Uninversion of control enables a nicer SoC, where `bar(..)` and `baz(..)` don't need to be involved in how `foo(..)` is called. Similarly, `foo(..)` doesn't need to know or care that `bar(..)` and `baz(..)` exist or are depending on the state if `foo(..)` completes or not (because once `foo(..)` resolves, `bar(..)` and `baz(..)` can only be continued PREDICTABLY from the resolved value of `foo(..)`).

function foo(x) {
	// start doing something that could take a while

	// construct and return an immediately fulfilled Promise that cannot be observed synchronously
	// The argument as a callback is synchronously/immediately called. This function is passed two function callbacks that act as resolution capabilities for the promise.
	return new Promise( function(resolve,reject){
		// `reject(..)` simply rejects the promise
		// but `resolve(..)` can either fulfill the promise or reject it, depending on what it's passed.
		// If `resolve(..)` is passed an immediate, non-Promise, non-thenable value, then the promise is fulfilled with that value.
        // But if `resolve(..)` is passed a genuine Promise or thenable value, that value is unwrapped recursively, and WHATEVER its final resolution/state is (fulfilled/rejected) will be adopted by the promise.
	} );
}

var p = foo( 42 );

function bar(fooPromise) {
	// listen for `foo(..)` to complete
	fooPromise.then(
		function(){
			// `foo(..)` has now finished, so
			// do `bar(..)`'s task
		},
		function(){
			// oops, something went wrong in `foo(..)`
		}
	);
}
// Same thing for `baz(..)`

bar( p );
baz( p );

//--------------- Or:

function bar() {
	// `foo(..)` has definitely finished, so
	// do `bar(..)`'s task
}

function oopsBar() {
	// oops, something went wrong in `foo(..)`,
	// so `bar(..)` didn't run
}

// ditto for `baz()` and `oopsBaz()`

p.then( bar, oopsBar );
p.then( baz, oopsBaz );
```

- Promises encapsulate the **time-dependent state** --  the fulfillment or rejection of the underlying *value* -- by waiting from the outside, therefore, Promise itself is time-independent, and thus Promises can be composed (combined) in predictable ways *regardless of the timing or outcome underneath*.
   Moreover, ==once ***a*** Promise is resolved, it stays that way forever -- it becomes an _immutable value_ at that point -- and can then be _observed_ as many times as necessary.==

- Promises don't get rid of callbacks, they just redirect the orchestration of those callbacks to a trustable intermediary mechanism that sits between us and another utility.
---

### "Thenable" Duck Typing
- The general term for "type checks" that make assumptions about a value's "type" based on its shape (what properties are present) is called "duck typing". 
   "Thenable duck typing" is a way to create a non-genuine-*but-Promise-like* value for recognizing a Promise (or something that behaves like a Promise) by defining something called a "thenable" as any object or function which has a `then(..)` method on it. It is assumed that any such value is a Promise-conforming thenable:

```js
// Thenable duck typing:
if (
	p !== null &&
	(
		typeof p === "object" ||
		typeof p === "function"
	) &&
	typeof p.then === "function"
) {
	// assume it's a thenable!
}
else {
	// not a thenable
}

// Use it:
var o = { then: function(){} };

// make `v` be `[[Prototype]]`-linked to `o`
var v = Object.create( o );

v.someStuff = "cool";
v.otherStuff = "not so cool";


// The thenable duck typing checks will think and assume `v` is a thenable, but actually isn't: 
v.hasOwnProperty( "then" );		// false
```

- Beware that thenable duck typing can be hazardous if it incorrectly identifies something as a Promise that isn't.
---

### Promise Trust
- Promises are designed to solve all of the _inversion of control_ trust issues with callbacks-only coding. When you pass a callback to a utility `foo(..)`, it might:
	-   Call the callback too early:
		- Solving "race condition" (when sometimes a task finishes synchronously and sometimes asynchronously): an immediately fulfilled Promise 
		  (like `new Promise(function(resolve){ resolve(42); })`) **cannot** be _observed_ synchronously.
		- When you call `then(..)` on a Promise, even if that Promise was already resolved, the callback you provide to `then(..)` will **always** be called asynchronously. No more need to insert your own `setTimeout(..,0)` hacks.
		
	-   Call the callback too late:
		- A good practice is to code in such a way where the ordering of multiple callbacks **doesn't** matter. 
		- When a Promise is resolved, all `then(..)` registered callbacks on it will be called, **in order**, immediately at the next asynchronous opportunity, and ==nothing that happens inside of one of those callbacks can affect/delay the calling of the other callbacks==
	```js
// You should never rely on anything about the ordering/scheduling of callbacks across Promises:
var p3 = new Promise( function(resolve,reject){
	resolve( "B" );
} );

var p1 = new Promise( function(resolve,reject){
	resolve( p3 );
} );

var p2 = new Promise( function(resolve,reject){
	resolve( "A" );
} );

p1.then( function(v){
	console.log( v );
} );

p2.then( function(v){
	console.log( v );
} );
// A B  <-- not  B A  as you might expect

// ---------------------------------
// all `then(..)` registered callbacks on it will be called, in order immediately at the next asynchronous opportunity:
p.then( function(){
	p.then( function(){
		console.log( "C" );
	} );
	console.log( "A" );
} );
p.then( function(){
	console.log( "B" );
} );
// A B C
// "C" cannot interrupt and precede "B", by virtue of how Promises are defined to operate
```

	-   Never Calling the Callback:
		- If you register both fulfillment and rejection callbacks for a Promise, and the Promise gets resolved, **one of the two** callbacks will *always* be called. 
		- Use a higher level abstraction called a `.race()` if the Promise itself never gets resolved:
			```js
// a utility for timing out a Promise 
function timeoutPromise(delay) {
	return new Promise( function(resolve,reject){
		setTimeout( function(){
			reject( "Timeout!" );
		}, delay );
	} );
}

// we can ensure a signal as to the outcome of a `foo()` Promise, to prevent it from hanging our program indefinitely:
// setup a timeout for `foo()` 
Promise.race( [
	foo(),					// attempt `foo()`
	timeoutPromise( 3000 )	// give it 3 seconds
] )
.then(
	function(){
		// `foo(..)` fulfilled in time!
	},
	function(err){
		// either `foo()` rejected, or it just
		// didn't finish in time, so inspect
		// `err` to know which
	}
);
```

	-   Calling Too Many Times:
		- Promises are designed to only be resolved once. If for some reason the Promise creation code tries to call `resolve(..)` or `reject(..)` multiple times, or tries to call both, the Promise will accept only the first resolution, and will silently ignore any subsequent attempts. Any `then(..)` registered callbacks will only ever be called once (each).
		- If you register the same callback more than once, (e.g. `p.then(f); p.then(f);`, **not** `p.then(f).then(f)`: this is a *chained* promise), it'll be called as many times as it was registered. The guarantee that a response function is called only once does not prevent you from shooting yourself in the foot.
		
	-   Failing to Pass Along Any Parameters/Environment:
		- Promises can have, **at most**, *one* resolution value (of fulfillment or rejection). If you don't explicitly resolve with a value either way, the value is `undefined`.
		- If you call `resolve(..)` or `reject(..)` with multiple arguments, all subsequent parameters beyond the first will be silently ignored. Therefore, if you want to pass along multiple values, you must wrap them in another single value that you pass, such as an `array` or an `object`.
		
	-   Swallowing Any Errors/Exceptions:
		- If at any point in the creation of a Promise, or in the observation of its resolution, a JS exception error occurs, such as a `TypeError` or `ReferenceError`, that exception will be caught, and it will force the Promise in question to become rejected:
		```js
var p = new Promise( function(resolve,reject){
	foo.bar();	// `foo` is not defined, so error!
	resolve( 42 );	// never gets here :(
} );

p.then(
	function fulfilled(msg){
	// never gets here :(
        console.log(msg)
	},
	function rejected(err){
        console.log(err) // ReferenceError: foo is not defined
	}
);

//-----------------------
// `f` was already fulfilled to the value `69`, so it can't later be changed to a rejection (hence the fundamental principle that Promises are IMMUTABLE once resolved) just because there's an error in observing `f`'s resolution:
var f = new Promise( function(resolve,reject){
	resolve( 69 );
} );

f.then(    // this is a promise itself
	function fulfilled(msg){
		// You'll get an auto-prompted error in the console, not from the `rejected(err)` function below:
		foo.bar();    
		console.log( msg );	// never gets here :(
	},
	function rejected(err){
		// never gets here either :(
		console.log(err)
	}
);

f.then(
	function fulfilled(msg){
		// Without `foo.bar()`, the `console.log(msg)` below returns the IMMUTABLE resolved value:
		console.log( msg );	     // 69
	},
	function rejected(err){
		// still never gets here because of the "immutable" 69
		console.log(err)
	}
);
```
		- If there were multiple `then(..)` registered callbacks on the same promise, be very careful with that because some would get called and others wouldn't, and it could be confusing as to why.
	
- `Promise.resolve(..)` will accept any "thenable", and will unwrap it to its non-thenable value. What you get back from `Promise.resolve(..)` is a real, genuine Promise in its place, **the one that you can trust**. If what you passed in is already a genuine Promise, you just get it right back, so there's no downside at all to filtering through `Promise.resolve(..)` to gain trust:

```js
var p1 = Promise.resolve( 42 );

// is similar to:
var p1 = new Promise( function(resolve,reject){
	resolve( 42 );
} );

var p2 = Promise.resolve( p1 );

p1 === p2; // true

//-------------
// A "thenable" object:
var p = {
	then: function(cb,errcb) {
		errcb( "evil laugh" );
		cb( 42 );
	}
};

Promise.resolve( p )
.then(
	function fulfilled(val){
	// never gets here
		console.log( val ); 
	},
	function rejected(err){
		console.log(err)    // evil laugh
	}
);

// If that thenable (or genuine promise) unwrapping reveals a rejected state, the Promise returned from `Promise.resolve(..)` is in fact in that same rejected state:
// Resolves this promise with a rejected promise:
var rejectedPr = Promise.resolve(Promise.reject( "Oops" ));

rejectedPr.then(
	function fulfilled(){
		// never gets here
	},
	function rejected(err){
		console.log( err );	// Oops
	}
);

// ----------- Good practice ----------- 
// Let's say we're calling a `foo(..)` utility and we're not sure we can trust its return value to be a well-behaving Promise, but we know it's at least a thenable. `Promise.resolve(..)` will give us a trustable Promise wrapper to chain off of:

// don't just do this:
foo( 42 )
.then( function(v){
	console.log( v );
} );

// instead, do this:
Promise.resolve( foo( 42 ) )
.then( function(v){
	console.log( v );
} );

```

- Another beneficial side effect of wrapping `Promise.resolve(..)` around any function's return value (thenable or not) is that it's an easy way *to normalize that function call into a well-behaving async task*. If `foo(42)` returns an immediate value sometimes, or a Promise other times, `Promise.resolve( foo(42) )` makes sure it's always a Promise result.
---

### Chain Flow
-   Every time you call `then(..)` on a Promise, it creates and returns a new Promise, which we can _chain_  with.
-   If the fulfillment or rejection handler returns a Promise, it is *unwrapped*, so that whatever its 
     resolution is *will become* the resolution of the subsequently chained Promise returned from the current `then(..)`:

```js
var p = Promise.resolve( 21 );

p.then( function(v){
	console.log( v );	// 21

	// create a promise to return
	return new Promise( function(resolve,reject){
		// introduce asynchrony!
		setTimeout( function(){
			// fulfill with value `42`
			resolve( v * 2 );
		}, 69 );
	} );
} )
.then( function(v){
	// runs after the 69ms delay from the previous step
	console.log( v );	// 42
} );
```

-   Inside the fulfillment/rejection handlers, if you return a value or an exception is thrown, the new returned (subsequent chainable) Promise is resolved accordingly. E.g:

 ```js
// When the error occurs in step 2, the rejection handler in step 3 catches it. The return value (`69` in this snippet), if any, from that rejection handler fulfills the promise for the next step (4), such that the chain is now back in a fulfillment state.
//------------------------------

// step 1:
var request = Promise.resolve( "http://some.url.1/" )

// step 2:
request.then( function(response1){
	foo.bar(); // undefined, error!

	// never gets here
	return request( "http://some.url.2/?v=" + response1 );
} )

// step 3:
.then(
	function fulfilled(response2){
		// never gets here
	},
	// rejection handler to catch the error
	function rejected(err){
		console.log( err );	// ReferenceError: foo is not defined
		return 69;
	}
)

// step 4:
.then( function(msg){
	console.log( msg );		// 69
} );
```

- The two callbacks provided to `then(..)` should be called `fulfilled(..)` and `rejected(..)` since calling `.then(..)` assumes a Promise has *resolved*.
---

### Error Handling
- `try..catch` would certainly be nice to have, but it's *synchronous-only* so it **doesn't** work across traditional async operations:

```js
function foo() {
	setTimeout( function(){
		baz.bar();
	}, 100 );
}

try {
	foo();
	// later throws global error from `baz.bar()`
}
catch (err) {
	// never gets here
}
```

- `catch(..)` also create and return a new promise and takes only the rejection callback as a parameter, and automatically substitutes the default fulfillment callback. If the fulfillment or rejection callbacks have an exception thrown, the returned promise is rejected. In other words, it's equivalent to `then(null,..)`:

```js

p.then( fulfilled );

p.then( fulfilled, rejected );

p.catch( rejected ); // or `p.then( null, rejected )`
```

- Promises could default to reporting (to the developer console) any rejection, on the next Job or event loop tick, if at that exact moment no error handler has been registered for the Promise.

- For the cases where you want a rejected Promise to hold onto its rejected state for an indefinite amount of time before observing, you could make an utility function called  `defer()` and then invoke it, which suppresses automatic error reporting on that Promise (or use a suitable library for that):

```js
// The promise returned from `foo(..)` gets an error handler attached _right away_, so it's implicitly opted out and no global reporting for it occurs either.
var p = Promise.reject( "Oops" ).defer();

// `foo(..)` is Promise-aware
// the promise returned from the `then(..)` call has no `defer()` or error handler attached, so if it rejects (from inside either resolution handler), then it will be reported to the developer console as an uncaught error.
foo( 42 )
.then(
	function fulfilled(){
		return p;
	},
	function rejected(err){
		// handle `foo(..)` error
	}
);
...
```

- If you construct a Promise chain that has no error handling in it, any error anywhere in the chain will propagate indefinitely down the chain, until observed (by registering a rejection handler at some step).
   So, in that specific case, having a reference to the _last_ promise in the chain is enough (`p` in the following snippet), because you can register a rejection handler there, and it will be notified of any propagated errors:

```js
// `foo(..)`, `STEP2(..)` and `STEP3(..)` are
// all promise-aware utilities

var p = foo( 42 )
.then( STEP2 )
.then( STEP3 );


// Although it may seem sneakily confusing, `p` here doesn't point to the first promise in the chain (the one from the `foo(42)` call),
// but instead it points to the settlement of the last promise (the one that comes from the `then(STEP3)` call) based on how the chain flow works.
// Also, no step in the promise chain is observably doing its own error handling. 
// That means that you could then register a rejection error handler on `p`, 
// and it would be notified if any errors occur anywhere in the chain:

p.catch( handleErrors );
```
---

### Promise Patterns
- In classic programming terminology, a "gate" is a mechanism that waits on two or more parallel/concurrent tasks to complete before continuing. It *doesn't matter what order* they finish in, just that ***all*** of them have to complete for the gate to open and let the flow control through. In the Promise API, we call this pattern `all([ .. ])`:

```js
/**
 * `Promise.all([ .. ])` expects a single argument, an array, consisting generally of Promise instances.
 * The SINGLE promise returned from the `Promise.all([ .. ])` call will receive a fulfillment message
 * that is an array of all the fulfillment messages from the passed in promises, in the same order as specified (regardless of fulfillment order).

 * Technically, the array of values passed into `Promise.all([ .. ])` can include Promises, thenables, or even immediate values.
 * Each value in the list is essentially passed through `Promise.resolve(..)` to make sure it's a genuine Promise to be waited on, so an immediate value will just be normalized into a Promise for that value.
 * If the array is empty, the main Promise is IMMEDIATELY fulfilled.
 * It rejects immediately upon ANY of the input promises rejecting or non-promises throwing an error, and will reject with this first rejection message/error.
 */
 
// `request(..)` is a Promise-aware Ajax utility
var p1 = request( "http://some.url.1/" );
var p2 = request( "http://some.url.2/" );

//The main promise returned from `Promise.all([ .. ])` will only be fulfilled if and when all its constituent promises are fulfilled. If any one of those promises instead is rejected, the main `Promise.all([ .. ])` promise is immediately rejected, discarding all results from any other promises.
Promise.all( [p1,p2] )
.then( function(msgs){
	// both `p1` and `p2` fulfill and pass in
	// their messages here
	return request(
		"http://some.url.3/?v=" + msgs.join(",")
	);
} )
.then( function(msg){
	console.log( msg );
} );
```

- Remember to always attach a rejection/error handler to every promise, even and especially the one that comes back from `Promise.all([ .. ])`.

-  `Promise.race([ .. ])` will fulfill if and when **any** Promise resolution is a fulfillment, and it will reject if and when **any** Promise resolution is a rejection. **Warning:** A "race" requires **at least one** "runner," so if you pass an empty array, instead of immediately resolving, the main `race([..])` Promise will never resolve. So be careful never to send in an empty array.

```js
// `request(..)` is a Promise-aware Ajax utility
var p1 = request( "http://some.url.1/" );
var p2 = request( "http://some.url.2/" );

// Because only ONE promise wins, the fulfillment value is a SINGLE message, not an `array` as it was for `Promise.all([ .. ])`
Promise.race( [p1,p2] )
.then( function(msg){

	// EITHER `p1` or `p2` will win the race
	return request(
		"http://some.url.3/?v=" + msg
	);
} )
.then( function(msg){
	console.log( msg );
} );
```

- You'll likely have places where you want to make sure your Promises aren't just silently ignored *by accident*. The `finally()` method will not receive any argument and can be useful if you want to do some processing or cleanup once the promise is settled, regardless of its outcome.
 
- A `finally` call will usually chain through an equivalent to the **original** promise:

```js
// Will return a new fulfilled promise with the value `77`: 
Promise.resolve(2).then(() => 77, () => {}) 

// Will return a new fulfilled promise with the value `2`:
Promise.resolve(2).finally(() => 77)

// Will return a new fulfilled promise with the value `88`
Promise.reject(3).then(() => {}, () => 88)

// Will return a new rejected promise with the reason `3`
Promise.reject(3).finally(() => 88)

// Will return a rejected promise with the reason `99`:
Promise.reject(3).finally(() => Promise.reject(99))
```

- Unlike `Promise.race()` which returns the first _settled_ value (either fulfillment or rejection), the `Promise.any()` method returns the first _fulfilled_ value. 
  It will ignore all rejected promises up until the first promise that fulfills and only returns a rejected promise if the _iterable_ passed is empty or all of the promises passed-in the iterable are rejected. ==This can be beneficial if we need **only one** promise to **fulfill** but we do not care which one does.
- ---

### Limitations
- One of the most intrinsic behaviors of Promises is that a Promise can only be resolved once (fulfillment or rejection). For many async use cases, you're only retrieving a value once, so this works fine.
   Imagine a scenario where you might want to fire off a sequence of async steps in response to a stimulus (like an event) that can in fact happen multiple times, like a button click:
   
```js
// `click(..)` binds the `"click"` event to a DOM element
// `request(..)` is a Promise-aware Ajax
// This probably won't work the way you want:
var p = new Promise( function(resolve,reject){
	click( "#mybtn", resolve );
} );

p.then( function(evt){
	var btnID = evt.currentTarget.id;
	return request( "http://some.url.1/?id=" + btnID );
} )
.then( function(text){
	console.log( text );
} );
// The behavior here only works if your application calls for the button to be clicked just once. 
// If the button is clicked a second time, the `p` promise has already been resolved, so the second `resolve(..)` call would be ignored.



//--------------------
// Instead, you'd probably need to invert the paradigm, creating a whole new Promise chain for each event firing.
// This approach will work in that a whole NEW Promise sequence will be fired off for each `"click"` event on the button:

click( "#mybtn", function(evt){
	var btnID = evt.currentTarget.id;

	request( "http://some.url.1/?id=" + btnID )
	.then( function(text){
		console.log( text );
	} );
} );
// This design in some respects violates the idea of SoC. You might very well want to define your event handler in a different place in your code from where you define the response to the event (the Promise chain). BUt c'est la vie.
```
