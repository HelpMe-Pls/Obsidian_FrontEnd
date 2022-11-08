- The callback function wraps or encapsulates the _continuation_ of the program.

- The first major deficiency to articulate about callbacks: they express asynchrony in code in ways our brains have to fight just to keep in sync with (try to mentally execute this piece of code):

```js
listen( "click", handler );

function handler() {
	setTimeout( request, 500 );
}

function request(){
	ajax( "http://some.url.1", response );
}

function response(text){
	if (text == "hello") {
		handler();
	}
	else if (text == "world") {
		request();
	}
}
```

---

### Trust issues
- Besides "hard-to-track" invocations, there's also "**inversion of control**," where you take part of your program and give over control of its execution to *another third party* to invoke the _continuation_ of your program. There's an unspoken "contract" that exists between your code and the 3rd-party utility -- a set of things you expect to be maintained:

```js
// What happens if under some unforeseen conditions, the API would retry the provided callback once per second, for five seconds, before failing with a timeout
// `chargeCreditCard()` would be 'accidentally' executed 5 times :))
analytics.trackPurchase( purchaseData, function(){
		chargeCreditCard();
		displayThankyouPage();
} );
```

Many possible things that could go wrong with them calling your callback. Here's roughly the list you come up with of ways the analytics API could misbehave:
-   Call the callback too early (before it's been tracked):
```js
// Will this code print `0` (sync callback invocation) or `1` (async callback invocation)? Depends... on the conditions.
function result(data) {
	console.log( a );
}

var a = 0;

ajax( "..pre-cached-url..", result );
a++;
```
-   Call the callback too late (or never)
-   Call the callback too few or too many times 
-   Fail to pass along any necessary environment/parameters to your callback
-   Swallow any errors/exceptions that may happen
-   ...

That should feel like a troubling list, because it is. You're probably slowly starting to realize that you're going to have to invent an awful lot of ad hoc logic **in each and every single callback** that's passed to an utility you're not positive you can trust.

- If you have code that uses callbacks, especially but not exclusively with third-party utilities, and you're not already applying some sort of mitigation logic for all these _inversion of control_ trust issues, your code _has_ bugs in it right now even though they may not have bitten you yet. Latent bugs are still bugs.
