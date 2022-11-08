## Spread operator
 - It **deep** copies the data *if it is **not** nested*. For nested data, it deeply copies the topmost data and shallow copies of the nested data:
```ts
const oldObj = {a: {b: 10}, c: 2};

const newObj = {...oldObj};

oldObj.a.b = 2; // It also changes the newObj `b` value as newObj and oldObj's `b` property allocates the same memory address.

oldObj.c = 5; // It changes the oldObj `c` but untouched at the newObj

console.log('oldObj:', oldObj);       // oldObj: { "a": { "b": 2 }, "c": 5 }

console.log('newObj:', newObj);       // newObj: { "a": { "b": 2 }, "c": 2 }
```

- Destructuring and alias:

```js
function bar() {
	return {
		x: 4,
		y: 5,
		z: 6
	};
}

// Destructuring:
var { x, y, z } = bar();

// Alias:
var { x: a, y: b, z: c } = bar();

console.log( a, b, c );	// 4 5 6
console.log( x, y, z );	// 4 5 6
```


## Assignments
- For the ***object*** destructuring form specifically, when leaving off a `var`/`let`/`const` declarator, we had to surround the *whole assignment expression* in `( )`, because otherwise the `{ .. }` on the lefthand side as the first element in the statement is taken to be a block statement instead of an object:

```js
// Map an object to an empty array:
var o1 = { a: 1, b: 2, c: 3 },
	a2 = [];

// Wrap the whole assignment expression in parentheses:
( { a: a2[0], b: a2[1], c: a2[2] } = o1 );

console.log( a2 );    // [1, 2, 3]
```

- Array transformation:

```js
// From an array to an5 object:
var a1 = [ 1, 2, 3 ],
	o2 = {};

[ o2.a, o2.b, o2.c ] = a1;

console.log( o2.a, o2.b, o2.c );   // 1 2 3


// Re-ordering an array:
var a1 = [ 1, 2, 3 ],
	a2 = [];

[ a2[2], a2[0], a2[1] ] = a1;

console.log( a2 );   // [2, 3, 1] 

// Rest:
var a = [2,3,4];
var [ b, ...c ] = a;

console.log( b, c );   // 2 [3, 4]
```

- Restructuring :

```js
var settings = {
	options: {
		remove: true,
		enable: false,
		instance: {}
	},
	log: {
		warn: true,
		error: true
	}
};

var config = {
	options: {
		remove: false,
		instance: null
	}
};

// merge `defaults` into `config`
{
	// destructure (with default value assignments)
	let {
		options: {
			remove = settings.options.remove,
			enable = settings.options.enable,
			instance = settings.options.instance
		} = {},
		log: {
			warn = settings.log.warn,
			error = settings.log.error
		} = {}
	} = config;

	// restructure: take the declared object from the expression above and assign it to `config`
	config = {
		options: { remove, enable, instance },
		log: { warn, error }
	};
}

console.log(config)
```
---

## Array traversing methods
#### .map()
- Returns a ***NEW*** (seperate ref) **array** with its elements based on what the `callbackFn` does:
```ts
const numbers = [1, 2, 3, 4];

const filteredNumbers = numbers.map((num, index) => {
  if (index < 3) {
    return num;
  }
});

// `filteredNumbers` is [1, 2, 3, undefined]
// `numbers` is still [1, 2, 3, 4]

```

- Caution:
	- `map` does ***not*** mutate the original array on which it is called (although `callbackFn`, if invoked, may do so).
	- Don't use `map` if: you're not using the array it returns; and/or you're not returning a value from the callback.
---

#### .filter()
- Returns a *shallow* copy of a portion of the original array, with its element(s) are the return value(s) of the `callbackFn`. If no elements pass the test, an ***empty array*** will be returned.
- Does ***not*** mutate the original array. However, `callbackFn` may do so:
```ts
// Modifying each word
let words = ['spray', 'limit', 'exuberant', 'destruction', 'elite', 'present'];

const modifiedWords = words.filter((word, index) => {
  words[index + 1] += ' extra';
  return word.length < 6;
});

console.log(modifiedWords);
// Notice there are three words below length 6, but since they've been modified one is returned
// ["spray"]

// Appending new words
words = ['spray', 'limit', 'exuberant', 'destruction', 'elite', 'present'];
const appendedWords = words.filter((word, index) => {
  words.push('new');
  return word.length < 6;
})

console.log(appendedWords);
// Only three fits the condition even though the `words` itself now has a lot more words with character length less than 6
// ["spray" ,"limit" ,"elite"]

// Deleting words
words = ['spray', 'limit', 'exuberant', 'destruction', 'elite', 'present'];
const deleteWords = words.filter((word, index) => {
  words.pop();
  return word.length < 6;
})

console.log(deleteWords);
// Notice 'elite' is not even obtained as it's been popped off 'words' before filter can even get there
// ["spray" ,"limit"]

console.log(words)   // ["spray", "limit", "exuberant"]
```
---

