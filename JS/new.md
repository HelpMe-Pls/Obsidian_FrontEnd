
- The `new` keyword: is used to initialize an object (or a subtype of it): `Object()`, `Array()`, `Function()`, `Date()`, `RegExp()`, `Error()` following these step:
1. Create a brand new empty object 
2. Link that object to an  `object.prototype` (depends what follows the `new`)
3. Call function with `this` set to the new object 
4. If function does not return an object, assume return of `this`

```javascript
function ask(question) {
    console.log(this.name, question)
}


// undefined 'Using the `new` keyword'
var newEmptyObject = new ask("Using the `new` keyword")
```