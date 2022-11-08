- Asynchrony: It's about what happens when part of your program runs _now_, and another part of your program runs _later_ -- there's a gap between _now_ and _later_ where your program isn't actively executing. 

- The problem most of us seem to have is that _later_ doesn't happen strictly and immediately after _now_. In other words, tasks that cannot complete _now_ are, by definition, going to complete asynchronously, and thus we will not have blocking behavior as you might intuitively expect or want.

- The simplest (but definitely not only, or necessarily even best!) way of "waiting" from _now_ until _later_ is to use a function, commonly called a callback function. For example, when your JS program makes an Ajax request to fetch some data from a server, you set up the "response" code in a function (commonly called a "_callback_"), and the JS engine tells the hosting environment, "Hey, I'm going to suspend execution for now, but whenever you finish with that network request, and you have some data, please _call_ this function _back_." :

```js
// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", function myCallbackFunction(data){

	console.log( data ); // Yay, I gots me some `data`!
	
});
```

- Any time you wrap a portion of code into a `function` and specify that it should be executed in response to some event (timer, mouse click, Ajax response, etc.), you are creating a _later_ chunk of your code, and thus introducing asynchrony to your program.
---

### Async Console gotcha
- There are some browsers and some conditions that `console.log(..)` does **not** actually _immediately_ output what it's given. The main reason this may happen is because I/O is a very slow and blocking part of many programs (not just JS). So, it may perform better (from the page/UI perspective) for a browser to handle `console` I/O asynchronously in the background, without you perhaps even knowing that occurred:
```js
var a = {
	index: 1
};

// later
console.log( a ); // ??

// even later
a.index++;
```

We'd normally expect to see the `a` object be snapshotted at the exact moment of the `console.log(..)` statement, printing something like `{ index: 1 }`, such that in the next statement when `a.index++` happens, it's modifying something different than, or just strictly after, the output of `a`.

Most of the time, the preceding code will probably produce an object representation in your developer tools' console that's what you'd expect. But it's possible this same code could run in a situation where the browser felt it needed to defer the console I/O to the background, in which case it's _possible_ that by the time the object is represented in the browser console, the `a.index++` has already happened, and the `console.log(a)` shows `{ index: 2 }`.

It's a moving target under what conditions exactly `console` I/O will be deferred, or even whether it will be observable. Just be aware of this possible asynchronicity in I/O in case you ever run into issues in debugging where objects have been modified _after_ a `console.log(..)` statement and yet you see the unexpected modifications show up.

**Note:** If you run into this rare scenario, the best option is to use breakpoints in your JS debugger instead of relying on `console` output. The next best option would be to force a "snapshot" of the object in question by serializing it to a `string`, like with `JSON.stringify(..)`

---

### Event Loop
- The mechanism which handles executing multiple chunks of your program _over time_, at each moment invoking the JS engine, called the "event loop". A vastly simplified pseudocode to illustrate the concepts. But it should be enough to help get a better understanding:

```js
// `eventLoop` is an array that acts as a queue (first-in, first-out)
var eventLoop = [ ];
var event;

// each iteration of this loop is called a "tick"
// For each tick, if an event is waiting on the queue, it's taken off and executed.
// These events are your function callbacks.
while (true) {
	// perform a "tick"
	if (eventLoop.length > 0) {
		// get the next event in the queue
		event = eventLoop.shift();

		// now, execute the next event
		try {
			event();
		}
		catch (err) {
			reportError(err);
		}
	}
}
```


- ==It's important to note that `setTimeout(..)` doesn't put your callback on the event loop queue immediately.== What it does is set up a timer; when the timer expires, the environment places your callback into the event loop, such that _some future tick_ will pick it up and execute it.

	 - What if there are already 20 items in the event loop at that moment? Your callback **waits**. It gets in line behind the others -- there's not normally a path for preempting the queue and skipping ahead in line. This explains why `setTimeout(..)` timers may not fire with perfect temporal accuracy. You're guaranteed (roughly speaking) that your callback won't fire _before_ the time interval you specify, but ==it can happen at or after that time, depending on the state of the event queue.

- In other words, your program is generally broken up into lots of small chunks, which happen one after the other in the event loop queue. And technically, other events not related directly to your program can be interleaved within the queue as well.

---
### Parallel Threading
- The most common tools for parallel computing are processes and threads. Processes and threads execute **independently** and may execute **simultaneously**: on separate processors, or even separate computers, but multiple threads can share the memory of a single process.

- In a single-threaded environment, it really doesn't matter that the items in the thread queue are low-level operations, because nothing can interrupt the thread. But if you have a parallel system, where two different threads are operating in the same program, you could very likely have unpredictable behavior.

- Because ==JavaScript is single-threading==, the code inside of `foo()` (and `bar()`) is atomic, which means that once `foo()` starts running, the entirety of its code will finish before any of the code in `bar()` can run, or vice versa. This is called "run-to-completion" behavior:

