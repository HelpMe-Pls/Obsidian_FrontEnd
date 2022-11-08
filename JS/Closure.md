- Closure is a *function **instance*** which "remembers" (able to access) its lexical **scope** even when it's executed **outside** that lexical scope (like a preservation of the *linkage* back to the original lexical scope where it was defined _no matter where_ it executes). E.g:

```javascript
function ask(question) {

  // At the time the callback `waitASec` is executed, the `ask` function has already finished, i.e. the `question` variable "should've been" garbage-collected

  // But because of "closure", the `waitASec` function "closes over" its lexical scope (in this case: the variable `question`)

  // i.e. its execution (which is outside of its lexical scope: 1s later than the execution of `ask`)

  // has access to its lexical scope (the variable `question`)

    setTimeout(function waitASec(){

        console.log(question)

    }, 1000)
}


ask("What is closure ?")    // print "what is closure ?" after 1 sec.

  
//-------------
// The linkage of closure may make your code behave unexpectedly (the common `var` vs `let` misuse):

for (var i = 1; i <=3; i++) {
    setTimeout(function(){

        console.log(`i: ${i}`)     // After the for loop has finished, `i` === 4

    }, i*1000)
}   // print "i: 4" every 1s for 3 times


// To fix it, use the `let` keyword to define a new independent variable
// (which doesn't has a linkage back to the original lexical scope) everytime the loop runs:

for (let i = 1; i <=3; i++) {
    setTimeout(function(){

        console.log(`i: ${i}`)    

    }, i*1000)
}  
```

- Be careful about the fact that the parameter *list* is its own scope if any of the parameters are **non-simple**:

```js
// There's a distinction between the lexical scope `var id` inside of `whatsTheDealHere` and the "closured" `id` param in this case: 
function whatsTheDealHere(id, defaultID = () => id) {
    var id;

    console.log(`local variable 'id': ${ id }`);
    console.log(
        `parameter 'id' (closure): ${ defaultID() }`
    );

    console.log("reassigning 'id' to 5");
    id = 5;

    console.log(`local variable 'id': ${ id }`);
    console.log(
        `parameter 'id' (closure): ${ defaultID() }`
    );
}

whatsTheDealHere(3);
// local variable 'id': 3   <--- This one is actually from the closure of `defaultID`
// parameter 'id' (closure): 3
// reassigning 'id' to 5  <--- After this line, the "local" `id` gets its value
// local variable 'id': 5
// parameter 'id' (closure): 3
```