#### .find()
- Does ***not*** mutate the original array. However, `callbackFn` may do so.
- Returns the ***first*** element found. If no values satisfy the `callbackFn`, returns `undefined`.
- If you need the **index** of the found element in the array, use `Array.findIndex()` instead (If no elements satisfy the testing function, returns `-1`).
- Nonexistent (`undefined`) and deleted elements ==***are visited***==, and the value passed to the callback is their value when visited:
```ts
// Declare array with no elements at indexes 2, 3, and 4
const array = [0,1,,,,5,6];

// Shows all indexes, not just those with assigned values
array.find((value, index) => {
  console.log('Visited index ', index, ' with value ', value);
});

// Shows all indexes, including deleted
array.find((value, index) => {
  // Delete element 5 on first iteration
  if (index === 0) {
    console.log('Deleting array[5] with value ', array[5]);
    delete array[5];
  }
  // Element 5 is still visited even though deleted
  console.log('Visited index ', index, ' with value ', value);
});
// expected output:
// Visited index 0 with value 0
// Visited index 1 with value 1
// Visited index 2 with value undefined
// Visited index 3 with value undefined
// Visited index 4 with value undefined
// Visited index 5 with value 5
// Visited index 6 with value 6
// Deleting array[5] with value 5
// Visited index 0 with value 0
// Visited index 1 with value 1
// Visited index 2 with value undefined
// Visited index 3 with value undefined
// Visited index 4 with value undefined
// Visited index 5 with value undefined
// Visited index 6 with value 6

```
---

#### .every()
- Does ***not*** mutate the original array. However, `callbackFn` may do so.
- Returns a **Boolean** value.
- Tests whether ***ALL*** elements pass the `callbackFn`:
```ts
const isBelowThreshold = (currentValue) => currentValue < 40;

const array1 = [1, 30, 39, 29, 10, 13];

console.log(array1.every(isBelowThreshold));  // true
```

----
#### .some()
- Like `.every()` returns `true` if ***AT LEAST ONE*** element passes the  `callbackFn`: 
```ts
[2, 5, 8, 1, 4].some((x) => x > 10);  // false
[12, 5, 8, 1, 4].some((x) => x > 10); // true
```

---

#### .includes()
- Like `.some()`, but search for primitive ***value***:
```ts
someArray.includes(searchElement, fromIndex)
// `fromIndex` is an optional argument
// `computedIndex` == array.length + `fromIndex`
// If the `computedIndex` is less than or equal to `0`, the entire array will be searched:
const arr = ['a', 'b', 'c'];

console.log(arr.includes('a')) // true
console.log(arr.includes('b', 69)) // false
console.log(arr.includes('c', 1)) // true
console.log(arr.includes('a', -2))   // false
console.log(arr.includes('b', -2))  // true
console.log(arr.includes('c', 2))  // true
```
---

#### .reduce()
- Does ***not*** mutate the original array. However, `callbackFn` may do so.
- Iterates and executes the "reducer" for all elements in the array.
- The returned value depends on the "reducer":
```ts
// `initialValue` is optional
someArray.reduce(callbackFn, initialValue)

// Inline `callbackFn`:
someArray.reduce((previousValue, currentValue) => { something... }, initialValue )

// If exists `initialValue`, it will be assigned to previousValue for the FIRST time the callback is called and the `currentValue` will be treated as the FIRST VALUE in the array.

// If there’s no `initialValue`, the first value of the array will be assigned to `previousValue` and `currentValue` will be the 2nd value of the array

// The `reduce` method executes { something... } on each element iteration of` the array, from left to right,  and its result is assigned to `previousValue`. Then the `currentValue` will be the param for the next execution of { something... }. The final result of running the reducer across all elements of the array is a SINGLE value (that value could be a number, string, boolean, object, or even a function). Example:

const arr = [21, 18, 42, 40, 69, 63, 34];

const maxNum = arr.reduce((max, value) => (value > max ? value : max), 0);

// maxNum returns 69

const nums = [1, 2, 3, 4];

const reducer = (prev, cur) => prev + cur

console.log(nums.reduce(reducer))  // 10
```

- `reduceRight()` method does the same thing but in reversed order (starts from the end of the array, right to left).
---

### .forEach()
- Does ***not*** mutate the original array. However, `callbackFn` may do so.
- Executes the `callbackFn` once for each array element.
- Nonexistent (`undefined`) and deleted elements ==***are NOT visited***==:
```ts
const logArrayElements = (element, index) => {
  console.log(`a[${index}] = ${element}`);
};

