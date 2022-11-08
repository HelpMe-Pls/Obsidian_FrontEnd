- Compile time: this phase is the where the lexical scope - lexical bindings being created and perceived as a "plan" of execution for the code, therefore, what it returns is the "executable" code (which contains all the *declaration/definition* of the identifiers).

- Run time: after recognizing all of the declarations, the next phase where it runs the "executable" code, i.e. references/calls the identifiers from the "plan" and move up to the outer scope if it didn't find that reference. Eventually, if it still didn't find that reference, it'll throw the "Reference Error" (there's no such identifier - the identifier is undeclared) or "Type Error" (you're doing something illegal with the identifier, e.g. calling a non-function identifier, accessing a property on a null/undefined thing,...)

- Be careful: if we use/assign a value to a **undeclared** variable **inside** of a function, the compiler will move up the outer scope and *auto* declare it ("Strict Mode" - `'use strict'` will prevent this from happening), which is prone to bugs later on. So remember to always _**declare before** using/calling_ an identifier.

- `undefined` means there's a variable but it has no value (yet), while "undeclared" means that variable has **never existed** in any scope that we have access to (which results in "Reference Error" in "Strict Mode").

- Dynamic scope is the kind of scope which is determined based on the conditions at run time (i.e. a function's references to its variables are depended upon where that function is called from) and it is **not exist** in Javascript (how `this` works is the closest you can get to it but still, it is not actually dynamic scope). E.g:

```javascript
var name = 'K'

function ask(question){

    // In lexical scope, the `name` below would be 'K'

    // But in dynamic scope, this function is called in the `otherClass` function,

    // so that's where the `name` get evaluated (to be 'L')

    console.log(name, question)

}


function otherClass(){

    var name = 'L'

  

    ask('Why ?')

}

otherClass()

```

- When a variable is declared as `const`, it means that the variable **can't be *reassign*** (**not** can't be mutated: _value immutability_ **is not** _assignment immutability_). The `const` keyword requires a variable to be initialized, so omitting an assignment from the declaration results in a `SyntaxError.
   It is best used to declare immutable primitive values (i.e. used as placeholders):

```javascript
var someone = 'S'

someone = 'D' // legit


const anotherOne = someone

anotherOne = 'F'  // TypeError


const empty;   // SyntaxError


const names = ['R', 'G']

names[1] = 'P'    // still legit

  

// Best cases for `const`:

const MAGIC_NUMBER = 69

const ENV_STRING = 'some-random-characters'

const API_URL = 'https://example.com'

const PAYLOAD = true
```