# React Testing Library

## Overview

React Testing Library (RTL) is the recommended testing library for React applications. Created by Kent C. Dodds, it encourages best practices by testing components from the user's perspective rather than implementation details. RTL is built on top of DOM Testing Library and works seamlessly with Jest.

## Table of Contents
- [RTL Philosophy](#rtl-philosophy)
- [Setup and Installation](#setup-and-installation)
- [Rendering Components](#rendering-components)
- [Queries](#queries)
- [User Interactions](#user-interactions)
- [Testing Async Behavior](#testing-async-behavior)
- [Testing Hooks](#testing-hooks)
- [Custom Render Functions](#custom-render-functions)
- [Common Patterns](#common-patterns)
- [Debugging Tests](#debugging-tests)
- [Best Practices](#best-practices)
- [Interview Questions](#interview-questions)

## RTL Philosophy

### Guiding Principles

```javascript
// ❌ Testing implementation details (Bad)
test('component uses useState', () => {
  const { result } = renderHook(() => useState(0));
  expect(result.current[0]).toBe(0);
});

// ✅ Testing user behavior (Good)
test('counter displays initial value', () => {
  render(<Counter />);
  expect(screen.getByText('Count: 0')).toBeInTheDocument();
});
```

**Core Philosophy:**
> "The more your tests resemble the way your software is used, the more confidence they can give you."

**Key Principles:**

1. **Test behavior, not implementation**
2. **Query by accessibility roles and labels**
3. **Avoid testing internal state**
4. **Test from the user's perspective**
5. **Use semantic queries**

```javascript
// Example: Testing like a user would interact
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!email || !password) {
      setError('Please fill in all fields');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input id="email" type="email" value={email} onChange={(e) => setEmail(e.target.value)} />

      <label htmlFor="password">Password</label>
      <input id="password" type="password" value={password} onChange={(e) => setPassword(e.target.value)} />

      <button type="submit">Login</button>
      {error && <div role="alert">{error}</div>}
    </form>
  );
}

// ✅ Test like a user - find by label, type, click, see error
test('shows error when fields are empty', async () => {
  render(<LoginForm />);

  // User sees and clicks the login button
  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  // User sees the error message
  expect(screen.getByRole('alert')).toHaveTextContent('Please fill in all fields');
});
```

## Setup and Installation

### Installation

```bash
# Install React Testing Library and Jest dependencies
npm install --save-dev @testing-library/react @testing-library/jest-dom @testing-library/user-event

# If using Create React App, these are already included
```

### Setup File

```javascript
// jest.setup.js
import '@testing-library/jest-dom';

// Optional: Configure React Testing Library
import { configure } from '@testing-library/react';

configure({
  testIdAttribute: 'data-testid', // Default
  asyncUtilTimeout: 1000, // Default timeout for async helpers
});
```

### Package.json

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "jest": {
    "testEnvironment": "jsdom",
    "setupFilesAfterEnv": ["<rootDir>/jest.setup.js"]
  }
}
```

## Rendering Components

### Basic Render

```javascript
import { render, screen } from '@testing-library/react';

test('renders a button', () => {
  render(<button>Click me</button>);

  const button = screen.getByRole('button', { name: /click me/i });
  expect(button).toBeInTheDocument();
});
```

### Render Returns

```javascript
test('render return values', () => {
  const { container, getByText, rerender, unmount } = render(
    <div>Hello World</div>
  );

  // container - the DOM node
  expect(container.firstChild).toHaveTextContent('Hello World');

  // Direct queries (use screen.getByText instead)
  expect(getByText('Hello World')).toBeInTheDocument();

  // Rerender with new props
  rerender(<div>Goodbye World</div>);
  expect(screen.getByText('Goodbye World')).toBeInTheDocument();

  // Unmount component
  unmount();
  expect(container.firstChild).toBeNull();
});
```

### Screen vs Render Destructuring

```javascript
// ✅ Recommended - use screen
test('use screen', () => {
  render(<MyComponent />);
  expect(screen.getByText('Hello')).toBeInTheDocument();
});

// ❌ Not recommended - destructuring from render
test('destructuring', () => {
  const { getByText } = render(<MyComponent />);
  expect(getByText('Hello')).toBeInTheDocument();
});
```

**Why use screen?**
- Less code to write
- Better error messages
- Automatic suggestions in IDEs
- Consistent across tests

## Queries

### Query Types

RTL provides three types of query variants:

```javascript
// getBy - Returns element or throws error
const button = screen.getByRole('button');
// ✅ Element found
// ❌ Throws if not found or multiple found

// queryBy - Returns element or null
const button = screen.queryByRole('button');
// ✅ Element found
// ✅ Returns null if not found
// ❌ Throws if multiple found

// findBy - Returns promise, waits for element
const button = await screen.findByRole('button');
// ✅ Waits for element to appear (async)
// ❌ Throws if not found after timeout
```

**When to use each:**

```javascript
// getBy - when element should be in the document
test('renders heading', () => {
  render(<h1>Welcome</h1>);
  expect(screen.getByRole('heading')).toBeInTheDocument();
});

// queryBy - when checking element is NOT in document
test('does not show error initially', () => {
  render(<Form />);
  expect(screen.queryByRole('alert')).not.toBeInTheDocument();
});

// findBy - when element appears asynchronously
test('shows data after loading', async () => {
  render(<UserProfile userId={1} />);
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
});
```

### Query Priority

**Priority order (recommended to use in this order):**

1. **Accessible to everyone** (screen readers, keyboard users):
   - `getByRole`
   - `getByLabelText`
   - `getByPlaceholderText`
   - `getByText`
   - `getByDisplayValue`

2. **Semantic queries**:
   - `getByAltText`
   - `getByTitle`

3. **Test IDs** (last resort):
   - `getByTestId`

```javascript
// 1. getByRole - BEST (most accessible)
test('finds button by role', () => {
  render(<button>Submit</button>);
  expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
});

// 2. getByLabelText - form elements
test('finds input by label', () => {
  render(
    <>
      <label htmlFor="email">Email</label>
      <input id="email" type="email" />
    </>
  );
  expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
});

// 3. getByPlaceholderText
test('finds input by placeholder', () => {
  render(<input placeholder="Enter email" />);
  expect(screen.getByPlaceholderText(/enter email/i)).toBeInTheDocument();
});

// 4. getByText - text content
test('finds paragraph by text', () => {
  render(<p>Hello World</p>);
  expect(screen.getByText(/hello world/i)).toBeInTheDocument();
});

// 5. getByTestId - LAST RESORT
test('finds element by test id', () => {
  render(<div data-testid="custom-element">Content</div>);
  expect(screen.getByTestId('custom-element')).toBeInTheDocument();
});
```

### Query Variants with Multiple Elements

```javascript
// getAllBy - returns array of all matching elements
test('finds all buttons', () => {
  render(
    <>
      <button>Save</button>
      <button>Cancel</button>
      <button>Delete</button>
    </>
  );

  const buttons = screen.getAllByRole('button');
  expect(buttons).toHaveLength(3);
});

// queryAllBy - returns empty array if none found
test('no error messages initially', () => {
  render(<Form />);
  expect(screen.queryAllByRole('alert')).toHaveLength(0);
});

// findAllBy - async, returns promise
test('finds all list items after loading', async () => {
  render(<TodoList />);
  const items = await screen.findAllByRole('listitem');
  expect(items).toHaveLength(5);
});
```

### getByRole Options

```javascript
test('advanced role queries', () => {
  render(
    <>
      <h1>Main Heading</h1>
      <h2>Subheading</h2>
      <button disabled>Save</button>
      <button>Cancel</button>
      <input type="checkbox" checked aria-label="Accept terms" />
    </>
  );

  // Query by name
  expect(screen.getByRole('heading', { name: /main heading/i })).toBeInTheDocument();

  // Query by level
  expect(screen.getByRole('heading', { level: 2 })).toHaveTextContent('Subheading');

  // Query by state
  expect(screen.getByRole('button', { name: /save/i })).toBeDisabled();

  // Query checked checkbox
  expect(screen.getByRole('checkbox', { checked: true })).toBeInTheDocument();
});
```

### Text Matching

```javascript
test('text matching options', () => {
  render(<div>Hello World 123</div>);

  // String match
  expect(screen.getByText('Hello World 123')).toBeInTheDocument();

  // Regex match (case insensitive)
  expect(screen.getByText(/hello world/i)).toBeInTheDocument();

  // Partial match
  expect(screen.getByText(/world/i)).toBeInTheDocument();

  // Function matcher
  expect(screen.getByText((content, element) => {
    return element.tagName.toLowerCase() === 'div' && content.startsWith('Hello');
  })).toBeInTheDocument();

  // Exact match (default: true)
  expect(screen.getByText('Hello World 123', { exact: true })).toBeInTheDocument();

  // Partial match (exact: false)
  expect(screen.getByText('Hello', { exact: false })).toBeInTheDocument();
});
```

## User Interactions

### userEvent vs fireEvent

```javascript
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

// ✅ userEvent - simulates real user interactions (RECOMMENDED)
test('userEvent types in input', async () => {
  render(<input />);
  const input = screen.getByRole('textbox');

  await userEvent.type(input, 'Hello');

  expect(input).toHaveValue('Hello');
});

// ❌ fireEvent - low-level DOM events (less realistic)
test('fireEvent changes input', () => {
  render(<input />);
  const input = screen.getByRole('textbox');

  fireEvent.change(input, { target: { value: 'Hello' } });

  expect(input).toHaveValue('Hello');
});
```

**Why userEvent is better:**
- Fires multiple events like a real user (focus, keydown, keyup, etc.)
- Handles browser behavior automatically
- More realistic interaction simulation
- Better edge case handling

### Click Events

```javascript
test('clicking a button', async () => {
  const handleClick = jest.fn();
  render(<button onClick={handleClick}>Click me</button>);

  await userEvent.click(screen.getByRole('button'));

  expect(handleClick).toHaveBeenCalledTimes(1);
});

// Double click
test('double clicking', async () => {
  const handleDoubleClick = jest.fn();
  render(<button onDoubleClick={handleDoubleClick}>Double click</button>);

  await userEvent.dblClick(screen.getByRole('button'));

  expect(handleDoubleClick).toHaveBeenCalledTimes(1);
});

// Right click
test('right clicking', async () => {
  const handleContextMenu = jest.fn();
  render(<div onContextMenu={handleContextMenu}>Right click me</div>);

  await userEvent.pointer({ target: screen.getByText(/right click/i), keys: '[MouseRight]' });

  expect(handleContextMenu).toHaveBeenCalled();
});
```

### Typing

```javascript
test('typing in an input', async () => {
  render(<input />);
  const input = screen.getByRole('textbox');

  await userEvent.type(input, 'Hello World');

  expect(input).toHaveValue('Hello World');
});

// Clear and type
test('clearing and typing', async () => {
  render(<input defaultValue="Old value" />);
  const input = screen.getByRole('textbox');

  await userEvent.clear(input);
  await userEvent.type(input, 'New value');

  expect(input).toHaveValue('New value');
});

// Special characters
test('typing special characters', async () => {
  render(<input />);
  const input = screen.getByRole('textbox');

  await userEvent.type(input, 'Hello{enter}World{backspace}');

  expect(input).toHaveValue('Hello\nWorl');
});
```

### Form Interactions

```javascript
test('selecting from dropdown', async () => {
  render(
    <select>
      <option value="">Choose...</option>
      <option value="apple">Apple</option>
      <option value="banana">Banana</option>
    </select>
  );

  const select = screen.getByRole('combobox');
  await userEvent.selectOptions(select, 'banana');

  expect(select).toHaveValue('banana');
});

// Checkbox
test('checking a checkbox', async () => {
  render(<input type="checkbox" />);
  const checkbox = screen.getByRole('checkbox');

  await userEvent.click(checkbox);
  expect(checkbox).toBeChecked();

  await userEvent.click(checkbox);
  expect(checkbox).not.toBeChecked();
});

// Radio buttons
test('selecting radio button', async () => {
  render(
    <>
      <input type="radio" name="size" value="small" />
      <input type="radio" name="size" value="large" />
    </>
  );

  const small = screen.getByDisplayValue('small');
  const large = screen.getByDisplayValue('large');

  await userEvent.click(large);
  expect(large).toBeChecked();
  expect(small).not.toBeChecked();
});
```

### File Upload

```javascript
test('uploading a file', async () => {
  const handleChange = jest.fn();
  render(<input type="file" onChange={handleChange} />);

  const file = new File(['hello'], 'hello.png', { type: 'image/png' });
  const input = screen.getByRole('textbox', { hidden: true });

  await userEvent.upload(input, file);

  expect(handleChange).toHaveBeenCalled();
  expect(input.files[0]).toBe(file);
  expect(input.files).toHaveLength(1);
});

// Multiple files
test('uploading multiple files', async () => {
  render(<input type="file" multiple />);

  const files = [
    new File(['file1'], 'file1.png', { type: 'image/png' }),
    new File(['file2'], 'file2.png', { type: 'image/png' }),
  ];

  const input = screen.getByRole('textbox', { hidden: true });
  await userEvent.upload(input, files);

  expect(input.files).toHaveLength(2);
});
```

### Keyboard Events

```javascript
test('keyboard interactions', async () => {
  const handleKeyDown = jest.fn();
  render(<input onKeyDown={handleKeyDown} />);

  const input = screen.getByRole('textbox');
  await userEvent.type(input, '{enter}');

  expect(handleKeyDown).toHaveBeenCalled();
});

// Tab navigation
test('tab navigation', async () => {
  render(
    <>
      <input data-testid="input1" />
      <input data-testid="input2" />
      <button>Submit</button>
    </>
  );

  const input1 = screen.getByTestId('input1');
  input1.focus();

  await userEvent.tab();
  expect(screen.getByTestId('input2')).toHaveFocus();

  await userEvent.tab();
  expect(screen.getByRole('button')).toHaveFocus();
});
```

## Testing Async Behavior

### findBy Queries

```javascript
// Component that fetches data
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(r => r.json())
      .then(setUser);
  }, [userId]);

  if (!user) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}

// Test async rendering
test('displays user name after loading', async () => {
  render(<UserProfile userId={1} />);

  // findBy waits for element to appear
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
});
```

### waitFor

```javascript
import { waitFor } from '@testing-library/react';

test('waits for error message', async () => {
  render(<Form />);

  await userEvent.click(screen.getByRole('button', { name: /submit/i }));

  await waitFor(() => {
    expect(screen.getByRole('alert')).toHaveTextContent('Error');
  });
});

// With timeout
test('waits with custom timeout', async () => {
  render(<SlowComponent />);

  await waitFor(
    () => {
      expect(screen.getByText('Loaded')).toBeInTheDocument();
    },
    { timeout: 3000 }
  );
});

// Multiple assertions
test('multiple async assertions', async () => {
  render(<DataGrid />);

  await waitFor(() => {
    expect(screen.getAllByRole('row')).toHaveLength(10);
    expect(screen.getByText('Page 1 of 5')).toBeInTheDocument();
  });
});
```

### waitForElementToBeRemoved

```javascript
test('loading spinner disappears', async () => {
  render(<AsyncComponent />);

  const spinner = screen.getByRole('status');

  await waitForElementToBeRemoved(spinner);

  expect(screen.getByText('Content loaded')).toBeInTheDocument();
});
```

### Testing Loading States

```javascript
test('shows loading then data', async () => {
  render(<UserList />);

  // Initially shows loading
  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  // Wait for data to load
  await waitFor(() => {
    expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
  });

  // Data is displayed
  expect(screen.getByText('John Doe')).toBeInTheDocument();
});
```

### Testing Error States

```javascript
test('shows error message on failed fetch', async () => {
  // Mock failed API call
  global.fetch = jest.fn(() =>
    Promise.reject(new Error('Network error'))
  );

  render(<UserProfile userId={1} />);

  // Wait for error to appear
  expect(await screen.findByRole('alert')).toHaveTextContent('Failed to load');
});
```

## Testing Hooks

### renderHook from @testing-library/react

```javascript
import { renderHook } from '@testing-library/react';

// Custom hook
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);

  return { count, increment, decrement };
}

// Test hook
test('useCounter hook', () => {
  const { result } = renderHook(() => useCounter(5));

  expect(result.current.count).toBe(5);

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(6);
});
```

### Testing Hook Updates

```javascript
test('hook updates state', () => {
  const { result } = renderHook(() => useCounter(0));

  act(() => {
    result.current.increment();
    result.current.increment();
  });

  expect(result.current.count).toBe(2);

  act(() => {
    result.current.decrement();
  });

  expect(result.current.count).toBe(1);
});
```

### Testing Hooks with Props

```javascript
// Hook with parameters
function useSearch(query) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    if (query) {
      fetch(`/api/search?q=${query}`)
        .then(r => r.json())
        .then(setResults);
    }
  }, [query]);

  return results;
}

// Test with changing props
test('useSearch updates when query changes', async () => {
  const { result, rerender } = renderHook(
    ({ query }) => useSearch(query),
    { initialProps: { query: '' } }
  );

  expect(result.current).toEqual([]);

  rerender({ query: 'react' });

  await waitFor(() => {
    expect(result.current).toHaveLength(3);
  });
});
```

### Testing Hooks with Context

```javascript
// Hook that uses context
function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// Test with wrapper
test('useAuth returns auth context', () => {
  const wrapper = ({ children }) => (
    <AuthProvider value={{ user: { name: 'John' } }}>
      {children}
    </AuthProvider>
  );

  const { result } = renderHook(() => useAuth(), { wrapper });

  expect(result.current.user.name).toBe('John');
});
```

## Custom Render Functions

### Creating Custom Render

```javascript
// test-utils.jsx
import { render } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { ThemeProvider } from 'styled-components';
import { AuthProvider } from './contexts/AuthContext';

const AllTheProviders = ({ children }) => {
  return (
    <BrowserRouter>
      <ThemeProvider theme={{ mode: 'light' }}>
        <AuthProvider>
          {children}
        </AuthProvider>
      </ThemeProvider>
    </BrowserRouter>
  );
};

const customRender = (ui, options) =>
  render(ui, { wrapper: AllTheProviders, ...options });

// Re-export everything
export * from '@testing-library/react';
export { customRender as render };
```

### Using Custom Render

```javascript
// component.test.js
import { render, screen } from './test-utils'; // Custom render

test('renders with all providers', () => {
  render(<MyComponent />);
  expect(screen.getByText('Hello')).toBeInTheDocument();
});
```

### Custom Render with Options

```javascript
// test-utils.jsx
function customRender(
  ui,
  {
    initialRoute = '/',
    user = null,
    theme = 'light',
    ...options
  } = {}
) {
  window.history.pushState({}, 'Test page', initialRoute);

  const Wrapper = ({ children }) => (
    <BrowserRouter>
      <ThemeProvider theme={{ mode: theme }}>
        <AuthProvider value={{ user }}>
          {children}
        </AuthProvider>
      </ThemeProvider>
    </BrowserRouter>
  );

  return render(ui, { wrapper: Wrapper, ...options });
}

// Usage
test('renders with custom options', () => {
  render(<Dashboard />, {
    user: { name: 'John', role: 'admin' },
    theme: 'dark',
    initialRoute: '/dashboard'
  });

  expect(screen.getByText('Welcome, John')).toBeInTheDocument();
});
```

## Common Patterns

### Testing Forms

```javascript
test('form submission', async () => {
  const handleSubmit = jest.fn();
  render(<ContactForm onSubmit={handleSubmit} />);

  // Fill out form
  await userEvent.type(screen.getByLabelText(/name/i), 'John Doe');
  await userEvent.type(screen.getByLabelText(/email/i), 'john@example.com');
  await userEvent.type(screen.getByLabelText(/message/i), 'Hello!');

  // Submit
  await userEvent.click(screen.getByRole('button', { name: /submit/i }));

  // Verify submission
  expect(handleSubmit).toHaveBeenCalledWith({
    name: 'John Doe',
    email: 'john@example.com',
    message: 'Hello!'
  });
});
```

### Testing Conditional Rendering

```javascript
test('shows content when logged in', () => {
  render(<Dashboard />, { user: { name: 'John' } });
  expect(screen.getByText('Welcome, John')).toBeInTheDocument();
});

test('shows login prompt when logged out', () => {
  render(<Dashboard />, { user: null });
  expect(screen.getByText('Please log in')).toBeInTheDocument();
  expect(screen.queryByText('Welcome')).not.toBeInTheDocument();
});
```

### Testing Lists

```javascript
test('renders list of items', () => {
  const items = [
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' }
  ];

  render(<ItemList items={items} />);

  const listItems = screen.getAllByRole('listitem');
  expect(listItems).toHaveLength(3);
  expect(listItems[0]).toHaveTextContent('Item 1');
});
```

### Testing Modal/Dialog

```javascript
test('opens and closes modal', async () => {
  render(<App />);

  // Modal not visible initially
  expect(screen.queryByRole('dialog')).not.toBeInTheDocument();

  // Open modal
  await userEvent.click(screen.getByRole('button', { name: /open modal/i }));
  expect(screen.getByRole('dialog')).toBeInTheDocument();

  // Close modal
  await userEvent.click(screen.getByRole('button', { name: /close/i }));
  expect(screen.queryByRole('dialog')).not.toBeInTheDocument();
});
```

### Testing Routing

```javascript
import { MemoryRouter } from 'react-router-dom';

test('navigates to different page', async () => {
  render(
    <MemoryRouter initialEntries={['/']}>
      <App />
    </MemoryRouter>
  );

  await userEvent.click(screen.getByRole('link', { name: /about/i }));

  expect(screen.getByRole('heading', { name: /about page/i })).toBeInTheDocument();
});
```

## Debugging Tests

### screen.debug()

```javascript
test('debugging example', () => {
  render(<MyComponent />);

  // Print entire document
  screen.debug();

  // Print specific element
  screen.debug(screen.getByRole('button'));

  // Limit output size
  screen.debug(undefined, 30000); // 30000 character limit
});
```

### logRoles

```javascript
import { logRoles } from '@testing-library/react';

test('log available roles', () => {
  const { container } = render(<MyComponent />);

  logRoles(container);
  // Prints all available roles and elements
});
```

### prettyDOM

```javascript
import { prettyDOM } from '@testing-library/react';

test('pretty print DOM', () => {
  const { container } = render(<MyComponent />);

  console.log(prettyDOM(container));
});
```

### Testing Playground

```javascript
test('get testing playground URL', () => {
  render(<MyComponent />);

  // Automatically opens Testing Playground
  screen.logTestingPlaygroundURL();
});
```

## Best Practices

### 1. Use Accessible Queries

```javascript
// ✅ Good - accessible queries
test('good queries', () => {
  render(<LoginForm />);

  screen.getByRole('button', { name: /login/i });
  screen.getByLabelText(/email/i);
  screen.getByText(/welcome/i);
});

// ❌ Bad - test IDs everywhere
test('bad queries', () => {
  render(<LoginForm />);

  screen.getByTestId('login-button');
  screen.getByTestId('email-input');
  screen.getByTestId('welcome-message');
});
```

### 2. Don't Test Implementation Details

```javascript
// ❌ Bad - testing state
test('bad test', () => {
  const { result } = renderHook(() => useState(0));
  expect(result.current[0]).toBe(0);
});

// ✅ Good - testing behavior
test('good test', () => {
  render(<Counter />);
  expect(screen.getByText('Count: 0')).toBeInTheDocument();

  userEvent.click(screen.getByRole('button', { name: /increment/i }));
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

### 3. Use userEvent over fireEvent

```javascript
// ❌ Less realistic
fireEvent.change(input, { target: { value: 'Hello' } });

// ✅ More realistic
await userEvent.type(input, 'Hello');
```

### 4. Prefer findBy for Async

```javascript
// ❌ Can be flaky
test('async test', async () => {
  render(<AsyncComponent />);
  await waitFor(() => {
    expect(screen.getByText('Loaded')).toBeInTheDocument();
  });
});

// ✅ Better
test('async test', async () => {
  render(<AsyncComponent />);
  expect(await screen.findByText('Loaded')).toBeInTheDocument();
});
```

### 5. Query by Label for Form Inputs

```javascript
// ✅ Best - by label
const input = screen.getByLabelText(/email/i);

// ⚠️ OK - by placeholder (less accessible)
const input = screen.getByPlaceholderText(/enter email/i);

// ❌ Avoid - by test ID
const input = screen.getByTestId('email-input');
```

## Interview Questions

### Question 1: What is React Testing Library and how does it differ from Enzyme?

**Answer:**

React Testing Library (RTL) is a testing library that encourages testing React components from the user's perspective, while Enzyme focuses on testing implementation details.

**Key Differences:**

**Philosophy:**
```javascript
// Enzyme - tests implementation details
wrapper.state('count'); // ❌ Accessing internal state
wrapper.instance().handleClick(); // ❌ Calling methods directly

// RTL - tests user behavior
screen.getByText('Count: 0'); // ✅ What user sees
userEvent.click(screen.getByRole('button')); // ✅ What user does
```

**RTL Advantages:**
1. **User-centric**: Tests how users interact with app
2. **Accessibility-focused**: Encourages semantic HTML and ARIA
3. **Refactor-friendly**: Tests don't break when implementation changes
4. **Better practices**: Forces you to write testable, accessible code

**Example:**
```javascript
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// Enzyme approach (implementation details)
test('enzyme test', () => {
  const wrapper = shallow(<Counter />);
  expect(wrapper.state('count')).toBe(0);
  wrapper.instance().setState({ count: 1 });
  expect(wrapper.state('count')).toBe(1);
});

// RTL approach (user behavior)
test('rtl test', async () => {
  render(<Counter />);
  expect(screen.getByText('Count: 0')).toBeInTheDocument();

  await userEvent.click(screen.getByRole('button', { name: /increment/i }));
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

**When to use RTL:**
- New projects (recommended by React team)
- Want accessible, maintainable tests
- Testing user flows and behavior

**When Enzyme might be needed:**
- Legacy projects already using it
- Need to test class component internals (migration period)

---

### Question 2: Explain the query priority in React Testing Library.

**Answer:**

RTL recommends a specific query priority based on accessibility and user experience.

**Priority Order:**

**1. Queries Accessible to Everyone:**
```javascript
// getByRole - HIGHEST PRIORITY
screen.getByRole('button', { name: /submit/i });
screen.getByRole('heading', { level: 1 });
screen.getByRole('textbox', { name: /email/i });

// getByLabelText - forms
screen.getByLabelText(/password/i);

// getByPlaceholderText
screen.getByPlaceholderText(/enter email/i);

// getByText - text content
screen.getByText(/welcome/i);

// getByDisplayValue - current input value
screen.getByDisplayValue('john@example.com');
```

**2. Semantic Queries:**
```javascript
// getByAltText - images
screen.getByAltText(/profile picture/i);

// getByTitle
screen.getByTitle(/close/i);
```

**3. Test IDs (Last Resort):**
```javascript
// getByTestId - only when other queries won't work
screen.getByTestId('custom-component');
```

**Why This Order?**

```javascript
// Example: Login form
function LoginForm() {
  return (
    <form>
      <label htmlFor="email">Email</label>
      <input id="email" type="email" placeholder="you@example.com" />

      <label htmlFor="password">Password</label>
      <input id="password" type="password" />

      <button type="submit">Login</button>
    </form>
  );
}

// ✅ BEST - getByRole (most accessible)
test('best approach', () => {
  render(<LoginForm />);
  screen.getByRole('textbox', { name: /email/i });
  screen.getByRole('button', { name: /login/i });
});

// ✅ GOOD - getByLabelText (semantic)
test('good approach', () => {
  render(<LoginForm />);
  screen.getByLabelText(/email/i);
  screen.getByLabelText(/password/i);
});

// ⚠️ OK - getByPlaceholderText (less reliable)
test('ok approach', () => {
  render(<LoginForm />);
  screen.getByPlaceholderText(/you@example.com/i);
});

// ❌ LAST RESORT - getByTestId
test('last resort', () => {
  render(<LoginForm />);
  screen.getByTestId('email-input');
  screen.getByTestId('password-input');
});
```

**Benefits of Priority:**
- Ensures accessibility
- Tests resemble user behavior
- More resilient to changes
- Encourages semantic HTML

---

### Question 3: What's the difference between getBy, queryBy, and findBy queries?

**Answer:**

These three query types differ in their behavior when elements are not found and their use with async code.

**getBy - Synchronous, Throws Error:**
```javascript
// ✅ Returns element if found
const button = screen.getByRole('button');

// ❌ Throws error if not found
const missing = screen.getByRole('button'); // Error: Unable to find

// ❌ Throws error if multiple found
const buttons = screen.getByRole('button'); // Error: Found multiple
```

**Use getBy when:**
- Element should be in the document
- You want test to fail if not found
- Synchronous operation

```javascript
test('uses getBy', () => {
  render(<h1>Welcome</h1>);
  expect(screen.getByRole('heading')).toBeInTheDocument();
});
```

**queryBy - Synchronous, Returns Null:**
```javascript
// ✅ Returns element if found
const button = screen.queryByRole('button');

// ✅ Returns null if not found (no error)
const missing = screen.queryByRole('button'); // null

// ❌ Throws error if multiple found
const buttons = screen.queryByRole('button'); // Error
```

**Use queryBy when:**
- Checking element is NOT in document
- Element might not exist
- Conditional rendering tests

```javascript
test('uses queryBy for absence', () => {
  render(<div>No errors</div>);
  expect(screen.queryByRole('alert')).not.toBeInTheDocument();
});
```

**findBy - Asynchronous, Returns Promise:**
```javascript
// ✅ Waits for element to appear (returns promise)
const button = await screen.findByRole('button');

// ❌ Throws error if not found after timeout (default 1000ms)
const missing = await screen.findByRole('button'); // Error after 1s

// ❌ Throws error if multiple found
const buttons = await screen.findByRole('button'); // Error
```

**Use findBy when:**
- Element appears asynchronously
- Waiting for API data
- Dynamic content loading

```javascript
test('uses findBy for async', async () => {
  render(<UserProfile userId={1} />);
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
});
```

**Comparison Table:**

| Aspect | getBy | queryBy | findBy |
|--------|-------|---------|--------|
| **Synchronous** | ✅ Yes | ✅ Yes | ❌ No (async) |
| **Not Found** | ❌ Throws | ✅ Returns null | ❌ Throws (after timeout) |
| **Multiple Found** | ❌ Throws | ❌ Throws | ❌ Throws |
| **Returns** | Element | Element or null | Promise<Element> |
| **Use Case** | Element exists | Element absent | Async elements |

**Practical Examples:**
```javascript
// getBy - element should exist
test('header is present', () => {
  render(<Header />);
  expect(screen.getByRole('banner')).toBeInTheDocument();
});

// queryBy - checking absence
test('no error message initially', () => {
  render(<Form />);
  expect(screen.queryByRole('alert')).not.toBeInTheDocument();
});

// findBy - async data loading
test('shows user data after fetch', async () => {
  render(<UserProfile userId={1} />);
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
});
```

---

### Question 4: How do you test asynchronous code with React Testing Library?

**Answer:**

RTL provides several utilities for testing async code: `findBy` queries, `waitFor`, and `waitForElementToBeRemoved`.

**1. findBy Queries (Recommended):**
```javascript
test('displays data after loading', async () => {
  render(<UserProfile userId={1} />);

  // findBy automatically waits (up to 1000ms default)
  const userName = await screen.findByText('John Doe');
  expect(userName).toBeInTheDocument();
});

// Equivalent to:
test('with waitFor', async () => {
  render(<UserProfile userId={1} />);

  await waitFor(() => {
    expect(screen.getByText('John Doe')).toBeInTheDocument();
  });
});
```

**2. waitFor - Complex Assertions:**
```javascript
test('waits for multiple conditions', async () => {
  render(<Dashboard />);

  await waitFor(() => {
    expect(screen.getAllByRole('row')).toHaveLength(10);
    expect(screen.getByText('Page 1 of 5')).toBeInTheDocument();
  });
});

// With custom timeout
test('waits longer', async () => {
  render(<SlowComponent />);

  await waitFor(
    () => {
      expect(screen.getByText('Loaded')).toBeInTheDocument();
    },
    { timeout: 3000 } // Wait up to 3 seconds
  );
});
```

**3. waitForElementToBeRemoved:**
```javascript
test('loading spinner disappears', async () => {
  render(<AsyncComponent />);

  const spinner = screen.getByRole('status');

  await waitForElementToBeRemoved(spinner);

  expect(screen.getByText('Data loaded')).toBeInTheDocument();
});
```

**4. Testing Loading States:**
```javascript
test('shows loading then content', async () => {
  render(<UserList />);

  // Initially loading
  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  // Wait for content
  expect(await screen.findByText('John Doe')).toBeInTheDocument();

  // Loading is gone
  expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
});
```

**5. Testing Error States:**
```javascript
test('shows error on failed fetch', async () => {
  // Mock failed API
  global.fetch = jest.fn(() =>
    Promise.reject(new Error('Network error'))
  );

  render(<UserProfile userId={1} />);

  // Wait for error message
  const error = await screen.findByRole('alert');
  expect(error).toHaveTextContent('Failed to load user');
});
```

**Common Patterns:**
```javascript
// Pattern 1: Loading → Content
test('loading to content', async () => {
  render(<AsyncComponent />);

  expect(screen.getByText('Loading...')).toBeInTheDocument();
  expect(await screen.findByText('Content')).toBeInTheDocument();
});

// Pattern 2: Loading → Error
test('loading to error', async () => {
  global.fetch = jest.fn().mockRejectedValue(new Error('Failed'));

  render(<AsyncComponent />);

  expect(await screen.findByText('Error occurred')).toBeInTheDocument();
});

// Pattern 3: Empty → Content
test('empty to content', async () => {
  render(<SearchResults />);

  expect(screen.getByText('No results')).toBeInTheDocument();

  await userEvent.type(screen.getByRole('searchbox'), 'query');

  expect(await screen.findByText('10 results')).toBeInTheDocument();
});
```

**Anti-Patterns to Avoid:**
```javascript
// ❌ Don't use setTimeout
test('bad async test', async () => {
  render(<AsyncComponent />);

  setTimeout(() => {
    expect(screen.getByText('Loaded')).toBeInTheDocument();
  }, 1000); // ❌ Arbitrary wait
});

// ✅ Use findBy or waitFor
test('good async test', async () => {
  render(<AsyncComponent />);
  expect(await screen.findByText('Loaded')).toBeInTheDocument();
});
```

---

### Question 5: Explain userEvent vs fireEvent.

**Answer:**

`userEvent` and `fireEvent` both simulate user interactions, but `userEvent` is more realistic and comprehensive.

**fireEvent - Low-Level DOM Events:**
```javascript
import { fireEvent } from '@testing-library/react';

test('fireEvent example', () => {
  render(<input />);
  const input = screen.getByRole('textbox');

  // Fires single change event
  fireEvent.change(input, { target: { value: 'Hello' } });

  expect(input).toHaveValue('Hello');
});
```

**userEvent - Real User Interactions:**
```javascript
import userEvent from '@testing-library/user-event';

test('userEvent example', async () => {
  render(<input />);
  const input = screen.getByRole('textbox');

  // Simulates actual typing: focus, keydown, keypress, input, keyup per character
  await userEvent.type(input, 'Hello');

  expect(input).toHaveValue('Hello');
});
```

**Key Differences:**

**1. Event Sequence:**
```javascript
// fireEvent - single event
fireEvent.click(button);
// Fires: click

// userEvent - multiple events like real user
await userEvent.click(button);
// Fires: mouseover → mousedown → focus → mouseup → click
```

**2. Realistic Behavior:**
```javascript
// Example: Disabled button
test('fireEvent vs userEvent with disabled button', async () => {
  const handleClick = jest.fn();
  render(<button disabled onClick={handleClick}>Click</button>);

  const button = screen.getByRole('button');

  // fireEvent still fires event (unrealistic)
  fireEvent.click(button);
  expect(handleClick).toHaveBeenCalled(); // ✅ Called

  // userEvent respects disabled state (realistic)
  await userEvent.click(button);
  // No error, but handler not called (correct behavior)
});
```

**3. Type Interactions:**
```javascript
// fireEvent - manual event object
test('fireEvent typing', () => {
  render(<input />);
  const input = screen.getByRole('textbox');

  fireEvent.change(input, { target: { value: 'test' } });
  expect(input).toHaveValue('test');
});

// userEvent - simulates actual keyboard
test('userEvent typing', async () => {
  render(<input />);
  const input = screen.getByRole('textbox');

  await userEvent.type(input, 'test');
  expect(input).toHaveValue('test');

  // Can use special characters
  await userEvent.type(input, '{backspace}{backspace}');
  expect(input).toHaveValue('te');
});
```

**When to Use Each:**

**Use userEvent (Recommended):**
```javascript
// ✅ More realistic
await userEvent.click(button);
await userEvent.type(input, 'Hello');
await userEvent.selectOptions(select, 'option1');
await userEvent.upload(fileInput, file);
```

**Use fireEvent (Only When Necessary):**
```javascript
// When userEvent doesn't support the event
fireEvent.mouseEnter(element);
fireEvent.focus(input);
fireEvent.scroll(window, { target: { scrollY: 100 } });
```

**Practical Example:**
```javascript
function SearchForm({ onSearch }) {
  const [query, setQuery] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    onSearch(query);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <button type="submit">Search</button>
    </form>
  );
}

// ✅ userEvent - realistic user flow
test('search with userEvent', async () => {
  const handleSearch = jest.fn();
  render(<SearchForm onSearch={handleSearch} />);

  const input = screen.getByPlaceholderText(/search/i);
  const button = screen.getByRole('button', { name: /search/i });

  // Types like a real user
  await userEvent.type(input, 'React testing');
  await userEvent.click(button);

  expect(handleSearch).toHaveBeenCalledWith('React testing');
});

// ❌ fireEvent - less realistic
test('search with fireEvent', () => {
  const handleSearch = jest.fn();
  render(<SearchForm onSearch={handleSearch} />);

  const input = screen.getByPlaceholderText(/search/i);

  // Doesn't simulate actual typing
  fireEvent.change(input, { target: { value: 'React testing' } });
  fireEvent.submit(screen.getByRole('form'));

  expect(handleSearch).toHaveBeenCalledWith('React testing');
});
```

**Best Practice:** Always prefer `userEvent` for better test reliability and realism.

---

### Question 6: How do you test custom hooks in React Testing Library?

**Answer:**

Use `renderHook` from `@testing-library/react` to test custom hooks in isolation.

**Basic Hook Testing:**
```javascript
import { renderHook } from '@testing-library/react';

// Custom hook
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset };
}

// Test
test('useCounter initial value', () => {
  const { result } = renderHook(() => useCounter(5));

  expect(result.current.count).toBe(5);
});

test('useCounter increment', () => {
  const { result } = renderHook(() => useCounter(0));

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

**Testing Hook Updates:**
```javascript
test('useCounter multiple operations', () => {
  const { result } = renderHook(() => useCounter(10));

  act(() => {
    result.current.increment();
    result.current.increment();
  });
  expect(result.current.count).toBe(12);

  act(() => {
    result.current.decrement();
  });
  expect(result.current.count).toBe(11);

  act(() => {
    result.current.reset();
  });
  expect(result.current.count).toBe(10);
});
```

**Testing Hooks with Props:**
```javascript
// Hook that depends on props
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(url)
      .then(r => r.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [url]);

  return { data, loading };
}

// Test with rerender
test('useFetch updates when URL changes', async () => {
  const { result, rerender } = renderHook(
    ({ url }) => useFetch(url),
    { initialProps: { url: '/api/users/1' } }
  );

  expect(result.current.loading).toBe(true);

  // Wait for data
  await waitFor(() => {
    expect(result.current.loading).toBe(false);
  });

  expect(result.current.data).toEqual({ id: 1, name: 'John' });

  // Change URL
  rerender({ url: '/api/users/2' });

  expect(result.current.loading).toBe(true);

  await waitFor(() => {
    expect(result.current.data).toEqual({ id: 2, name: 'Jane' });
  });
});
```

**Testing Hooks with Context:**
```javascript
// Hook using context
const ThemeContext = createContext();

function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// Test with wrapper
test('useTheme returns theme from context', () => {
  const wrapper = ({ children }) => (
    <ThemeContext.Provider value={{ mode: 'dark' }}>
      {children}
    </ThemeContext.Provider>
  );

  const { result } = renderHook(() => useTheme(), { wrapper });

  expect(result.current.mode).toBe('dark');
});

test('useTheme throws without provider', () => {
  // Suppress console.error for this test
  const spy = jest.spyOn(console, 'error').mockImplementation(() => {});

  expect(() => {
    renderHook(() => useTheme());
  }).toThrow('useTheme must be used within ThemeProvider');

  spy.mockRestore();
});
```

**Async Hook Testing:**
```javascript
function useData(id) {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetchData(id).then(setData);
  }, [id]);

  return data;
}

test('useData fetches data', async () => {
  global.fetchData = jest.fn().mockResolvedValue({ id: 1, name: 'Test' });

  const { result } = renderHook(() => useData(1));

  expect(result.current).toBeNull();

  await waitFor(() => {
    expect(result.current).toEqual({ id: 1, name: 'Test' });
  });
});
```

**Alternative: Test Hook in Component:**
```javascript
// Sometimes testing hook through component is better
function TestComponent() {
  const { count, increment } = useCounter(0);

  return (
    <div>
      <span>Count: {count}</span>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

test('useCounter in component', async () => {
  render(<TestComponent />);

  expect(screen.getByText('Count: 0')).toBeInTheDocument();

  await userEvent.click(screen.getByRole('button'));

  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

**Best Practice:** Test simple hooks with `renderHook`, but consider testing complex hooks through actual components for more realistic tests.

---

### Question 7: How do you create and use custom render functions in RTL?

**Answer:**

Custom render functions wrap components with necessary providers (Router, Theme, Context, etc.) to avoid repeating setup in every test.

**Creating Custom Render:**
```javascript
// test-utils.jsx
import { render } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { ThemeProvider } from 'styled-components';
import { AuthProvider } from './contexts/AuthContext';
import { QueryClient, QueryClientProvider } from 'react-query';

// Create wrapper with all providers
function AllProviders({ children }) {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false }
    }
  });

  return (
    <BrowserRouter>
      <QueryClientProvider client={queryClient}>
        <ThemeProvider theme={{ mode: 'light' }}>
          <AuthProvider>
            {children}
          </AuthProvider>
        </ThemeProvider>
      </QueryClientProvider>
    </BrowserRouter>
  );
}

