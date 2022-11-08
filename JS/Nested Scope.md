- Function _declaration_ is a function definition with **nothing else** precedes it (i.e. it only starts with the `function` keyword).

- Function _expression_ is a function declaration with **anything** precedes it. Function expression nests their own scope and is created when the execution reaches it and is usable only from that moment:

```javascript

// in global scope:

function fnDeclaration(){  /* do something...*/ }

  

// fnExpression is in global scope while otherFn() is one level lower (i.e. in its own scope):

var fnExpression = function otherFn(){  /* do something... */ }

  

// It is a good practice to have the same names in function expression:

const fnExpr = function fnExpr(){ /* do something... */ }

```

- Named function declaration > Named function expression > Anonymous function expression.

- The IIFE (Immediately Invoked Function Expression) pattern is often used to make your code to follow the PoLP (Principle of Least Privilege) e.g. to avoid naming collisions:

```javascript
var name = 'K'

var name = 'L'

console.log(name) // 'L'

console.log(name) // naming collision, the output is still 'L'

// ------------------------------

  

// To fix it (i.e. you still want to use the same variable `name` in the same scope

// without having to deal with naming collision), use the IIFE pattern:

var name = 'K';

  
// Use IIFE to create a one-time scope just for that variable:

(function anotherName(){

    var name = 'L'

    console.log(name) // 'L'

})()


// Or use block scoping with the `let` keyword:

{
    let name = 'M'

    console.log(name) // 'M'
}


console.log(name)   // 'K'

// In other cases, IIFE can also be used to make your intentions clearer when someone else read your code:

// So instead of this:

var user

try {

    user = fetchUser(1)

}
catch (err) {

    user = 'K'

}

  

// Do this:

var user = (function getUser(){

    try {

        return fetchUser(1)

    }

    catch (err) {

        return 'K'

    }

})()
```

---

### `var` vs `let`
- `var` is used in a bigger scope than `let` and can survive ***block*** scopes where `let` can't. If you see `let`, it tells you that you're dealing with a *localized declaration*. If you see `var`, it tells you that you're dealing with a function-wide declaration: 

```javascript
function getStudents(data) {
    var studentRecords = [];

    for (let record of data.records) {
        let id = `student-${ record.id }`;
        studentRecords.push({
            id,
            record.name
        });
    }

    return studentRecords;
}

//--------------------------
function commitAction() {
    do {
        let result = commit();
        var done = result && result.code == 1;
    } while (!done);
}

//--------------------------------
// `let` wouldn't work in this case:

function lookupRecord(query) {
  // prefer initial declarations to always be as close as possible (ideally, same line) to the first usage of the variable
  // `id` is in the same memory slot (because of lexical scope) eventhough it's redeclared
  // think of `var` more like a declarative annotation that's reminding you, each usage, where the variable comes from, no matter where I am in the function:
    try {
        var id = getRecord(query)
        // imagine a very long piece of code here...
    }
    
    catch (err) {
	    // a remind that this variable belongs to this function's scope:
        var id = -1
    }
    return id
}

//--------`var` vs `let`-----------
for (var i = 0; i < 3; i++) {
    setTimeout(()=> console.log(`i: ${ i }`), 1000)
}
// i: 3
// i: 3
// i: 3

for (let i = 0; i < 3; i++) {
    setTimeout(()=> console.log(`i: ${ i }`), 1000)
}
// i: 0
// i: 1
// i: 2
```


- Use `let` when you need to explicitly narrow down the scope of your variables, and it's a best practice to put it on the first lines of the block:

```javascript
function formatStr(str) {

    { let prefix, rest;

          prefix = str.slice(0,3)

          rest = str.slice(3)

          str = prefix.toUpperCase() + rest
    }

    if (/^FOO:/test(str)) return str;

    return str.slice(4)
}
```
