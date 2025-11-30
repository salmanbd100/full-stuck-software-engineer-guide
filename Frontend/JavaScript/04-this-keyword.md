# The 'this' Keyword

Understanding `this` is crucial for JavaScript interviews. The value of `this` depends on how a function is called, not where it's defined.

## 📚 Core Concepts

### 1. The Four Binding Rules

JavaScript determines `this` using these rules (in priority order):

1. **new binding** (Constructor call)
2. **Explicit binding** (call, apply, bind)
3. **Implicit binding** (Method call)
4. **Default binding** (Fallback)

### 2. Default Binding

When function is called standalone, `this` refers to global object (or `undefined` in strict mode).

```javascript
function showThis() {
    console.log(this);
}

showThis(); // window (in browser) or global (in Node.js)

// Strict mode
'use strict';
function showThisStrict() {
    console.log(this);
}

showThisStrict(); // undefined
```

### 3. Implicit Binding

When function is called as object method, `this` refers to the object.

```javascript
const user = {
    name: 'Alice',
    greet: function() {
        console.log(`Hello, I'm ${this.name}`);
    }
};

user.greet(); // "Hello, I'm Alice" (this = user)

// Losing implicit binding
const greetFunc = user.greet;
greetFunc(); // "Hello, I'm undefined" (this = window/undefined)
```

**Nested Objects**
```javascript
const company = {
    name: 'TechCorp',
    department: {
        name: 'Engineering',
        show: function() {
            console.log(this.name); // Only looks at immediate parent
        }
    }
};