// Custom render function
function customRender(ui, options) {
  return render(ui, { wrapper: AllProviders, ...options });
}

// Re-export everything from RTL
export * from '@testing-library/react';

// Override render with custom version
export { customRender as render };
```

**Using Custom Render:**
```javascript
// component.test.js
import { render, screen } from './test-utils'; // Custom render

test('component with all providers', () => {
  render(<MyComponent />);

  // Component automatically has access to:
  // - Router
  // - React Query
  // - Theme
  // - Auth context

  expect(screen.getByText('Hello')).toBeInTheDocument();
});
```

**Custom Render with Options:**
```javascript
// test-utils.jsx
function customRender(
  ui,
  {
    // Default values
    route = '/',
    user = null,
    theme = 'light',
    ...renderOptions
  } = {}
) {
  // Set initial route
  window.history.pushState({}, 'Test', route);

  function Wrapper({ children }) {
    const queryClient = new QueryClient({
      defaultOptions: { queries: { retry: false } }
    });

    return (
      <BrowserRouter>
        <QueryClientProvider client={queryClient}>
          <ThemeProvider theme={{ mode: theme }}>
            <AuthProvider value={{ user }}>
              {children}
            </AuthProvider>
          </ThemeProvider>
        </QueryClientProvider>
      </BrowserRouter>
    );
  }

  return render(ui, { wrapper: Wrapper, ...renderOptions });
}

