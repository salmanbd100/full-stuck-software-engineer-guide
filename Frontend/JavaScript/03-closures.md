# Closures

## Concept

A **closure** is a function that has access to variables in its outer (enclosing) lexical scope, even after the outer function has returned. Closures are created every time a function is created, and they "remember" the environment in which they were created.

### Key Points
- Functions have access to variables from their outer scope
- Inner functions keep references to outer scope variables
- Closures enable data privacy and function factories
- Created at function creation time, not invocation time

---

## Example 1: Basic Closure

**Closure Fundamentals** - Demonstrates how inner functions retain access to outer function variables even after the outer function has returned, forming a closure.

```javascript
function outerFunction() {
    const outerVariable = 'I am from outer scope';

    function innerFunction() {
        console.log(outerVariable); // Can access outer variable
    }

    return innerFunction;
}

const closure = outerFunction();
closure(); // Output: "I am from outer scope"

// Even though outerFunction has finished executing,
// innerFunction still has access to outerVariable
```

### How it works:
1. `outerFunction` creates a variable `outerVariable`
2. `innerFunction` is defined inside `outerFunction`
3. `innerFunction` has access to `outerVariable` (lexical scoping)
4. `outerFunction` returns `innerFunction`
5. Even after `outerFunction` completes, the returned function maintains access to `outerVariable`

---

## Example 2: Counter with Closure (Data Privacy)

**Data Encapsulation** - Uses closures to create private variables that can only be accessed and modified through exposed methods, ensuring data privacy.

```javascript
function createCounter() {
    let count = 0; // Private variable

    return {
        increment: function() {
            count++;
            return count;
        },
        decrement: function() {
            count--;
            return count;
        },
        getCount: function() {
            return count;
        }
    };
}

const counter = createCounter();

console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.decrement()); // 1
console.log(counter.getCount());  // 1

// Cannot directly access or modify count
console.log(counter.count); // undefined

// Each counter instance has its own private count
const counter2 = createCounter();
console.log(counter2.increment()); // 1 (independent from counter)
```

### Real-world Use Case:
Data encapsulation - `count` is private and can only be modified through defined methods.

---

## Example 3: Function Factory

**Function Factory with Closures** - Creates specialized functions by capturing different parameter values in separate closures, enabling function customization.

```javascript
function createMultiplier(multiplier) {
    return function(number) {
        return number * multiplier;
    };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15

// Each function "remembers" its own multiplier value
```

### Real-world Use Case:
Creating specialized functions from a generic function template.

---

## Common Pitfalls

### Pitfall 1: Closures in Loops with var

**Loop Variable Capture Problem** - Shows the classic issue where var in loops causes all closures to share the same variable, and solutions using let or IIFE.

```javascript
// WRONG - Common mistake
for (var i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // Prints: 3, 3, 3
    }, 1000);
}

// Why? Because var is function-scoped, all callbacks share
// the same 'i' variable, which is 3 after the loop ends

// SOLUTION 1: Use let (block-scoped)
for (let i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // Prints: 0, 1, 2
    }, 1000);
}

// SOLUTION 2: Use IIFE to create new scope
for (var i = 0; i < 3; i++) {
    (function(j) {
        setTimeout(function() {
            console.log(j); // Prints: 0, 1, 2
        }, 1000);
    })(i);
}
```

### Pitfall 2: Memory Leaks

**Closure Memory Management** - Demonstrates how closures can unintentionally keep large objects in memory, and how to extract only needed values to prevent leaks.

```javascript
// Potential memory leak
function createHugeArray() {
    const hugeArray = new Array(1000000).fill('data');

    return function() {
        console.log('Function created');
        // The closure keeps reference to hugeArray even if not used
    };
}

const func = createHugeArray(); // hugeArray stays in memory

// SOLUTION: Only close over what you need
function createOptimized() {
    const hugeArray = new Array(1000000).fill('data');
    const needed = hugeArray[0]; // Extract only what's needed

    return function() {
        console.log(needed); // Only 'needed' is in closure
        // hugeArray can be garbage collected
    };
}
```

### Pitfall 3: Unexpected Behavior with this

**'this' Binding in Closures** - Shows how arrow functions capture 'this' from outer scope while regular functions have their own 'this' binding.

```javascript
const obj = {
    value: 42,
    getValue: function() {
        // Arrow function captures 'this' from surrounding scope
        const innerArrow = () => {
            console.log(this.value); // 42 - works as expected
        };

        // Regular function has its own 'this'
        const innerRegular = function() {
            console.log(this.value); // undefined (or global value)
        };

        innerArrow();
        innerRegular();
    }
};

obj.getValue();
```