company.department.show(); // "Engineering" (not "TechCorp")
```

### 4. Explicit Binding

Using `call`, `apply`, or `bind` to explicitly set `this`.

**call()**
```javascript
function greet(greeting, punctuation) {
    console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const user = { name: 'Alice' };

greet.call(user, 'Hello', '!');
// "Hello, I'm Alice!"
// First arg: this value
// Remaining args: function arguments
```

**apply()**
```javascript
function greet(greeting, punctuation) {
    console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const user = { name: 'Bob' };

greet.apply(user, ['Hi', '.']);
// "Hi, I'm Bob."
// First arg: this value
// Second arg: array of function arguments
```

**bind()**
```javascript
function greet(greeting) {
    console.log(`${greeting}, I'm ${this.name}`);
}

const user = { name: 'Charlie' };

const boundGreet = greet.bind(user, 'Hey');
// Returns new function with this permanently bound

boundGreet(); // "Hey, I'm Charlie"

// Common use case: Event handlers
const button = {
    text: 'Click me',
    click: function() {
        console.log(`Button text: ${this.text}`);
    }
};

// Without bind, 'this' would be the DOM element
// document.addEventListener('click', button.click); // Wrong
document.addEventListener('click', button.click.bind(button)); // Correct
```

### 5. new Binding

When function is called with `new`, a new object is created and `this` refers to it.

```javascript
function User(name, age) {
    this.name = name;
    this.age = age;
    this.greet = function() {
        console.log(`I'm ${this.name}, ${this.age} years old`);
    };
}

const alice = new User('Alice', 25);
alice.greet(); // "I'm Alice, 25 years old"

// What 'new' does:
// 1. Creates empty object
// 2. Sets prototype
// 3. Binds 'this' to new object
// 4. Returns the object (unless function returns object explicitly)
```

### 6. Arrow Functions and 'this'

Arrow functions DON'T have their own `this` - they inherit from enclosing scope (lexical `this`).

```javascript
const user = {
    name: 'Alice',
    hobbies: ['reading', 'coding'],

    // Regular function
    showHobbiesRegular: function() {
        this.hobbies.forEach(function(hobby) {
            // 'this' is undefined (or window)
            console.log(`${this.name} likes ${hobby}`);
        });
    },

    // Arrow function
    showHobbiesArrow: function() {
        this.hobbies.forEach(hobby => {
            // 'this' inherited from showHobbiesArrow
            console.log(`${this.name} likes ${hobby}`);
        });
    }
};

// user.showHobbiesRegular(); // Error
user.showHobbiesArrow();
// Alice likes reading
// Alice likes coding
```

**Arrow Functions Can't Be Bound**
```javascript
const user = { name: 'Alice' };

const greet = () => {
    console.log(`Hello, ${this.name}`);
};

// call, apply, bind have NO effect on arrow functions
greet.call(user); // "Hello, undefined" (this from outer scope)
```

**When NOT to Use Arrow Functions**
```javascript
const obj = {
    value: 42,

    // Wrong: arrow function as method
    getValue: () => {
        return this.value; // 'this' is NOT obj
    },

    // Correct: regular function
    getValueCorrect: function() {
        return this.value;
    }
};

console.log(obj.getValue()); // undefined
console.log(obj.getValueCorrect()); // 42
```

## 🎯 Common Interview Questions

### Q1: What will this code output?

```javascript
const obj = {
    name: 'Object',
    getName: function() {
        return this.name;
    }
};

const getName = obj.getName;
console.log(getName()); // ?
```

**Answer:** `undefined` (or error in strict mode)
**Reason:** Lost implicit binding. `getName` is called standalone, so `this` is global/undefined.

**Fix:**
```javascript
const getName = obj.getName.bind(obj);
console.log(getName()); // "Object"

// Or
console.log(obj.getName()); // "Object"
```

### Q2: Fix this code

```javascript
const timer = {
    seconds: 0,
    start: function() {
        setInterval(function() {
            this.seconds++;
            console.log(this.seconds);
        }, 1000);
    }
};

timer.start(); // NaN, NaN, NaN... (this.seconds is undefined)
```

**Solution 1: Arrow Function**
```javascript
const timer = {
    seconds: 0,
    start: function() {
        setInterval(() => {
            this.seconds++;
            console.log(this.seconds);
        }, 1000);
    }
};
```

**Solution 2: bind()**
```javascript
const timer = {
    seconds: 0,
    start: function() {
        setInterval(function() {
            this.seconds++;
            console.log(this.seconds);
        }.bind(this), 1000);
    }
};
```

**Solution 3: Store 'this'**
```javascript
const timer = {
    seconds: 0,
    start: function() {
        const self = this;
        setInterval(function() {
            self.seconds++;
            console.log(self.seconds);
        }, 1000);
    }
};
```

### Q3: Predict the output

```javascript
const person = {
    name: 'Alice',
    greet: () => {
        console.log(`Hello, ${this.name}`);
    }
};

person.greet(); // ?
```

**Answer:** `"Hello, undefined"`
**Reason:** Arrow function doesn't bind `this` to the object. It uses `this` from outer scope (global).

## 💡 Practical Examples

### Example 1: Event Handler

```javascript
class Button {
    constructor(text) {
        this.text = text;
        this.clickCount = 0;
    }

    // Wrong way
    handleClickWrong() {
        this.clickCount++;
        console.log(`${this.text} clicked ${this.clickCount} times`);
    }

    // Right way: Arrow function
    handleClick = () => {
        this.clickCount++;
        console.log(`${this.text} clicked ${this.clickCount} times`);
    }
}

const btn = new Button('Submit');

// DOM event
const domButton = document.querySelector('#myButton');
// domButton.addEventListener('click', btn.handleClickWrong); // Wrong!
domButton.addEventListener('click', btn.handleClick); // Correct
// Or
// domButton.addEventListener('click', btn.handleClickWrong.bind(btn));
```

### Example 2: Method Borrowing

```javascript
const person1 = {
    name: 'Alice',
    greet: function() {
        console.log(`Hello, I'm ${this.name}`);
    }
};

const person2 = {
    name: 'Bob'
};

// Borrow method
person1.greet.call(person2); // "Hello, I'm Bob"

// Real-world: Array-like objects
const arrayLike = {
    0: 'a',
    1: 'b',
    2: 'c',
    length: 3
};

// Borrow Array methods
const arr = Array.prototype.slice.call(arrayLike);
console.log(arr); // ['a', 'b', 'c']

// ES6 way
const arr2 = Array.from(arrayLike);
```

### Example 3: Function Currying with bind

```javascript
function multiply(a, b) {
    return a * b;
}

// Create specific multipliers
const double = multiply.bind(null, 2);
const triple = multiply.bind(null, 3);

console.log(double(5)); // 10
console.log(triple(5)); // 15

// More practical example
function log(level, message) {
    console.log(`[${level}] ${message}`);
}

const logError = log.bind(null, 'ERROR');
const logInfo = log.bind(null, 'INFO');

logError('Something went wrong'); // [ERROR] Something went wrong
logInfo('App started'); // [INFO] App started
```

## 🚨 Common Pitfalls

### 1. Callback Functions

```javascript
const user = {
    name: 'Alice',
    friends: ['Bob', 'Charlie'],

    printFriends: function() {
        // Wrong
        this.friends.forEach(function(friend) {
            console.log(`${this.name} is friends with ${friend}`);
            // TypeError: Cannot read 'name' of undefined
        });
    }
};

// Solutions:
// 1. Arrow function
printFriends: function() {
    this.friends.forEach(friend => {
        console.log(`${this.name} is friends with ${friend}`);
    });
}

// 2. forEach second parameter
printFriends: function() {
    this.friends.forEach(function(friend) {
        console.log(`${this.name} is friends with ${friend}`);
    }, this); // Pass 'this' as second argument
}
```

### 2. Constructor Functions Without 'new'

```javascript
function User(name) {
    this.name = name;
}

const user1 = new User('Alice'); // Correct
console.log(user1.name); // "Alice"

const user2 = User('Bob'); // Forgot 'new'!
console.log(user2); // undefined
console.log(window.name); // "Bob" (polluted global!)

// Solution: Use class or check
function UserSafe(name) {
    if (!(this instanceof UserSafe)) {
        return new UserSafe(name);
    }
    this.name = name;
}

const user3 = UserSafe('Charlie'); // Works without 'new'
```

### 3. Method Extraction

```javascript
const calculator = {
    value: 0,
    add: function(n) {
        this.value += n;
        return this;
    }
};

const add = calculator.add;
add(5); // Error! 'this' is undefined/global

// Solution: Bind
const addBound = calculator.add.bind(calculator);
addBound(5);
console.log(calculator.value); // 5
```

## 🎓 Best Practices

1. **Use arrow functions for callbacks** (preserves `this`)
2. **Use regular functions for methods** (needs `this` binding)
3. **Use `bind` for event handlers** (or class properties with arrow functions)
4. **Be careful with method extraction** (use `bind` or keep context)
5. **Use strict mode** (catches `this` mistakes early)
6. **In classes, use arrow functions for event handlers**

## 📊 Quick Reference Chart

| Call Type | this Value |
|-----------|------------|
| `func()` | global / undefined (strict) |
| `obj.method()` | obj |
| `func.call(obj)` | obj |
| `func.apply(obj)` | obj |
| `func.bind(obj)()` | obj |
| `new Func()` | new object |
| `() => {}` | lexical (outer scope) |

## 🔗 Related Topics

- [Functions & Scope](./02-functions-scope.md)
- [Closures](./03-closures.md)
- [Prototypes & Inheritance](./05-prototypes-inheritance.md)

---

[← Back to JavaScript](./README.md) | [Next: Prototypes →](./05-prototypes-inheritance.md)
