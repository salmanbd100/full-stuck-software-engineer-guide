# Data Types & Variables

## Concept

JavaScript has **dynamic typing**, meaning variables can hold values of any type without explicit type declaration. Understanding data types and how JavaScript handles them is fundamental for writing reliable code.

### Key Points
- Two categories: Primitives and Reference types
- Primitives are immutable, stored by value
- Objects are mutable, stored by reference
- Type coercion can lead to unexpected behavior
- Variables: var (function-scoped), let & const (block-scoped)

---

## Example 1: Primitive vs Reference Types

```javascript
// PRIMITIVE TYPES (stored by value)
// string, number, boolean, undefined, null, symbol, bigint

let a = 10;
let b = a;  // Copy value
b = 20;
console.log(a); // 10 (unchanged)
console.log(b); // 20

// REFERENCE TYPES (stored by reference)
// objects, arrays, functions

let obj1 = { name: 'Alice' };
let obj2 = obj1;  // Copy reference, not value
obj2.name = 'Bob';
console.log(obj1.name); // 'Bob' (changed!)
console.log(obj2.name); // 'Bob'

// Arrays are objects
let arr1 = [1, 2, 3];
let arr2 = arr1;
arr2.push(4);
console.log(arr1); // [1, 2, 3, 4] (changed!)
console.log(arr2); // [1, 2, 3, 4]
```

---

## Example 2: Type Coercion

```javascript
// Implicit type coercion
console.log('5' + 3);      // '53' (number to string)
console.log('5' - 3);      // 2 (string to number)
console.log('5' * '2');    // 10 (both to numbers)
console.log(true + 1);     // 2 (true = 1)
console.log(false + 1);    // 1 (false = 0)

// Comparison coercion
console.log(5 == '5');     // true (loose equality, coerces)
console.log(5 === '5');    // false (strict equality, no coercion)
console.log(null == undefined);  // true
console.log(null === undefined); // false

// Falsy values
console.log(Boolean(0));        // false
console.log(Boolean(''));       // false
console.log(Boolean(null));     // false
console.log(Boolean(undefined)); // false
console.log(Boolean(NaN));      // false
console.log(Boolean(false));    // false

// Everything else is truthy
console.log(Boolean('0'));      // true
console.log(Boolean('false'));  // true
console.log(Boolean([]));       // true
console.log(Boolean({}));       // true
```

---

## Example 3: Variable Declarations

```javascript
// VAR (function-scoped, hoisted)
function varExample() {
    console.log(x); // undefined (hoisted)
    var x = 10;

    if (true) {
        var x = 20; // Same variable!
    }
    console.log(x); // 20
}

// LET (block-scoped)
function letExample() {
    // console.log(y); // ReferenceError (temporal dead zone)
    let y = 10;

    if (true) {
        let y = 20; // Different variable
        console.log(y); // 20
    }
    console.log(y); // 10
}

// CONST (block-scoped, cannot reassign)
const PI = 3.14159;
// PI = 3.14; // TypeError

// But can mutate objects/arrays
const person = { name: 'Alice' };
person.name = 'Bob'; // OK
person.age = 30;     // OK
// person = {};      // TypeError

const arr = [1, 2, 3];
arr.push(4);  // OK
// arr = [];  // TypeError
```

---

## Common Pitfalls

### Pitfall 1: Type Coercion Surprises

```javascript
// Array to string coercion
console.log([1, 2] + [3, 4]); // '1,23,4'

// Object coercion
console.log({} + []);  // '[object Object]'
console.log([] + {});  // '[object Object]'

// Comparison gotchas
console.log([] == ![]);    // true (complex coercion)
console.log('' == 0);      // true
console.log('0' == 0);     // true
console.log('0' == false); // true

// SOLUTION: Always use strict equality
console.log([] === ![]);   // false
console.log('' === 0);     // false
```

### Pitfall 2: Reference Type Mutations

