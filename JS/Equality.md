- Double equals (`==`) and triple equals (`===`) are exactly the same if their operands are **the same type** (in fact, JS performs the `===` in that case):

```javascript
let a = "some string"

let b = `${a}`

  
const c = {

  random: "prop"

}


const d = {

  random: "prop"

}

  
// These are exactly the same:
a == b    // true

a === b   // true

c == d    // false

c === d   // false
```


- The way that triple equals works is that it returns `false` when the operands types **isn't** match (except for the `NaN` and `-0` special values), i.e it **does not** allow *coercion* if the types are **the same** while the double equals does the exact opposite.

- The double equals coerces `null` and `undefined` and returns `true`, which is useful for cases where you need to check empty values:

```javascript
null == undefined   // true

undefined == null   // true
  

let w1 = {topic: null}

let w2 = {}

if (w1.topic == undefined && w2.topic == null)  // run this

```


- The double equals prefers numeric coercion (which may helpful if you already narrowed down the type of your operands):

```javascript
let x = 69

let y = event.target.value  // which is essentially a number in form of a string

  

// The double coerces y to a number then perform the comparison with x:

if (x == y)   // run this ?

  

// If we're using the triple equal then the code would be UNNECESSARILY explicit:

if (+x === +y) // run this ?

```


- The double equals performs the `ToPrimitive()` abstract operation if the operands are not primitive values:

```javascript
var m = 69

var n = [96]

  
// The process of comparing `m` and `n`:

m: 69 == n: '96'

m: 69 == n: 96

m == n

m === n     // false
```


- You should **never** perform an equality check with primitive `boolean`:

```javascript

var v = []

  

// Do this:

if (v)  // run this

  

// DON'T:

// v == true --> '' == true --> 0 === 1 --> false

if (v == true)    // this won't run

if (v === true)   // this won't run

if (v == false)   // this RUNS !!!

```


- **Don't** use the double equal with `0` or any kind of empty string, with non-primitives.

- Double equal is recommended for cases where you already know the type (and their edge cases) of the operands. The opposite is for triple equals.