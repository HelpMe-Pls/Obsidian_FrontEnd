- A function's `this` references the *execution __context__* for that call, determined entirely by *__how__ the function was called*. A `this`-aware function can thus have a different **context** each time it's called, which makes it more flexible and reusable (sort of like *having dynamic scope*):

```javascript
// Implicit binding:

var w = {
    name: 'K',

    ask(question) {
        console.log(this.name, question)
    }
}

w.ask("??")   // "K ??"

  
  

//-----------------------

function ask(question) {
    console.log(this.name, question)
}

  

var w1 = {
    name: 'K',

    ask: ask
}

  

var w2 = {
    name: 'L',

    ask: ask
}

  

function otherClass() {
    var myContext = {

        name: 'S'

    }

    ask.call(myContext, "?")   // explicit `this` binding
}

  

otherClass()    // "S ?"

  

// Dynamic binding:

w1.ask("?")     // "K ?"

w2.ask("?")     // "L ?"

  

// `.call()` method treats their first argument as a `this` keyword (i.e. explicit binding):  

ask.call(w2, "explicitly set context.")  // "L explicitly set context."

ask.call(w1, "explicitly set context.")  // "K explicitly set context."

  
  

//---------Hard binding with `.bind() method----------

// You only need to "bind" if the method/function is "`this`-aware"

var w = {
    name: 'K',

    ask(question) {
        console.log(this.name, question)
    }
}

  

// Because of the implementation of `setTimeout()`, its first argument binds `this` to the global context

setTimeout(w.ask, 10, "?")   // undefined ?

  

// Which is why we need to hard bind `this` in this case to give it the context of `w` (i.e. "lexical `this`" behavior):

setTimeout(w.ask.bind(w), 10, "?") // K ?
```


- If we want a flexible dynamic, use `this`. If we want predictability, use closure (lexical scope), i.e. if you find yourself using `.bind()` very often, then best switch to use closure.

- The `this` keyword points at these corresponding context following the order: `new` > `.call()`, `.apply()`, `.bind()` > context object > global context (except strict mode):

```javascript

// context object:

var w = {
    name: 'K',

    ask(question) {
       console.log(this.name, question)
    }
}

w.ask("??")   // "K ??"

  

//---------- global context ----------

var name = 'K'

  

function ask(question) {
    console.log(this.name, question)
}

  

function askAgain(question) {
    "use strict"

    console.log(this.name, question)
}

  

ask("in non-strict mode")     // "K in non-strict mode"

askAgain("in strict mode")    // TypeError

```


- `this` *__doesn't__ mean anything* inside of an arrow function. Therefore, it's the best practice to use arrow function if you want the "lexical `this`" behavior for a function:

```javascript
var this1 = {
    number: 123,

    logFunction: function () { console.log(this.number); },

    logArrow: () => console.log(this.number) // `this` is unrecognized, therefore it’s just like console.log(number)
};

this1.logFunction();        // 123, because the `logFunction` is called from `this1`, so that's where its `this` points to

this1.logArrow();         // undefined, because `number` is undeclared in the scope of `logArrow` function, so it goes up in the lexical scope, in this case, is the global (which has no `number` in it)

  

var w = {
    name: 'K',

    ask(question) {
      // It goes up in the lexical scope and in the same scope of `ask`, it found `name`:
      
      setTimeout(() => console.log(this.name, question), 100)
    }
}

w.ask("??") // "K ??"
```

- How to know ***when*** to use the arrow function: 

![[When to use the arrow function.png]]