```javascript
// Unexpected mutation
function addItem(arr, item) {
    arr.push(item);
    return arr;
}

const original = [1, 2, 3];
const modified = addItem(original, 4);
console.log(original); // [1, 2, 3, 4] - Mutated!

// SOLUTION: Create copy
function addItemSafe(arr, item) {
    return [...arr, item]; // Spread operator
}

const original2 = [1, 2, 3];
const modified2 = addItemSafe(original2, 4);
console.log(original2); // [1, 2, 3] - Unchanged
console.log(modified2); // [1, 2, 3, 4]
```

### Pitfall 3: var Hoisting Issues

```javascript
// Problem with var in loops
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}
// Prints: 3, 3, 3 (var is function-scoped)

// SOLUTION: Use let
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}
// Prints: 0, 1, 2 (let is block-scoped)
```

---

## Best Practices

### 1. Use const by default, let when needed, avoid var

```javascript
// Good
const MAX_SIZE = 100;
const user = { name: 'Alice' };

let counter = 0;
counter++;

// Avoid
var x = 10; // Don't use var in modern JavaScript
```

### 2. Always use strict equality (===)

```javascript
// Good
if (value === 0) { }
if (user !== null) { }

// Avoid
if (value == 0) { }
if (user != null) { }

// Exception: checking for null or undefined
if (value == null) { } // Checks both null and undefined
```

### 3. Be explicit with type conversions

```javascript
// Good - explicit conversion
const num = Number(str);
const str2 = String(num);
const bool = Boolean(value);

// Avoid - implicit coercion
const num2 = +str;
const str3 = num + '';
const bool2 = !!value;
```

---

## Real-world Scenarios

### Scenario 1: Deep Cloning Objects

```javascript
// Shallow copy (only top level)
const original = { a: 1, b: { c: 2 } };
const shallow = { ...original };
shallow.b.c = 3;
console.log(original.b.c); // 3 (nested object still shared!)

// Deep copy solutions
// 1. JSON method (limited)
const deep1 = JSON.parse(JSON.stringify(original));

// 2. structuredClone (modern browsers)
const deep2 = structuredClone(original);

// 3. Recursive function
function deepClone(obj) {
    if (obj === null || typeof obj !== 'object') return obj;
    if (Array.isArray(obj)) return obj.map(deepClone);

    const cloned = {};
    for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
            cloned[key] = deepClone(obj[key]);
        }
    }
    return cloned;
}
```

### Scenario 2: Type Checking

```javascript
function processValue(value) {
    // Check primitive types
    if (typeof value === 'string') {
        return value.toUpperCase();
    }
    if (typeof value === 'number') {
        return value * 2;
    }
    if (typeof value === 'boolean') {
        return !value;
    }

    // Check for null (typeof null === 'object')
    if (value === null) {
        return 'Value is null';
    }

    // Check for array
    if (Array.isArray(value)) {
        return value.length;
    }

    // Check for object
    if (typeof value === 'object') {
        return Object.keys(value).length;
    }
}
```

### Scenario 3: Immutable Updates

```javascript
// State update pattern (React-style)
const state = {
    user: { name: 'Alice', age: 30 },
    settings: { theme: 'dark' }
};

// Update nested property immutably
const newState = {
    ...state,
    user: {
        ...state.user,
        age: 31
    }
};

// Array immutable operations
const numbers = [1, 2, 3, 4, 5];

// Add item
const added = [...numbers, 6];

// Remove item
const removed = numbers.filter(n => n !== 3);

// Update item
const updated = numbers.map(n => n === 3 ? 30 : n);
```

---

## External Resources

- [MDN: JavaScript data types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures)
- [MDN: var, let, const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements)
- [JavaScript.info: Data types](https://javascript.info/types)
- [Equality comparisons](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness)

---

[← Back to JavaScript](./README.md) | [Next: Functions & Scope →](./02-functions-scope.md)
