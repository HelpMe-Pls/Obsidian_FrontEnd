1. Undefined
2. Null
3. Boolean
4. Number
5. BigInt
6. String
7. Symbol
8. Object: only objects (and subtypes of Object) can have this type, other primitive values can be sorted into one of the types above.

- `typeof`: returns a `string` which represents the type of the **value** that a variable holds.
    - Caution: if you're doing a `typeof` check on an object, make sure it's *not accidentally* a `null` because ==there's a historical bug in JS that returns `"object"` when doing `typeof null`==. Another one is doing `typeof` on an array which also returns `"object"`, so if you want to type check an array, explicitly use `Array.isArray()` instead.

```javascript
var v = Symbol()  

typeof v  // "symbol"

  

typeof null //  "object"

// `void 0` can be used in place of `undefined`
typeof undefined   // "undefined"
typeof void 0      // "undefined"
void 0 === undefined   // true

typeof [3,6,9] // "object"

  

Array.isArray([1, 2, 3]);  // true

Array.isArray({foo: 369});  // false
```


- `NaN`: should be understood as a "**not valid** *number*". It is **falsy**. A triple equal with itself is **always** false. Use the `Number.isNaN()` utility to check if some value is `NaN`. E.g:

```javascript
!!NaN     // false

69 > NaN  // false

  

typeof NaN === Number // false

typeof NaN  // returns "number"

  

// A triple equal with itself is always `false`:

NaN === NaN   // false

  

// Any operation that involves a string and is NOT an addition operation returns `NaN`:

69 - "some random number" === NaN  // false

"foo" / 3     // NaN

  

// Operand of an argument is NaN returns `NaN`:

6 ** NaN  // NaN

9 + NaN   // NaN

  

// Number cannot be parsed returns `NaN`:

parseInt("blabla")    // NaN

Number(undefined)     // NaN

  

// Math operation where the result is not a real number

Math.sqrt(-1)   // NaN

  

// Use the `Number.isNaN()` utility to check if some value is `NaN`:

Number.isNaN("some random string")  // false

Number.isNaN(69 ** "string")        // true

```

  
- Special value `-0`: used in cases where the sign of a number matters (as in direction of some moving things) e.g:

```javascript
function formatTrend(trendRate){
  /**

   *  Object.is() behaves like a quadruple equal:

   *  0 === -0   // true

   *  Object.is(0, -0)   // false  

   * */
  var direction = (trendRate < 0 || Object.is(trendRate, -0)) ? "▼" : "▲"

  return `${direction} ${Math.abs(trendRate)}`
}

formatTrend(-69)    // "▼ 69"

formatTrend(69)     // "▲ 69"

formatTrend(0)      // "▲ 0"

formatTrend(-0)     // "▼ 0"
```