// Notice that index 2 is skipped, since there is no item at
// that position in the array.
[2, 5, , 9].forEach(logArrayElements);
// logs:
// a[0] = 2
// a[1] = 5
// a[3] = 9

-------
// Flatten an array implementation:
const flatten = (arr) => {
  const result = [];
  arr.forEach((i) => {
    if (Array.isArray(i)) {
      result.push(...flatten(i));
    } else {
      result.push(i);
    }
  });
  return result;
}

// Usage:
const nested = [1, 2, 3, [4, 5, [6, 7], 8, 9]];
console.log(flatten(nested)); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```
---

#### .slice()
- Useful when you want to extract or remove a portion of an array.
- Does ***not*** mutate the original array.
- Returns a **new** array (a *shallow* copy portion) of the original array with its element(s) selected from `start` to `end` (`end` is ==***not***== included).
- Returns an **empty** array when:
	- `start` is `>=` array length.
	- `end` is `0`.
	- `start` and `end` are overlapped.
	
```ts
const animals = ['ant', 'bison', 'camel', 'duck', 'elephant'];

// If `start` is `undefined` or `-Infinity`, `slice` starts from the index `0`.
console.log(animals.slice(undefined,2));  // ["ant", "bison"]

// If `start` is >= the array length, an empty array is returned.
console.log(animals.slice(9, 3));    // []
console.log(animals.slice(9));       // []

// If `start` is negative, the absolute of it is how many element(s) TAKEN from the last index (right to left). If that absolute is >= the array length, ALL elements are returned.
console.log(animals.slice(-3));     // ["camel", "duck", "elephant"]
console.log(animals.slice(-3, 4));  // ["camel", "duck"]
console.log(animals.slice(-5));     // ['ant', 'bison', 'camel', 'duck', 'elephant']

// If `end` is `undefined` or >= the array length, `slice` goes through the last element
console.log(animals.slice(3,));     // ["duck", "elephant"]
console.log(animals.slice(1, undefined));   // ["bison", "camel", "duck", "elephant"]
console.log(animals.slice(2, 5));     // ["camel", "duck", "elephant"]
console.log(animals.slice(1, 9));     // ["bison", "camel", "duck", "elephant"]

// If `end` is `0`, returns empty array
console.log(animals.slice(1, 0));          // []
console.log(animals.slice(-3, 0));         // []
console.log(animals.slice(2, -0));         // []
console.log(animals.slice(undefined, 0));  // []

// If `end` is negative, the absolute of it is how many element(s) REMOVED from the last index (right to left)
console.log(animals.slice(0, -1));    // ["ant", "bison", "camel", "duck"]
console.log(animals.slice(1, -3))     // ["bison"]
console.log(animals.slice(1, -5));    // []
console.log(animals.slice(0, -69));   // []
console.log(animals.slice(-3, -1));   // ["camel", "duck"]

// If both `start` and `end` is omitted, returns a fully SHALLOW copied version of the original array
console.log(animals.slice());   // ['ant', 'bison', 'camel', 'duck', 'elephant']

// If `start` and `end` overlapped, returns an empty array
console.log(animals.slice(2, -3));   // []
console.log(animals.slice(3, -4));   // []
-----------
// Proof of shallow copy:
const myHonda = { color: 'red', wheels: 4, engine: { cylinders: 4, size: 2.2 } };
const myCar = [myHonda, 2, 'cherry condition', 'purchased 1997'];

const newCar = myCar.slice(0, 2); // create newCar from myCar.

// myCar = [
//   { color: 'red', wheels: 4, engine: { cylinders: 4, size: 2.2 } },
//   2,
//   'cherry condition',
//   'purchased 1997'
// ]
console.log('myCar = ', myCar); 

// newCar = [{color: 'red', wheels: 4, engine: {cylinders: 4, size: 2.2}}, 2]
console.log('newCar = ', newCar);

// Change the color of myHonda.
myHonda.color = 'purple';
console.log('The new color of my Honda is ', myHonda.color);  // 'purple'

console.log('myCar[0].color = ', myCar[0].color);      // 'purple'
console.log('newCar[0].color = ', newCar[0].color);    // 'purple'
```
---

## Array mutating methods
#### .push()
- Adds one (*or more*) elements to the ***end*** of an array.
- Returns the new **length** of the array.
```ts
const sports = ['soccer', 'baseball'];
const total = sports.push('football', 'swimming');

console.log(sports); // ['soccer', 'baseball', 'football', 'swimming']
console.log(total);  // 4
--------------

// Merging arrays:
const vegetables = ['parsnip', 'potato'];
const moreVegs = ['celery', 'beetroot'];

// Merge the second array into the first one
vegetables.push(...moreVegs);

