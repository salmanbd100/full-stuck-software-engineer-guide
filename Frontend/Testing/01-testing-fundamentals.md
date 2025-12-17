# Testing Fundamentals

## Overview

Testing is the foundation of reliable software development. Understanding testing fundamentals is crucial for building maintainable applications and succeeding in technical interviews. This guide covers the core concepts, types of tests, testing strategies, and best practices that every frontend developer should master.

## Table of Contents
- [Why Testing Matters](#why-testing-matters)
- [Types of Tests](#types-of-tests)
- [Unit Tests](#unit-tests)
- [Integration Tests](#integration-tests)
- [End-to-End Tests](#end-to-end-tests)
- [Testing Pyramid vs Testing Trophy](#testing-pyramid-vs-testing-trophy)
- [AAA Pattern](#aaa-pattern-arrange-act-assert)
- [Test Coverage Metrics](#test-coverage-metrics)
- [When to Write Tests](#when-to-write-tests)
- [Testing Mindset](#testing-mindset)
- [Common Testing Mistakes](#common-testing-mistakes)
- [Interview Questions](#interview-questions)

## Why Testing Matters

### Business Value

```javascript
// Testing provides tangible business benefits
const testingBenefits = {
  quality: 'Catch bugs before production',
  confidence: 'Deploy with certainty',
  documentation: 'Tests serve as living documentation',
  refactoring: 'Safe code improvements',
  speed: 'Faster development in the long run',
  cost: 'Cheaper than production bugs'
};

// Cost of bugs increases exponentially
const bugCost = {
  development: '$100',      // Caught during coding
  testing: '$1,000',        // Caught in QA
  production: '$10,000+'    // Caught by users
};
```

### Developer Benefits

**Immediate Feedback:**
- Know if code works instantly
- Catch regressions immediately
- Debug faster with isolated tests

**Better Design:**
- Testable code is better structured
- Forces modular design
- Encourages separation of concerns

**Confidence:**
- Refactor without fear
- Add features safely
- Delete dead code confidently

## Types of Tests

### The Testing Spectrum

```javascript
// Different test types serve different purposes
const testTypes = {
  unit: {
    scope: 'Single function/component',
    speed: 'Very fast (milliseconds)',
    cost: 'Low',
    confidence: 'Low to medium',
    quantity: 'Many (70%)'
  },
  integration: {
    scope: 'Multiple components together',
    speed: 'Medium (seconds)',
    cost: 'Medium',
    confidence: 'Medium to high',
    quantity: 'Some (20%)'
  },
  e2e: {
    scope: 'Full user flows',
    speed: 'Slow (minutes)',
    cost: 'High',
    confidence: 'Very high',
    quantity: 'Few (10%)'
  }
};
```

### Comparison Matrix

| Aspect | Unit Tests | Integration Tests | E2E Tests |
|--------|-----------|------------------|-----------|
| **Scope** | Single function | Multiple units | Full application |
| **Speed** | ⚡ Very Fast | 🏃 Medium | 🐌 Slow |
| **Isolation** | ✅ Complete | ⚠️ Partial | ❌ None |
| **Confidence** | 📊 Low-Medium | 📈 Medium-High | 📉 Very High |
| **Maintenance** | ✅ Easy | ⚠️ Medium | ❌ Hard |
| **Flakiness** | ✅ Stable | ⚠️ Sometimes | ❌ Often flaky |
| **Cost** | 💰 Cheap | 💰💰 Medium | 💰💰💰 Expensive |

## Unit Tests

### What Are Unit Tests?

Unit tests verify individual functions or components in **complete isolation** from the rest of the application.

```javascript
// Unit test example - testing a single function
function calculateDiscount(price, discountPercent) {
  if (price < 0 || discountPercent < 0 || discountPercent > 100) {
    throw new Error('Invalid input');
  }
  return price * (discountPercent / 100);
}

// ✅ Unit test - tests ONE function in isolation
describe('calculateDiscount', () => {
  test('calculates 10% discount correctly', () => {
    expect(calculateDiscount(100, 10)).toBe(10);
  });

  test('calculates 50% discount correctly', () => {
    expect(calculateDiscount(200, 50)).toBe(100);
  });

  test('throws error for negative price', () => {
    expect(() => calculateDiscount(-100, 10)).toThrow('Invalid input');
  });

  test('throws error for discount over 100%', () => {
    expect(() => calculateDiscount(100, 150)).toThrow('Invalid input');
  });

  test('handles zero discount', () => {
    expect(calculateDiscount(100, 0)).toBe(0);
  });
});
```

### Unit Test Characteristics

**✅ Good Unit Tests:**
```javascript
// 1. Fast - runs in milliseconds
test('adds two numbers', () => {
  expect(add(2, 3)).toBe(5);
}); // ~1ms

// 2. Isolated - no external dependencies
test('validates email format', () => {
  expect(isValidEmail('test@example.com')).toBe(true);
}); // No API calls, no database

// 3. Deterministic - same input = same output
test('formats currency', () => {
  expect(formatCurrency(100)).toBe('$100.00');
}); // Always returns same result

// 4. Single responsibility - tests ONE thing
test('returns empty array when no items match filter', () => {
  const items = [{ active: false }, { active: false }];
  expect(filterActive(items)).toEqual([]);
});
```

**❌ Bad Unit Tests:**
```javascript
// 1. Testing multiple things
test('processes user data', () => {
  const user = createUser();
  saveUser(user);
  sendEmail(user.email);
  logAnalytics(user);
  // Too much in one test!
});

// 2. Dependent on external systems
test('fetches user from API', async () => {
  const user = await api.getUser(1); // ❌ Real API call
  expect(user.name).toBe('John');
});

// 3. Testing implementation details
test('calls useState hook', () => {
  // ❌ Testing React internals
  expect(component.state.count).toBe(0);
});
```

### When to Write Unit Tests

**✅ Perfect for:**
- Pure functions (same input → same output)
- Business logic and algorithms
- Utility functions
- Data transformations
- Validation logic
- Edge case handling

```javascript
// ✅ Great candidate for unit testing
function calculateShipping(weight, distance, isPriority) {
  const baseRate = 5;
  const weightCost = weight * 0.5;
  const distanceCost = distance * 0.1;
  const priorityMultiplier = isPriority ? 2 : 1;

  return (baseRate + weightCost + distanceCost) * priorityMultiplier;
}

// Easy to test all combinations
test.each([
  [10, 100, false, 20],   // weight, distance, priority, expected
  [10, 100, true, 40],
  [0, 0, false, 5],
  [5, 50, false, 12.5],
])('calculates shipping for %i kg, %i km, priority: %s',
  (weight, distance, priority, expected) => {
    expect(calculateShipping(weight, distance, priority)).toBe(expected);
  }
);
```

**❌ Not ideal for:**
```javascript
// ❌ Complex component interactions
function ShoppingCart({ items, onCheckout, onRemove }) {
  // Multiple dependencies, state, side effects
  // Better suited for integration tests
}

// ❌ DOM manipulation
function setupEventListeners() {
  document.getElementById('btn').addEventListener('click', handler);
  // Better tested in integration or E2E
}
```

## Integration Tests

### What Are Integration Tests?

Integration tests verify that multiple units work together correctly.

```javascript
// Integration test example - testing component + API interaction
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import UserProfile from './UserProfile';
import { server } from './mocks/server';

describe('UserProfile Integration', () => {
  test('loads and displays user data on mount', async () => {
    // Arrange - render component
    render(<UserProfile userId={1} />);

    // Act - component fetches data automatically
    // (Integration: component + API)

    // Assert - verify data is displayed
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
      expect(screen.getByText('john@example.com')).toBeInTheDocument();
    });
  });

  test('updates profile when form is submitted', async () => {
    // Testing: Component + Form + API + State
    render(<UserProfile userId={1} />);

    // Wait for initial load
    await screen.findByText('John Doe');

    // Click edit button
    const editBtn = screen.getByRole('button', { name: /edit/i });
    await userEvent.click(editBtn);

    // Update form
    const nameInput = screen.getByLabelText(/name/i);
    await userEvent.clear(nameInput);
    await userEvent.type(nameInput, 'Jane Doe');

    // Submit
    const saveBtn = screen.getByRole('button', { name: /save/i });
    await userEvent.click(saveBtn);

    // Verify update
    await waitFor(() => {
      expect(screen.getByText('Jane Doe')).toBeInTheDocument();
      expect(screen.queryByLabelText(/name/i)).not.toBeInTheDocument();
    });
  });
});
```

### Integration Test Characteristics

**Key Differences from Unit Tests:**

```javascript
// Unit test - isolated function
test('formats name', () => {
  expect(formatName('john', 'doe')).toBe('John Doe');
});

// Integration test - multiple components working together
test('user registration flow', async () => {
  // Tests: Form + Validation + API + Success Message + Navigation
  render(<RegistrationForm />);

  await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
  await userEvent.type(screen.getByLabelText(/password/i), 'password123');
  await userEvent.click(screen.getByRole('button', { name: /register/i }));

  await waitFor(() => {
    expect(screen.getByText(/registration successful/i)).toBeInTheDocument();
  });
});
```

### When to Write Integration Tests

**✅ Perfect for:**
- Component interactions
- Form submissions
- API integrations
- State management flows
- User workflows
- Data flow between components

```javascript
// ✅ Great candidate for integration testing
function ShoppingCart() {
  // Integrates: CartItems + PriceCalculator + CheckoutButton + API
  const [items, setItems] = useState([]);
  const total = calculateTotal(items);

  const handleCheckout = async () => {
    await api.processCheckout(items, total);
    setItems([]);
  };

  return (
    <div>
      <CartItems items={items} onRemove={removeItem} />
      <PriceDisplay total={total} />
      <CheckoutButton onClick={handleCheckout} />
    </div>
  );
}

// Integration test
test('checkout flow clears cart after successful payment', async () => {
  render(<ShoppingCart initialItems={mockItems} />);

  expect(screen.getByText('3 items')).toBeInTheDocument();

  await userEvent.click(screen.getByRole('button', { name: /checkout/i }));

  await waitFor(() => {
    expect(screen.getByText('0 items')).toBeInTheDocument();
  });
});
```

## End-to-End Tests

### What Are E2E Tests?

E2E tests verify complete user flows in a production-like environment.

```javascript
// E2E test example - using Playwright
import { test, expect } from '@playwright/test';

test.describe('E2E: User Authentication Flow', () => {
  test('user can sign up, login, and access dashboard', async ({ page }) => {
    // 1. Navigate to sign up
    await page.goto('https://app.example.com/signup');

    // 2. Fill registration form
    await page.fill('[name="email"]', 'newuser@example.com');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.fill('[name="confirmPassword"]', 'SecurePass123!');
    await page.click('button[type="submit"]');

    // 3. Verify redirect to dashboard
    await expect(page).toHaveURL(/.*dashboard/);
    await expect(page.locator('h1')).toContainText('Welcome');

    // 4. Verify user can access protected features
    await page.click('nav >> text=Profile');
    await expect(page.locator('[data-testid="user-email"]'))
      .toContainText('newuser@example.com');

    // 5. Test logout
    await page.click('button:has-text("Logout")');
    await expect(page).toHaveURL(/.*login/);
  });

  test('prevents unauthorized access to dashboard', async ({ page }) => {
    // Try to access dashboard without login
    await page.goto('https://app.example.com/dashboard');

    // Should redirect to login
    await expect(page).toHaveURL(/.*login/);
    await expect(page.locator('.error-message'))
      .toContainText('Please log in to continue');
  });
});
```

### E2E Test Characteristics

**What Makes E2E Different:**

```javascript
// E2E tests the ENTIRE application stack
const e2eScope = {
  frontend: 'React application',
  backend: 'Real API server',
  database: 'Test database',
  authentication: 'Real auth flow',
  thirdParty: 'External services (can be mocked)',
  browser: 'Real browser environment'
};

// E2E test - tests everything
test('complete order flow', async ({ page }) => {
  // Login (authentication)
  await page.goto('/login');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'password');
  await page.click('button[type="submit"]');

  // Browse products (frontend + API)
  await page.click('nav >> text=Products');
  await page.click('text=Laptop >> .. >> button:has-text("Add to Cart")');

  // Checkout (forms + validation + API)
  await page.click('text=Cart');
  await page.click('button:has-text("Checkout")');
  await page.fill('[name="cardNumber"]', '4242424242424242');
  await page.fill('[name="expiry"]', '12/25');
  await page.fill('[name="cvv"]', '123');
  await page.click('button:has-text("Place Order")');

  // Verify success (database + email notification)
  await expect(page.locator('.success-message'))
    .toContainText('Order placed successfully');
});
```

### When to Write E2E Tests

**✅ Perfect for:**
- Critical user journeys
- Authentication flows
- Payment/checkout processes
- Multi-step forms
- Cross-page workflows
- Smoke tests

**❌ Not ideal for:**
- Testing every edge case (too slow)
- Unit-level logic testing
- Rapid feedback during development

## Testing Pyramid vs Testing Trophy

### The Testing Pyramid

```
         /\
        /  \      E2E Tests (10%)
       /    \     - Few, critical paths only
      /------\    - Slow, expensive
     /        \
    / Integration \ (20%)
   /    Tests     \
  /--------------\ - Important workflows
 /                \ - Medium speed
/   Unit Tests     \ (70%)
/                   \
---------------------
- Many, fast tests
- Pure functions, logic
- Cheap to maintain
```

**Pyramid Philosophy:**
```javascript
const testingPyramid = {
  base: {
    type: 'Unit Tests',
    quantity: '70%',
    focus: 'Individual functions, pure logic',
    speed: 'Very fast (milliseconds)',
    example: 'calculateTotal(items) returns correct sum'
  },
  middle: {
    type: 'Integration Tests',
    quantity: '20%',
    focus: 'Component interactions, API calls',
    speed: 'Medium (seconds)',
    example: 'Form submission updates UI and saves to API'
  },
  top: {
    type: 'E2E Tests',
    quantity: '10%',
    focus: 'Critical user journeys',
    speed: 'Slow (minutes)',
    example: 'User can sign up, login, and make purchase'
  }
};
```

### The Testing Trophy

```
      _____
     |     |    E2E (10%)
     |_____|
    /       \   Integration (50%)
   /         \  - Most valuable
  |           | - Test behavior
  |           |
  |___________|
      |   |      Unit (30%)
      |   |
      -----
     Static (10%)
   (TypeScript, Linting)
```

**Trophy Philosophy (Kent C. Dodds):**

```javascript
const testingTrophy = {
  static: {
    percentage: '10%',
    tools: ['TypeScript', 'ESLint', 'Prettier'],
    value: 'Catch typos and type errors',
    cost: 'Very low'
  },
  unit: {
    percentage: '30%',
    focus: 'Pure logic, utilities',
    value: 'Fast feedback on algorithms',
    cost: 'Low'
  },
  integration: {
    percentage: '50%', // ⭐ Most important
    focus: 'Component + behavior testing',
    value: 'Tests how users interact with app',
    cost: 'Medium',
    note: 'Sweet spot for confidence vs speed'
  },
  e2e: {
    percentage: '10%',
    focus: 'Critical paths only',
    value: 'Highest confidence',
    cost: 'Very high'
  }
};

// Quote from Kent C. Dodds:
// "Write tests. Not too many. Mostly integration."
```

### Pyramid vs Trophy: Which to Use?

```javascript
// Use Testing Pyramid when:
const usePyramid = {
  scenario: 'Backend/API heavy applications',
  when: [
    'Lots of business logic in functions',
    'Complex algorithms',
    'Data transformations',
    'Calculations and validations'
  ],
  example: 'E-commerce pricing engine, data analytics'
};

// Use Testing Trophy when:
const useTrophy = {
  scenario: 'Frontend/UI heavy applications',
  when: [
    'User interaction focused',
    'React/Vue/Angular apps',
    'Form-heavy applications',
    'UI state management critical'
  ],
  example: 'Social media app, SaaS dashboard'
};
```

## AAA Pattern (Arrange, Act, Assert)

### The AAA Structure

```javascript
// AAA Pattern - standard test structure
test('adds item to shopping cart', () => {
  // 🔧 ARRANGE - Set up test data and conditions
  const cart = new ShoppingCart();
  const item = { id: 1, name: 'Laptop', price: 999 };

  // ⚡ ACT - Execute the function being tested
  cart.addItem(item);

  // ✅ ASSERT - Verify the result
  expect(cart.items).toHaveLength(1);
  expect(cart.items[0]).toEqual(item);
  expect(cart.total).toBe(999);
});
```

### Detailed AAA Examples

**Example 1: Simple Function Test**

```javascript
// Function to test
function calculateTax(price, taxRate) {
  return price * taxRate;
}

test('calculates tax correctly', () => {
  // ARRANGE - prepare inputs
  const price = 100;
  const taxRate = 0.08; // 8%

  // ACT - call the function
  const tax = calculateTax(price, taxRate);

  // ASSERT - verify output
  expect(tax).toBe(8);
});
```

**Example 2: React Component Test**

```javascript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('counter increments when button is clicked', async () => {
  // ARRANGE - render component
  render(<Counter initialCount={0} />);

  // ACT - simulate user interaction
  const button = screen.getByRole('button', { name: /increment/i });
  await userEvent.click(button);

  // ASSERT - verify UI updated
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

**Example 3: Async API Test**

```javascript
test('fetches and displays user data', async () => {
  // ARRANGE - setup mock data
  const mockUser = { id: 1, name: 'John Doe' };
  jest.spyOn(api, 'getUser').mockResolvedValue(mockUser);

  // ACT - trigger the action
  render(<UserProfile userId={1} />);

  // ASSERT - verify async result
  const userName = await screen.findByText('John Doe');
  expect(userName).toBeInTheDocument();
  expect(api.getUser).toHaveBeenCalledWith(1);
});
```

### Given-When-Then (BDD Variation)

```javascript
// BDD style - similar to AAA
test('user login flow', async () => {
  // GIVEN - user is on login page
  render(<LoginPage />);
  const emailInput = screen.getByLabelText(/email/i);
  const passwordInput = screen.getByLabelText(/password/i);
  const submitButton = screen.getByRole('button', { name: /login/i });

  // WHEN - user enters credentials and submits
  await userEvent.type(emailInput, 'test@example.com');
  await userEvent.type(passwordInput, 'password123');
  await userEvent.click(submitButton);

  // THEN - user is redirected to dashboard
  await waitFor(() => {
    expect(window.location.pathname).toBe('/dashboard');
  });
});
```

## Test Coverage Metrics

### What is Code Coverage?

```javascript
// Code coverage measures how much code is executed by tests
const coverageMetrics = {
  line: 'Percentage of lines executed',
  branch: 'Percentage of if/else paths taken',
  function: 'Percentage of functions called',
  statement: 'Percentage of statements executed'
};

// Example function with coverage annotations
function getUserStatus(user) {
  // Line 1: Covered ✅
  if (!user) {
    // Line 2: Covered ✅
    return 'guest';
  }

  // Line 3: Not covered ❌
  if (user.isPremium) {
    // Line 4: Not covered ❌
    return 'premium';
  }

  // Line 5: Covered ✅
  return 'regular';
}

// Current test only covers guest and regular paths
test('returns guest for no user', () => {
  expect(getUserStatus(null)).toBe('guest');
});

test('returns regular for basic user', () => {
  expect(getUserStatus({ isPremium: false })).toBe('regular');
});

// Missing test for premium path! Coverage: 60%
```

### Coverage Report Example

```bash
# Jest coverage report
npm test -- --coverage

# Output:
--------------------------|---------|----------|---------|---------|
File                      | % Stmts | % Branch | % Funcs | % Lines |
--------------------------|---------|----------|---------|---------|
All files                 |   85.5  |   78.2   |   90.1  |   85.5  |
 utils/                   |   95.0  |   88.0   |  100.0  |   95.0  |
  calculate.js            |   98.0  |   92.0   |  100.0  |   98.0  |
  format.js               |   92.0  |   84.0   |  100.0  |   92.0  |
 components/              |   78.0  |   70.0   |   85.0  |   78.0  |
  UserProfile.jsx         |   75.0  |   66.0   |   80.0  |   75.0  | ⚠️
  ShoppingCart.jsx        |   81.0  |   74.0   |   90.0  |   81.0  |
--------------------------|---------|----------|---------|---------|
```

### Understanding Coverage Metrics

**Statement Coverage:**
```javascript
function divide(a, b) {
  const result = a / b;        // Statement 1
  return result;               // Statement 2
}

// 100% statement coverage with one test
test('divides numbers', () => {
  expect(divide(10, 2)).toBe(5);
}); // Both statements executed ✅
```

**Branch Coverage:**
```javascript
function getDiscount(age) {
  if (age < 18) {              // Branch point
    return 0.1;                // Branch 1
  } else {
    return 0.05;               // Branch 2
  }
}

// 50% branch coverage - only one branch tested
test('adult discount', () => {
  expect(getDiscount(25)).toBe(0.05);
}); // Only 'else' branch ❌

// 100% branch coverage - both branches tested
test('child discount', () => {
  expect(getDiscount(15)).toBe(0.1);
}); // Now both branches ✅
```

**Function Coverage:**
```javascript
// Module with multiple functions
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export function multiply(a, b) { return a * b; }

// 33% function coverage - only testing add
test('addition works', () => {
  expect(add(2, 3)).toBe(5);
}); // 1 of 3 functions tested
```

### Coverage Goals

```javascript
const coverageGoals = {
  greenFlag: {
    statements: '> 80%',
    branches: '> 75%',
    functions: '> 80%',
    lines: '> 80%'
  },
  yellowFlag: {
    statements: '60-80%',
    branches: '50-75%',
    note: 'Needs improvement'
  },
  redFlag: {
    statements: '< 60%',
    branches: '< 50%',
    note: 'Inadequate testing'
  }
};

// Important: 100% coverage ≠ bug-free code!
```

### Coverage Pitfalls

**❌ Mistake 1: Chasing 100% Coverage**
```javascript
// Meaningless test just for coverage
test('imports module', () => {
  expect(MyComponent).toBeDefined(); // ❌ Useless test
});

// Testing getters/setters
test('sets and gets name', () => {
  const user = new User();
  user.name = 'John';
  expect(user.name).toBe('John'); // ❌ Testing language features
});
```

**❌ Mistake 2: Not Testing Edge Cases**
```javascript
// 100% line coverage but missing edge cases!
function divide(a, b) {
  return a / b; // What about b = 0? ⚠️
}

test('divides numbers', () => {
  expect(divide(10, 2)).toBe(5); // ✅ Covers line
  // Missing: divide(10, 0) => Infinity or error?
});
```

**✅ Better Approach: Quality over Coverage**
```javascript
// Focus on meaningful tests
test('divide throws error for division by zero', () => {
  expect(() => divide(10, 0)).toThrow('Cannot divide by zero');
});

test('divide handles negative numbers', () => {
  expect(divide(-10, 2)).toBe(-5);
});

test('divide handles decimals', () => {
  expect(divide(1, 3)).toBeCloseTo(0.333, 2);
});
```

## When to Write Tests

### Test-First Approach (TDD)

```javascript
// 1. Write test first (it will fail - RED)
test('calculateTotal adds item prices', () => {
  const items = [
    { price: 10 },
    { price: 20 },
    { price: 30 }
  ];
  expect(calculateTotal(items)).toBe(60);
}); // ❌ Fails - function doesn't exist yet

// 2. Write minimal code to pass (GREEN)
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price, 0);
} // ✅ Test passes

// 3. Refactor (REFACTOR)
function calculateTotal(items) {
  return items?.reduce((sum, item) => sum + (item?.price || 0), 0) || 0;
} // ✅ Test still passes, code is better
```

### Test-After Approach

```javascript
// 1. Write code first
function fetchUserPosts(userId) {
  return fetch(`/api/users/${userId}/posts`)
    .then(res => res.json())
    .then(data => data.posts);
}

// 2. Then write tests
describe('fetchUserPosts', () => {
  test('fetches posts for valid user', async () => {
    // Mock fetch
    global.fetch = jest.fn(() =>
      Promise.resolve({
        json: () => Promise.resolve({ posts: [{ id: 1 }] })
      })
    );

    const posts = await fetchUserPosts(123);
    expect(posts).toHaveLength(1);
    expect(fetch).toHaveBeenCalledWith('/api/users/123/posts');
  });
});
```

### When to Write Each Type

**Write Unit Tests When:**
```javascript
// ✅ Building utility functions
function formatPhoneNumber(number) {
  // Pure logic - perfect for unit tests
  return number.replace(/(\d{3})(\d{3})(\d{4})/, '($1) $2-$3');
}

// ✅ Implementing algorithms
function findShortestPath(graph, start, end) {
  // Complex logic - unit test each case
}

// ✅ Creating validation logic
function validateCreditCard(number) {
  // Business rules - test all edge cases
}
```

**Write Integration Tests When:**
```javascript
// ✅ Building forms with validation + API
function RegistrationForm() {
  // Test: Form validation + API call + Success message
}

// ✅ Creating data flows
function ShoppingCart() {
  // Test: Add item → Update total → Save to storage
}

// ✅ Component interactions
function Dashboard() {
  // Test: Filter changes → Table updates → Export works
}
```

**Write E2E Tests When:**
```javascript
// ✅ Critical user journeys
test('user can complete purchase', async () => {
  // Login → Browse → Add to cart → Checkout → Payment → Confirmation
});

// ✅ Authentication flows
test('user can sign up and access protected pages', async () => {
  // Sign up → Email verification → Login → Dashboard
});

// ✅ Multi-page workflows
test('user can create and publish blog post', async () => {
  // Login → Create → Edit → Preview → Publish → View
});
```

### Decision Framework

```javascript
const testingDecisionTree = {
  question1: 'Is it pure logic/calculation?',
  yes1: 'Write unit test',
  no1: {
    question2: 'Does it involve multiple components/modules?',
    yes2: 'Write integration test',
    no2: {
      question3: 'Is it a critical end-to-end user flow?',
      yes3: 'Write E2E test',
      no3: 'Consider if test is needed at all'
    }
  }
};
```

## Testing Mindset

### Think Like a User

```javascript
// ❌ Developer mindset - testing implementation
test('useState hook initializes with 0', () => {
  const { result } = renderHook(() => useState(0));
  expect(result.current[0]).toBe(0);
});

// ✅ User mindset - testing behavior
test('counter shows 0 on initial render', () => {
  render(<Counter />);
  expect(screen.getByText('Count: 0')).toBeInTheDocument();
});
```

### Test Behavior, Not Implementation

```javascript
// Component using internal state
function ToggleButton() {
  const [isOn, setIsOn] = useState(false);

  return (
    <button onClick={() => setIsOn(!isOn)}>
      {isOn ? 'ON' : 'OFF'}
    </button>
  );
}

// ❌ Bad - testing internal state
test('toggles isOn state', () => {
  const { result } = renderHook(() => useState(false));
  // Testing React internals ❌
});

// ✅ Good - testing visible behavior
test('button text toggles between ON and OFF', async () => {
  render(<ToggleButton />);
  const button = screen.getByRole('button');

  expect(button).toHaveTextContent('OFF');

  await userEvent.click(button);
  expect(button).toHaveTextContent('ON');

  await userEvent.click(button);
  expect(button).toHaveTextContent('OFF');
});
```

### Test Edge Cases

```javascript
// Testing happy path only is not enough
function parseUserInput(input) {
  return input.trim().toLowerCase();
}

// ❌ Incomplete - only happy path
test('parses normal input', () => {
  expect(parseUserInput('Hello')).toBe('hello');
});

// ✅ Complete - covers edge cases
describe('parseUserInput', () => {
  test('handles normal input', () => {
    expect(parseUserInput('Hello')).toBe('hello');
  });

  test('handles empty string', () => {
    expect(parseUserInput('')).toBe('');
  });

  test('handles whitespace', () => {
    expect(parseUserInput('  Hello  ')).toBe('hello');
  });

  test('handles special characters', () => {
    expect(parseUserInput('Hello!@#')).toBe('hello!@#');
  });

  test('handles null/undefined', () => {
    expect(() => parseUserInput(null)).toThrow();
  });
});
```

### Write Tests That Provide Value

```javascript
// ❌ Low-value tests
test('component renders', () => {
  render(<MyComponent />);
  expect(screen.getByTestId('my-component')).toBeInTheDocument();
}); // Every component renders if no error - useless test

test('constant has correct value', () => {
  expect(MAX_COUNT).toBe(100);
}); // Testing a constant - no value

// ✅ High-value tests
test('shows error message when form submission fails', async () => {
  // Tests error handling - important!
  server.use(
    rest.post('/api/submit', (req, res, ctx) => {
      return res(ctx.status(500));
    })
  );

  render(<Form />);
  await userEvent.click(screen.getByRole('button', { name: /submit/i }));

  expect(await screen.findByText(/submission failed/i)).toBeInTheDocument();
});

test('disables submit button while request is in flight', async () => {
  // Tests UX behavior - valuable!
  render(<Form />);
  const submitBtn = screen.getByRole('button', { name: /submit/i });

  await userEvent.click(submitBtn);

  expect(submitBtn).toBeDisabled();

  await waitFor(() => expect(submitBtn).not.toBeDisabled());
});
```

## Common Testing Mistakes

### 1. Testing Implementation Details

```javascript
// ❌ Bad - coupled to implementation
test('uses useEffect to fetch data', () => {
  const spy = jest.spyOn(React, 'useEffect');
  render(<UserProfile />);
  expect(spy).toHaveBeenCalled();
});

// ✅ Good - tests behavior
test('displays user data after loading', async () => {
  render(<UserProfile />);
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
});
```

### 2. Brittle Selectors

```javascript
// ❌ Bad - breaks when structure changes
const button = wrapper.find('div').at(2).find('button').first();

// ✅ Good - semantic queries
const button = screen.getByRole('button', { name: /submit/i });
```

### 3. Not Cleaning Up

```javascript
// ❌ Bad - tests affect each other
let user;
beforeEach(() => {
  user = { name: 'John' };
});

test('test 1', () => {
  user.name = 'Jane'; // Mutates shared state ❌
  expect(user.name).toBe('Jane');
});

test('test 2', () => {
  expect(user.name).toBe('John'); // Might fail! ❌
});

// ✅ Good - isolated tests
test('test 1', () => {
  const user = { name: 'John' };
  user.name = 'Jane';
  expect(user.name).toBe('Jane');
});

test('test 2', () => {
  const user = { name: 'John' };
  expect(user.name).toBe('John');
});
```

### 4. Testing Too Much in One Test

```javascript
// ❌ Bad - tests multiple things
test('user registration flow', async () => {
  // Too much in one test!
  await registerUser();
  expect(success).toBe(true);

  await loginUser();
  expect(loggedIn).toBe(true);

  await updateProfile();
  expect(updated).toBe(true);

  await deleteAccount();
  expect(deleted).toBe(true);
});

// ✅ Good - separate tests
describe('User Management', () => {
  test('user can register', async () => {
    await registerUser();
    expect(success).toBe(true);
  });

  test('user can login after registration', async () => {
    await registerUser();
    await loginUser();
    expect(loggedIn).toBe(true);
  });

  test('user can update profile', async () => {
    // ... setup
    await updateProfile();
    expect(updated).toBe(true);
  });
});
```

### 5. Ignoring Async Behavior

```javascript
// ❌ Bad - race condition
test('displays loaded data', () => {
  render(<AsyncComponent />);
  expect(screen.getByText('Data loaded')).toBeInTheDocument(); // ❌ Fails!
});

// ✅ Good - wait for async
test('displays loaded data', async () => {
  render(<AsyncComponent />);
  expect(await screen.findByText('Data loaded')).toBeInTheDocument(); // ✅
});
```

## Interview Questions

### Question 1: Explain the difference between unit, integration, and E2E tests.

**Answer:**

The three test types differ in scope, speed, and confidence level:

**Unit Tests:**
- Test individual functions or components in complete isolation
- Very fast (milliseconds)
- Low confidence but high quantity
- Example: Testing a `formatCurrency(100)` function returns `"$100.00"`

**Integration Tests:**
- Test multiple units working together
- Medium speed (seconds)
- Medium-high confidence, medium quantity
- Example: Testing a form component that validates input, calls an API, and displays results

**E2E Tests:**
- Test complete user flows through the entire application
- Slow (minutes)
- Very high confidence but low quantity
- Example: Testing a user can login, add items to cart, checkout, and receive confirmation

**Code Example:**
```javascript
// Unit test - isolated function
test('calculateDiscount returns 10% off', () => {
  expect(calculateDiscount(100, 10)).toBe(10);
});

// Integration test - component + API
test('form submits data and shows success', async () => {
  render(<ContactForm />);
  await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
  await userEvent.click(screen.getByRole('button', { name: /submit/i }));
  expect(await screen.findByText(/success/i)).toBeInTheDocument();
});

// E2E test - full user journey
test('user can complete purchase', async ({ page }) => {
  await page.goto('/products');
  await page.click('text=Add to Cart');
  await page.click('text=Checkout');
  await page.fill('[name="card"]', '4242424242424242');
  await page.click('button:has-text("Pay")');
  await expect(page.locator('.confirmation')).toBeVisible();
});
```

The key is using the right type for each scenario: unit tests for logic, integration for behavior, and E2E for critical paths.

---

### Question 2: What is the testing pyramid and how does it differ from the testing trophy?

**Answer:**

Both are strategies for balancing different types of tests, but they prioritize differently:

**Testing Pyramid:**
- Base: 70% unit tests (fast, isolated, many)
- Middle: 20% integration tests (medium speed)
- Top: 10% E2E tests (slow, few, critical paths only)
- Philosophy: Build foundation with fast unit tests
- Best for: Backend/API applications with lots of business logic

**Testing Trophy (Kent C. Dodds):**
- Bottom: 10% static analysis (TypeScript, linting)
- Lower: 30% unit tests
- Middle (widest): 50% integration tests (most valuable)
- Top: 10% E2E tests
- Philosophy: "Write tests. Not too many. Mostly integration."
- Best for: Frontend applications where user interaction matters most

**Key Difference:**
```javascript
// Pyramid approach - emphasizes unit tests
const pyramidBreakdown = {
  unitTests: 700,        // 70%
  integrationTests: 200, // 20%
  e2eTests: 100         // 10%
};

// Trophy approach - emphasizes integration
const trophyBreakdown = {
  staticAnalysis: 100,   // 10%
  unitTests: 300,        // 30%
  integrationTests: 500, // 50% ⭐ Most important
  e2eTests: 100         // 10%
};
```

The trophy argues integration tests provide the best return on investment because they test how users actually interact with your application while remaining faster than E2E tests.

---

### Question 3: What is the AAA pattern in testing?

**Answer:**

AAA stands for **Arrange, Act, Assert** - a standard structure for writing clear, maintainable tests:

**Arrange:** Set up test data and conditions
**Act:** Execute the code being tested
**Assert:** Verify the expected outcome

**Example:**
```javascript
test('shopping cart calculates total correctly', () => {
  // ARRANGE - set up test data
  const cart = new ShoppingCart();
  const item1 = { name: 'Book', price: 20 };
  const item2 = { name: 'Pen', price: 5 };

  // ACT - perform the action
  cart.addItem(item1);
  cart.addItem(item2);

  // ASSERT - verify the result
  expect(cart.total).toBe(25);
  expect(cart.items).toHaveLength(2);
});
```

**Benefits:**
- Consistent structure across all tests
- Easy to understand test intent
- Clear separation of concerns
- Makes debugging easier

**BDD Variation (Given-When-Then):**
```javascript
test('user login flow', async () => {
  // GIVEN - user is on login page
  render(<LoginPage />);

  // WHEN - user enters credentials and submits
  await userEvent.type(screen.getByLabelText(/email/i), 'user@example.com');
  await userEvent.type(screen.getByLabelText(/password/i), 'password');
  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  // THEN - user sees dashboard
  expect(await screen.findByText(/welcome/i)).toBeInTheDocument();
});
```

---

### Question 4: What test coverage percentage should you aim for?

**Answer:**

**Short Answer:** 80% is a good target, but coverage percentage is not the goal - quality and meaningful tests are.

**Detailed Explanation:**

```javascript
const coverageGuidelines = {
  statement: '80-90%',  // Most common metric
  branch: '75-85%',     // Ensures all code paths tested
  function: '80-90%',
  line: '80-90%'
};
```

**Why 100% coverage is NOT the goal:**

1. **Diminishing returns** - Last 20% often requires testing trivial code
2. **False confidence** - High coverage doesn't mean good tests
3. **Maintenance burden** - Tests need updating when code changes

**Example of meaningless coverage:**
```javascript
// ❌ 100% coverage but useless test
function add(a, b) {
  return a + b;
}

test('add function exists', () => {
  expect(add).toBeDefined(); // ✅ Coverage but no value
});

// ✅ Lower coverage but valuable tests
test('add handles positive numbers', () => {
  expect(add(2, 3)).toBe(5);
});

test('add handles negative numbers', () => {
  expect(add(-2, 3)).toBe(1);
});

test('add handles decimals', () => {
  expect(add(0.1, 0.2)).toBeCloseTo(0.3);
});
```

**Better Metrics to Track:**
- Mutation test score (measures test quality)
- Code review feedback on tests
- Bug escape rate to production
- Test execution time
- Flaky test percentage

**Golden Rule:** 80% coverage with well-written, meaningful tests beats 100% coverage with brittle, implementation-focused tests.

---

### Question 5: When should you write tests - before or after writing code?

**Answer:**

**It depends** on the context, but both approaches have merit:

**Test-Driven Development (TDD) - Write Tests First:**

**When to use:**
- Building complex algorithms
- Implementing well-defined requirements
- Creating API endpoints
- When you know exactly what you need

**Process:**
```javascript
// 1. RED - Write failing test
test('validateEmail rejects invalid emails', () => {
  expect(validateEmail('invalid')).toBe(false);
}); // ❌ Fails - function doesn't exist

// 2. GREEN - Write minimal code to pass
function validateEmail(email) {
  return email.includes('@');
} // ✅ Passes

// 3. REFACTOR - Improve implementation
function validateEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
} // ✅ Still passes, better code
```

**Benefits:**
- Forces you to think about requirements first
- Ensures testable code design
- Prevents over-engineering
- Provides immediate feedback

**Test-After - Write Code First:**

**When to use:**
- Prototyping and exploring solutions
- UI/UX experimentation
- Spike/proof-of-concept work
- Learning new technologies

**Process:**
```javascript
// 1. Write working code
function calculateShipping(weight, distance) {
  const baseRate = 5;
  const weightCost = weight * 0.5;
  const distanceCost = distance * 0.1;
  return baseRate + weightCost + distanceCost;
}

// 2. Add tests to ensure it keeps working
describe('calculateShipping', () => {
  test('calculates basic shipping', () => {
    expect(calculateShipping(10, 100)).toBe(20);
  });

  test('handles edge cases', () => {
    expect(calculateShipping(0, 0)).toBe(5);
  });
});
```

**Pragmatic Approach:**
- Use TDD for critical business logic
- Write tests immediately after for new features
- Add tests when fixing bugs (regression tests)
- Always have tests before pushing to production

---

### Question 6: How do you test asynchronous code?

**Answer:**

Testing async code requires handling promises, callbacks, and timing properly. Modern testing frameworks provide several approaches:

**1. Async/Await (Recommended):**
```javascript
test('fetches user data', async () => {
  const user = await fetchUser(1);
  expect(user.name).toBe('John Doe');
});
```

**2. Using Promises:**
```javascript
test('fetches user data', () => {
  return fetchUser(1).then(user => {
    expect(user.name).toBe('John Doe');
  });
  // Must return the promise!
});
```

**3. React Testing Library - findBy Queries:**
```javascript
test('displays loaded data', async () => {
  render(<UserProfile userId={1} />);

  // findBy automatically waits for element
  const userName = await screen.findByText('John Doe');
  expect(userName).toBeInTheDocument();
});
```

**4. waitFor Helper:**
```javascript
test('error message appears after failed submission', async () => {
  render(<ContactForm />);

  await userEvent.click(screen.getByRole('button', { name: /submit/i }));

  await waitFor(() => {
    expect(screen.getByText(/error/i)).toBeInTheDocument();
  });
});
```

**5. Testing Timers:**
```javascript
test('shows notification after delay', () => {
  jest.useFakeTimers();

  const { getByText } = render(<DelayedNotification />);

  expect(queryByText('Notification')).not.toBeInTheDocument();

  jest.advanceTimersByTime(3000);

  expect(getByText('Notification')).toBeInTheDocument();

  jest.useRealTimers();
});
```

**Common Pitfalls:**
```javascript
// ❌ Forgetting to wait
test('displays data', () => {
  render(<AsyncComponent />);
  expect(screen.getByText('Data')).toBeInTheDocument(); // ❌ Fails!
});

// ✅ Properly waiting
test('displays data', async () => {
  render(<AsyncComponent />);
  expect(await screen.findByText('Data')).toBeInTheDocument(); // ✅
});

// ❌ Not returning promise
test('fetches data', () => {
  fetchData().then(data => {
    expect(data).toBeDefined();
  });
  // Test finishes before assertion! ❌
});

// ✅ Returning promise
test('fetches data', () => {
  return fetchData().then(data => {
    expect(data).toBeDefined();
  }); // ✅
});
```

---

### Question 7: What is the difference between mocks, stubs, and spies?

**Answer:**

These are all test doubles but serve different purposes:

**Mock:**
- Pre-programmed object with expectations
- Verifies interactions (was it called? with what arguments?)
- Fails test if expectations not met

```javascript
// Mock - verifies function was called correctly
test('saves user to database', () => {
  const mockSave = jest.fn();
  const user = { name: 'John' };

  saveUser(user, mockSave);

  expect(mockSave).toHaveBeenCalledWith(user);
  expect(mockSave).toHaveBeenCalledTimes(1);
});
```

**Stub:**
- Provides canned answers to calls
- Controls indirect inputs
- Doesn't verify interactions

```javascript
// Stub - provides fake data
test('displays user data', async () => {
  // Stub the API call
  jest.spyOn(api, 'getUser').mockResolvedValue({
    id: 1,
    name: 'John Doe'
  });

  render(<UserProfile userId={1} />);

  expect(await screen.findByText('John Doe')).toBeInTheDocument();
  // Don't care about HOW api.getUser was called
});
```

**Spy:**
- Records information about calls
- Lets real function execute
- Verifies behavior without changing it

```javascript
// Spy - monitors real function
test('logs user action', () => {
  const consoleSpy = jest.spyOn(console, 'log');

  performUserAction();

  expect(consoleSpy).toHaveBeenCalledWith('Action performed');

  consoleSpy.mockRestore();
});
```

**Comparison:**
```javascript
const apiService = {
  fetchData: () => fetch('/api/data').then(r => r.json())
};

// Mock - replace implementation AND verify calls
const mock = jest.fn(() => Promise.resolve({ data: 'mock' }));
apiService.fetchData = mock;
await apiService.fetchData();
expect(mock).toHaveBeenCalled(); // ✅ Verifies interaction

// Stub - replace implementation only
jest.spyOn(apiService, 'fetchData')
  .mockResolvedValue({ data: 'stub' });
const result = await apiService.fetchData();
expect(result.data).toBe('stub'); // ✅ Uses fake data

// Spy - keep implementation, monitor calls
const spy = jest.spyOn(apiService, 'fetchData');
await apiService.fetchData(); // ✅ Real function executes
expect(spy).toHaveBeenCalled(); // ✅ But we can verify it
```

**Rule of Thumb:**
- Use **mocks** when testing interactions
- Use **stubs** when controlling dependencies
- Use **spies** when monitoring real behavior

---

### Question 8: What makes a test "flaky" and how do you prevent it?

**Answer:**

A flaky test is one that **passes and fails inconsistently** without code changes - a major problem for CI/CD pipelines.

**Common Causes:**

**1. Timing Issues:**
```javascript
// ❌ Flaky - arbitrary timeout
test('shows notification', () => {
  render(<Notification />);

  setTimeout(() => {
    expect(screen.getByText('Alert')).toBeInTheDocument();
  }, 1000); // Might be too short on slow CI ❌
});

// ✅ Fixed - wait for element
test('shows notification', async () => {
  render(<Notification />);

  expect(await screen.findByText('Alert')).toBeInTheDocument();
  // Waits up to 1000ms by default ✅
});
```

**2. Shared State:**
```javascript
// ❌ Flaky - shared state
let counter = 0;

test('increments counter', () => {
  counter++;
  expect(counter).toBe(1); // ❌ Fails if tests run in parallel
});

// ✅ Fixed - isolated state
test('increments counter', () => {
  let counter = 0;
  counter++;
  expect(counter).toBe(1); // ✅
});
```

**3. Network Dependence:**
```javascript
// ❌ Flaky - real network request
test('fetches users', async () => {
  const users = await fetch('https://api.example.com/users');
  expect(users).toHaveLength(5); // ❌ API might be down
});

// ✅ Fixed - mock network
test('fetches users', async () => {
  jest.spyOn(global, 'fetch').mockResolvedValue({
    json: () => Promise.resolve([{ id: 1 }, { id: 2 }])
  });

  const users = await fetchUsers();
  expect(users).toHaveLength(2); // ✅
});
```

**4. Random Data:**
```javascript
// ❌ Flaky - random values
test('generates ID', () => {
  const id = generateId();
  expect(id).toBe(12345); // ❌ Random ID changes
});

// ✅ Fixed - seed randomness
test('generates ID', () => {
  jest.spyOn(Math, 'random').mockReturnValue(0.5);
  const id = generateId();
  expect(id).toBe(12345); // ✅ Predictable
});
```

**5. Test Order Dependency:**
```javascript
// ❌ Flaky - depends on test order
let user;

test('creates user', () => {
  user = { name: 'John' };
  expect(user).toBeDefined();
});

test('user has name', () => {
  expect(user.name).toBe('John'); // ❌ Fails if run first
});

// ✅ Fixed - independent tests
test('creates user', () => {
  const user = { name: 'John' };
  expect(user).toBeDefined();
});

test('user has name', () => {
  const user = { name: 'John' };
  expect(user.name).toBe('John'); // ✅
});
```

**Prevention Strategies:**
- Use proper async testing (async/await, waitFor)
- Isolate each test (no shared state)
- Mock external dependencies
- Avoid hardcoded timeouts
- Use faker with seeds for random data
- Ensure tests can run in any order
- Clean up after each test (afterEach)

---

### Question 9: How do you test error states and edge cases?

**Answer:**

Testing error states and edge cases is crucial because bugs often hide in unusual scenarios.

**Error State Testing:**

**1. API Errors:**
```javascript
test('displays error message when API fails', async () => {
  // Mock failed API call
  server.use(
    rest.get('/api/users', (req, res, ctx) => {
      return res(ctx.status(500), ctx.json({ error: 'Server error' }));
    })
  );

  render(<UserList />);

  expect(await screen.findByText(/failed to load/i)).toBeInTheDocument();
  expect(screen.queryByRole('list')).not.toBeInTheDocument();
});
```

**2. Form Validation Errors:**
```javascript
test('shows validation errors for invalid email', async () => {
  render(<SignupForm />);

  const emailInput = screen.getByLabelText(/email/i);
  const submitBtn = screen.getByRole('button', { name: /submit/i });

  await userEvent.type(emailInput, 'invalid-email');
  await userEvent.click(submitBtn);

  expect(await screen.findByText(/invalid email format/i))
    .toBeInTheDocument();
});
```

**3. Exception Handling:**
```javascript
test('handles divide by zero gracefully', () => {
  expect(() => divide(10, 0)).toThrow('Cannot divide by zero');
});

test('catches and displays unexpected errors', async () => {
  // Mock function that throws
  jest.spyOn(api, 'saveData').mockRejectedValue(
    new Error('Unexpected error')
  );

  render(<DataForm />);

  await userEvent.click(screen.getByRole('button', { name: /save/i }));

  expect(await screen.findByText(/something went wrong/i))
    .toBeInTheDocument();
});
```

**Edge Case Testing:**

**1. Boundary Values:**
```javascript
describe('pagination', () => {
  test('handles first page', () => {
    expect(getPagination(1, 10)).toEqual({ prev: null, next: 2 });
  });

  test('handles last page', () => {
    expect(getPagination(10, 10)).toEqual({ prev: 9, next: null });
  });

  test('handles single page', () => {
    expect(getPagination(1, 1)).toEqual({ prev: null, next: null });
  });
});
```

**2. Empty States:**
```javascript
test('shows empty state when no items', () => {
  render(<TodoList items={[]} />);
  expect(screen.getByText(/no todos yet/i)).toBeInTheDocument();
});

test('handles empty search results', async () => {
  render(<SearchResults query="nonexistent" />);
  expect(await screen.findByText(/no results found/i))
    .toBeInTheDocument();
});
```

**3. Null/Undefined:**
```javascript
test('handles missing user data gracefully', () => {
  render(<UserProfile user={null} />);
  expect(screen.getByText(/user not found/i)).toBeInTheDocument();
});

describe('formatName', () => {
  test('handles null', () => {
    expect(formatName(null)).toBe('');
  });

  test('handles undefined', () => {
    expect(formatName(undefined)).toBe('');
  });

  test('handles empty string', () => {
    expect(formatName('')).toBe('');
  });
});
```

**4. Maximum/Minimum Values:**
```javascript
describe('text input validation', () => {
  test('accepts minimum length', () => {
    expect(validateInput('ab')).toBe(true); // min: 2
  });

  test('accepts maximum length', () => {
    expect(validateInput('a'.repeat(100))).toBe(true); // max: 100
  });

  test('rejects too short', () => {
    expect(validateInput('a')).toBe(false);
  });

  test('rejects too long', () => {
    expect(validateInput('a'.repeat(101))).toBe(false);
  });
});
```

**Systematic Approach:**
```javascript
// Use test.each for comprehensive edge case coverage
describe('calculateDiscount', () => {
  test.each([
    // [price, discount, expected, description]
    [100, 0, 0, 'zero discount'],
    [100, 100, 100, 'full discount'],
    [0, 50, 0, 'zero price'],
    [100, 50, 50, 'normal case'],
    [-100, 10, null, 'negative price'],
    [100, -10, null, 'negative discount'],
    [100, 150, null, 'discount over 100%'],
  ])('calculateDiscount(%i, %i) = %s (%s)',
    (price, discount, expected, description) => {
      if (expected === null) {
        expect(() => calculateDiscount(price, discount)).toThrow();
      } else {
        expect(calculateDiscount(price, discount)).toBe(expected);
      }
    }
  );
});
```

---

### Question 10: What is the difference between white-box and black-box testing?

**Answer:**

These approaches differ in **how much knowledge of internal implementation** the tester has:

**Black-Box Testing:**
- Tests from **user's perspective**
- No knowledge of internal code
- Focuses on inputs and outputs
- Tests behavior, not implementation

```javascript
// Black-box test - treats component as a black box
test('login form submits credentials', async () => {
  render(<LoginForm />);

  // Don't care HOW it works internally
  await userEvent.type(screen.getByLabelText(/email/i), 'user@test.com');
  await userEvent.type(screen.getByLabelText(/password/i), 'password');
  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  // Only care about observable outcome
  expect(await screen.findByText(/welcome/i)).toBeInTheDocument();
});
```

**White-Box Testing:**
- Tests with **full knowledge of internals**
- Tests code paths, branches, logic
- Focuses on implementation details
- Common for unit tests

```javascript
// White-box test - knows internal structure
test('login form validates before submission', () => {
  const mockValidate = jest.fn();
  const mockSubmit = jest.fn();

  render(<LoginForm validate={mockValidate} onSubmit={mockSubmit} />);

  // Testing internal flow: validate → submit
  userEvent.click(screen.getByRole('button', { name: /login/i }));

  expect(mockValidate).toHaveBeenCalled();
  expect(mockValidate).toHaveBeenCalledBefore(mockSubmit);
});

// Testing internal logic branches
describe('calculatePrice', () => {
  test('uses premium discount if user is premium', () => {
    const price = calculatePrice(100, { isPremium: true });
    // We know internally it should apply 20% discount
    expect(price).toBe(80);
  });

  test('uses regular discount if user is not premium', () => {
    const price = calculatePrice(100, { isPremium: false });
    // We know internally it should apply 10% discount
    expect(price).toBe(90);
  });
});
```

**Comparison:**

| Aspect | Black-Box | White-Box |
|--------|-----------|-----------|
| **Knowledge** | No internal code knowledge | Full code knowledge |
| **Focus** | External behavior | Internal logic |
| **Tester** | Can be non-developer | Requires developer |
| **Coverage** | Functional requirements | Code coverage |
| **Brittleness** | More robust to refactoring | Breaks with refactoring |
| **Examples** | E2E, acceptance tests | Unit tests, branch testing |

**In Practice - Combine Both:**
```javascript
// Black-box approach (Recommended for React Testing Library)
test('user can filter products', async () => {
  render(<ProductList />);

  // User perspective - what they see and do
  await userEvent.click(screen.getByRole('button', { name: /filter/i }));
  await userEvent.click(screen.getByLabelText(/electronics/i));

  expect(screen.getByText('Laptop')).toBeInTheDocument();
  expect(screen.queryByText('T-Shirt')).not.toBeInTheDocument();
});

// White-box approach (for complex logic)
test('filter function correctly filters array', () => {
  const products = [
    { name: 'Laptop', category: 'electronics' },
    { name: 'T-Shirt', category: 'clothing' }
  ];

  // Testing internal function with knowledge of implementation
  const filtered = filterByCategory(products, 'electronics');

  expect(filtered).toHaveLength(1);
  expect(filtered[0].name).toBe('Laptop');
});
```

**Best Practice:**
- Use **black-box** for integration and E2E tests
- Use **white-box** for unit testing complex algorithms
- Prefer black-box when possible (more maintainable)
- Use white-box when you need full code coverage

**React Testing Library encourages black-box testing:**
```javascript
// ✅ Recommended - black-box
test('shows loading spinner while fetching', async () => {
  render(<DataComponent />);
  expect(screen.getByRole('status')).toBeInTheDocument();
  await waitFor(() => {
    expect(screen.queryByRole('status')).not.toBeInTheDocument();
  });
});

// ❌ Discouraged - white-box
test('component uses loading state', () => {
  const { result } = renderHook(() => useState(true));
  expect(result.current[0]).toBe(true); // Testing React internals
});
```

---

## Summary

Testing fundamentals form the foundation of reliable software development. Key takeaways:

**Core Concepts:**
- **Three test types:** Unit (isolation), Integration (collaboration), E2E (full flow)
- **Testing pyramid/trophy:** Balance different test types for optimal coverage
- **AAA pattern:** Structure tests with Arrange, Act, Assert
- **Coverage metrics:** Aim for 80% but prioritize quality over quantity

**Testing Philosophy:**
- Test behavior, not implementation
- Write tests for confidence, not coverage
- Think like a user, not a developer
- Test edge cases and error states
- Keep tests simple, readable, and maintainable

**When to Write Tests:**
- Unit tests: Pure functions, business logic, algorithms
- Integration tests: Component interactions, forms, workflows
- E2E tests: Critical user journeys, authentication, checkout

**Common Mistakes to Avoid:**
- Testing implementation details
- Ignoring async behavior
- Flaky tests from timing issues
- Over-testing trivial code
- Not cleaning up between tests

**Next Steps:**
- Master Jest for test execution and mocking
- Learn React Testing Library for component testing
- Practice writing integration tests
- Understand E2E testing tools

Remember: Good tests give you confidence to refactor, deploy, and evolve your codebase safely.

---

**Next:** [Jest Basics →](./02-jest-basics.md)

**Previous:** [← Testing README](./README.md)

---

[← Back to Testing Interview Prep](./README.md) | [↑ Back to Frontend](../README.md)