export { customRender as render };
```

**Using Custom Options:**
```javascript
import { render, screen } from './test-utils';

test('renders with custom options', () => {
  render(<Dashboard />, {
    route: '/dashboard',
    user: { id: 1, name: 'John', role: 'admin' },
    theme: 'dark'
  });

  expect(screen.getByText('Welcome, John')).toBeInTheDocument();
  expect(screen.getByText('Admin Panel')).toBeInTheDocument();
});

test('renders for guest user', () => {
  render(<Dashboard />, {
    user: null
  });

  expect(screen.getByText('Please log in')).toBeInTheDocument();
});
```

**Provider-Specific Custom Renders:**
```javascript
// For components that only need specific providers
export function renderWithRouter(ui, { route = '/' } = {}) {
  window.history.pushState({}, 'Test', route);

  return render(ui, {
    wrapper: ({ children }) => <BrowserRouter>{children}</BrowserRouter>
  });
}

export function renderWithTheme(ui, { theme = 'light' } = {}) {
  return render(ui, {
    wrapper: ({ children }) => (
      <ThemeProvider theme={{ mode: theme }}>{children}</ThemeProvider>
    )
  });
}

// Usage
import { renderWithRouter } from './test-utils';

test('renders with router only', () => {
  renderWithRouter(<MyComponent />, { route: '/about' });
});
```

**Benefits:**
- Reduces boilerplate in tests
- Ensures consistent provider setup
- Makes tests more readable
- Easy to update provider config globally
- Supports custom options per test

---

### Question 8: What are the best practices for writing React Testing Library tests?

**Answer:**

**1. Use Accessible Queries (Priority Order):**
```javascript
// ✅ Best - getByRole
screen.getByRole('button', { name: /submit/i });
screen.getByRole('heading', { level: 1 });

