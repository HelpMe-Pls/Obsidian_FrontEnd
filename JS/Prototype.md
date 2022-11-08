- Javascript's `class` is a form of syntactic sugar for `prototype` (i.e. `class` is actually a subset of `prototype`):

```javascript
// Prototype:
function Workshop(teacher) {
    this.teacher = teacher
}

Workshop.prototype.ask = function (question) {
    console.log(this.teacher, question)
}

  

var deepJS = new Workshop("K")

var reactJS = new Workshop("S")

  

deepJS.ask("say something ?")   // "K say something ?"

reactJS.ask("say what ?")       // "S say swhat ?"

  

//-------------- Class --------------

class Workshop {
    constructor(teacher) {
        this.teacher = teacher
    }

    ask(question) {
        console.log(this.teacher, question)
    }  
}

  

var deepJS = new Workshop("K")

var reactJS = new Workshop("S")


deepJS.ask("say something ?")   // "K say something ?"

reactJS.ask("say what ?")       // "S say swhat ?"
```


- Objects (instances of `class`) are built by "constructor calls" (via `new`). A "constructor call" makes an object **linked to** its own `prototype`. Arrow functions don't have their `prototype`.

- You can change the "prototype chain" by using `Object.create()` or using the "dunder proto" `__proto__`:

```javascript
function Workshop(teacher) {
    this.teacher = teacher
}

// Add an `ask` method to `Workshop`:
Workshop.prototype.ask = function (question) {
    console.log(this.teacher, question)
}

  
// `AnotherWorkshop` now has a `teacher` "constructor"
function AnotherWorkshop(teacher) {
	// `call()` allows for a function/method belonging to one object to be assigned and called for a different object.
    Workshop.call(this, teacher)
}

  

// Same thing as `AnotherWorkshop.prototype.__proto__ = Workshop.prototype`:
AnotherWorkshop.prototype = Object.create(Workshop.prototype)

  
// `AnotherWorkshop` now "inherited" the `ask` method from `Workshop`:
AnotherWorkshop.prototype.speakUp = function (msg) {
    this.ask(msg.toUpperCase())
}

  

var JSRecentParts = new AnotherWorkshop("K")

JSRecentParts.speakUp("this is kinda like inheritance")

// "K THIS IS KINDA LIKE INHERITANCE"
```