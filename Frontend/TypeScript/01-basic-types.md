# TypeScript Basic Types

## Table of Contents
- [Introduction](#introduction)
- [Primitive Types](#primitive-types)
- [Special Types](#special-types)
- [Type Annotations](#type-annotations)
- [Type Inference](#type-inference)
- [Arrays and Tuples](#arrays-and-tuples)
- [Objects](#objects)
- [Functions](#functions)
- [Interview Questions](#interview-questions)

---

## Introduction

TypeScript is a superset of JavaScript that adds static type checking. Understanding basic types is fundamental to writing type-safe code.

### Why TypeScript?
- Catch errors at compile time, not runtime
- Better IDE support with autocomplete and refactoring
- Self-documenting code
- Easier to maintain large codebases

---

## Primitive Types

### string

**String Type Annotation** - Explicitly declare string types for variables. TypeScript enforces type safety, preventing assignment of incompatible types like numbers.

```typescript
let name: string = "Alice";
let greeting: string = 'Hello';
let message: string = `Welcome, ${name}`;

// Error
name = 42; // Type 'number' is not assignable to type 'string'
```

### number

**Number Type** - All numeric values (integers, floats, hex, binary, octal) use the number type. TypeScript treats all numbers as floating-point values.

```typescript
let age: number = 25;
let price: number = 19.99;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;

// All numbers are floating point
let float: number = 3.14;
```

### boolean

**Boolean Type** - Represents true/false values. TypeScript prevents assigning strings like "yes" or numbers like 1/0 to boolean variables.

```typescript
let isActive: boolean = true;
let hasPermission: boolean = false;

// Error
isActive = "yes"; // Type 'string' is not assignable to type 'boolean'
```

### null and undefined

**Null and Undefined with strictNullChecks** - With strict mode enabled, null and undefined are not assignable to other types unless explicitly allowed using union types.

```typescript
let nothing: null = null;
let notDefined: undefined = undefined;

// With strictNullChecks enabled (recommended)
let name: string = null; // Error
let age: number = undefined; // Error

// Correct way
let name: string | null = null; // Union type
let age: number | undefined = undefined;
```

### symbol

**Symbol Type** - Unique, immutable primitive values often used as object keys. Each symbol is guaranteed to be unique, even with the same description.

```typescript
let sym1: symbol = Symbol("key");
let sym2: symbol = Symbol("key");

console.log(sym1 === sym2); // false - symbols are unique
```

### bigint (ES2020+)

**BigInt Type** - Represents arbitrarily large integers beyond Number.MAX_SAFE_INTEGER. bigint and number are separate types and not compatible.

```typescript
let big: bigint = 100n;
let huge: bigint = BigInt(9007199254740991);

// Error
let mixed: bigint = 100; // Type 'number' is not assignable to type 'bigint'
```

---

## Special Types

### any
Disables type checking - avoid when possible!

**any Type** - Opts out of type checking, allowing any operation. Use only when migrating from JavaScript or dealing with truly dynamic data. Avoid in production code.

```typescript
let anything: any = "hello";
anything = 42; // OK
anything = true; // OK
anything.foo.bar(); // No error, but will crash at runtime!

// When to use `any`:
// - Migrating from JavaScript
// - Working with dynamic content
// - Third-party libraries without types
```

### unknown
Safer alternative to `any` - requires type checking before use.

**unknown Type** - Type-safe alternative to any that requires type guards before use. Forces you to narrow the type before performing operations.

```typescript
let value: unknown = "hello";

// Error - must check type first
// let str: string = value;

// Correct - type guard
if (typeof value === "string") {
  let str: string = value; // OK
}

// unknown is assignable to any
let anything: any = value; // OK
```

### void
Function returns nothing.

**void Type** - Indicates a function returns no value (undefined). Used for functions that perform side effects without returning data.

```typescript
function logMessage(message: string): void {
  console.log(message);
  // No return statement, or return undefined
}

let result: void = logMessage("Hello");
console.log(result); // undefined

// void variables can only be null or undefined
let nothing: void = undefined;
```

### never
Function never returns (throws error or infinite loop).

**never Type** - Represents values that never occur. Used for functions that always throw errors, infinite loops, or exhaustive type checking in switch statements.

```typescript
// Function that throws
function throwError(message: string): never {
  throw new Error(message);
}

// Function that never completes
function infiniteLoop(): never {
  while (true) {
    // ...
  }
}

// never is useful in exhaustive checks
type Status = "pending" | "success" | "error";

function handleStatus(status: Status) {
  switch (status) {
    case "pending":
      return "Loading...";
    case "success":
      return "Done!";
    case "error":
      return "Failed!";
    default:
      const exhaustiveCheck: never = status;
      return exhaustiveCheck;
  }
}
```

---

## Type Annotations

Explicitly specify types:

**Type Annotations** - Explicitly declare types for variables, function parameters, and return values to enable compile-time type checking and better IDE support.

```typescript
// Variables
let name: string = "Alice";
let age: number = 25;
let isStudent: boolean = true;

// Function parameters
function greet(name: string, age: number): string {
  return `Hello ${name}, you are ${age} years old`;
}

// Function return type
function add(a: number, b: number): number {
  return a + b;
}

// Object properties
let user: { name: string; age: number } = {
  name: "Alice",
  age: 25
};
```

---

## Type Inference

TypeScript automatically infers types when not explicitly annotated.

**Type Inference** - TypeScript automatically determines types from assigned values. Best practice is to let TypeScript infer obvious types and be explicit only when necessary.

```typescript
// Type is inferred as string
let name = "Alice";
name = 42; // Error

// Type is inferred as number
let count = 0;
count = "zero"; // Error

// Function return type inferred
function add(a: number, b: number) {
  return a + b; // Inferred as number
}

// Best practice: Let TypeScript infer when obvious
let message = "Hello"; // Inferred - good
let age = 25; // Inferred - good

// Be explicit when not obvious
function processData(data: unknown): string {
  // Explicit return type makes intent clear
  return String(data);
}
```

---

## Arrays and Tuples

### Arrays

**Array Type Syntax** - Two equivalent syntaxes for arrays: T[] or Array<T>. Arrays are type-safe collections where all methods enforce the element type.

```typescript
// Array of strings
let names: string[] = ["Alice", "Bob", "Charlie"];
let names2: Array<string> = ["Alice", "Bob"]; // Generic syntax

// Array of numbers
let numbers: number[] = [1, 2, 3, 4, 5];

// Array of mixed types (union)
let mixed: (string | number)[] = [1, "two", 3];

// Array methods are type-safe
names.push("Dave"); // OK
names.push(42); // Error

// Multi-dimensional arrays
let matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6]
];
```

### Tuples
Fixed-length arrays with specific types at each position.

**Tuple Types** - Fixed-length arrays where each position has a specific type. Useful for representing structured data like coordinates, key-value pairs, or function return values.

```typescript
// Tuple: [string, number]
let person: [string, number] = ["Alice", 25];

// Access by index
let name = person[0]; // string
let age = person[1]; // number

// Error
person = [25, "Alice"]; // Wrong order
person = ["Alice", 25, true]; // Wrong length

// Tuple with optional elements
let point: [number, number, number?] = [10, 20]; // OK
point = [10, 20, 30]; // Also OK

// Tuple with rest elements
let tuple: [string, ...number[]] = ["Alice", 1, 2, 3, 4];

// Named tuples (TypeScript 4.0+)
let employee: [name: string, age: number, salary: number] = ["Alice", 25, 50000];
```

### readonly Arrays and Tuples

**readonly Modifier** - Prevents modification of arrays and tuples. Useful for immutable data structures and function parameters that shouldn't be mutated.

```typescript
// readonly array
let numbers: readonly number[] = [1, 2, 3];
numbers.push(4); // Error - readonly
numbers[0] = 10; // Error - readonly

// readonly tuple
let point: readonly [number, number] = [10, 20];
point[0] = 5; // Error - readonly

// Use ReadonlyArray utility type
let names: ReadonlyArray<string> = ["Alice", "Bob"];
names.push("Charlie"); // Error
```

---

## Objects

### Object Type Annotations

**Object Type Literals** - Inline type definitions for objects with optional and readonly properties. Provides type safety for object shape and property access.

```typescript
// Inline type
let user: { name: string; age: number } = {
  name: "Alice",
  age: 25
};

// Optional properties
let person: { name: string; age?: number } = {
  name: "Bob"
  // age is optional
};

// Readonly properties
let config: { readonly apiKey: string; timeout: number } = {
  apiKey: "abc123",
  timeout: 5000
};

config.timeout = 3000; // OK
config.apiKey = "new"; // Error - readonly
```

### Index Signatures
For objects with dynamic keys.

**Index Signatures** - Define types for objects with dynamic property names. Allows any string or number key while enforcing value type consistency.

```typescript
// String keys, number values
let scores: { [key: string]: number } = {
  math: 95,
  english: 87,
  science: 92
};

scores.history = 88; // OK
scores.physics = "A"; // Error

// Number keys
let array: { [index: number]: string } = {
  0: "first",
  1: "second"
};
```

### Object Type

**object Type** - Represents any non-primitive type (objects, arrays, functions). Different from Object (capital O) which includes primitives via boxing.

```typescript
// Accepts any non-primitive value
let obj: object = { name: "Alice" };
obj = [1, 2, 3]; // OK - arrays are objects
obj = () => {}; // OK - functions are objects

obj = "string"; // Error
obj = 42; // Error
obj = null; // Error
```

---

## Functions

### Function Type Annotations

**Function Signatures** - Type annotations for parameters and return values ensure type safety in function calls. Includes support for optional, default, and rest parameters.

```typescript
// Named function
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function
const multiply = (a: number, b: number): number => {
  return a * b;
};

// Optional parameters
function greet(name: string, greeting?: string): string {
  return `${greeting || "Hello"}, ${name}`;
}

greet("Alice"); // "Hello, Alice"
greet("Bob", "Hi"); // "Hi, Bob"

// Default parameters
function createUser(name: string, role: string = "user"): void {
  console.log(`${name} is a ${role}`);
}

createUser("Alice"); // "Alice is a user"
createUser("Bob", "admin"); // "Bob is an admin"

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3, 4); // 10
```

### Function Type Expressions

**Function Types** - Define reusable function signatures using type aliases. Useful for callbacks, event handlers, and higher-order functions.

```typescript
// Function type
type MathOperation = (a: number, b: number) => number;

const add: MathOperation = (a, b) => a + b;
const subtract: MathOperation = (a, b) => a - b;

// Callback types
function processData(data: string, callback: (result: string) => void): void {
  callback(data.toUpperCase());
}

processData("hello", (result) => {
  console.log(result); // "HELLO"
});
```

### void vs undefined

**void vs undefined Return Types** - void allows ignoring return values, while undefined requires explicitly returning undefined. Important distinction for callback functions.

```typescript
// void - return value is ignored
function logMessage(message: string): void {
  console.log(message);
  // Can return undefined or nothing
}

// undefined - must explicitly return undefined
function doNothing(): undefined {
  return undefined; // Must return undefined
}

// Difference in practice
type VoidFunc = () => void;
type UndefinedFunc = () => undefined;

const voidFn: VoidFunc = () => {
  return 123; // OK - return value ignored
};

const undefinedFn: UndefinedFunc = () => {
  return 123; // Error - must return undefined
};
```

---

## Interview Questions

### Q1: What's the difference between `any` and `unknown`?

**Answer:**
- `any`: Disables all type checking. You can do anything with it.
- `unknown`: Type-safe alternative. Must narrow the type before use.

**any vs unknown Comparison** - Demonstrates the safety difference: any bypasses type checking completely, while unknown requires type guards before operations.

```typescript
let anyValue: any = "hello";
anyValue.toUpperCase(); // OK (no type checking)

let unknownValue: unknown = "hello";
unknownValue.toUpperCase(); // Error - must check type first

if (typeof unknownValue === "string") {
  unknownValue.toUpperCase(); // OK
}
```

**When to use:**
- `any`: Migrating code, working with truly dynamic data (use sparingly)
- `unknown`: When type is uncertain but you want type safety

---

### Q2: What's the difference between `void` and `never`?

**Answer:**
- `void`: Function returns undefined or nothing
- `never`: Function never returns (throws error or infinite loop)

**void vs never Comparison** - void functions complete normally and return undefined, while never functions never complete (throw or loop infinitely).

```typescript
function logMessage(): void {
  console.log("Hello");
  // Returns undefined implicitly
}

function throwError(): never {
  throw new Error("Error!");
  // Never reaches end of function
}
```

---

### Q3: How do you type a function that accepts a callback?

**Answer:**

**Callback Function Types** - Two approaches: inline callback types or type aliases. Type aliases improve reusability and readability for complex function signatures.

```typescript
// Method 1: Inline type
function processData(
  data: string,
  callback: (result: string) => void
): void {
  callback(data.toUpperCase());
}

// Method 2: Type alias
type Callback = (result: string) => void;

function processData(data: string, callback: Callback): void {
  callback(data.toUpperCase());
}

// Usage
processData("hello", (result) => {
  console.log(result); // "HELLO"
});
```

---

### Q4: What's the difference between `Array<string>` and `string[]`?

**Answer:** They're identical - just different syntax:

**Array Syntax Comparison** - Both string[] and Array<string> are equivalent. Prefer T[] for simple types, Array<T> for complex generic types.

```typescript
let names1: string[] = ["Alice", "Bob"];
let names2: Array<string> = ["Alice", "Bob"];

// Prefer string[] for simple types
// Use Array<T> for complex types
let complexArray: Array<[string, number]> = [["Alice", 25]];
```

---

### Q5: How do you create a readonly array?

**Answer:**

**readonly Array Methods** - Three ways to create immutable arrays: readonly modifier, ReadonlyArray type, or const assertion. All prevent push, pop, and index assignment.

```typescript
// Method 1: readonly modifier
let numbers: readonly number[] = [1, 2, 3];

// Method 2: ReadonlyArray utility type
let names: ReadonlyArray<string> = ["Alice", "Bob"];

// Method 3: const assertion
let values = [1, 2, 3] as const;

// All prevent modification
numbers.push(4); // Error
numbers[0] = 10; // Error
```

---

## Key Takeaways

1. **TypeScript has 7 primitive types**: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`
2. **Special types**:
   - `any` - disable type checking (avoid!)
   - `unknown` - safer alternative to `any`
   - `void` - function returns nothing
   - `never` - function never returns
3. **Type inference** - Let TypeScript infer when obvious
4. **Arrays** - Use `string[]` or `Array<string>` syntax
5. **Tuples** - Fixed-length arrays with specific types
6. **Functions** - Always type parameters and return values
7. **readonly** - Prevents modification of arrays and objects

---

## Next Steps

- [Interfaces & Types](./02-interfaces-types.md)
- [Generics](./03-generics.md)
- [Type Guards](./05-type-guards.md)

---

[← Back to TypeScript](./README.md)
