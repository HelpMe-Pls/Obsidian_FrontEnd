- If you have a method that has the same name in a child class as in its parent, you can refer to parent from the child by using the `super.`:

```javascript
class Workshop {
    constructor(teacher) {
        this.teacher = teacher
    }

    ask(question) {
        console.log(this.teacher, question)
    }  
}

  

class AnotherWorkshop extends Workshop {
    ask(msg) {
        super.ask(msg.toUpperCase())
    }
}

var JSRecentParts = new AnotherWorkshop("K")

JSRecentParts.speakUp("using super")      // "K USING SUPER"

```