console.log(vegetables); // ['parsnip', 'potato', 'celery', 'beetroot']
```
---

#### .unshift()
- Like `.push()`, but adds element(s) to the ***beginning*** of the original array:
```ts
const arr = [1, 2]

arr.unshift(0)               // result of the call is 3, which is the new array length
// arr is [0, 1, 2]

arr.unshift(-2, -1)          // the new array length is 5
// arr is [-2, -1, 0, 1, 2]

arr.unshift([-4, -3])        // the new array length is 6
// arr is [[-4, -3], -2, -1, 0, 1, 2]

arr.unshift([-7, -6], [-5])  // the new array length is 8
// arr is [ [-7, -6], [-5], [-4, -3], -2, -1, 0, 1, 2 ]
```
---

#### .pop()
- Removes the **last** element from an array ==and returns ***that element==*** (returns `undefined` if the array is empty).
- This method changes the length of the array and it ***doesn't*** receive any argument.

```ts
const plants = ['broccoli', 'cauliflower', 'cabbage', 'kale', 'tomato'];

console.log(plants.pop());  // 'tomato'
-----------------------

arr = [1, 2, 3, 4];
arr.every((elem, index) => {
  arr.pop()
  console.log(`[${arr}][${index}] -> ${elem}`)
  return elem < 4
})

// Loop runs for 2 iterations only, as the remaining items are `pop()`ed off
// 1st iteration: [1,2,3][0] -> 1
// 2nd iteration: [1,2][1] -> 2
```
---

#### .shift()
- Like `.pop()`, but applied to the ***first*** element in an array.
----

#### .reverse()
- It is what it is.
- Add the `length` property to an object and `.call()` it if you wanna reverse an object:
```ts
const obj = {0: 1, 1: 2, 2: 3, length: 3};
console.log(obj); // {0: 1, 1: 2, 2: 3, length: 3}

Array.prototype.reverse.call(obj); //same syntax for using apply()
console.log(obj); // {0: 3, 1: 2, 2: 1, length: 3}
-----

const array1 = ['one', 'two', 'three'];
console.log('array1:', array1);
// expected output: "array1:" Array ["one", "two", "three"]

const reversed = array1.reverse();
console.log('reversed:', reversed);
// expected output: "reversed:" Array ["three", "two", "one"]

// Careful: reverse is destructive -- it changes the original array.
console.log('array1:', array1);
// expected output: "array1:" Array ["three", "two", "one"]
```
---

#### .splice()
- Useful when you want to add/remove/***replace*** element(s) at (or *from*) a specific index in an array.
- ==***Changes***== the original array.
- Returns an array containing the ***deleted*** element(s). Returns empty array if no elements are removed.
```ts
someArray.splice(start, deleteCount, item1, item2, itemN)
// `start` is similar to `.slice()`, except that if it's >= the array length, `.splice()` will act as `.push()` if the third argument (`item1, item2, itemN`) is passed in.
// `deleteCount` is number of elements in the array to be removed FROM the index of `start` (`start` element is included)
// `deleteCount` MUST be included if the third argument is passed in.
// If `deleteCount` is `undefined`-ish (`0`, negative number, -Infinity, NaN), you should pass in the third argument. Then, `splice()` will act as `.push()` FROM the index of `start`

------------------------
const myItems = ['parrot', 'anemone', 'blue', 'trumpet'];

// Add element(s):
const added = myItems.splice(2, 0, 'shit');  
console.log(added)        // []
console.log(myItems)      // ["parrot", "anemone", "shit", "blue", "trumpet"]
const added1 = myItems.splice(69, 3, 'pussy'); 
console.log(added1)  // []
console.log(myItems) // ["parrot", "anemone", "shit", "blue", "trumpet", "pussy"]
const added2 = myItems.splice(2, undefined, 'cunt'); 
console.log(added2)  // []
console.log(myItems) // ["parrot", "anemone", "cunt", "shit", "blue", "trumpet", "pussy"]

// Remove element(s): 
const removed = myItems.splice(4, 3);
console.log(removed)     // ["blue", "trumpet", "pussy"]
console.log(myItems)     // ["parrot", "anemone", "cunt", "shit"]
const removed1 = myItems.splice(1, 1);
console.log(removed1)    // ["anemone"]
console.log(myItems)     // ["parrot", "cunt", "shit"]

// Replace element(s);
const replaced = myItems.splice(1, 1, 'cock', 'fart')
console.log(replaced)   // ["cunt"]
console.log(myItems)    // ["parrot", "cock", "fart", "shit"]
const replaced1 = myItems.splice(0, 1, 'ass')
console.log(replaced1)  // ["parrot"]
console.log(myItems)    // ["ass", "cock", "fart", "shit"]
```
