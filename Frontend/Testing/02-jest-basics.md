# Jest Basics

## Overview

Jest is the most popular JavaScript testing framework, created by Facebook (Meta). It's a zero-configuration, all-in-one testing solution that includes a test runner, assertion library, mocking capabilities, and coverage reporting. Jest is the de facto standard for testing React applications and is widely used across the JavaScript ecosystem.

## Table of Contents
- [What is Jest](#what-is-jest)
- [Jest Setup and Configuration](#jest-setup-and-configuration)
- [Writing Test Suites](#writing-test-suites)
- [Matchers and Assertions](#matchers-and-assertions)
- [Setup and Teardown](#setup-and-teardown)
- [Mocking Functions](#mocking-functions)
- [Mocking Modules](#mocking-modules)
- [Snapshot Testing](#snapshot-testing)
- [Coverage Reports](#coverage-reports)
- [Advanced Jest Features](#advanced-jest-features)
- [Best Practices](#best-practices)
- [Interview Questions](#interview-questions)

## What is Jest

### Key Features

```javascript
// Jest provides everything you need out of the box
const jestFeatures = {
  testRunner: 'Executes tests with watch mode',
  assertions: 'Rich matcher library (expect)',
  mocking: 'Function, module, and timer mocking',
  coverage: 'Built-in code coverage reports',
  snapshots: 'Snapshot testing for React components',
  parallel: 'Runs tests in parallel for speed',
  isolated: 'Each test file runs in isolated environment',
  watch: 'Smart watch mode for rapid development'
};
```

### Why Jest?

**Zero Configuration:**
```bash
# Install Jest
npm install --save-dev jest

# Add to package.json
{
  "scripts": {
    "test": "jest"
  }
}

# Run tests - that's it!
npm test
```

**Developer Experience:**
- Fast: Runs tests in parallel
- Interactive: Watch mode with intelligent test filtering
- Clear: Excellent error messages
- Complete: Everything built-in, no extra libraries needed

## Jest Setup and Configuration

### Basic Setup

```bash
# 1. Install Jest
npm install --save-dev jest

# 2. Install Babel support for modern JavaScript (if needed)
npm install --save-dev babel-jest @babel/core @babel/preset-env

# 3. Install React testing utilities (for React projects)
npm install --save-dev @testing-library/react @testing-library/jest-dom
```

### Package.json Configuration

```json
{
  "name": "my-project",
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:debug": "node --inspect-brk node_modules/.bin/jest --runInBand"
  },
  "jest": {
    "testEnvironment": "jsdom",
    "setupFilesAfterEnv": ["<rootDir>/jest.setup.js"],
    "moduleNameMapper": {
      "\\.(css|less|scss)$": "identity-obj-proxy"
    },
    "collectCoverageFrom": [
      "src/**/*.{js,jsx}",
      "!src/index.js",
      "!src/**/*.test.{js,jsx}"
    ]
  },
  "devDependencies": {
    "jest": "^29.7.0",
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^6.1.0"
  }
}
```

### Jest Configuration File (jest.config.js)

```javascript
// jest.config.js
module.exports = {
  // Test environment
  testEnvironment: 'jsdom', // or 'node' for backend

  // Setup files
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],

  // Module paths
  moduleNameMapper: {
    // Handle CSS imports
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    // Handle image imports
    '\\.(jpg|jpeg|png|gif|svg)$': '<rootDir>/__mocks__/fileMock.js',
    // Handle module aliases
    '^@/(.*)$': '<rootDir>/src/$1'
  },

  // Transform files
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': 'babel-jest'
  },

  // Coverage configuration
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js',
    '!src/**/*.test.{js,jsx,ts,tsx}'
  ],

  // Coverage thresholds
  coverageThresholds: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },

  // Test match patterns
  testMatch: [
    '**/__tests__/**/*.[jt]s?(x)',
    '**/?(*.)+(spec|test).[jt]s?(x)'
  ],

  // Ignore patterns
  testPathIgnorePatterns: [
    '/node_modules/',
    '/build/',
    '/dist/'
  ]
};
```

### Setup File (jest.setup.js)

```javascript
// jest.setup.js
// Import custom matchers from jest-dom
import '@testing-library/jest-dom';

// Global test configuration
global.console = {
  ...console,
  // Suppress console.error in tests (optional)
  error: jest.fn(),
  // Keep console.warn visible
  warn: console.warn,
};

// Mock window.matchMedia (not implemented in JSDOM)
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  takeRecords() {
    return [];
  }
  unobserve() {}
};
```

## Writing Test Suites

### Test Structure

```javascript
// describe - groups related tests
describe('Calculator', () => {
  // test (or it) - individual test case
  test('adds two numbers', () => {
    expect(add(2, 3)).toBe(5);
  });

  it('subtracts two numbers', () => { // 'it' is alias for 'test'
    expect(subtract(5, 3)).toBe(2);
  });
});
```

### Nested Describe Blocks

```javascript
describe('User Authentication', () => {
  describe('login', () => {
    test('succeeds with valid credentials', () => {
      const result = login('user@test.com', 'password123');
      expect(result.success).toBe(true);
    });

    test('fails with invalid password', () => {
      const result = login('user@test.com', 'wrongpass');
      expect(result.success).toBe(false);
    });

    test('fails with non-existent user', () => {
      const result = login('fake@test.com', 'password');
      expect(result.success).toBe(false);
    });
  });

  describe('logout', () => {
    test('clears user session', () => {
      logout();
      expect(getSession()).toBeNull();
    });

    test('redirects to login page', () => {
      const redirectUrl = logout();
      expect(redirectUrl).toBe('/login');
    });
  });
});
```

### Test Naming Best Practices

```javascript
// ✅ Good - descriptive test names
describe('ShoppingCart', () => {
  test('calculates total price of all items', () => {});
  test('applies discount when coupon code is valid', () => {});
  test('shows error message when item is out of stock', () => {});
});

// ❌ Bad - vague test names
describe('ShoppingCart', () => {
  test('works', () => {});
  test('test1', () => {});
  test('cart', () => {});
});

// ✅ Good - using describe for context
describe('calculateShipping', () => {
  describe('when user is premium member', () => {
    test('returns free shipping', () => {});
  });

  describe('when order total is over $50', () => {
    test('returns free shipping', () => {});
  });

  describe('when order is international', () => {
    test('adds international shipping fee', () => {});
  });
});
```

### Only and Skip

```javascript
// Run only this test (useful for debugging)
test.only('this test will run', () => {
  expect(true).toBe(true);
});

test('this test will be skipped', () => {
  expect(false).toBe(true);
});

// Skip this test (temporarily disable)
test.skip('this test is skipped', () => {
  expect(complexFunction()).toBeTruthy();
});

// Run only this describe block
describe.only('UserProfile', () => {
  test('test 1', () => {});
  test('test 2', () => {});
});

describe('Other tests', () => {
  test('will be skipped', () => {});
});
```

### Test.each - Parameterized Tests

```javascript
// Test multiple cases with same logic
test.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 2, 4],
])('add(%i, %i) returns %i', (a, b, expected) => {
  expect(add(a, b)).toBe(expected);
});

// With objects for better readability
test.each([
  { a: 1, b: 1, expected: 2 },
  { a: 1, b: 2, expected: 3 },
  { a: 2, b: 2, expected: 4 },
])('add($a, $b) returns $expected', ({ a, b, expected }) => {
  expect(add(a, b)).toBe(expected);
});

// Template literal syntax
test.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${1} | ${2} | ${3}
  ${2} | ${2} | ${4}
`('returns $expected when $a is added to $b', ({ a, b, expected }) => {
  expect(add(a, b)).toBe(expected);
});
```

## Matchers and Assertions

### Common Matchers

```javascript
// Equality
test('equality matchers', () => {
  expect(2 + 2).toBe(4);                    // Strict equality (===)
  expect({ name: 'John' }).toEqual({ name: 'John' }); // Deep equality
  expect(['a', 'b']).toEqual(['a', 'b']);   // Arrays
});

// Truthiness
test('truthiness matchers', () => {
  expect(null).toBeNull();
  expect(undefined).toBeUndefined();
  expect(5).toBeDefined();
  expect(true).toBeTruthy();
  expect(false).toBeFalsy();
});

// Numbers
test('number matchers', () => {
  expect(4).toBeGreaterThan(3);
  expect(4).toBeGreaterThanOrEqual(4);
  expect(3).toBeLessThan(4);
  expect(3).toBeLessThanOrEqual(3);

  // Floating point equality
  expect(0.1 + 0.2).toBeCloseTo(0.3);       // Avoids rounding errors
});

// Strings
test('string matchers', () => {
  expect('team').not.toMatch(/I/);          // Regex
  expect('Christoph').toMatch(/stop/);
  expect('hello world').toContain('world'); // Substring
});

// Arrays and Iterables
test('array matchers', () => {
  const fruits = ['apple', 'banana', 'orange'];

  expect(fruits).toContain('banana');
  expect(fruits).toHaveLength(3);
  expect(new Set(fruits)).toContain('apple');
});

// Objects
test('object matchers', () => {
  const user = {
    name: 'John',
    age: 30,
    address: {
      city: 'NYC'
    }
  };

  expect(user).toHaveProperty('name');
  expect(user).toHaveProperty('address.city', 'NYC');
  expect(user).toMatchObject({ name: 'John' }); // Partial match
});

// Exceptions
test('exception matchers', () => {
  expect(() => {
    throw new Error('Error!');
  }).toThrow();

  expect(() => {
    throw new Error('Invalid input');
  }).toThrow('Invalid input');

  expect(() => {
    throw new Error('Invalid input');
  }).toThrow(/Invalid/);
});
```

### jest-dom Custom Matchers

```javascript
import '@testing-library/jest-dom';

test('DOM matchers', () => {
  const button = document.createElement('button');
  button.textContent = 'Click me';
  button.disabled = true;

  expect(button).toBeInTheDocument();
  expect(button).toHaveTextContent('Click me');
  expect(button).toBeDisabled();
  expect(button).toBeVisible();
});

// Common jest-dom matchers
test('form matchers', () => {
  const input = document.createElement('input');
  input.value = 'test@example.com';

  expect(input).toHaveValue('test@example.com');
  expect(input).toBeValid();
  expect(input).toBeRequired();
});
```

### Not Modifier

```javascript
test('using .not modifier', () => {
  expect(2 + 2).not.toBe(5);
  expect('hello').not.toMatch(/bye/);
  expect([1, 2, 3]).not.toContain(4);
  expect(null).not.toBeDefined();
});
```

### Async Matchers

```javascript
// Promises
test('resolves matcher', async () => {
  await expect(Promise.resolve('success')).resolves.toBe('success');
});

test('rejects matcher', async () => {
  await expect(Promise.reject(new Error('fail'))).rejects.toThrow('fail');
});

// Alternative syntax
test('async with return', () => {
  return expect(fetchData()).resolves.toEqual({ data: 'value' });
});
```

## Setup and Teardown

### beforeEach and afterEach

```javascript
describe('Database operations', () => {
  let db;

  // Runs before each test
  beforeEach(() => {
    db = new Database();
    db.connect();
  });

  // Runs after each test
  afterEach(() => {
    db.disconnect();
    db = null;
  });

  test('saves user to database', () => {
    db.save({ name: 'John' });
    expect(db.users).toHaveLength(1);
  });

  test('retrieves user from database', () => {
    db.save({ name: 'John' });
    const user = db.findByName('John');
    expect(user).toBeDefined();
  });
});
```

### beforeAll and afterAll

```javascript
describe('Expensive setup', () => {
  let server;

  // Runs once before all tests
  beforeAll(() => {
    server = setupTestServer();
  });

  // Runs once after all tests
  afterAll(() => {
    server.close();
  });

  test('test 1', () => {
    expect(server.isRunning()).toBe(true);
  });

  test('test 2', () => {
    expect(server.port).toBe(3000);
  });
});
```

### Scoping

```javascript
// Global setup/teardown
beforeEach(() => {
  console.log('Before every test in the file');
});

describe('Group 1', () => {
  beforeEach(() => {
    console.log('Before every test in Group 1');
  });

  test('test 1', () => {
    // Logs: "Before every test in the file"
    // Then: "Before every test in Group 1"
    expect(true).toBe(true);
  });
});

describe('Group 2', () => {
  test('test 2', () => {
    // Only logs: "Before every test in the file"
    expect(true).toBe(true);
  });
});
```

### Order of Execution

```javascript
// Execution order example
beforeAll(() => console.log('1 - beforeAll'));
afterAll(() => console.log('1 - afterAll'));
beforeEach(() => console.log('1 - beforeEach'));
afterEach(() => console.log('1 - afterEach'));

test('test 1', () => console.log('1 - test 1'));

describe('Scoped', () => {
  beforeAll(() => console.log('2 - beforeAll'));
  afterAll(() => console.log('2 - afterAll'));
  beforeEach(() => console.log('2 - beforeEach'));
  afterEach(() => console.log('2 - afterEach'));

  test('test 2', () => console.log('2 - test 2'));
});

// Output:
// 1 - beforeAll
// 1 - beforeEach
// 1 - test 1
// 1 - afterEach
// 2 - beforeAll
// 1 - beforeEach
// 2 - beforeEach
// 2 - test 2
// 2 - afterEach
// 1 - afterEach
// 2 - afterAll
// 1 - afterAll
```

## Mocking Functions

### jest.fn() - Mock Functions

```javascript
// Create a mock function
const mockCallback = jest.fn();

// Use it
[1, 2].forEach(mockCallback);

// Verify it was called
expect(mockCallback).toHaveBeenCalledTimes(2);
expect(mockCallback).toHaveBeenCalledWith(1, 0, [1, 2]);
expect(mockCallback).toHaveBeenNthCalledWith(2, 2, 1, [1, 2]);
```

### Mock Return Values

```javascript
// Return a specific value
const mockFn = jest.fn();
mockFn.mockReturnValue(42);
expect(mockFn()).toBe(42);

// Return different values on consecutive calls
const mockFn2 = jest.fn();
mockFn2
  .mockReturnValueOnce(10)
  .mockReturnValueOnce(20)
  .mockReturnValue(30);

console.log(mockFn2()); // 10
console.log(mockFn2()); // 20
console.log(mockFn2()); // 30
console.log(mockFn2()); // 30
```

### Mock Async Functions

```javascript
// Mock async function with resolved value
const fetchUser = jest.fn();
fetchUser.mockResolvedValue({ id: 1, name: 'John' });

test('fetches user', async () => {
  const user = await fetchUser();
  expect(user.name).toBe('John');
});

// Mock async function with rejected value
const failingFetch = jest.fn();
failingFetch.mockRejectedValue(new Error('Network error'));

test('handles fetch error', async () => {
  await expect(failingFetch()).rejects.toThrow('Network error');
});

// Multiple async calls
const mockFetch = jest.fn();
mockFetch
  .mockResolvedValueOnce({ data: 'first' })
  .mockResolvedValueOnce({ data: 'second' })
  .mockRejectedValueOnce(new Error('third fails'));
```

### Mock Implementation

```javascript
// Provide custom implementation
const mockFn = jest.fn((x) => x * 2);
expect(mockFn(3)).toBe(6);

// Or use mockImplementation
const mockFn2 = jest.fn();
mockFn2.mockImplementation((a, b) => a + b);
expect(mockFn2(1, 2)).toBe(3);

// One-time implementation
const mockFn3 = jest.fn();
mockFn3
  .mockImplementationOnce(() => 'first')
  .mockImplementationOnce(() => 'second')
  .mockImplementation(() => 'default');

console.log(mockFn3()); // 'first'
console.log(mockFn3()); // 'second'
console.log(mockFn3()); // 'default'
console.log(mockFn3()); // 'default'
```

### Tracking Mock Calls

```javascript
const mockFn = jest.fn();
mockFn('arg1', 'arg2');
mockFn('arg3', 'arg4');

// Check if called
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledTimes(2);

// Check arguments
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
expect(mockFn).toHaveBeenLastCalledWith('arg3', 'arg4');
expect(mockFn).toHaveBeenNthCalledWith(1, 'arg1', 'arg2');

// Access mock data
console.log(mockFn.mock.calls); // [['arg1', 'arg2'], ['arg3', 'arg4']]
console.log(mockFn.mock.calls[0]); // ['arg1', 'arg2']
console.log(mockFn.mock.results[0]); // { type: 'return', value: undefined }
```

### Resetting Mocks

```javascript
const mockFn = jest.fn();
mockFn('test');

// Clear call history but keep implementation
mockFn.mockClear();
expect(mockFn).not.toHaveBeenCalled();

// Reset implementation to undefined
mockFn.mockReset();

// Reset and restore original implementation
mockFn.mockRestore(); // Only works with jest.spyOn
```

### Spy on Methods

```javascript
// Spy on existing method
const video = {
  play() {
    return true;
  }
};

const spy = jest.spyOn(video, 'play');
video.play();

expect(spy).toHaveBeenCalled();
spy.mockRestore(); // Restore original implementation

// Spy and mock implementation
const spy2 = jest.spyOn(video, 'play').mockImplementation(() => false);
expect(video.play()).toBe(false);
spy2.mockRestore();
```

## Mocking Modules

### Automatic Mocking

```javascript
// __mocks__/axios.js
export default {
  get: jest.fn(() => Promise.resolve({ data: {} })),
  post: jest.fn(() => Promise.resolve({ data: {} }))
};

// In test file
import axios from 'axios';

jest.mock('axios');

test('fetches data', async () => {
  axios.get.mockResolvedValue({ data: { name: 'John' } });

  const response = await axios.get('/users/1');
  expect(response.data.name).toBe('John');
  expect(axios.get).toHaveBeenCalledWith('/users/1');
});
```

### Manual Mocking

```javascript
// api.js
export const fetchUser = (id) => {
  return fetch(`/api/users/${id}`).then(r => r.json());
};

// api.test.js
import { fetchUser } from './api';

jest.mock('./api', () => ({
  fetchUser: jest.fn()
}));

test('mocks fetchUser', async () => {
  fetchUser.mockResolvedValue({ id: 1, name: 'John' });

  const user = await fetchUser(1);
  expect(user.name).toBe('John');
});
```

### Partial Mocking

```javascript
// utils.js
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
export const multiply = (a, b) => a * b;

// utils.test.js
import * as utils from './utils';

jest.mock('./utils', () => ({
  ...jest.requireActual('./utils'), // Keep original implementations
  multiply: jest.fn() // Mock only multiply
}));

test('partial mock', () => {
  utils.multiply.mockReturnValue(100);

  expect(utils.add(2, 3)).toBe(5);        // Real implementation
  expect(utils.subtract(5, 3)).toBe(2);   // Real implementation
  expect(utils.multiply(2, 3)).toBe(100); // Mocked
});
```

### Mock Implementation for Specific Tests

```javascript
import { fetchData } from './api';

jest.mock('./api');

describe('API tests', () => {
  test('successful fetch', async () => {
    fetchData.mockResolvedValue({ success: true });
    const result = await fetchData();
    expect(result.success).toBe(true);
  });

  test('failed fetch', async () => {
    fetchData.mockRejectedValue(new Error('Network error'));
    await expect(fetchData()).rejects.toThrow('Network error');
  });
});
```

### Mocking ES6 Classes

```javascript
// SoundPlayer.js
export default class SoundPlayer {
  constructor() {
    this.sounds = [];
  }

  play(sound) {
    this.sounds.push(sound);
  }
}

// SoundPlayer.test.js
import SoundPlayer from './SoundPlayer';

jest.mock('./SoundPlayer');

test('mock class', () => {
  const mockPlaySound = jest.fn();
  SoundPlayer.mockImplementation(() => {
    return {
      play: mockPlaySound
    };
  });

  const player = new SoundPlayer();
  player.play('song.mp3');

  expect(mockPlaySound).toHaveBeenCalledWith('song.mp3');
});
```

### Mocking Timers

```javascript
// Code with setTimeout
function delayedGreeting(callback) {
  setTimeout(() => {
    callback('Hello!');
  }, 1000);
}

// Test with fake timers
test('calls callback after 1 second', () => {
  jest.useFakeTimers();
  const callback = jest.fn();

  delayedGreeting(callback);

  // Fast-forward time
  jest.advanceTimersByTime(1000);

  expect(callback).toHaveBeenCalledWith('Hello!');

  jest.useRealTimers();
});

// Test with runAllTimers
test('runs all timers', () => {
  jest.useFakeTimers();
  const callback = jest.fn();

  setTimeout(() => callback('first'), 100);
  setTimeout(() => callback('second'), 500);

  jest.runAllTimers();

  expect(callback).toHaveBeenCalledTimes(2);
  jest.useRealTimers();
});
```

## Snapshot Testing

### Basic Snapshot Testing

```javascript
// Component to test
function Link({ url, children }) {
  return <a href={url}>{children}</a>;
}

// Snapshot test
import renderer from 'react-test-renderer';

test('Link renders correctly', () => {
  const tree = renderer
    .create(<Link url="https://example.com">Example</Link>)
    .toJSON();

  expect(tree).toMatchSnapshot();
});

// First run creates snapshot file:
// __snapshots__/Link.test.js.snap
```

### Generated Snapshot

```javascript
// Auto-generated snapshot file
exports[`Link renders correctly 1`] = `
<a
  href="https://example.com"
>
  Example
</a>
`;
```

### Updating Snapshots

```bash
# Update all snapshots
npm test -- -u

# Update specific snapshot interactively
# Press 'u' in watch mode when snapshot fails
npm test -- --watch
```

### Inline Snapshots

```javascript
test('inline snapshot', () => {
  const user = {
    name: 'John',
    age: 30
  };

  expect(user).toMatchInlineSnapshot(`
    {
      "age": 30,
      "name": "John",
    }
  `);
  // Snapshot stored in test file, not separate file
});
```

### Property Matchers

```javascript
// Handle dynamic values in snapshots
test('user snapshot with dynamic date', () => {
  const user = {
    id: 1,
    name: 'John',
    createdAt: new Date()
  };

  expect(user).toMatchSnapshot({
    createdAt: expect.any(Date) // Matches any Date object
  });
});

// Generated snapshot
exports[`user snapshot 1`] = `
{
  "createdAt": Any<Date>,
  "id": 1,
  "name": "John",
}
`;
```

### When to Use Snapshots

```javascript
// ✅ Good use cases for snapshots
test('error message format', () => {
  const error = formatError({ code: 404, message: 'Not found' });
  expect(error).toMatchSnapshot();
});

test('API response structure', () => {
  const response = buildUserResponse(user);
  expect(response).toMatchSnapshot();
});

// ❌ Bad use cases
test('current timestamp', () => {
  expect(Date.now()).toMatchSnapshot(); // ❌ Changes every run
});

test('random values', () => {
  expect(Math.random()).toMatchSnapshot(); // ❌ Non-deterministic
});
```

### Snapshot Best Practices

```javascript
// ✅ Good - small, focused snapshots
test('button renders correctly', () => {
  const { container } = render(<Button>Click me</Button>);
  expect(container.firstChild).toMatchSnapshot();
});

// ❌ Bad - large, complex snapshots
test('entire app', () => {
  const { container } = render(<App />);
  expect(container).toMatchSnapshot(); // ❌ Too large
});

// ✅ Good - test specific props
test('button with different variants', () => {
  expect(<Button variant="primary">Primary</Button>).toMatchSnapshot();
  expect(<Button variant="secondary">Secondary</Button>).toMatchSnapshot();
});
```

## Coverage Reports

### Running Coverage

```bash
# Generate coverage report
npm test -- --coverage

# Generate and open in browser
npm test -- --coverage --coverageReporters=html
open coverage/index.html

# Watch mode with coverage
npm test -- --watch --coverage
```

### Coverage Output

```bash
# Console output
--------------------|---------|----------|---------|---------|-------------------
File                | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
--------------------|---------|----------|---------|---------|-------------------
All files           |   85.71 |    83.33 |      80 |   85.71 |
 calculator.js      |     100 |      100 |     100 |     100 |
 user.js            |   71.43 |    66.67 |   66.67 |   71.43 | 12,18-20
 utils.js           |      90 |       80 |      75 |      90 | 45
--------------------|---------|----------|---------|---------|-------------------
```

### Coverage Configuration

```javascript
// jest.config.js
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{js,jsx}',
    '!src/**/*.test.{js,jsx}',
    '!src/index.js',
    '!src/setupTests.js'
  ],

  coverageThresholds: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/components/': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90
    }
  },

  coverageReporters: [
    'text',        // Console output
    'lcov',        // HTML report
    'json',        // JSON report
    'text-summary' // Summary in console
  ]
};
```

### Ignoring Code from Coverage

```javascript
// Ignore next line
function myFunction() {
  /* istanbul ignore next */
  if (process.env.NODE_ENV === 'development') {
    console.log('Debug info');
  }
}

// Ignore entire function
/* istanbul ignore next */
function debugFunction() {
  // This entire function ignored from coverage
}

// Ignore else branch
function getUserRole(user) {
  if (user.isAdmin) {
    return 'admin';
  }
  /* istanbul ignore else */
  else {
    return 'user';
  }
}
```

## Advanced Jest Features

### Custom Matchers

```javascript
// Define custom matcher
expect.extend({
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling;
    if (pass) {
      return {
        message: () =>
          `expected ${received} not to be within range ${floor} - ${ceiling}`,
        pass: true,
      };
    } else {
      return {
        message: () =>
          `expected ${received} to be within range ${floor} - ${ceiling}`,
        pass: false,
      };
    }
  },
});

// Use custom matcher
test('is within range', () => {
  expect(100).toBeWithinRange(90, 110);
  expect(101).not.toBeWithinRange(0, 100);
});
```

### Global Setup and Teardown

```javascript
// globalSetup.js
module.exports = async () => {
  // Start test database
  global.__DB__ = await startDatabase();
  console.log('Test database started');
};

// globalTeardown.js
module.exports = async () => {
  // Stop test database
  await global.__DB__.stop();
  console.log('Test database stopped');
};

// jest.config.js
module.exports = {
  globalSetup: './globalSetup.js',
  globalTeardown: './globalTeardown.js'
};
```

### Test Retry

```javascript
// Retry flaky tests
test('flaky test', () => {
  // This will retry up to 2 times if it fails
}, 2);

// Configure globally
// jest.config.js
module.exports = {
  testRetryCount: 2
};
```

### Concurrent Tests

```javascript
// Run tests concurrently (faster but can't share state)
describe.concurrent('concurrent suite', () => {
  test.concurrent('test 1', async () => {
    await someAsyncOperation();
    expect(result).toBe(expected);
  });

  test.concurrent('test 2', async () => {
    await anotherAsyncOperation();
    expect(result).toBe(expected);
  });
});
```

## Best Practices

### 1. Keep Tests Isolated

```javascript
// ❌ Bad - shared state
let user;
beforeEach(() => {
  user = { name: 'John', age: 30 };
});

test('test 1', () => {
  user.age = 31; // Mutates shared object
  expect(user.age).toBe(31);
});

test('test 2', () => {
  expect(user.age).toBe(30); // Might fail!
});

// ✅ Good - factory function
function createUser() {
  return { name: 'John', age: 30 };
}

test('test 1', () => {
  const user = createUser();
  user.age = 31;
  expect(user.age).toBe(31);
});

test('test 2', () => {
  const user = createUser();
  expect(user.age).toBe(30); // Always works
});
```

### 2. Use Descriptive Test Names

```javascript
// ✅ Good
test('returns null when user is not found', () => {});
test('throws error when email is invalid', () => {});
test('calculates 10% discount for premium members', () => {});

// ❌ Bad
test('works', () => {});
test('test user', () => {});
test('it does the thing', () => {});
```

### 3. Test One Thing Per Test

```javascript
// ❌ Bad - testing too much
test('user registration', () => {
  const user = registerUser('test@example.com', 'password');
  expect(user).toBeDefined();
  expect(user.email).toBe('test@example.com');

  const loggedIn = loginUser('test@example.com', 'password');
  expect(loggedIn).toBe(true);

  const profile = getUserProfile(user.id);
  expect(profile.email).toBe('test@example.com');
});

// ✅ Good - separate tests
test('registerUser creates user with correct email', () => {
  const user = registerUser('test@example.com', 'password');
  expect(user.email).toBe('test@example.com');
});

test('loginUser returns true for valid credentials', () => {
  registerUser('test@example.com', 'password');
  expect(loginUser('test@example.com', 'password')).toBe(true);
});
```

### 4. Avoid Nesting Describes Too Deeply

```javascript
// ❌ Bad - too nested
describe('UserService', () => {
  describe('create', () => {
    describe('validation', () => {
      describe('email', () => {
        describe('format', () => {
          test('accepts valid email', () => {});
        });
      });
    });
  });
});

// ✅ Good - flatter structure
describe('UserService', () => {
  describe('create validation', () => {
    test('accepts valid email format', () => {});
    test('rejects invalid email format', () => {});
  });
});
```

### 5. Mock External Dependencies

```javascript
// ✅ Good - mock external API
jest.mock('./api');
import { fetchUser } from './api';

test('displays user data', async () => {
  fetchUser.mockResolvedValue({ id: 1, name: 'John' });
  // Test runs fast and reliably
});

// ❌ Bad - real network call
test('displays user data', async () => {
  const user = await fetch('https://api.example.com/users/1');
  // Slow, unreliable, dependent on network
});
```

## Interview Questions

### Question 1: What is Jest and what are its main features?

**Answer:**

Jest is a zero-configuration JavaScript testing framework created by Facebook. It's the most popular testing solution for React applications and provides a complete testing solution out of the box.

**Main Features:**

1. **Zero Configuration**: Works out of the box for most JavaScript projects
2. **Fast and Parallel**: Runs tests in parallel for maximum performance
3. **Built-in Coverage**: Code coverage reports without additional tools
4. **Snapshot Testing**: Test React components and other outputs
5. **Powerful Mocking**: Mock functions, modules, and timers easily
6. **Great DX**: Interactive watch mode, clear error messages
7. **Isolated Tests**: Each test file runs in its own environment

**Example:**
```javascript
// No setup needed - just install and run
npm install --save-dev jest

// package.json
{
  "scripts": {
    "test": "jest"
  }
}

// Write test
test('adds two numbers', () => {
  expect(2 + 2).toBe(4);
});

// Run tests
npm test
```

**Why Jest is Popular:**
- Works with React out of the box
- Fast test execution
- Rich API (matchers, mocking, assertions)
- Active development and community
- Used by major companies (Facebook, Airbnb, Twitter)

---

### Question 2: Explain the difference between `toBe()` and `toEqual()` matchers.

**Answer:**

The key difference is that `toBe()` uses **strict equality (===)** while `toEqual()` performs **deep equality** comparison.

**toBe() - Reference Equality:**
```javascript
// Primitive values - both work the same
expect(5).toBe(5);           // ✅ Pass
expect('hello').toBe('hello'); // ✅ Pass

// Objects - checks reference, not content
const user1 = { name: 'John' };
const user2 = { name: 'John' };
expect(user1).toBe(user2);    // ❌ Fail - different references
expect(user1).toBe(user1);    // ✅ Pass - same reference
```

**toEqual() - Deep Equality:**
```javascript
// Primitive values - works like toBe
expect(5).toEqual(5);         // ✅ Pass
expect('hello').toEqual('hello'); // ✅ Pass

// Objects - checks content, not reference
const user1 = { name: 'John', age: 30 };
const user2 = { name: 'John', age: 30 };
expect(user1).toEqual(user2); // ✅ Pass - same content

// Works with nested objects
const user3 = {
  name: 'John',
  address: {
    city: 'NYC',
    zip: '10001'
  }
};
const user4 = {
  name: 'John',
  address: {
    city: 'NYC',
    zip: '10001'
  }
};
expect(user3).toEqual(user4); // ✅ Pass - deep equality

// Arrays
expect([1, 2, 3]).toEqual([1, 2, 3]); // ✅ Pass
expect([1, 2, 3]).toBe([1, 2, 3]);    // ❌ Fail
```

**When to Use Each:**

Use `toBe()` for:
- Primitive values (numbers, strings, booleans)
- Checking reference equality
- Null/undefined checks

Use `toEqual()` for:
- Objects and arrays
- Deep value comparison
- When structure matters, not reference

```javascript
// Practical examples
test('number equality', () => {
  expect(calculateTotal([10, 20])).toBe(30); // ✅ toBe for primitives
});

test('object equality', () => {
  const user = createUser({ name: 'John' });
  expect(user).toEqual({ // ✅ toEqual for objects
    id: expect.any(Number),
    name: 'John',
    createdAt: expect.any(Date)
  });
});
```

---

### Question 3: How do you test asynchronous code in Jest?

**Answer:**

Jest provides several ways to test async code. The key is ensuring Jest waits for promises to resolve before completing the test.

**1. Async/Await (Recommended):**
```javascript
test('fetches user data', async () => {
  const user = await fetchUser(1);
  expect(user.name).toBe('John');
});

// Testing rejections
test('handles fetch error', async () => {
  await expect(fetchUser(999)).rejects.toThrow('User not found');
});
```

**2. Returning Promises:**
```javascript
test('fetches user data', () => {
  return fetchUser(1).then(user => {
    expect(user.name).toBe('John');
  });
  // MUST return the promise!
});
```

**3. Done Callback:**
```javascript
test('callback-based async', (done) => {
  function callback(data) {
    try {
      expect(data).toBe('success');
      done(); // MUST call done()
    } catch (error) {
      done(error);
    }
  }

  fetchDataWithCallback(callback);
});
```

**4. Resolves/Rejects Matchers:**
```javascript
test('promise resolves', async () => {
  await expect(Promise.resolve('success')).resolves.toBe('success');
});

test('promise rejects', async () => {
  await expect(Promise.reject(new Error('fail'))).rejects.toThrow('fail');
});

// Without async/await
test('promise resolves', () => {
  return expect(fetchData()).resolves.toEqual({ data: 'value' });
});
```

**5. Testing with Fake Timers:**
```javascript
test('delayed execution', () => {
  jest.useFakeTimers();

  const callback = jest.fn();
  setTimeout(callback, 1000);

  jest.advanceTimersByTime(1000);

  expect(callback).toHaveBeenCalled();
  jest.useRealTimers();
});
```

**Common Mistakes:**
```javascript
// ❌ Not waiting for async
test('fetches data', () => {
  fetchData().then(data => {
    expect(data).toBeDefined();
  });
  // Test completes before promise resolves!
});

// ✅ Proper async testing
test('fetches data', async () => {
  const data = await fetchData();
  expect(data).toBeDefined();
});

// ❌ Forgetting to call done()
test('callback test', (done) => {
  asyncFunction((result) => {
    expect(result).toBe('success');
    // Forgot done() - test times out!
  });
});

// ✅ Calling done
test('callback test', (done) => {
  asyncFunction((result) => {
    expect(result).toBe('success');
    done(); // ✅
  });
});
```

---

### Question 4: What is the purpose of `beforeEach()`, `afterEach()`, `beforeAll()`, and `afterAll()`?

**Answer:**

These are **setup and teardown hooks** that help you prepare test environments and clean up afterward.

**beforeEach() - Runs before each test:**
```javascript
describe('User tests', () => {
  let user;

  beforeEach(() => {
    // Fresh user for each test
    user = { name: 'John', age: 30 };
  });

  test('test 1', () => {
    user.age = 31;
    expect(user.age).toBe(31);
  });

  test('test 2', () => {
    expect(user.age).toBe(30); // ✅ Fresh user, not mutated
  });
});
```

**afterEach() - Runs after each test:**
```javascript
describe('Database tests', () => {
  afterEach(() => {
    // Clean up after each test
    database.clear();
    jest.clearAllMocks();
  });

  test('saves user', () => {
    database.save({ name: 'John' });
    expect(database.users).toHaveLength(1);
  });

  test('starts with empty database', () => {
    expect(database.users).toHaveLength(0); // ✅ Cleaned up
  });
});
```

**beforeAll() - Runs once before all tests:**
```javascript
describe('API tests', () => {
  let server;

  beforeAll(() => {
    // Expensive setup - do once
    server = startTestServer();
  });

  afterAll(() => {
    // Cleanup once after all tests
    server.close();
  });

  test('test 1', () => {
    expect(server.isRunning()).toBe(true);
  });

  test('test 2', () => {
    expect(server.port).toBe(3000);
  });
});
```

**Execution Order:**
```javascript
beforeAll(() => console.log('1 - beforeAll'));
afterAll(() => console.log('1 - afterAll'));
beforeEach(() => console.log('1 - beforeEach'));
afterEach(() => console.log('1 - afterEach'));

test('test 1', () => console.log('1 - test'));

describe('Inner', () => {
  beforeAll(() => console.log('2 - beforeAll'));
  afterAll(() => console.log('2 - afterAll'));
  beforeEach(() => console.log('2 - beforeEach'));
  afterEach(() => console.log('2 - afterEach'));

  test('test 2', () => console.log('2 - test'));
});

// Output:
// 1 - beforeAll
// 1 - beforeEach
// 1 - test
// 1 - afterEach
// 2 - beforeAll
// 1 - beforeEach
// 2 - beforeEach
// 2 - test
// 2 - afterEach
// 1 - afterEach
// 2 - afterAll
// 1 - afterAll
```

**When to Use:**
- `beforeEach/afterEach`: Test isolation, reset state, clear mocks
- `beforeAll/afterAll`: Expensive operations (DB connections, servers)

---

### Question 5: How do you mock a function in Jest?

**Answer:**

Jest provides `jest.fn()` to create mock functions, which lets you control behavior and verify interactions.

**Basic Mock Function:**
```javascript
// Create mock
const mockFn = jest.fn();

// Use it
mockFn('arg1', 'arg2');

// Verify calls
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
expect(mockFn).toHaveBeenCalledTimes(1);
```

**Mock Return Values:**
```javascript
// Return specific value
const mockFn = jest.fn().mockReturnValue(42);
expect(mockFn()).toBe(42);

// Different values per call
const mockFn2 = jest.fn()
  .mockReturnValueOnce('first')
  .mockReturnValueOnce('second')
  .mockReturnValue('default');

console.log(mockFn2()); // 'first'
console.log(mockFn2()); // 'second'
console.log(mockFn2()); // 'default'
console.log(mockFn2()); // 'default'
```

**Mock Async Functions:**
```javascript
// Resolved promise
const fetchUser = jest.fn().mockResolvedValue({ id: 1, name: 'John' });
const user = await fetchUser();
expect(user.name).toBe('John');

// Rejected promise
const failFetch = jest.fn().mockRejectedValue(new Error('Failed'));
await expect(failFetch()).rejects.toThrow('Failed');
```

**Mock Implementation:**
```javascript
// Custom logic
const mockFn = jest.fn((x, y) => x + y);
expect(mockFn(2, 3)).toBe(5);

// Or using mockImplementation
const mockFn2 = jest.fn();
mockFn2.mockImplementation((x, y) => x * y);
expect(mockFn2(2, 3)).toBe(6);
```

**Spy on Existing Methods:**
```javascript
const calculator = {
  add: (a, b) => a + b
};

// Spy tracks calls but keeps original implementation
const spy = jest.spyOn(calculator, 'add');
expect(calculator.add(2, 3)).toBe(5);
expect(spy).toHaveBeenCalledWith(2, 3);

// Spy and change implementation
spy.mockImplementation(() => 100);
expect(calculator.add(2, 3)).toBe(100);

// Restore original
spy.mockRestore();
expect(calculator.add(2, 3)).toBe(5);
```

**Practical Example:**
```javascript
// Component that uses callback
function LoginForm({ onSubmit }) {
  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit({ email: 'test@example.com', password: 'pass' });
  };

  return <form onSubmit={handleSubmit}>...</form>;
}

// Test with mock
test('calls onSubmit with credentials', () => {
  const mockSubmit = jest.fn();
  render(<LoginForm onSubmit={mockSubmit} />);

  fireEvent.submit(screen.getByRole('form'));

  expect(mockSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'pass'
  });
  expect(mockSubmit).toHaveBeenCalledTimes(1);
});
```

---

### Question 6: Explain Jest module mocking with `jest.mock()`.

**Answer:**

`jest.mock()` replaces an entire module with a mock implementation, useful for mocking external dependencies like APIs, databases, or third-party libraries.

**Automatic Mocking:**
```javascript
// api.js
export const fetchUser = (id) => {
  return fetch(`/api/users/${id}`).then(r => r.json());
};

// test.js
import { fetchUser } from './api';

jest.mock('./api'); // Automatically mocks all exports

test('mocked API', async () => {
  fetchUser.mockResolvedValue({ id: 1, name: 'John' });

  const user = await fetchUser(1);
  expect(user.name).toBe('John');
  expect(fetchUser).toHaveBeenCalledWith(1);
});
```

**Manual Mock Implementation:**
```javascript
// Mock with custom implementation
jest.mock('./api', () => ({
  fetchUser: jest.fn((id) => Promise.resolve({ id, name: 'Mock User' })),
  deleteUser: jest.fn(() => Promise.resolve(true))
}));

import { fetchUser, deleteUser } from './api';

test('uses manual mock', async () => {
  const user = await fetchUser(5);
  expect(user).toEqual({ id: 5, name: 'Mock User' });
});
```

**Partial Mocking:**
```javascript
// Keep some real implementations
jest.mock('./utils', () => ({
  ...jest.requireActual('./utils'), // Keep original implementations
  expensiveFunction: jest.fn(() => 'mocked') // Mock only this one
}));

import * as utils from './utils';

test('partial mock', () => {
  expect(utils.add(2, 3)).toBe(5); // Real implementation
  expect(utils.expensiveFunction()).toBe('mocked'); // Mocked
});
```

**Mocking Node Modules:**
```javascript
// Mock axios
jest.mock('axios');
import axios from 'axios';

test('mocks axios', async () => {
  axios.get.mockResolvedValue({ data: { users: [] } });

  const response = await axios.get('/api/users');
  expect(response.data.users).toEqual([]);
});
```

**Manual Mock Files:**
```javascript
// __mocks__/axios.js
export default {
  get: jest.fn(() => Promise.resolve({ data: {} })),
  post: jest.fn(() => Promise.resolve({ data: {} })),
  put: jest.fn(() => Promise.resolve({ data: {} })),
  delete: jest.fn(() => Promise.resolve({ data: {} }))
};

// test.js
jest.mock('axios'); // Automatically uses __mocks__/axios.js
import axios from 'axios';

test('uses manual mock file', async () => {
  axios.get.mockResolvedValue({ data: { name: 'John' } });
  const response = await axios.get('/user');
  expect(response.data.name).toBe('John');
});
```

**Clearing Mocks:**
```javascript
describe('tests with mocks', () => {
  afterEach(() => {
    jest.clearAllMocks(); // Clear call history
  });

  test('test 1', () => {
    mockFn('test');
    expect(mockFn).toHaveBeenCalledTimes(1);
  });

  test('test 2', () => {
    // Mock is cleared, call count is 0
    expect(mockFn).not.toHaveBeenCalled();
  });
});
```

---

### Question 7: What is snapshot testing and when should you use it?

**Answer:**

Snapshot testing captures the output of a component or function and saves it to a file. Future test runs compare new output against the saved snapshot to detect unexpected changes.

**How It Works:**
```javascript
import renderer from 'react-test-renderer';

function Button({ children, onClick }) {
  return (
    <button onClick={onClick} className="btn">
      {children}
    </button>
  );
}

test('Button renders correctly', () => {
  const tree = renderer
    .create(<Button onClick={() => {}}>Click me</Button>)
    .toJSON();

  expect(tree).toMatchSnapshot();
});

// First run creates: __snapshots__/Button.test.js.snap
exports[`Button renders correctly 1`] = `
<button
  className="btn"
  onClick={[Function]}
>
  Click me
</button>
`;
```

**Updating Snapshots:**
```bash
# When component changes intentionally
npm test -- -u

# Interactive update in watch mode
npm test -- --watch
# Press 'u' to update failing snapshots
```

**Inline Snapshots:**
```javascript
test('config object', () => {
  const config = {
    apiUrl: 'https://api.example.com',
    timeout: 5000
  };

  expect(config).toMatchInlineSnapshot(`
    {
      "apiUrl": "https://api.example.com",
      "timeout": 5000,
    }
  `);
});
```

**When to Use Snapshots:**

**✅ Good Use Cases:**
```javascript
// 1. Testing component structure
test('UserCard renders correctly', () => {
  const { container } = render(<UserCard user={mockUser} />);
  expect(container.firstChild).toMatchSnapshot();
});

// 2. Testing formatted output
test('formats error message', () => {
  const error = formatError({ code: 404, message: 'Not found' });
  expect(error).toMatchSnapshot();
});

// 3. Testing API response structure
test('user API response', () => {
  const response = buildUserResponse(user);
  expect(response).toMatchSnapshot({
    id: expect.any(String),
    createdAt: expect.any(String)
  });
});
```

**❌ When NOT to Use:**
```javascript
// 1. Large components
test('entire app', () => {
  expect(<App />).toMatchSnapshot(); // ❌ Too large
});

// 2. Dynamic/random data
test('timestamp', () => {
  expect(Date.now()).toMatchSnapshot(); // ❌ Changes every run
});

// 3. Implementation details
test('internal state', () => {
  expect(component.state).toMatchSnapshot(); // ❌ Test behavior instead
});
```

**Best Practices:**
- Keep snapshots small and focused
- Review snapshot changes in code review
- Update snapshots intentionally, not blindly
- Use property matchers for dynamic values
- Commit snapshot files to version control

---

### Question 8: How do you test code that uses timers (setTimeout, setInterval)?

**Answer:**

Jest provides fake timers to control time in tests, making timer-dependent code testable without waiting for real time to pass.

**Basic Fake Timers:**
```javascript
function delayedGreeting(callback) {
  setTimeout(() => {
    callback('Hello!');
  }, 1000);
}

test('calls callback after 1 second', () => {
  jest.useFakeTimers();
  const callback = jest.fn();

  delayedGreeting(callback);

  // Fast-forward 1 second
  jest.advanceTimersByTime(1000);

  expect(callback).toHaveBeenCalledWith('Hello!');

  jest.useRealTimers();
});
```

**Run All Timers:**
```javascript
test('runs all pending timers', () => {
  jest.useFakeTimers();
  const callback = jest.fn();

  setTimeout(() => callback('first'), 100);
  setTimeout(() => callback('second'), 500);
  setTimeout(() => callback('third'), 1000);

  jest.runAllTimers();

  expect(callback).toHaveBeenCalledTimes(3);
  expect(callback).toHaveBeenNthCalledWith(3, 'third');

  jest.useRealTimers();
});
```

**Run Only Pending Timers:**
```javascript
test('runs only currently pending timers', () => {
  jest.useFakeTimers();
  let count = 0;

  function recursiveTimer() {
    count++;
    if (count < 5) {
      setTimeout(recursiveTimer, 100);
    }
  }

  setTimeout(recursiveTimer, 100);

  // Run only timers pending now (not recursively scheduled ones)
  jest.runOnlyPendingTimers();

  expect(count).toBe(1); // Only first call

  jest.runAllTimers();
  expect(count).toBe(5); // All calls

  jest.useRealTimers();
});
```

**Advance Time by Time:**
```javascript
test('progresses through multiple timeouts', () => {
  jest.useFakeTimers();
  const callback = jest.fn();

  setTimeout(() => callback('100ms'), 100);
  setTimeout(() => callback('500ms'), 500);
  setTimeout(() => callback('1000ms'), 1000);

  jest.advanceTimersByTime(250);
  expect(callback).toHaveBeenCalledTimes(1);
  expect(callback).toHaveBeenCalledWith('100ms');

  jest.advanceTimersByTime(500);
  expect(callback).toHaveBeenCalledTimes(2);
  expect(callback).toHaveBeenLastCalledWith('500ms');

  jest.useRealTimers();
});
```

**Testing setInterval:**
```javascript
function counter(callback) {
  let count = 0;
  const id = setInterval(() => {
    count++;
    callback(count);
    if (count >= 3) {
      clearInterval(id);
    }
  }, 1000);
}

test('interval calls callback multiple times', () => {
  jest.useFakeTimers();
  const callback = jest.fn();

  counter(callback);

  jest.advanceTimersByTime(1000);
  expect(callback).toHaveBeenCalledWith(1);

  jest.advanceTimersByTime(1000);
  expect(callback).toHaveBeenCalledWith(2);

  jest.advanceTimersByTime(1000);
  expect(callback).toHaveBeenCalledWith(3);

  expect(callback).toHaveBeenCalledTimes(3);

  jest.useRealTimers();
});
```

**Modern Timers (Jest 27+):**
```javascript
test('modern fake timers', async () => {
  jest.useFakeTimers('modern');

  const promise = new Promise(resolve => {
    setTimeout(() => resolve('done'), 1000);
  });

  jest.advanceTimersByTime(1000);

  await expect(promise).resolves.toBe('done');

  jest.useRealTimers();
});
```

---

### Question 9: What is code coverage and what's a good coverage percentage to aim for?

**Answer:**

Code coverage measures how much of your code is executed by your tests. It's reported as percentages across different metrics.

**Coverage Metrics:**
```javascript
// Example function
function getUserStatus(user) {
  if (!user) {              // Line 1
    return 'guest';          // Line 2
  }

  if (user.isPremium) {      // Line 3
    return 'premium';        // Line 4
  }

  return 'regular';          // Line 5
}

// With this test:
test('returns guest for no user', () => {
  expect(getUserStatus(null)).toBe('guest');
});

// Coverage:
// Lines: 40% (2 of 5 lines executed)
// Branches: 50% (1 of 2 if branches taken)
// Functions: 100% (function was called)
// Statements: 40% (2 of 5 statements executed)
```

**The Four Metrics:**

1. **Statement Coverage**: % of statements executed
2. **Branch Coverage**: % of if/else branches taken
3. **Function Coverage**: % of functions called
4. **Line Coverage**: % of lines executed

**Generating Coverage:**
```bash
# Run tests with coverage
npm test -- --coverage

# Example output:
--------------------|---------|----------|---------|---------|
File                | % Stmts | % Branch | % Funcs | % Lines |
--------------------|---------|----------|---------|---------|
All files           |   85.71 |    75.00 |   90.00 |   85.71 |
 utils.js           |   90.00 |    80.00 |  100.00 |   90.00 |
 calculator.js      |  100.00 |   100.00 |  100.00 |  100.00 |
 user.js            |   71.43 |    66.67 |   80.00 |   71.43 |
--------------------|---------|----------|---------|---------|
```

**Good Coverage Targets:**
```javascript
// jest.config.js
module.exports = {
  coverageThresholds: {
    global: {
      statements: 80,
      branches: 75,
      functions: 80,
      lines: 80
    },
    // Higher standards for critical code
    './src/core/': {
      statements: 95,
      branches: 90,
      functions: 95,
      lines: 95
    }
  }
};
```

**Industry Standards:**
- **80%+**: Good coverage for most projects
- **90%+**: Excellent coverage for critical systems
- **100%**: Often impractical and not always valuable

**Important Caveats:**

```javascript
// ❌ 100% coverage doesn't mean bug-free
function divide(a, b) {
  return a / b;
}

test('divides numbers', () => {
  expect(divide(10, 2)).toBe(5);
});
// 100% coverage but missing edge case: divide(10, 0) => Infinity!

// ✅ Good coverage tests edge cases
test('handles division by zero', () => {
  expect(() => divide(10, 0)).toThrow('Cannot divide by zero');
});
```

**Best Practices:**
- Aim for 80% as baseline
- Focus on **quality** tests, not just coverage numbers
- Test edge cases and error conditions
- Don't write tests just to increase coverage
- Use coverage to find untested code, not as a goal itself

---

### Question 10: How do you organize and structure your Jest tests?

**Answer:**

Proper test organization makes tests maintainable, readable, and easy to debug.

**File Structure:**
```
src/
├── components/
│   ├── Button/
│   │   ├── Button.jsx
│   │   ├── Button.test.jsx        # Co-located tests
│   │   └── Button.module.css
│   └── UserCard/
│       ├── UserCard.jsx
│       └── UserCard.test.jsx
├── utils/
│   ├── calculations.js
│   └── calculations.test.js
└── __tests__/                      # Alternative: centralized tests
    ├── integration/
    │   └── userFlow.test.js
    └── e2e/
        └── checkout.test.js
```

**Test File Naming:**
```javascript
// Good naming conventions
Button.test.jsx         // Component tests
Button.spec.jsx         // Alternative (BDD style)
calculations.test.js    // Utility tests
userFlow.integration.test.js  // Integration tests
```

**Describe Blocks for Organization:**
```javascript
describe('ShoppingCart', () => {
  describe('adding items', () => {
    test('adds item to empty cart', () => {});
    test('adds item to cart with existing items', () => {});
    test('prevents adding duplicate items', () => {});
  });

  describe('removing items', () => {
    test('removes item from cart', () => {});
    test('handles removing non-existent item', () => {});
  });

  describe('calculating totals', () => {
    test('calculates total for single item', () => {});
    test('calculates total for multiple items', () => {});
    test('applies discount to total', () => {});
  });
});
```

**Setup with beforeEach:**
```javascript
describe('UserService', () => {
  let userService;
  let mockDatabase;

  beforeEach(() => {
    mockDatabase = {
      users: [],
      save: jest.fn(),
      find: jest.fn()
    };
    userService = new UserService(mockDatabase);
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  describe('createUser', () => {
    test('saves user to database', () => {
      userService.createUser({ name: 'John' });
      expect(mockDatabase.save).toHaveBeenCalled();
    });
  });
});
```

**Helper Functions:**
```javascript
// test-utils.js
export function createMockUser(overrides = {}) {
  return {
    id: 1,
    name: 'Test User',
    email: 'test@example.com',
    ...overrides
  };
}

export function createMockDatabase() {
  return {
    users: [],
    save: jest.fn(),
    find: jest.fn(),
    delete: jest.fn()
  };
}

// In tests
import { createMockUser, createMockDatabase } from './test-utils';

test('example', () => {
  const user = createMockUser({ name: 'John' });
  const db = createMockDatabase();
  // ... test code
});
```

**Shared Setup File:**
```javascript
// jest.setup.js
import '@testing-library/jest-dom';

// Global test utilities
global.testUtils = {
  createMockUser: (overrides) => ({
    id: 1,
    name: 'Test',
    ...overrides
  })
};

// Global mocks
jest.mock('axios');
```

**Test Naming Convention:**
```javascript
// ✅ Good - describes behavior
test('shows error message when email is invalid', () => {});
test('disables submit button while loading', () => {});
test('redirects to dashboard after successful login', () => {});

// ❌ Bad - vague or implementation-focused
test('test login', () => {});
test('it works', () => {});
test('calls useState', () => {});
```

**Grouping Related Tests:**
```javascript
describe('Authentication', () => {
  describe('when user is logged out', () => {
    test('shows login form', () => {});
    test('redirects to login on protected route', () => {});
  });

  describe('when user is logged in', () => {
    test('shows user menu', () => {});
    test('allows access to protected routes', () => {});
  });
});
```

---

## Summary

Jest is a comprehensive testing framework that provides everything needed for JavaScript testing:

**Key Features:**
- Zero configuration setup
- Fast parallel test execution
- Built-in code coverage
- Powerful mocking capabilities
- Snapshot testing
- Excellent developer experience

**Essential Concepts:**
- Use `describe` to group related tests
- Write tests with `test` or `it`
- Use appropriate matchers (`toBe`, `toEqual`, etc.)
- Setup/teardown with `beforeEach`, `afterEach`, `beforeAll`, `afterAll`
- Mock functions with `jest.fn()` and modules with `jest.mock()`
- Test async code with `async/await` or promises
- Control timers with `jest.useFakeTimers()`

**Best Practices:**
- Keep tests isolated and independent
- Use descriptive test names
- Test one thing per test
- Mock external dependencies
- Aim for 80% coverage but prioritize quality
- Organize tests logically with describe blocks

**Next Steps:**
- Learn React Testing Library for component testing
- Practice mocking complex scenarios
- Explore integration testing patterns
- Master async testing techniques

Remember: Jest provides the foundation, but writing meaningful tests requires understanding what and how to test effectively.

---

**Next:** [React Testing Library →](./03-react-testing-library.md)

**Previous:** [← Testing Fundamentals](./01-testing-fundamentals.md)

---

[← Back to Testing Interview Prep](./README.md) | [↑ Back to Frontend](../README.md)