// ✅ Good - getByLabelText (forms)
screen.getByLabelText(/email/i);

// ⚠️ OK - getByText
screen.getByText(/welcome/i);

// ❌ Last resort - getByTestId
screen.getByTestId('custom-element');
```

**2. Test User Behavior, Not Implementation:**
```javascript
// ❌ Bad - testing implementation
test('bad test', () => {
  const { result } = renderHook(() => useState(0));
  expect(result.current[0]).toBe(0);
});

// ✅ Good - testing behavior
test('good test', async () => {
  render(<Counter />);
  expect(screen.getByText('Count: 0')).toBeInTheDocument();

  await userEvent.click(screen.getByRole('button', { name: /increment/i }));
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

**3. Use userEvent Instead of fireEvent:**
```javascript
// ❌ Less realistic
fireEvent.change(input, { target: { value: 'test' } });

// ✅ More realistic
await userEvent.type(input, 'test');
```

**4. Use findBy for Async Instead of waitFor + getBy:**
```javascript
// ❌ Verbose
await waitFor(() => {
  expect(screen.getByText('Loaded')).toBeInTheDocument();
});

// ✅ Concise
expect(await screen.findByText('Loaded')).toBeInTheDocument();
```

**5. Use screen Instead of Destructuring:**
```javascript
// ❌ Destructuring
const { getByRole } = render(<MyComponent />);
expect(getByRole('button')).toBeInTheDocument();

// ✅ Screen
render(<MyComponent />);
expect(screen.getByRole('button')).toBeInTheDocument();
```

**6. Query by Accessibility Features:**
```javascript
// ✅ Good - accessible form
<form>
  <label htmlFor="email">Email</label>
  <input id="email" type="email" />
  <button type="submit">Submit</button>
</form>

test('accessible test', async () => {
  render(<Form />);

  await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
  await userEvent.click(screen.getByRole('button', { name: /submit/i }));
});
```

**7. Test Error States and Edge Cases:**
```javascript
test('shows error for invalid email', async () => {
  render(<LoginForm />);

  await userEvent.type(screen.getByLabelText(/email/i), 'invalid');
  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  expect(await screen.findByRole('alert')).toHaveTextContent('Invalid email');
});

test('disables button while submitting', async () => {
  render(<LoginForm />);

  const button = screen.getByRole('button', { name: /login/i });
  await userEvent.click(button);

  expect(button).toBeDisabled();
});
```

**8. Use Custom Render for Providers:**
```javascript
// Create custom render once
function customRender(ui, options) {
  return render(ui, {
    wrapper: AllProviders,
    ...options
  });
}

// Use in all tests
test('test with providers', () => {
  customRender(<MyComponent />);
});
```

**9. Descriptive Test Names:**
```javascript
// ✅ Good
test('shows error message when login fails with invalid credentials', () => {});
test('disables submit button while form is submitting', () => {});

// ❌ Bad
test('login test', () => {});
test('works', () => {});
```

**10. Clean Up and Isolation:**
```javascript
// Automatic cleanup with @testing-library/react
afterEach(() => {
  cleanup(); // Usually automatic
  jest.clearAllMocks();
});

// Each test should be independent
test('test 1', () => {
  render(<Component />);
  // Test 1 logic
});

test('test 2', () => {
  render(<Component />); // Fresh render
  // Test 2 logic
});
```

---

### Question 9: How do you test components that use React Router?

**Answer:**

Test routing using `MemoryRouter` or custom render with `BrowserRouter`.

**Using MemoryRouter:**
```javascript
import { MemoryRouter } from 'react-router-dom';

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/users/:id" element={<UserProfile />} />
    </Routes>
  );
}

test('renders home page', () => {
  render(
    <MemoryRouter initialEntries={['/']}>
      <App />
    </MemoryRouter>
  );

  expect(screen.getByRole('heading', { name: /home/i })).toBeInTheDocument();
});

test('renders about page', () => {
  render(
    <MemoryRouter initialEntries={['/about']}>
      <App />
    </MemoryRouter>
  );

  expect(screen.getByRole('heading', { name: /about/i })).toBeInTheDocument();
});

test('renders user profile with params', () => {
  render(
    <MemoryRouter initialEntries={['/users/123']}>
      <App />
    </MemoryRouter>
  );

  expect(screen.getByText('User ID: 123')).toBeInTheDocument();
});
```

**Testing Navigation:**
```javascript
test('navigates to about page', async () => {
  render(
    <MemoryRouter initialEntries={['/']}>
      <App />
    </MemoryRouter>
  );

  // Click link to navigate
  await userEvent.click(screen.getByRole('link', { name: /about/i }));

  // Verify navigation
  expect(screen.getByRole('heading', { name: /about/i })).toBeInTheDocument();
});
```

**Custom Render with Router:**
```javascript
// test-utils.jsx
function renderWithRouter(ui, { route = '/' } = {}) {
  return render(
    <MemoryRouter initialEntries={[route]}>
      {ui}
    </MemoryRouter>
  );
}

// Usage
test('renders with router', () => {
  renderWithRouter(<App />, { route: '/about' });
  expect(screen.getByRole('heading', { name: /about/i })).toBeInTheDocument();
});
```

**Testing useNavigate:**
```javascript
function LoginPage() {
  const navigate = useNavigate();

  const handleLogin = () => {
    // Login logic
    navigate('/dashboard');
  };

  return <button onClick={handleLogin}>Login</button>;
}

test('navigates to dashboard after login', async () => {
  render(
    <MemoryRouter initialEntries={['/login']}>
      <Routes>
        <Route path="/login" element={<LoginPage />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </MemoryRouter>
  );

  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  expect(screen.getByRole('heading', { name: /dashboard/i })).toBeInTheDocument();
});
```

**Testing Protected Routes:**
```javascript
test('redirects to login when not authenticated', () => {
  render(
    <MemoryRouter initialEntries={['/dashboard']}>
      <Routes>
        <Route path="/dashboard" element={<ProtectedRoute><Dashboard /></ProtectedRoute>} />
        <Route path="/login" element={<Login />} />
      </Routes>
    </MemoryRouter>,
    { user: null }
  );

  expect(screen.getByRole('heading', { name: /login/i })).toBeInTheDocument();
});
```

---

### Question 10: How do you debug failing tests in React Testing Library?

**Answer:**

RTL provides several debugging utilities to help diagnose test failures.

**1. screen.debug():**
```javascript
test('debugging with screen.debug', () => {
  render(<MyComponent />);

  // Print entire document
  screen.debug();

  // Print specific element
  screen.debug(screen.getByRole('button'));

  // Limit output size
  screen.debug(undefined, 30000);
});
```

**2. logRoles():**
```javascript
import { logRoles } from '@testing-library/react';

test('log available roles', () => {
  const { container } = render(<MyComponent />);

  logRoles(container);
  // Prints all available roles and their elements
});

// Output example:
// heading:
//   <h1>Welcome</h1>
// button:
//   <button>Submit</button>
//   <button>Cancel</button>
```

**3. Testing Playground:**
```javascript
test('get testing playground URL', () => {
  render(<MyComponent />);

  screen.logTestingPlaygroundURL();
  // Opens interactive playground in browser
});
```

**4. prettyDOM():**
```javascript
import { prettyDOM } from '@testing-library/react';

test('pretty print DOM', () => {
  const { container } = render(<MyComponent />);

  console.log(prettyDOM(container, 10000));
});
```

**5. Using --verbose Flag:**
```bash
# Run tests with verbose output
npm test -- --verbose

# Run specific test file with debug info
npm test -- MyComponent.test.js --verbose
```

**6. Add Console Logs Strategically:**
```javascript
test('debug test', async () => {
  render(<UserProfile userId={1} />);

  console.log('Before findBy');
  const userName = await screen.findByText('John Doe');
  console.log('After findBy', userName);

  expect(userName).toBeInTheDocument();
});
```

**7. Check Query Errors:**
```javascript
// Error messages are helpful
test('debugging query', () => {
  render(<div>Hello World</div>);

  // This will show helpful error with available roles and text
  screen.getByRole('button'); // ❌

  // Error output shows:
  // Unable to find an accessible element with the role "button"
  //
  // Here are the accessible roles:
  //
  //   document:
  //   <body />
  //
  // --------------------------------------------------
  //   <div>Hello World</div>
});
```

**8. Use getAllBy to See All Matches:**
```javascript
test('see all matches', () => {
  render(
    <>
      <button>Save</button>
      <button>Cancel</button>
    </>
  );

  // See all buttons
  const buttons = screen.getAllByRole('button');
  console.log(buttons.map(b => b.textContent));
  // ['Save', 'Cancel']
});
```

**9. Check Async Timing:**
```javascript
test('debug async', async () => {
  render(<AsyncComponent />);

  try {
    const element = await screen.findByText('Loaded', {}, { timeout: 5000 });
    console.log('Found:', element);
  } catch (error) {
    console.log('Not found after 5 seconds');
    screen.debug();
  }
});
```

**10. VS Code Debugger:**
```javascript
// Add debugger statement
test('with debugger', () => {
  render(<MyComponent />);

  debugger; // Pause here

  expect(screen.getByText('Hello')).toBeInTheDocument();
});

// Run with:
// node --inspect-brk node_modules/.bin/jest --runInBand
```

---

## Summary

React Testing Library is the recommended way to test React components:

**Core Philosophy:**
- Test from user's perspective
- Focus on behavior, not implementation
- Use accessible queries
- Encourage best practices

**Key Concepts:**
- **Queries**: getBy (sync), queryBy (returns null), findBy (async)
- **Priority**: Role > Label > Text > TestId
- **Interactions**: Use userEvent over fireEvent
- **Async**: Use findBy and waitFor
- **Hooks**: Test with renderHook or through components
- **Providers**: Create custom render functions

**Best Practices:**
- Use semantic queries
- Test user behavior
- Handle async properly
- Test error states
- Keep tests isolated
- Use descriptive test names

**Debugging:**
- screen.debug()
- logRoles()
- Testing Playground
- Console logs
- Error messages

Master RTL to write maintainable, reliable tests that give you confidence in your React applications.

---

**Next:** [Integration Testing →](./04-integration-testing.md)

**Previous:** [← Jest Basics](./02-jest-basics.md)

---

[← Back to Testing Interview Prep](./README.md) | [↑ Back to Frontend](../README.md)
