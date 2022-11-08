- Modules **encapsulate** data and behavior (methods) together. The state (data) of a module is held by its methods via closure. It's the idea to take some behavior and the data which that behavior operates on, then encapsulate it into a data structure, hide what we don't need to show and expose only the minimal necessary API (PoLP):

```javascript
var workshop = (function Module(teacher){
    var publicAPI = {ask, }
    return publicAPI

    function ask(question){
        console.log(question, ",", teacher)
    }
})('K')

  
workshop.ask("Yep it's a module")   // "Yep it's a module, K"

workshop.teacher     // `undefined`, because it's private

// If you want `teacher` as a module, add it to the return value:

// var publicAPI = {ask, teacher}
```

- ES6 modules (syntactic sugar for modules) are files that has the `.mjs` extension. Exported code is public and the rest is private:

```javascript
var name = 'K'  // private

export function ask(question){    // public
    console.log(question, ",", name)
}
```