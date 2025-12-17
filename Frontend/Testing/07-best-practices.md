# Testing Best Practices

## Overview

Writing effective tests is a skill that goes beyond just making assertions pass. Professional testing requires understanding test organization, maintaining test quality, avoiding common pitfalls, and building tests that provide confidence without becoming maintenance burdens. This guide covers battle-tested practices that separate senior developers from juniors and ensure your test suite remains valuable as your codebase grows.

## Table of Contents
- [Test Organization and Structure](#test-organization-and-structure)
- [Naming Conventions](#naming-conventions)
- [Test Isolation and Independence](#test-isolation-and-independence)
- [Avoiding Flaky Tests](#avoiding-flaky-tests)
- [Testing Anti-Patterns](#testing-anti-patterns)
- [Performance Optimization](#performance-optimization)
- [Continuous Testing Strategies](#continuous-testing-strategies)
- [Code Coverage Best Practices](#code-coverage-best-practices)
- [Maintainable Tests](#maintainable-tests)
- [Interview Questions](#interview-questions)

## Test Organization and Structure

### File Structure

```javascript
// ✅ Good: Organized by feature
src/
├── components/
│   ├── UserProfile/
│   │   ├── UserProfile.tsx
│   │   ├── UserProfile.test.tsx
│   │   └── UserProfile.module.css
│   ├── LoginForm/
│   │   ├── LoginForm.tsx
│   │   ├── LoginForm.test.tsx
│   │   └── LoginForm.module.css
├── hooks/
│   ├── useAuth.ts
│   └── useAuth.test.ts
├── utils/
│   ├── formatters.ts
│   └── formatters.test.ts

// ❌ Bad: Separated test directories
src/
├── components/
│   ├── UserProfile.tsx
│   └── LoginForm.tsx
├── tests/
│   ├── UserProfile.test.tsx
│   └── LoginForm.test.tsx
// Tests are far from implementation
```

### Test Suite Structure

```javascript
// ✅ Good: Well-organized test suite
describe('ShoppingCart', () => {
  // Setup
  let cart;

  beforeEach(() => {
    cart = new ShoppingCart();
  });

  // Group related tests
  describe('initialization', () => {
    test('starts with zero items', () => {
      expect(cart.itemCount()).toBe(0);
    });

    test('has zero total', () => {
      expect(cart.total()).toBe(0);
    });
  });

  describe('adding items', () => {
    test('increases item count', () => {
      cart.addItem({ id: 1, price: 100 });
      expect(cart.itemCount()).toBe(1);
    });

    test('updates total price', () => {
      cart.addItem({ id: 1, price: 100 });
      cart.addItem({ id: 2, price: 50 });
      expect(cart.total()).toBe(150);
    });

    test('throws error for invalid item', () => {
      expect(() => cart.addItem(null)).toThrow('Invalid item');
    });
  });

  describe('removing items', () => {
    beforeEach(() => {
      cart.addItem({ id: 1, price: 100 });
      cart.addItem({ id: 2, price: 50 });
    });

    test('decreases item count', () => {
      cart.removeItem(1);
      expect(cart.itemCount()).toBe(1);
    });

    test('updates total price', () => {
      cart.removeItem(1);
      expect(cart.total()).toBe(50);
    });
  });

  describe('edge cases', () => {
    test('handles removing non-existent item', () => {
      expect(() => cart.removeItem(999)).toThrow('Item not found');
    });

    test('prevents negative quantities', () => {
      expect(() => cart.addItem({ id: 1, quantity: -1 })).toThrow();
    });
  });
});

// ❌ Bad: Flat, unorganized
describe('ShoppingCart', () => {
  test('test1', () => {});
  test('test2', () => {});
  test('test3', () => {});
  // No logical grouping, hard to navigate
});
```

### AAA Pattern (Arrange-Act-Assert)

```javascript
// ✅ Good: Clear AAA structure
test('user can update profile', () => {
  // Arrange - Set up test data
  const user = {
    id: '123',
    name: 'John Doe',
    email: 'john@example.com'
  };
  const userService = new UserService(mockDatabase);

  // Act - Perform the action
  const updatedUser = userService.updateProfile(user.id, {
    name: 'Jane Smith'
  });

  // Assert - Verify the outcome
  expect(updatedUser.name).toBe('Jane Smith');
  expect(mockDatabase.update).toHaveBeenCalledWith('123', {
    name: 'Jane Smith'
  });
});

// ❌ Bad: Mixed concerns
test('user can update profile', () => {
  const user = { id: '123', name: 'John' };
  expect(updateProfile(user.id, { name: 'Jane' })).toBeDefined();
  const service = new UserService();
  // Arrange/Act/Assert all mixed up
});
```

### Helper Functions

```javascript
// ✅ Good: Reusable test helpers
describe('UserProfile', () => {
  // Helper function
  function renderUserProfile(props = {}) {
    const defaultProps = {
      user: { name: 'John', email: 'john@example.com' },
      onEdit: jest.fn(),
      onDelete: jest.fn()
    };

    return render(<UserProfile {...defaultProps} {...props} />);
  }

  test('displays user name', () => {
    renderUserProfile();
    expect(screen.getByText('John')).toBeInTheDocument();
  });

  test('calls onEdit when edit button clicked', () => {
    const onEdit = jest.fn();
    renderUserProfile({ onEdit });

    fireEvent.click(screen.getByText('Edit'));
    expect(onEdit).toHaveBeenCalled();
  });
});

// Create test utilities file for common helpers
// __tests__/utils/testUtils.js
export function createMockUser(overrides = {}) {
  return {
    id: '123',
    name: 'Test User',
    email: 'test@example.com',
    ...overrides
  };
}

export function renderWithProviders(ui, options = {}) {
  return render(
    <AuthProvider>
      <ThemeProvider>
        {ui}
      </ThemeProvider>
    </AuthProvider>,
    options
  );
}
```

## Naming Conventions

### Test Names

```javascript
// ✅ Good: Descriptive, action-based names
test('displays error message when email is invalid', () => {});
test('calls onSubmit with form data when form is submitted', () => {});
test('disables submit button while request is pending', () => {});
test('redirects to dashboard after successful login', () => {});

// ❌ Bad: Vague or implementation-focused names
test('test1', () => {});
test('it works', () => {});
test('button test', () => {});
test('state is set correctly', () => {}); // Testing implementation
```

### Naming Pattern

```javascript
// Pattern: [Action/State] + [Expected Outcome]

// User actions
test('clicking logout button logs out user', () => {});
test('submitting form with valid data creates new user', () => {});

// State descriptions
test('loading spinner is visible while data is fetching', () => {});
test('error boundary displays fallback UI when error occurs', () => {});

// Edge cases
test('handles empty search results gracefully', () => {});
test('prevents duplicate form submissions', () => {});

// BDD-style naming
describe('UserAuthentication', () => {
  describe('when credentials are valid', () => {
    it('should log in the user', () => {});
    it('should redirect to dashboard', () => {});
    it('should save auth token', () => {});
  });

  describe('when credentials are invalid', () => {
    it('should display error message', () => {});
    it('should not redirect', () => {});
    it('should clear password field', () => {});
  });
});
```

### File Naming

```javascript
// ✅ Good: Consistent naming convention
UserProfile.test.tsx          // Component tests
UserProfile.integration.test.tsx  // Integration tests
useAuth.test.ts               // Hook tests
formatters.test.ts            // Utility tests
api.test.ts                   // API tests

// ❌ Bad: Inconsistent naming
UserProfile.spec.tsx
UserProfile-test.tsx
test-UserProfile.tsx
UserProfileTests.tsx
```

## Test Isolation and Independence

### Independent Tests

```javascript
// ✅ Good: Each test is independent
describe('Counter', () => {
  test('increments from 0 to 1', () => {
    const counter = new Counter();
    counter.increment();
    expect(counter.value).toBe(1);
  });

  test('decrements from 0 to -1', () => {
    const counter = new Counter(); // Fresh instance
    counter.decrement();
    expect(counter.value).toBe(-1);
  });
});

// ❌ Bad: Tests depend on each other
describe('Counter', () => {
  let counter = new Counter();

  test('increments to 1', () => {
    counter.increment();
    expect(counter.value).toBe(1);
  });

  test('increments to 2', () => {
    counter.increment(); // Depends on previous test!
    expect(counter.value).toBe(2);
  });
});
```

### Setup and Teardown

```javascript
// ✅ Good: Clean setup/teardown
describe('DatabaseService', () => {
  let db;
  let service;

  beforeEach(() => {
    // Fresh state for each test
    db = new MockDatabase();
    service = new DatabaseService(db);
  });

  afterEach(() => {
    // Clean up after each test
    db.clear();
  });

  test('saves user successfully', async () => {
    await service.saveUser({ name: 'John' });
    expect(db.users).toHaveLength(1);
  });

  test('updates existing user', async () => {
    await service.saveUser({ id: '1', name: 'John' });
    await service.saveUser({ id: '1', name: 'Jane' });
    expect(db.users).toHaveLength(1); // Not 2
  });
});

// ❌ Bad: Shared state between tests
describe('DatabaseService', () => {
  const db = new MockDatabase(); // Shared!
  const service = new DatabaseService(db);

  test('saves user', async () => {
    await service.saveUser({ name: 'John' });
    expect(db.users).toHaveLength(1);
  });

  test('saves another user', async () => {
    await service.saveUser({ name: 'Jane' });
    expect(db.users).toHaveLength(2); // Depends on first test!
  });
});
```

### Avoiding Global State

```javascript
// ✅ Good: No global state
test('formatDate works correctly', () => {
  const date = new Date('2024-01-15');
  const formatted = formatDate(date, 'en-US');
  expect(formatted).toBe('1/15/2024');
});

// ❌ Bad: Global state
let currentLocale = 'en-US'; // Global variable

test('formatDate works in English', () => {
  currentLocale = 'en-US';
  expect(formatDate(new Date())).toContain('/');
});

test('formatDate works in French', () => {
  currentLocale = 'fr-FR'; // Modifies global state
  expect(formatDate(new Date())).toContain('/');
});
```

## Avoiding Flaky Tests

### Proper Async Handling

```javascript
// ✅ Good: Proper async/await
test('loads user data', async () => {
  render(<UserProfile userId="123" />);

  // Wait for async operation
  await waitFor(() => {
    expect(screen.getByText('John Doe')).toBeInTheDocument();
  });
});

// ✅ Good: Using findBy queries (auto-wait)
test('displays error message', async () => {
  render(<LoginForm />);
  fireEvent.click(screen.getByText('Login'));

  // findBy waits automatically
  const error = await screen.findByText('Invalid credentials');
  expect(error).toBeInTheDocument();
});

// ❌ Bad: Arbitrary timeouts
test('loads user data', (done) => {
  render(<UserProfile userId="123" />);

  setTimeout(() => {
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    done();
  }, 1000); // Fragile! What if it takes 1001ms?
});

// ❌ Bad: Not waiting for async
test('displays success message', () => {
  render(<Form />);
  fireEvent.click(screen.getByText('Submit'));

  // Doesn't wait! Test may pass/fail randomly
  expect(screen.getByText('Success')).toBeInTheDocument();
});
```

### Stable Selectors

```javascript
// ✅ Good: Stable selectors
test('user can submit form', () => {
  render(<LoginForm />);

  // data-testid won't change
  fireEvent.change(screen.getByTestId('email-input'), {
    target: { value: 'user@example.com' }
  });

  // ARIA roles are stable
  fireEvent.click(screen.getByRole('button', { name: 'Login' }));
});

// ❌ Bad: Fragile selectors
test('user can submit form', () => {
  render(<LoginForm />);

  // CSS classes can change
  fireEvent.change(document.querySelector('.input-primary.email'), {
    target: { value: 'user@example.com' }
  });

  // Position-based selectors break easily
  fireEvent.click(document.querySelectorAll('button')[2]);
});
```

### Avoiding Race Conditions

```javascript
// ✅ Good: Proper sequencing
test('sequential actions work correctly', async () => {
  render(<App />);

  // Wait for first action to complete
  fireEvent.click(screen.getByText('Load Data'));
  await screen.findByText('Data loaded');

  // Then perform second action
  fireEvent.click(screen.getByText('Process Data'));
  await screen.findByText('Processing complete');
});

// ❌ Bad: Race condition
test('actions work', async () => {
  render(<App />);

  fireEvent.click(screen.getByText('Load Data'));
  fireEvent.click(screen.getByText('Process Data')); // May execute before load completes!

  // Unpredictable results
});
```

### Deterministic Test Data

```javascript
// ✅ Good: Fixed test data
test('sorts users by name', () => {
  const users = [
    { id: 1, name: 'Charlie' },
    { id: 2, name: 'Alice' },
    { id: 3, name: 'Bob' }
  ];

  const sorted = sortUsers(users);

  expect(sorted[0].name).toBe('Alice');
  expect(sorted[1].name).toBe('Bob');
  expect(sorted[2].name).toBe('Charlie');
});

// ❌ Bad: Random test data
test('sorts users by name', () => {
  const users = generateRandomUsers(); // Different each run!

  const sorted = sortUsers(users);

  expect(sorted[0].name).toBeDefined(); // Weak assertion
});
```

## Testing Anti-Patterns

### 1. Testing Implementation Details

```javascript
// ❌ Bad: Testing internal state
test('counter increments state', () => {
  const wrapper = shallow(<Counter />);
  wrapper.find('button').simulate('click');

  expect(wrapper.state('count')).toBe(1); // Implementation detail!
});

// ✅ Good: Testing user-visible behavior
test('counter displays incremented value', () => {
  render(<Counter />);
  fireEvent.click(screen.getByText('Increment'));

  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

### 2. Too Many Mocks

```javascript
// ❌ Bad: Over-mocking
jest.mock('./api');
jest.mock('./utils');
jest.mock('./helpers');
jest.mock('./validators');
jest.mock('./formatters');
// Testing nothing real!

test('user registration', async () => {
  // All mocked, no real behavior tested
  await registerUser('user@example.com', 'password');
  expect(mockApi.post).toHaveBeenCalled();
});

// ✅ Good: Mock only external dependencies
jest.mock('./api'); // Mock external API only

test('user registration validates email', async () => {
  // Real validation, mocked API
  await expect(
    registerUser('invalid', 'password')
  ).rejects.toThrow('Invalid email');
});
```

### 3. Testing Multiple Things

```javascript
// ❌ Bad: Testing everything in one test
test('login flow', async () => {
  render(<App />);

  // Too many assertions
  expect(screen.getByText('Login')).toBeInTheDocument();
  fireEvent.click(screen.getByText('Login'));
  expect(screen.getByLabelText('Email')).toBeInTheDocument();
  fireEvent.change(screen.getByLabelText('Email'), {
    target: { value: 'user@example.com' }
  });
  expect(screen.getByLabelText('Email')).toHaveValue('user@example.com');
  // ... 20 more lines
  expect(screen.getByText('Dashboard')).toBeInTheDocument();
});

// ✅ Good: One test, one concern
test('login form displays email input', () => {
  render(<LoginForm />);
  expect(screen.getByLabelText('Email')).toBeInTheDocument();
});

test('email input accepts user input', () => {
  render(<LoginForm />);
  const input = screen.getByLabelText('Email');
  fireEvent.change(input, { target: { value: 'user@example.com' } });
  expect(input).toHaveValue('user@example.com');
});

test('redirects to dashboard after successful login', async () => {
  render(<App />);
  await performLogin('user@example.com', 'password');
  expect(screen.getByText('Dashboard')).toBeInTheDocument();
});
```

### 4. Fragile Tests

```javascript
// ❌ Bad: Depends on text content
test('displays welcome message', () => {
  render(<Home />);
  expect(screen.getByText('Welcome to our amazing platform!')).toBeInTheDocument();
  // Breaks if marketing changes text!
});

// ✅ Good: Tests meaningful behavior
test('displays welcome message', () => {
  render(<Home />);
  expect(screen.getByRole('heading', { level: 1 })).toContainText('Welcome');
  // More resilient
});
```

### 5. Duplicate Tests

```javascript
// ❌ Bad: Redundant tests
test('add returns sum', () => {
  expect(add(2, 3)).toBe(5);
});

test('add adds numbers', () => {
  expect(add(2, 3)).toBe(5); // Same test!
});

test('add function works', () => {
  expect(add(2, 3)).toBe(5); // Again!
});

// ✅ Good: Distinct test cases
test('add returns sum of positive numbers', () => {
  expect(add(2, 3)).toBe(5);
});

test('add handles negative numbers', () => {
  expect(add(-2, 3)).toBe(1);
});

test('add handles zero', () => {
  expect(add(0, 5)).toBe(5);
});
```

## Performance Optimization

### Parallel Test Execution

```javascript
// jest.config.js
module.exports = {
  // Run tests in parallel
  maxWorkers: '50%', // Use 50% of CPU cores

  // Or specific number
  maxWorkers: 4,

  // Disable for debugging
  maxWorkers: 1
};
```

### Test Splitting

```javascript
// package.json
{
  "scripts": {
    "test": "jest",
    "test:unit": "jest --testPathPattern='\\.test\\.(ts|tsx)$'",
    "test:integration": "jest --testPathPattern='\\.integration\\.test\\.(ts|tsx)$'",
    "test:e2e": "playwright test"
  }
}

// Run fast tests frequently, slow tests less often
```

### Efficient Setup

```javascript
// ❌ Bad: Expensive setup in every test
describe('UserService', () => {
  test('creates user', async () => {
    const db = await createDatabaseConnection(); // Slow!
    await db.migrate(); // Slow!
    const service = new UserService(db);

    await service.createUser({ name: 'John' });
  });

  test('updates user', async () => {
    const db = await createDatabaseConnection(); // Slow!
    await db.migrate(); // Slow!
    const service = new UserService(db);

    await service.updateUser('1', { name: 'Jane' });
  });
});

// ✅ Good: Shared setup with cleanup
describe('UserService', () => {
  let db;
  let service;

  beforeAll(async () => {
    // One-time expensive setup
    db = await createDatabaseConnection();
    await db.migrate();
  });

  beforeEach(() => {
    // Fast per-test setup
    service = new UserService(db);
  });

  afterEach(async () => {
    // Clean data, not connection
    await db.clear();
  });

  afterAll(async () => {
    // Cleanup connection
    await db.close();
  });

  test('creates user', async () => {
    await service.createUser({ name: 'John' });
  });

  test('updates user', async () => {
    await service.updateUser('1', { name: 'Jane' });
  });
});
```

### Mocking Heavy Dependencies

```javascript
// ✅ Good: Mock slow operations
jest.mock('./emailService', () => ({
  sendEmail: jest.fn().mockResolvedValue({ success: true })
}));

test('user registration sends welcome email', async () => {
  await registerUser('user@example.com', 'password');

  // Fast mock instead of real email service
  expect(emailService.sendEmail).toHaveBeenCalledWith(
    'user@example.com',
    'Welcome!'
  );
});
```

### Test Timeouts

```javascript
// Configure global timeout
// jest.config.js
module.exports = {
  testTimeout: 5000, // 5 seconds default
};

// Override for specific test
test('slow operation', async () => {
  await performSlowOperation();
}, 30000); // 30 second timeout

// Or for entire suite
describe('Integration tests', () => {
  jest.setTimeout(30000);

  test('database migration', async () => {
    await runMigration();
  });
});
```

## Continuous Testing Strategies

### Watch Mode

```javascript
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:watch:all": "jest --watchAll"
  }
}

// jest.config.js - Configure watch mode
module.exports = {
  watchPlugins: [
    'jest-watch-typeahead/filename',
    'jest-watch-typeahead/testname'
  ]
};
```

### Pre-commit Hooks

```javascript
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "jest --bail --findRelatedTests"
    ]
  }
}

// Only runs tests for changed files
```

### CI/CD Integration

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm run test:unit

      - name: Run integration tests
        run: npm run test:integration

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json

      - name: Comment coverage
        uses: romeovs/lcov-reporter-action@v0.3.1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          lcov-file: ./coverage/lcov.info
```

### Test Matrices

```yaml
# Test across multiple versions
jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16, 18, 20]
        react-version: [17, 18]

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}

      - run: npm install react@${{ matrix.react-version }}
      - run: npm test
```

## Code Coverage Best Practices

### Coverage Configuration

```javascript
// jest.config.js
module.exports = {
  collectCoverage: true,
  coverageDirectory: 'coverage',

  // What to include
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.tsx',
    '!src/index.tsx'
  ],

  // Coverage thresholds
  coverageThresholds: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    // Stricter for critical files
    './src/utils/': {
      branches: 90,
      functions: 95,
      lines: 90,
      statements: 90
    }
  },

  // Reporter types
  coverageReporters: ['text', 'lcov', 'html'],
};
```

### Meaningful Coverage

```javascript
// ❌ Bad: 100% coverage, poor tests
function divide(a, b) {
  return a / b;
}

test('divide returns result', () => {
  divide(10, 2); // No assertion!
  // 100% coverage but tests nothing
});

// ✅ Good: Good coverage with meaningful tests
function divide(a, b) {
  if (b === 0) {
    throw new Error('Division by zero');
  }
  return a / b;
}

test('divide returns quotient', () => {
  expect(divide(10, 2)).toBe(5);
});

test('divide handles decimals', () => {
  expect(divide(5, 2)).toBe(2.5);
});

test('divide throws error for zero divisor', () => {
  expect(() => divide(10, 0)).toThrow('Division by zero');
});
```

### Coverage Gaps

```javascript
// Identify untested code
// Run: npm test -- --coverage

// Example output:
/*
File                | % Stmts | % Branch | % Funcs | % Lines | Uncovered Lines
--------------------|---------|----------|---------|---------|----------------
src/utils/math.ts   |   80.00 |    66.67 |   75.00 |   80.00 | 15-20
*/

// View HTML report
// open coverage/lcov-report/index.html

// Focus on critical paths, not 100%
const priorities = {
  high: 'Business logic, security, data handling',
  medium: 'UI components, utils',
  low: 'Constants, types, simple getters'
};
```

## Maintainable Tests

### DRY Principle

```javascript
// ✅ Good: Reusable test utilities
const mockUser = {
  id: '123',
  name: 'John Doe',
  email: 'john@example.com'
};

function renderUserProfile(props = {}) {
  return render(
    <UserProfile user={mockUser} {...props} />
  );
}

test('displays user name', () => {
  renderUserProfile();
  expect(screen.getByText('John Doe')).toBeInTheDocument();
});

test('displays user email', () => {
  renderUserProfile();
  expect(screen.getByText('john@example.com')).toBeInTheDocument();
});

// ❌ Bad: Repeated setup
test('displays user name', () => {
  const user = {
    id: '123',
    name: 'John Doe',
    email: 'john@example.com'
  };
  render(<UserProfile user={user} />);
  expect(screen.getByText('John Doe')).toBeInTheDocument();
});

test('displays user email', () => {
  const user = {
    id: '123',
    name: 'John Doe',
    email: 'john@example.com'
  };
  render(<UserProfile user={user} />);
  expect(screen.getByText('john@example.com')).toBeInTheDocument();
});
```

### Self-Documenting Tests

```javascript
// ✅ Good: Clear and self-explanatory
test('shopping cart calculates correct total with discount', () => {
  const cart = new ShoppingCart();

  // Given: Cart has items
  cart.addItem({ name: 'Laptop', price: 1000 });
  cart.addItem({ name: 'Mouse', price: 50 });

  // When: 10% discount applied
  cart.applyDiscount(10);

  // Then: Total is 945 (1050 - 10%)
  expect(cart.total()).toBe(945);
});

// Comments only when needed for clarity
test('validates email with special characters', () => {
  // Plus addressing is valid per RFC 5322
  expect(validateEmail('user+tag@example.com')).toBe(true);
});
```

### Test Data Builders

```javascript
// Test data builder pattern
class UserBuilder {
  constructor() {
    this.user = {
      id: '123',
      name: 'Test User',
      email: 'test@example.com',
      role: 'user',
      isActive: true
    };
  }

  withName(name) {
    this.user.name = name;
    return this;
  }

  withEmail(email) {
    this.user.email = email;
    return this;
  }

  asAdmin() {
    this.user.role = 'admin';
    return this;
  }

  inactive() {
    this.user.isActive = false;
    return this;
  }

  build() {
    return this.user;
  }
}

// Usage
test('admin can delete users', () => {
  const admin = new UserBuilder()
    .withName('Admin User')
    .asAdmin()
    .build();

  expect(canDeleteUsers(admin)).toBe(true);
});

test('inactive users cannot login', () => {
  const user = new UserBuilder()
    .inactive()
    .build();

  expect(canLogin(user)).toBe(false);
});
```

## Interview Questions

**Q1: What is test isolation and why is it important?**

A: Test isolation means each test runs independently without affecting or being affected by other tests.

**Why it matters:**
- **Reliable results:** Test order doesn't matter
- **Easier debugging:** Failures are localized
- **Parallel execution:** Tests can run simultaneously
- **Maintainability:** Tests don't have hidden dependencies

**Example:**
```javascript
// ✅ Good: Isolated tests
describe('Counter', () => {
  test('increments to 1', () => {
    const counter = new Counter(); // Fresh instance
    counter.increment();
    expect(counter.value).toBe(1);
  });

  test('decrements to -1', () => {
    const counter = new Counter(); // Fresh instance
    counter.decrement();
    expect(counter.value).toBe(-1);
  });
});

// ❌ Bad: Shared state
describe('Counter', () => {
  const counter = new Counter(); // Shared!

  test('increments to 1', () => {
    counter.increment();
    expect(counter.value).toBe(1);
  });

  test('increments to 2', () => {
    counter.increment(); // Depends on previous test!
    expect(counter.value).toBe(2); // Fails if run alone
  });
});
```

**How to achieve isolation:**
- Use `beforeEach` for setup
- Don't share variables between tests
- Clean up in `afterEach`
- Mock external dependencies

**Q2: How do you avoid flaky tests?**

A: Flaky tests pass/fail randomly. Avoid them with these practices:

**1. Proper async handling:**
```javascript
// ❌ Flaky: Timing-dependent
test('shows message', () => {
  render(<App />);
  setTimeout(() => {
    expect(screen.getByText('Hello')).toBeInTheDocument();
  }, 1000); // May fail if takes 1001ms
});

// ✅ Reliable: Wait for condition
test('shows message', async () => {
  render(<App />);
  await screen.findByText('Hello'); // Waits automatically
});
```

**2. Stable selectors:**
```javascript
// ❌ Flaky: CSS classes change
fireEvent.click(document.querySelector('.btn-primary'));

// ✅ Reliable: data-testid
fireEvent.click(screen.getByTestId('submit-button'));
```

**3. No race conditions:**
```javascript
// ✅ Wait for each step
fireEvent.click(screen.getByText('Load'));
await screen.findByText('Loaded');
fireEvent.click(screen.getByText('Process'));
await screen.findByText('Done');
```

**4. Deterministic data:**
```javascript
// ❌ Random data
const user = generateRandomUser();

// ✅ Fixed data
const user = { id: '123', name: 'John' };
```

**Q3: What are common testing anti-patterns?**

A: **1. Testing implementation details:**
```javascript
// ❌ Bad
expect(wrapper.state('count')).toBe(1);

// ✅ Good
expect(screen.getByText('Count: 1')).toBeInTheDocument();
```

**2. Over-mocking:**
```javascript
// ❌ Bad: Mock everything
jest.mock('./api');
jest.mock('./utils');
jest.mock('./helpers');

// ✅ Good: Mock only external dependencies
jest.mock('./api');
```

**3. Multiple assertions:**
```javascript
// ❌ Bad: Testing too much
test('login flow', () => {
  // 50 lines of setup and assertions
});

// ✅ Good: One concern per test
test('displays login form', () => {});
test('validates email format', () => {});
test('redirects after login', () => {});
```

**4. Fragile selectors:**
```javascript
// ❌ Bad
document.querySelector('div > button:nth-child(3)');

// ✅ Good
screen.getByRole('button', { name: 'Submit' });
```

**Q4: How should tests be organized?**

A: **File structure:**
```
src/
├── components/
│   ├── UserProfile/
│   │   ├── UserProfile.tsx
│   │   └── UserProfile.test.tsx    ← Co-located
```

**Test suite structure:**
```javascript
describe('ShoppingCart', () => {
  // Group by functionality
  describe('initialization', () => {
    test('starts empty', () => {});
  });

  describe('adding items', () => {
    test('increases count', () => {});
    test('updates total', () => {});
  });

  describe('edge cases', () => {
    test('handles invalid item', () => {});
  });
});
```

**AAA pattern:**
```javascript
test('user can checkout', () => {
  // Arrange
  const cart = new ShoppingCart();
  cart.addItem({ price: 100 });

  // Act
  const result = cart.checkout();

  // Assert
  expect(result.total).toBe(100);
});
```

**Q5: What makes a good test name?**

A: Good test names are descriptive and action-based:

**✅ Good names:**
```javascript
test('displays error message when email is invalid', () => {});
test('calls onSubmit with form data when form is submitted', () => {});
test('disables button while request is pending', () => {});
```

**❌ Bad names:**
```javascript
test('test1', () => {});
test('it works', () => {});
test('button', () => {});
```

**Pattern:** `[Action/State] + [Expected Outcome]`

**BDD style:**
```javascript
describe('when user submits form', () => {
  describe('with valid data', () => {
    it('should create new user', () => {});
    it('should redirect to dashboard', () => {});
  });

  describe('with invalid data', () => {
    it('should display error message', () => {});
  });
});
```

**Q6: How do you improve test performance?**

A: **1. Parallel execution:**
```javascript
// jest.config.js
maxWorkers: '50%', // Use 50% of CPU cores
```

**2. Efficient setup:**
```javascript
beforeAll(async () => {
  // Expensive one-time setup
  db = await createConnection();
});

beforeEach(() => {
  // Fast per-test setup
  service = new UserService(db);
});

afterEach(() => {
  // Clean data, not connection
  db.clear();
});
```

**3. Mock slow operations:**
```javascript
jest.mock('./emailService');
// Fast mock instead of real email service
```

**4. Split test suites:**
```json
{
  "scripts": {
    "test:unit": "jest --testPathPattern='\\.test\\.ts$'",
    "test:integration": "jest --testPathPattern='\\.integration\\.test\\.ts$'"
  }
}
```

**5. Watch mode:**
```bash
npm test -- --watch
# Only re-runs affected tests
```

**Q7: What is code coverage and how should it be used?**

A: Code coverage measures how much code is executed by tests.

**Types:**
- **Lines:** % of code lines executed
- **Branches:** % of if/else paths taken
- **Functions:** % of functions called
- **Statements:** % of statements executed

**Configuration:**
```javascript
// jest.config.js
coverageThresholds: {
  global: {
    branches: 80,
    functions: 80,
    lines: 80,
    statements: 80
  }
}
```

**Best practices:**
- **Don't chase 100%:** Diminishing returns
- **Focus on critical code:** Business logic, security
- **Quality over quantity:** Good tests matter more than coverage number
- **Use as guide:** Identify untested code paths

**❌ Bad:**
```javascript
test('divide', () => {
  divide(10, 2); // No assertion! 100% coverage, useless test
});
```

**✅ Good:**
```javascript
test('divide returns quotient', () => {
  expect(divide(10, 2)).toBe(5);
});

test('divide throws on zero', () => {
  expect(() => divide(10, 0)).toThrow();
});
```

**Q8: How do you handle testing async code?**

A: **async/await:**
```javascript
test('fetches user data', async () => {
  const user = await fetchUser('123');
  expect(user.name).toBe('John');
});
```

**waitFor:**
```javascript
test('displays loaded data', async () => {
  render(<UserProfile userId="123" />);

  await waitFor(() => {
    expect(screen.getByText('John')).toBeInTheDocument();
  });
});
```

**findBy queries (auto-wait):**
```javascript
test('shows error', async () => {
  render(<Form />);
  fireEvent.click(screen.getByText('Submit'));

  const error = await screen.findByText('Invalid');
  expect(error).toBeInTheDocument();
});
```

**Wait for API calls:**
```javascript
test('loads data on mount', async () => {
  const { result, waitForNextUpdate } = renderHook(() => useUsers());

  expect(result.current.loading).toBe(true);

  await waitForNextUpdate();

  expect(result.current.loading).toBe(false);
  expect(result.current.users).toHaveLength(3);
});
```

**Q9: What is the AAA pattern?**

A: AAA stands for Arrange-Act-Assert, a test structure pattern:

**Arrange:** Set up test data and dependencies
**Act:** Execute the code being tested
**Assert:** Verify the expected outcome

**Example:**
```javascript
test('shopping cart calculates total', () => {
  // Arrange - Set up
  const cart = new ShoppingCart();
  const item1 = { name: 'Laptop', price: 1000 };
  const item2 = { name: 'Mouse', price: 50 };

  // Act - Execute
  cart.addItem(item1);
  cart.addItem(item2);
  const total = cart.total();

  // Assert - Verify
  expect(total).toBe(1050);
});
```

**Benefits:**
- **Clarity:** Easy to understand test flow
- **Consistency:** Standard structure across tests
- **Maintainability:** Changes are easier to make

**With comments (optional):**
```javascript
test('applies discount correctly', () => {
  // Given a cart with items
  const cart = new ShoppingCart();
  cart.addItem({ price: 100 });

  // When 10% discount is applied
  cart.applyDiscount(10);

  // Then total is reduced
  expect(cart.total()).toBe(90);
});
```

**Q10: How do you maintain tests as code evolves?**

A: **1. Keep tests close to code:**
```
UserProfile.tsx
UserProfile.test.tsx    ← Same directory
```

**2. Use test utilities:**
```javascript
// testUtils.js
export function renderWithProviders(ui) {
  return render(
    <AuthProvider>
      <ThemeProvider>
        {ui}
      </ThemeProvider>
    </AuthProvider>
  );
}
```

**3. Builder pattern for test data:**
```javascript
const user = new UserBuilder()
  .withEmail('test@example.com')
  .asAdmin()
  .build();
```

**4. Don't test implementation:**
```javascript
// ✅ Good: Tests behavior
expect(screen.getByText('Count: 1')).toBeInTheDocument();

// ❌ Bad: Tests implementation
expect(wrapper.state('count')).toBe(1);
```

**5. Regular refactoring:**
- Remove duplicate test code
- Update outdated tests
- Delete tests for removed features

**6. CI fails = update tests:**
- Don't disable failing tests
- Update or fix them immediately

**7. Code review for tests:**
- Review tests like production code
- Check for anti-patterns
- Ensure meaningful assertions

## Summary

**Testing Best Practices Checklist:**
- [ ] Co-locate tests with code
- [ ] Use descriptive test names
- [ ] Follow AAA pattern
- [ ] Ensure test isolation
- [ ] Handle async properly
- [ ] Use stable selectors
- [ ] Mock only external dependencies
- [ ] Keep tests fast
- [ ] Aim for meaningful coverage (not 100%)
- [ ] Test behavior, not implementation

**Key Principles:**
1. **Independence:** Each test runs in isolation
2. **Readability:** Tests are documentation
3. **Reliability:** No flaky tests
4. **Maintainability:** Tests are easy to update
5. **Speed:** Fast feedback loop

**Red Flags:**
- Tests depend on each other
- Arbitrary timeouts
- Testing implementation details
- Fragile selectors
- Too many mocks
- Flaky tests
- Slow test suite

**Test Quality Metrics:**
- ✅ Tests fail when they should
- ✅ Tests pass reliably
- ✅ Easy to understand
- ✅ Fast execution
- ✅ Catch real bugs

---

[← Test-Driven Development](./06-test-driven-development.md) | [Back to README](./README.md)