```js
// Chunk 1:
var a = 1;
var b = 2;

// Chunk 2:
function foo() {
	a++;
	b = b * a;
	a = b + 3;
}

// Chunk 3:
function bar() {
	b--;
	a = 8 + b;
	b = a * 2;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

Chunk 1 is synchronous (happens _now_), but chunks 2 and 3 are asynchronous (happen _later_), which means their execution will be separated by a gap of time. Chunks 2 and 3 may happen in either-first order, so there are two possible outcomes for this program:

```js
// Chunk 2 happens first:
a; // 11
b; // 22

// Chunk 3 happens first:
a; // 183
b; // 180
```

- As applied to JavaScript's behavior, this *function-ordering nondeterminism* is the common term "==race condition==", as `foo()` and `bar()` are racing against each other to see which runs first. Specifically, it's a "race condition" because you cannot predict **reliably** how `a` and `b` will turn out. Another example that *may look like* it's having a "race condition", but actually not:

```js
/* `foo()` and `bar()` are two concurrent "processes," and it's nondeterminated in which order they will be fired in. 
* But we've constructed the program so it doesn't matter what order they fire in
* because they act independently and the code will always work correctly, 
* regardless of the ordering, therefore, it's NOT a "race condition".
*/ 

var res = {};

function foo(results) {
	res.foo = results;
}

function bar(results) {
	res.bar = results;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

- To avoid race condition, you can make the output determinated by coordinate ordering interaction:

```js
var a, b;

function foo(x) {
	a = x * 2;
	if (a && b) {
		baz();
	}
}

function bar(y) {
	b = y * 2;
	if (a && b) {
		baz();
	}
}

function baz() {
	console.log( a + b );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

---

### Concurrency
- Concurrency is when two or more "processes" are executing simultaneously over the same period, regardless of whether their individual constituent operations happen _in parallel_ (at the same instant on separate processors or cores) or not. You can think of concurrency then as "process"-level (or task-level) parallelism, as opposed to operation-level parallelism (separate-processor threads).

- From a high-level perspective, they appear to be running _simultaneously_ (even though at any given moment **only one event** is being processed)

- Let's imagine a site that displays a list of status updates (like a social network news feed) that progressively loads as the user scrolls down the list. For a given window of time (a few seconds worth of a user scrolling), let's visualize each independent "process" as a series of events/operations and the interleaving of all these events onto the event loop queue:

```js
// "Process" 1 (`onscroll` events)
// "Process" 2 (Ajax response events)
// "Process 1" and "Process 2" run concurrently (task-level parallel), but their individual events run sequentially on the event loop queue:
onscroll, request 1   <--- Process 1 starts
onscroll, request 2
response 1            <--- Process 2 starts
onscroll, request 3
response 2
response 3
onscroll, request 4
onscroll, request 5
onscroll, request 6
response 4
onscroll, request 7   <--- Process 1 finishes
response 6
response 5
response 7            <--- Process 2 finishes
```


- If a "process" is too heavy, it can be _cooperated_ by breaking into smaller chunks and to allow other "process" interleaving:

```js
var res = [];

// `response(..)` receives array of results from the Ajax call
// Imagine `data` has 10 million records, that can take a while to run (several seconds on a powerful laptop, much longer on a mobile device, etc.)
// While such a "process" is running, nothing else in the page can happen,
// including no other `response(..)` calls, no UI updates, not even user events like scrolling, typing, button clicking, and the like. That's pretty painful.
function response(data) {
	// add onto existing `res` array
	res = res.concat(
		// make a new transformed array with all `data` values doubled
		data.map( function(val){
			return val * 2;
		} )
	);
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );

//---------------------
// So, to make a more cooperatively concurrent system, one that's friendlier and doesn't clog up the event loop queue,
// you can process these results in asynchronous batches, after each one "yielding" back to the event loop to let other waiting events happen: 
var res = [];

function response(data) {
	// let's just do 1000 at a time
	var chunk = data.splice( 0, 1000 );

	// add onto existing `res` array
	res = res.concat(
		// make a new transformed array with all `chunk` values doubled
		chunk.map( function(val){
			return val * 2;
		} )
	);

	// anything left to process?
	if (data.length > 0) {
		// async schedule next batch
		setTimeout( function(){
			response( data );
		}, 0 );
	}
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

---

## Jobs
- At any given moment, only one event can be processed from the queue at a time. While an event is executing, it can directly or indirectly cause one or more subsequent events.

- The `setTimeout(..0)` (hack) for async scheduling, which basically just means "add this function at the end of the current event loop queue". Jobs are kind of like the spirit of the `setTimeout(..0)` hack, but implemented in such a way as to have a much more well-defined and guaranteed ordering: **later, but as soon as possible** (make sure it happens right away before anything else can happen). "Job queue" is a queue hanging off the end of every **tick** in the event loop queue:

```js
// You might expect this to print out `A B C D`
// but instead it would print out `A C D B`,
// because the Jobs happen at the end of the current event loop tick, and the timer fires to schedule for the NEXT event loop tick (IF AVAILABLE!).
console.log( "A" );

setTimeout( function(){
	console.log( "B" );
}, 0 );

// theoretical "Job API"
schedule( function(){
	console.log( "C" );

	schedule( function(){
		console.log( "D" );
	} );
} );
```
---


Related topics:
[[Callbacks]]
[[Promises]]
[[Generators]]
[[Performance]]
[[Advanced Patterns]]