---

## Best Practices

### 1. Use Closures for Data Privacy

**Private State Management** - Implements a bank account with private balance using closures, demonstrating secure state management with controlled access.

```javascript
// Good: Encapsulated state
function createBankAccount(initialBalance) {
    let balance = initialBalance;

    return {
        deposit: (amount) => {
            if (amount > 0) {
                balance += amount;
                return balance;
            }
        },
        withdraw: (amount) => {
            if (amount > 0 && amount <= balance) {
                balance -= amount;
                return balance;
            }
            return 'Insufficient funds';
        },
        getBalance: () => balance
    };
}

const account = createBankAccount(1000);
console.log(account.deposit(500));    // 1500
console.log(account.withdraw(200));   // 1300
console.log(account.getBalance());    // 1300
```

### 2. Module Pattern

**Module Pattern with IIFE** - Uses immediately-invoked function expression with closures to create modules with private variables and public API.

```javascript
const calculator = (function() {
    // Private variables and functions
    let result = 0;

    function log(message) {
        console.log(`Calculator: ${message}`);
    }

    // Public API
    return {
        add: function(a, b) {
            result = a + b;
            log(`${a} + ${b} = ${result}`);
            return result;
        },
        multiply: function(a, b) {
            result = a * b;
            log(`${a} * ${b} = ${result}`);
            return result;
        },
        getResult: function() {
            return result;
        }
    };
})();

calculator.add(5, 3);      // Calculator: 5 + 3 = 8
calculator.multiply(4, 2); // Calculator: 4 * 2 = 8
console.log(calculator.result); // undefined (private)
```

### 3. Memoization with Closures

**Caching Function Results** - Creates a memoization wrapper using closures to cache expensive function results, improving performance through result reuse.

```javascript
function memoize(fn) {
    const cache = {};

    return function(...args) {
        const key = JSON.stringify(args);

        if (cache[key]) {
            console.log('Returning from cache');
            return cache[key];
        }

        console.log('Calculating...');
        const result = fn.apply(this, args);
        cache[key] = result;
        return result;
    };
}

// Example: Expensive fibonacci calculation
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

const memoizedFib = memoize(fibonacci);

console.log(memoizedFib(10)); // Calculating... 55
console.log(memoizedFib(10)); // Returning from cache 55
```

---

## Real-world Scenarios

### Scenario 1: Event Handlers with Dynamic Data

**Dynamic Event Handlers** - Creates event handlers that capture specific data through closures, enabling unique behavior for each handler instance.

```javascript
function createButtonHandler(buttonId, message) {
    return function() {
        console.log(`Button ${buttonId} clicked: ${message}`);
        // Can access buttonId and message even after setup
    };
}

// Setup multiple buttons
const buttons = ['btn1', 'btn2', 'btn3'];
buttons.forEach((id, index) => {
    const handler = createButtonHandler(id, `Message ${index}`);
    // In real code: document.getElementById(id).addEventListener('click', handler);
});
```

### Scenario 2: Partial Application

**Partial Function Application** - Uses closures to pre-fill function arguments, creating specialized versions of generic functions for reusability.

```javascript
function partial(fn, ...fixedArgs) {
    return function(...remainingArgs) {
        return fn.apply(this, [...fixedArgs, ...remainingArgs]);
    };
}

function greet(greeting, name) {
    return `${greeting}, ${name}!`;
}

const sayHello = partial(greet, 'Hello');
const sayHi = partial(greet, 'Hi');

console.log(sayHello('Alice')); // Hello, Alice!
console.log(sayHi('Bob'));      // Hi, Bob!
```

### Scenario 3: React Hooks Pattern

**useState Implementation Concept** - Simplified version showing how React's useState uses closures to maintain state between function calls.

```javascript
// Simplified version of how useState works internally
function createUseState() {
    let state = null;

    function useState(initialValue) {
        if (state === null) {
            state = initialValue;
        }

        function setState(newValue) {
            state = newValue;
            // In React, this would trigger re-render
        }

        return [state, setState];
    }

    return useState;
}

// Usage
const useState = createUseState();
const [count, setCount] = useState(0);
console.log(count); // 0
setCount(5);
const [newCount] = useState(); // Gets current state
console.log(newCount); // 5
```

---

## External Resources

- [MDN: Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
- [JavaScript.info: Closure](https://javascript.info/closure)
- [You Don't Know JS: Scope & Closures](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/README.md)

---

[← Back to JavaScript](./README.md) | [Next: This Keyword →](./04-this-keyword.md)
