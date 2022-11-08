- `toString()`: a method which returns a string representing the object. The `String()` function also does the same thing:

```javascript
let a = false

a.toString()  // 'false'

  

a = 96

a.toString()  // '96'

  

String(69)    // '69'

String(-0)    // 0

String({})    // '[object Object]'

  

// To detect an object's class:

Object.prototype.toString.call({some: 'thing'});    // [object Object]

Object.prototype.toString.call(new Date);           // [object Date]

Object.prototype.toString.call(new String);         // [object String]

```


- `Number()`: in most cases, it's used as a function: `Number(value)` converts a string or other value to the `Number` type. If the value can't be converted, it returns `NaN`. An alternative is to use the unary plus operator (`+`) which is the fastest and preferred way of converting something into a number, because it does not perform any other operations on the number.

```javascript
+true       // 1

+false      // 0

+null       // 0

+undefined  // NaN

+'string'   // NaN

+96         // 96

  

Number("")            // 0

Number(" \t \n ")     // 0

Number('69') === 69   // true

Number(null)          // 0

Number(undefined)     // NaN

  

// For arrays and objects, it performs toString() first

// then coerces the value into a number:

Number({a: "b"})  // NaN

Number([null])    // 0

Number([69])      // 69

Number([6, 9])    // NaN
```


- `Boolean()`: object wrapper for a boolean value. Most typically used as a function (or as the `!!` operator) to convert a non-boolean value to a boolean value.

  - Be careful: Do not confuse the **primitive** `Boolean` values `true` and `false` with the `true` and `false` values of the `Boolean` **object**. When `Boolean` is called as a constructor (with `new`), it creates a `Boolean` **object**, which is **not** a primitive. If you specify *any* object, including a `Boolean` object whose value is `false` (as the initial value of a `Boolean` object), the new `Boolean` object has a value of `true`. The implicit coercion of a primitive `Boolean` to a `Number` is also a source of confusion.

```js
const a = [];       !!a   // true

const b = {};       Boolean(b)  // true

const c = "";       !!c // false

  
// Any object of which the value is not undefined or null
// (including a Boolean object whose value is false), evaluates to true:
Boolean(undefined)    // false

  
new Boolean(undefined)  // Boolean {false}

const f = new Boolean(null)

!!f   // true

  

const x = new Boolean(false);

if (x) {

  // this code is executed

}

  

// supposed to be `true` but it's actually `false`:
3 > 2 > 1   // false

(3 > 2) > 1  

true > 1    

1 > 1       // false  
```
