# Test-Driven Development (TDD)

## Overview

Test-Driven Development (TDD) is a software development methodology where you write tests before writing the actual code. The cycle is simple: write a failing test, write just enough code to make it pass, then refactor. While TDD requires discipline and practice, it leads to better-designed, more maintainable code with comprehensive test coverage. Understanding TDD is essential for senior developer roles and demonstrates professional development maturity.

## Table of Contents
- [What is TDD](#what-is-tdd)
- [The Red-Green-Refactor Cycle](#the-red-green-refactor-cycle)
- [Benefits of TDD](#benefits-of-tdd)
- [Challenges of TDD](#challenges-of-tdd)
- [TDD with React Components](#tdd-with-react-components)
- [When to Use TDD](#when-to-use-tdd)
- [Common TDD Patterns](#common-tdd-patterns)
- [Behavior-Driven Development (BDD)](#behavior-driven-development-bdd)
- [TDD vs Traditional Testing](#tdd-vs-traditional-testing)
- [Interview Questions](#interview-questions)

## What is TDD

### The TDD Philosophy

```javascript
// Traditional approach: Code first, test later
// 1. Write implementation
function add(a, b) {
  return a + b;
}

// 2. Write tests (maybe)
test('add function', () => {
  expect(add(2, 3)).toBe(5);
});

// TDD approach: Test first, code second
// 1. Write failing test FIRST
test('add function', () => {
  expect(add(2, 3)).toBe(5); // ❌ Fails - add doesn't exist
});

// 2. Write minimal code to pass
function add(a, b) {
  return a + b; // ✅ Passes
}

// 3. Refactor if needed
```

### Core Principles

```javascript
const tddPrinciples = {
  testFirst: 'Always write test before implementation',
  minimal: 'Write just enough code to pass the test',
  refactor: 'Improve code structure without changing behavior',
  incremental: 'Build features one test at a time',
  feedback: 'Fast feedback on code correctness'
};
```

## The Red-Green-Refactor Cycle

### The Three Phases

```javascript
// 🔴 RED: Write a failing test
test('calculateDiscount applies 10% discount', () => {
  const price = 100;
  const discount = 10;
  const result = calculateDiscount(price, discount);
  expect(result).toBe(90); // ❌ Fails - function doesn't exist
});

// 🟢 GREEN: Write minimal code to pass
function calculateDiscount(price, discount) {
  return price - (price * discount / 100);
} // ✅ Test passes

// 🔵 REFACTOR: Improve the code
function calculateDiscount(price, discountPercent) {
  const discountAmount = price * (discountPercent / 100);
  return price - discountAmount;
} // ✅ Test still passes, code is cleaner
```

### Complete TDD Example

```javascript
// Step 1: RED - Write failing test for first requirement
describe('ShoppingCart', () => {
  test('starts with zero items', () => {
    const cart = new ShoppingCart();
    expect(cart.itemCount()).toBe(0); // ❌ ShoppingCart doesn't exist
  });
});

// Step 2: GREEN - Minimal implementation
class ShoppingCart {
  constructor() {
    this.items = [];
  }

  itemCount() {
    return this.items.length;
  }
}
// ✅ Test passes

// Step 3: RED - Write next test
test('can add an item', () => {
  const cart = new ShoppingCart();
  cart.addItem({ name: 'Laptop', price: 1000 });
  expect(cart.itemCount()).toBe(1); // ❌ addItem doesn't exist
});

// Step 4: GREEN - Implement addItem
class ShoppingCart {
  constructor() {
    this.items = [];
  }

  addItem(item) {
    this.items.push(item);
  }

  itemCount() {
    return this.items.length;
  }
}
// ✅ Test passes

// Step 5: RED - Add total calculation test
test('calculates total price', () => {
  const cart = new ShoppingCart();
  cart.addItem({ name: 'Laptop', price: 1000 });
  cart.addItem({ name: 'Mouse', price: 50 });
  expect(cart.total()).toBe(1050); // ❌ total doesn't exist
});

// Step 6: GREEN - Implement total
class ShoppingCart {
  constructor() {
    this.items = [];
  }

  addItem(item) {
    this.items.push(item);
  }

  itemCount() {
    return this.items.length;
  }

  total() {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
}
// ✅ All tests pass

// Step 7: REFACTOR - No changes needed, code is clean
```

### Real-World TDD Flow

```javascript
// Building a user validator with TDD

// Test 1: RED
test('validateUser returns true for valid user', () => {
  const user = { email: 'user@example.com', age: 25 };
  expect(validateUser(user)).toBe(true); // ❌ Fails
});

// GREEN
function validateUser(user) {
  return true; // Simplest code to pass
}

// Test 2: RED - Email validation
test('validateUser returns false for invalid email', () => {
  const user = { email: 'invalid', age: 25 };
  expect(validateUser(user)).toBe(false); // ❌ Fails
});

// GREEN
function validateUser(user) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(user.email)) {
    return false;
  }
  return true;
}

// Test 3: RED - Age validation
test('validateUser returns false for underage users', () => {
  const user = { email: 'user@example.com', age: 16 };
  expect(validateUser(user)).toBe(false); // ❌ Fails
});

// GREEN
function validateUser(user) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(user.email)) {
    return false;
  }
  if (user.age < 18) {
    return false;
  }
  return true;
}

// REFACTOR - Clean up
function validateUser(user) {
  const isValidEmail = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(user.email);
  const isAdult = user.age >= 18;

  return isValidEmail && isAdult;
}
// ✅ All tests still pass, code is cleaner
```

## Benefits of TDD

### 1. Better Design

```javascript
// TDD forces you to think about API design first

// ❌ Without TDD: Implementation-first thinking
function processOrder(orderId) {
  const db = new Database();
  const order = db.query('SELECT * FROM orders WHERE id = ?', orderId);
  // Tightly coupled to database
}

// ✅ With TDD: Interface-first thinking
test('processOrder returns order details', async () => {
  const orderService = new OrderService(mockDatabase);
  const order = await orderService.processOrder('123');
  expect(order).toEqual({ id: '123', status: 'processed' });
});

// Result: Better abstraction
class OrderService {
  constructor(database) {
    this.database = database; // Dependency injection
  }

  async processOrder(orderId) {
    const order = await this.database.findOrder(orderId);
    order.status = 'processed';
    return order;
  }
}
```

### 2. Comprehensive Test Coverage

```javascript
// TDD naturally achieves high coverage

// Building a calculator with TDD
describe('Calculator', () => {
  let calculator;

  beforeEach(() => {
    calculator = new Calculator();
  });

  // Each feature built with test first
  test('adds two numbers', () => {
    expect(calculator.add(2, 3)).toBe(5);
  });

  test('subtracts two numbers', () => {
    expect(calculator.subtract(5, 3)).toBe(2);
  });

  test('multiplies two numbers', () => {
    expect(calculator.multiply(2, 3)).toBe(6);
  });

  test('divides two numbers', () => {
    expect(calculator.divide(6, 3)).toBe(2);
  });

  test('throws error when dividing by zero', () => {
    expect(() => calculator.divide(6, 0)).toThrow('Cannot divide by zero');
  });
});

// Result: 100% code coverage naturally achieved
```

### 3. Living Documentation

```javascript
// Tests serve as documentation

describe('UserAuthentication', () => {
  // Documents behavior: What does the system do?
  test('successful login returns user token', async () => {
    const auth = new UserAuthentication();
    const result = await auth.login('user@example.com', 'password123');
    expect(result.token).toBeDefined();
  });

  test('failed login throws InvalidCredentialsError', async () => {
    const auth = new UserAuthentication();
    await expect(
      auth.login('user@example.com', 'wrongpassword')
    ).rejects.toThrow(InvalidCredentialsError);
  });

  test('login requires email and password', async () => {
    const auth = new UserAuthentication();
    await expect(auth.login('', '')).rejects.toThrow('Email and password required');
  });

  // Reading tests = understanding requirements
});
```

### 4. Refactoring Safety

```javascript
// TDD enables confident refactoring

// Original implementation
function calculateShipping(order) {
  if (order.weight < 5 && order.country === 'US') {
    return 10;
  } else if (order.weight >= 5 && order.country === 'US') {
    return 20;
  } else if (order.country !== 'US') {
    return 50;
  }
}

// Tests protect behavior
describe('calculateShipping', () => {
  test('US orders under 5kg cost $10', () => {
    expect(calculateShipping({ weight: 3, country: 'US' })).toBe(10);
  });

  test('US orders 5kg or more cost $20', () => {
    expect(calculateShipping({ weight: 6, country: 'US' })).toBe(20);
  });

  test('International orders cost $50', () => {
    expect(calculateShipping({ weight: 3, country: 'CA' })).toBe(50);
  });
});

// Refactor with confidence
function calculateShipping(order) {
  const { weight, country } = order;

  if (country !== 'US') return 50;

  return weight < 5 ? 10 : 20;
}
// ✅ All tests still pass - refactor successful
```

### 5. Faster Debugging

```javascript
// TDD catches bugs immediately

// Test fails immediately when new code breaks existing functionality
test('discount cannot exceed 100%', () => {
  expect(() => applyDiscount(100, 150)).toThrow('Invalid discount');
});

// Without TDD: Bug might not be caught until production
// With TDD: Fails immediately when you write the code
```

## Challenges of TDD

### 1. Initial Time Investment

```javascript
// TDD feels slower at first

// Without TDD: Quick implementation (5 minutes)
function sum(a, b) {
  return a + b;
}

// With TDD: More upfront time (15 minutes)
// 1. Write test (2 min)
test('sum adds two numbers', () => {
  expect(sum(2, 3)).toBe(5);
});

// 2. Run test, see it fail (1 min)
// 3. Implement (2 min)
function sum(a, b) {
  return a + b;
}

// 4. Run test, see it pass (1 min)
// 5. Add edge case tests (5 min)
test('sum handles negative numbers', () => {
  expect(sum(-2, 3)).toBe(1);
});

test('sum handles zero', () => {
  expect(sum(0, 5)).toBe(5);
});

// 6. Refactor (4 min)

// However: Saves debugging time later (30+ minutes saved)
```

### 2. Learning Curve

```javascript
// TDD requires new thinking patterns

// Traditional: "How do I implement this?"
// TDD: "How do I test this?"

// Requires practice to internalize
const learningCurve = {
  week1: 'Feels unnatural and slow',
  week2: 'Starting to see benefits',
  month1: 'Getting comfortable',
  month3: 'Natural workflow',
  month6: 'Faster than non-TDD'
};
```

### 3. UI/Visual Testing Challenges

```javascript
// TDD works great for logic, harder for UI

// ✅ Easy: Business logic
test('calculateTax returns correct amount', () => {
  expect(calculateTax(100, 0.1)).toBe(10);
});

// ⚠️ Harder: UI components
test('button has correct styling', () => {
  // How do you test CSS in a meaningful way?
  // Visual regression testing might be better
});

// Solution: Separate logic from presentation
// Test logic with TDD, use visual testing for UI
```

### 4. Over-Testing Risk

```javascript
// Risk: Testing implementation details

// ❌ Bad: Testing internal state
test('Counter component', () => {
  const counter = new Counter();
  counter.increment();
  expect(counter.state.count).toBe(1); // Testing implementation
});

// ✅ Good: Testing behavior
test('Counter displays incremented value', () => {
  render(<Counter />);
  fireEvent.click(screen.getByText('Increment'));
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

## TDD with React Components

### Basic Component TDD

```javascript
// Step 1: RED - Write failing test
import { render, screen, fireEvent } from '@testing-library/react';

test('Counter starts at 0', () => {
  render(<Counter />);
  expect(screen.getByText('Count: 0')).toBeInTheDocument(); // ❌ Fails
});

// Step 2: GREEN - Minimal implementation
function Counter() {
  return <div>Count: 0</div>;
} // ✅ Passes

// Step 3: RED - Add increment test
test('Counter increments when button clicked', () => {
  render(<Counter />);
  fireEvent.click(screen.getByText('Increment'));
  expect(screen.getByText('Count: 1')).toBeInTheDocument(); // ❌ Fails
});

// Step 4: GREEN - Add increment functionality
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <div>Count: {count}</div>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
} // ✅ Passes

// Step 5: RED - Add decrement test
test('Counter decrements when button clicked', () => {
  render(<Counter />);
  fireEvent.click(screen.getByText('Decrement'));
  expect(screen.getByText('Count: -1')).toBeInTheDocument(); // ❌ Fails
});

// Step 6: GREEN - Add decrement
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <div>Count: {count}</div>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
    </div>
  );
} // ✅ Passes
```

### Form Component TDD

```javascript
// Building a login form with TDD

// Test 1: RED - Render form
test('LoginForm renders email and password inputs', () => {
  render(<LoginForm />);
  expect(screen.getByLabelText('Email')).toBeInTheDocument();
  expect(screen.getByLabelText('Password')).toBeInTheDocument(); // ❌ Fails
});

// GREEN
function LoginForm() {
  return (
    <form>
      <label htmlFor="email">Email</label>
      <input id="email" type="email" />
      <label htmlFor="password">Password</label>
      <input id="password" type="password" />
    </form>
  );
}

// Test 2: RED - Submit with valid data
test('calls onSubmit with email and password', () => {
  const handleSubmit = jest.fn();
  render(<LoginForm onSubmit={handleSubmit} />);

  fireEvent.change(screen.getByLabelText('Email'), {
    target: { value: 'user@example.com' }
  });
  fireEvent.change(screen.getByLabelText('Password'), {
    target: { value: 'password123' }
  });
  fireEvent.click(screen.getByRole('button', { name: 'Login' }));

  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'user@example.com',
    password: 'password123'
  }); // ❌ Fails
});

// GREEN
function LoginForm({ onSubmit }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit({ email, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <label htmlFor="password">Password</label>
      <input
        id="password"
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button type="submit">Login</button>
    </form>
  );
}

// Test 3: RED - Validation
test('shows error for invalid email', () => {
  render(<LoginForm />);

  fireEvent.change(screen.getByLabelText('Email'), {
    target: { value: 'invalid' }
  });
  fireEvent.click(screen.getByRole('button', { name: 'Login' }));

  expect(screen.getByText('Invalid email address')).toBeInTheDocument(); // ❌ Fails
});

// GREEN - Add validation
function LoginForm({ onSubmit }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    setError('');

    if (!email.includes('@')) {
      setError('Invalid email address');
      return;
    }

    onSubmit({ email, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      {error && <div className="error">{error}</div>}
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <label htmlFor="password">Password</label>
      <input
        id="password"
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

### Custom Hook TDD

```javascript
// Building useCounter hook with TDD

// Test 1: RED
test('useCounter starts at initial value', () => {
  const { result } = renderHook(() => useCounter(5));
  expect(result.current.count).toBe(5); // ❌ Fails
});

// GREEN
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  return { count };
}

// Test 2: RED
test('useCounter increments', () => {
  const { result } = renderHook(() => useCounter(0));
  act(() => {
    result.current.increment();
  });
  expect(result.current.count).toBe(1); // ❌ Fails
});

// GREEN
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(count + 1);

  return { count, increment };
}

// Test 3: RED
test('useCounter decrements', () => {
  const { result } = renderHook(() => useCounter(5));
  act(() => {
    result.current.decrement();
  });
  expect(result.current.count).toBe(4); // ❌ Fails
});

// GREEN
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  return { count, increment, decrement };
}

// Test 4: RED
test('useCounter resets to initial value', () => {
  const { result } = renderHook(() => useCounter(5));
  act(() => {
    result.current.increment();
    result.current.increment();
    result.current.reset();
  });
  expect(result.current.count).toBe(5); // ❌ Fails
});

// GREEN - Final implementation
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset };
}
```

## When to Use TDD

### Good Candidates for TDD

```javascript
// ✅ Business logic and algorithms
function calculateLoanPayment(principal, rate, years) {
  // Complex calculation - perfect for TDD
}

// ✅ Data transformations
function transformUserData(apiResponse) {
  // Clear inputs/outputs - great for TDD
}

// ✅ Utility functions
function formatCurrency(amount, locale) {
  // Pure function - ideal for TDD
}

// ✅ API integration
class UserService {
  async fetchUser(id) {
    // Clear contract - good for TDD
  }
}

// ✅ Form validation
function validateForm(formData) {
  // Well-defined rules - perfect for TDD
}
```

### When TDD May Not Be Ideal

```javascript
// ⚠️ Exploratory/prototype code
// When you're not sure what you're building yet
// Write code first, add tests once requirements are clear

// ⚠️ Simple CRUD operations
function saveUser(user) {
  return database.save(user);
}
// Thin wrapper - integration test might be better

// ⚠️ UI experimentation
// Rapid UI prototyping where design is evolving
// Add tests once design stabilizes

// ⚠️ Configuration files
// webpack.config.js, next.config.js
// Not much logic to test

// ⚠️ One-off scripts
// Data migration, one-time imports
// May not warrant TDD overhead
```

## Common TDD Patterns

### 1. Fake It Till You Make It

```javascript
// Start with hard-coded values, then generalize

// Test
test('sum adds two numbers', () => {
  expect(sum(2, 3)).toBe(5);
});

// Step 1: Fake it (hard-code)
function sum() {
  return 5; // ✅ Test passes
}

// Step 2: Make it real
function sum(a, b) {
  return a + b; // ✅ Test still passes, now correct
}
```

### 2. Triangulation

```javascript
// Add multiple test cases to drive generalization

// Test 1
test('sum(2, 3) returns 5', () => {
  expect(sum(2, 3)).toBe(5);
});

// Implementation 1: Fake it
function sum() {
  return 5;
}

// Test 2: Add another case
test('sum(1, 4) returns 5', () => {
  expect(sum(1, 4)).toBe(5);
});
// Still passes with hard-coded 5!

// Test 3: Force generalization
test('sum(10, 20) returns 30', () => {
  expect(sum(10, 20)).toBe(30); // ❌ Fails with hard-coded 5
});

// Implementation 2: Now must generalize
function sum(a, b) {
  return a + b; // ✅ All tests pass
}
```

### 3. Obvious Implementation

```javascript
// For simple cases, write the correct implementation immediately

// Test
test('double returns input times 2', () => {
  expect(double(5)).toBe(10);
});

// Implementation: Just write it (no need to fake it)
function double(n) {
  return n * 2;
}
```

### 4. One to Many

```javascript
// Start with single item, then generalize to collections

// Test 1: Single item
test('processItem returns uppercase', () => {
  expect(processItem('hello')).toBe('HELLO');
});

// Implementation 1
function processItem(item) {
  return item.toUpperCase();
}

// Test 2: Multiple items
test('processItems returns all uppercase', () => {
  expect(processItems(['hello', 'world'])).toEqual(['HELLO', 'WORLD']);
});

// Implementation 2
function processItems(items) {
  return items.map(item => processItem(item));
}
```

## Behavior-Driven Development (BDD)

### BDD vs TDD

```javascript
// TDD: Technical perspective
test('validateEmail returns true for valid email', () => {
  expect(validateEmail('user@example.com')).toBe(true);
});

// BDD: Business perspective
describe('User Registration', () => {
  describe('when user enters valid email', () => {
    it('should accept the email', () => {
      expect(validateEmail('user@example.com')).toBe(true);
    });
  });

  describe('when user enters invalid email', () => {
    it('should reject the email', () => {
      expect(validateEmail('invalid')).toBe(false);
    });

    it('should show error message', () => {
      // More descriptive, user-focused
    });
  });
});
```

### BDD Style Tests

```javascript
// Given-When-Then pattern
describe('Shopping Cart', () => {
  describe('given an empty cart', () => {
    describe('when user adds an item', () => {
      it('then cart should contain one item', () => {
        // Arrange (Given)
        const cart = new ShoppingCart();

        // Act (When)
        cart.addItem({ id: 1, name: 'Laptop', price: 1000 });

        // Assert (Then)
        expect(cart.itemCount()).toBe(1);
      });
    });
  });

  describe('given a cart with items', () => {
    describe('when user removes all items', () => {
      it('then cart should be empty', () => {
        const cart = new ShoppingCart();
        cart.addItem({ id: 1, name: 'Laptop', price: 1000 });

        cart.clear();

        expect(cart.itemCount()).toBe(0);
      });
    });
  });
});
```

### BDD with Cucumber-Style Syntax

```javascript
// Using jest-cucumber or similar
const { defineFeature, loadFeature } = require('jest-cucumber');

const feature = loadFeature('./features/login.feature');

defineFeature(feature, test => {
  test('Successful login', ({ given, when, then }) => {
    let loginPage;
    let response;

    given('I am on the login page', () => {
      loginPage = new LoginPage();
    });

    when('I enter valid credentials', async () => {
      response = await loginPage.login('user@example.com', 'password123');
    });

    then('I should see the dashboard', () => {
      expect(response.redirectUrl).toBe('/dashboard');
    });
  });
});
```

## TDD vs Traditional Testing

### Workflow Comparison

```javascript
// Traditional: Code first, test later
const traditionalWorkflow = {
  step1: 'Write implementation',
  step2: 'Manual testing',
  step3: 'Find bugs',
  step4: 'Fix bugs',
  step5: 'Write tests (maybe)',
  time: '1 hour code + 2 hours debugging'
};

// TDD: Test first, code second
const tddWorkflow = {
  step1: 'Write test',
  step2: 'Write minimal code',
  step3: 'Test passes',
  step4: 'Refactor',
  step5: 'Repeat',
  time: '2 hours total (less debugging)'
};
```

### Code Quality Comparison

```javascript
// Traditional approach: Tightly coupled
class OrderProcessor {
  processOrder(orderId) {
    const db = new Database('connection-string');
    const emailer = new EmailService('api-key');
    const logger = new Logger();

    const order = db.getOrder(orderId);
    order.status = 'processed';
    db.updateOrder(order);
    emailer.send(order.userEmail, 'Order processed');
    logger.log('Order processed: ' + orderId);

    return order;
  }
}
// Hard to test, tightly coupled

// TDD approach: Loosely coupled
class OrderProcessor {
  constructor(database, emailService, logger) {
    this.database = database;
    this.emailService = emailService;
    this.logger = logger;
  }

  async processOrder(orderId) {
    const order = await this.database.getOrder(orderId);
    order.status = 'processed';

    await this.database.updateOrder(order);
    await this.emailService.send(order.userEmail, 'Order processed');
    this.logger.log(`Order processed: ${orderId}`);

    return order;
  }
}
// Easy to test with mocks, loosely coupled
```

## Interview Questions

**Q1: What is Test-Driven Development (TDD)?**

A: TDD is a development methodology where you write tests before writing implementation code, following the Red-Green-Refactor cycle.

**The TDD Cycle:**
1. **Red:** Write a failing test
2. **Green:** Write minimal code to pass
3. **Refactor:** Improve code without changing behavior

**Example:**
```javascript
// RED: Write failing test
test('add returns sum of two numbers', () => {
  expect(add(2, 3)).toBe(5); // ❌ add doesn't exist
});

// GREEN: Minimal implementation
function add(a, b) {
  return a + b; // ✅ Test passes
}

// REFACTOR: (if needed)
// Code is already clean
```

**Benefits:**
- Better design (forces you to think about API)
- Comprehensive test coverage
- Living documentation
- Refactoring confidence

**Q2: Explain the Red-Green-Refactor cycle.**

A: The three phases of TDD:

**🔴 RED - Write a failing test:**
```javascript
test('calculateTax returns correct amount', () => {
  expect(calculateTax(100, 0.2)).toBe(20); // ❌ Function doesn't exist
});
```

**🟢 GREEN - Write minimal code to pass:**
```javascript
function calculateTax(amount, rate) {
  return amount * rate; // ✅ Test passes
}
```

**🔵 REFACTOR - Improve the code:**
```javascript
function calculateTax(amount, taxRate) {
  const tax = amount * taxRate;
  return Number(tax.toFixed(2)); // Better: rounded to 2 decimals
}
// ✅ Test still passes, code is better
```

**Why this cycle works:**
- **Red:** Confirms test actually tests something
- **Green:** Ensures you write just enough code
- **Refactor:** Improves code safely with test protection

**Q3: What are the benefits of TDD?**

A: Key benefits of Test-Driven Development:

**1. Better Design:**
```javascript
// TDD forces dependency injection
class UserService {
  constructor(database, emailService) { // Dependencies injected
    this.database = database;
    this.emailService = emailService;
  }
}
```

**2. Comprehensive Test Coverage:**
- Every line of code has a test (100% coverage naturally)
- No forgotten edge cases

**3. Living Documentation:**
```javascript
test('user cannot purchase with insufficient funds', () => {
  // Test documents behavior
});
```

**4. Refactoring Safety:**
- Change implementation confidently
- Tests catch regressions immediately

**5. Faster Debugging:**
- Bugs caught immediately when writing code
- Not in QA or production

**6. Less Coupling:**
- TDD naturally leads to modular, testable code

**Q4: What are the challenges of TDD?**

A: Common challenges:

**1. Initial Time Investment:**
```javascript
// Without TDD: 5 minutes
function sum(a, b) { return a + b; }

// With TDD: 15 minutes (but saves 30+ in debugging)
```

**2. Learning Curve:**
- Requires new thinking patterns
- Takes practice (3-6 months to feel natural)

**3. UI/Visual Testing Difficulty:**
```javascript
// Easy with TDD: Logic
test('calculateTotal works', () => {});

// Hard with TDD: Visual appearance
test('button is beautiful', () => {}); // How?
```

**4. Over-Testing Risk:**
```javascript
// ❌ Bad: Testing implementation
expect(component.state.count).toBe(1);

// ✅ Good: Testing behavior
expect(screen.getByText('Count: 1')).toBeInTheDocument();
```

**5. Legacy Code:**
- Hard to apply TDD to untested codebases

**Q5: How do you practice TDD with React components?**

A: Follow the Red-Green-Refactor cycle for components:

**Example: Building a Counter**
```javascript
// RED: Test doesn't pass
test('Counter displays initial count', () => {
  render(<Counter initialCount={5} />);
  expect(screen.getByText('Count: 5')).toBeInTheDocument(); // ❌
});

// GREEN: Minimal implementation
function Counter({ initialCount }) {
  return <div>Count: {initialCount}</div>; // ✅
}

// RED: Next feature
test('Counter increments when button clicked', () => {
  render(<Counter initialCount={0} />);
  fireEvent.click(screen.getByText('Increment'));
  expect(screen.getByText('Count: 1')).toBeInTheDocument(); // ❌
});

// GREEN: Add functionality
function Counter({ initialCount }) {
  const [count, setCount] = useState(initialCount);
  return (
    <div>
      <div>Count: {count}</div>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  ); // ✅
}
```

**Key points:**
- Test user-visible behavior, not implementation
- Use React Testing Library
- Test interactions (clicks, form submissions)

**Q6: When should you use TDD and when should you avoid it?**

A: **Use TDD for:**
```javascript
// ✅ Business logic
function calculateShipping(weight, distance) {}

// ✅ Algorithms
function sortProducts(products, criteria) {}

// ✅ Data transformations
function parseAPIResponse(data) {}

// ✅ Utilities
function formatDate(date, locale) {}

// ✅ Form validation
function validateForm(formData) {}
```

**Avoid TDD for:**
```javascript
// ⚠️ Exploratory/prototype code
// When requirements are unclear

// ⚠️ Simple CRUD
function saveUser(user) {
  return db.save(user);
}

// ⚠️ UI experimentation
// Rapid design iteration

// ⚠️ Configuration files
// webpack.config.js

// ⚠️ One-off scripts
// Data migrations
```

**Decision factors:**
- **Complexity:** High complexity → Use TDD
- **Requirements clarity:** Clear requirements → Use TDD
- **Code lifespan:** Long-lived code → Use TDD
- **Prototype phase:** Exploratory → Skip TDD initially

**Q7: What is the difference between TDD and BDD?**

A: **TDD (Test-Driven Development):**
- Technical perspective
- Developer-focused
- Tests implementation behavior

```javascript
// TDD style
test('validateEmail returns true for valid email', () => {
  expect(validateEmail('user@example.com')).toBe(true);
});
```

**BDD (Behavior-Driven Development):**
- Business perspective
- Stakeholder-focused
- Tests business requirements

```javascript
// BDD style
describe('User Registration', () => {
  describe('when user provides valid email', () => {
    it('should accept the registration', () => {
      expect(validateEmail('user@example.com')).toBe(true);
    });
  });
});
```

**BDD Given-When-Then:**
```javascript
describe('Shopping Cart', () => {
  // Given - Initial context
  const cart = new ShoppingCart();

  // When - Action
  cart.addItem({ name: 'Laptop', price: 1000 });

  // Then - Expected outcome
  expect(cart.total()).toBe(1000);
});
```

**Key difference:** BDD uses more natural language and focuses on business value.

**Q8: How does TDD improve code design?**

A: TDD naturally leads to better design:

**1. Forces Modularity:**
```javascript
// Hard to test: God object
class OrderSystem {
  processOrder(orderId) {
    // Connects to DB
    // Sends email
    // Logs
    // Processes payment
    // Updates inventory
  }
}

// Easy to test: Separated concerns
class OrderProcessor {
  constructor(db, emailer, logger, payment, inventory) {
    // Dependencies injected
  }
}
```

**2. Encourages Interfaces:**
```javascript
// TDD pushes you to define clear contracts
interface UserRepository {
  findById(id: string): Promise<User>;
  save(user: User): Promise<void>;
}
```

**3. Promotes Single Responsibility:**
```javascript
// One function, one purpose
function validateEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

function validateAge(age) {
  return age >= 18;
}
```

**4. Reduces Coupling:**
- Dependencies are explicit (constructor injection)
- Easy to swap implementations (mocks in tests)

**Q9: What are common TDD patterns?**

A: **1. Fake It Till You Make It:**
```javascript
// Start simple, generalize later
test('sum(2, 3) returns 5', () => {
  expect(sum(2, 3)).toBe(5);
});

// Step 1: Fake it
function sum() { return 5; }

// Step 2: Generalize
function sum(a, b) { return a + b; }
```

**2. Triangulation:**
```javascript
// Multiple tests drive generalization
test('sum(2, 3) returns 5', () => {});
test('sum(1, 1) returns 2', () => {});
test('sum(10, 5) returns 15', () => {});
// Now you MUST generalize the implementation
```

**3. Obvious Implementation:**
```javascript
// For simple cases, just write it
test('double returns number times 2', () => {
  expect(double(5)).toBe(10);
});

function double(n) {
  return n * 2; // No need to fake it
}
```

**4. One to Many:**
```javascript
// Start with one, extend to collection
test('processItem uppercases string', () => {
  expect(processItem('hello')).toBe('HELLO');
});

test('processItems uppercases all strings', () => {
  expect(processItems(['a', 'b'])).toEqual(['A', 'B']);
});
```

**Q10: How do you know what test to write next?**

A: Follow this strategy:

**1. Start with simplest case:**
```javascript
// Test 1: Simplest scenario
test('empty cart has 0 items', () => {
  const cart = new ShoppingCart();
  expect(cart.itemCount()).toBe(0);
});
```

**2. Add basic functionality:**
```javascript
// Test 2: Core feature
test('can add item to cart', () => {
  const cart = new ShoppingCart();
  cart.addItem({ name: 'Laptop' });
  expect(cart.itemCount()).toBe(1);
});
```

**3. Handle edge cases:**
```javascript
// Test 3: Edge case
test('cannot add duplicate item', () => {
  const cart = new ShoppingCart();
  const item = { id: 1, name: 'Laptop' };
  cart.addItem(item);
  cart.addItem(item);
  expect(cart.itemCount()).toBe(1); // Not 2
});
```

**4. Test error conditions:**
```javascript
// Test 4: Error handling
test('throws error when adding null item', () => {
  const cart = new ShoppingCart();
  expect(() => cart.addItem(null)).toThrow();
});
```

**Strategy:**
- Simple → Complex
- Happy path → Edge cases
- Core features → Nice-to-haves

## Summary

**TDD Checklist:**
- [ ] Write test first (RED)
- [ ] Write minimal code to pass (GREEN)
- [ ] Refactor for quality (REFACTOR)
- [ ] Repeat for each feature
- [ ] Keep tests simple and readable
- [ ] Test behavior, not implementation
- [ ] Commit with green tests

**Key Principles:**
1. **Test first** - Always write failing test before code
2. **Minimal code** - Write just enough to pass
3. **Refactor often** - Improve code with test safety net
4. **Small steps** - One test, one feature at a time
5. **Fast feedback** - Run tests frequently

**When to Use TDD:**
- Business logic and algorithms
- Data transformations
- API integrations
- Form validation
- Utility functions

**When to Avoid TDD:**
- Exploratory prototyping
- Simple CRUD operations
- UI design experimentation
- Configuration files
- One-off scripts

**TDD Benefits:**
- Better code design
- Comprehensive coverage
- Living documentation
- Refactoring confidence
- Faster debugging

---

[← E2E Testing](./05-e2e-testing.md) | [Next: Testing Best Practices →](./07-best-practices.md)
