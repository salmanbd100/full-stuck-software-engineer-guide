# Integration Testing

## Overview

Integration testing verifies that multiple components, modules, or systems work together correctly. Unlike unit tests that isolate individual functions, integration tests examine the interactions between different parts of your application. This guide covers integration testing strategies, API mocking with Mock Service Worker (MSW), testing data flow, and common integration patterns.

## Table of Contents
- [Integration vs Unit Tests](#integration-vs-unit-tests)
- [Testing Component Interactions](#testing-component-interactions)
- [API Integration Testing](#api-integration-testing)
- [Mock Service Worker (MSW)](#mock-service-worker-msw)
- [Testing Data Flow](#testing-data-flow)
- [Testing Forms and Workflows](#testing-forms-and-workflows)
- [Testing State Management](#testing-state-management)
- [Database and API Mocking](#database-and-api-mocking)
- [Integration Test Patterns](#integration-test-patterns)
- [Best Practices](#best-practices)
- [Interview Questions](#interview-questions)

## Integration vs Unit Tests

### Comparing Approaches

```javascript
// Example: Shopping cart functionality

// UNIT TEST - Tests isolated function
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

test('calculateTotal sums item prices', () => {
  const items = [
    { price: 10, quantity: 2 },
    { price: 5, quantity: 1 }
  ];

  expect(calculateTotal(items)).toBe(25);
});

// INTEGRATION TEST - Tests multiple components working together
function ShoppingCart() {
  const [items, setItems] = useState([]);

  const addItem = (item) => setItems([...items, item]);
  const removeItem = (id) => setItems(items.filter(i => i.id !== id));
  const total = calculateTotal(items);

  return (
    <div>
      <CartItems items={items} onRemove={removeItem} />
      <CartTotal total={total} />
      <AddItemForm onAdd={addItem} />
    </div>
  );
}

test('shopping cart adds item and updates total', async () => {
  render(<ShoppingCart />);

  // Initially empty
  expect(screen.getByText('Total: $0')).toBeInTheDocument();

  // Add item through form
  await userEvent.type(screen.getByLabelText(/product name/i), 'Laptop');
  await userEvent.type(screen.getByLabelText(/price/i), '999');
  await userEvent.click(screen.getByRole('button', { name: /add/i }));

  // Verify item added and total updated
  expect(screen.getByText('Laptop')).toBeInTheDocument();
  expect(screen.getByText('Total: $999')).toBeInTheDocument();
});
```

### When to Use Each

```javascript
// Use UNIT tests for:
const unitTestCandidates = {
  pureFunctions: 'formatCurrency(100) => "$100.00"',
  utilities: 'validateEmail("test@example.com") => true',
  algorithms: 'sortItems(items, "price") => sortedItems',
  calculations: 'calculateDiscount(100, 10) => 10'
};

// Use INTEGRATION tests for:
const integrationTestCandidates = {
  componentInteractions: 'Parent → Child component communication',
  userWorkflows: 'Login → Dashboard → Profile flow',
  apiIntegration: 'Component fetches and displays API data',
  formSubmission: 'Form validation → API call → Success message',
  stateManagement: 'Redux/Context updates propagate to UI'
};
```

### Test Pyramid Application

```javascript
// Typical test distribution
const testDistribution = {
  unit: {
    percentage: '70%',
    count: 700,
    examples: [
      'formatDate(date) returns correct format',
      'isValidEmail() validates email correctly',
      'calculateShipping() computes correct cost'
    ]
  },
  integration: {
    percentage: '20%',
    count: 200,
    examples: [
      'Form submits data and shows success message',
      'Search component fetches and displays results',
      'Cart updates when item is added/removed'
    ]
  },
  e2e: {
    percentage: '10%',
    count: 100,
    examples: [
      'User can complete checkout flow',
      'User can register, login, and access dashboard',
      'User can create and publish a post'
    ]
  }
};
```

## Testing Component Interactions

### Parent-Child Communication

```javascript
// Parent component
function UserList() {
  const [users, setUsers] = useState([]);
  const [selectedUser, setSelectedUser] = useState(null);

  useEffect(() => {
    fetch('/api/users')
      .then(r => r.json())
      .then(setUsers);
  }, []);

  return (
    <div>
      <UserTable users={users} onSelect={setSelectedUser} />
      {selectedUser && <UserDetails user={selectedUser} />}
    </div>
  );
}

// Integration test - tests parent and children together
test('selecting user shows details', async () => {
  // Setup MSW to mock API
  server.use(
    rest.get('/api/users', (req, res, ctx) => {
      return res(ctx.json([
        { id: 1, name: 'John Doe', email: 'john@example.com' },
        { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
      ]));
    })
  );

  render(<UserList />);

  // Wait for users to load
  await screen.findByText('John Doe');

  // Initially no details shown
  expect(screen.queryByText(/email:/i)).not.toBeInTheDocument();

  // Click on user
  await userEvent.click(screen.getByText('John Doe'));

  // Details appear
  expect(screen.getByText('Email: john@example.com')).toBeInTheDocument();
});
```

### Sibling Component Communication

```javascript
// Components communicate through parent state
function Dashboard() {
  const [filter, setFilter] = useState('all');

  return (
    <div>
      <FilterBar onFilterChange={setFilter} />
      <DataGrid filter={filter} />
    </div>
  );
}

test('filter updates grid data', async () => {
  render(<Dashboard />);

  // Initially shows all items
  await screen.findByText('Showing 10 items');

  // Change filter
  await userEvent.selectOptions(
    screen.getByLabelText(/filter/i),
    'active'
  );

  // Grid updates
  await screen.findByText('Showing 5 items');
});
```

### Prop Drilling Test

```javascript
// Deep component tree
function App() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Header />
      <Main>
        <Sidebar />
        <Content>
          <Article />
        </Content>
      </Main>
    </ThemeContext.Provider>
  );
}

// Test theme changes propagate through entire tree
test('theme change applies to all components', async () => {
  render(<App />);

  // Check initial theme on various components
  expect(screen.getByRole('banner')).toHaveClass('theme-light');
  expect(screen.getByRole('main')).toHaveClass('theme-light');
  expect(screen.getByRole('article')).toHaveClass('theme-light');

  // Toggle theme
  await userEvent.click(screen.getByRole('button', { name: /toggle theme/i }));

  // All components update
  expect(screen.getByRole('banner')).toHaveClass('theme-dark');
  expect(screen.getByRole('main')).toHaveClass('theme-dark');
  expect(screen.getByRole('article')).toHaveClass('theme-dark');
});
```

### Callback Props

```javascript
function SearchPage() {
  const [results, setResults] = useState([]);

  const handleSearch = async (query) => {
    const data = await api.search(query);
    setResults(data);
  };

  return (
    <div>
      <SearchBar onSearch={handleSearch} />
      <SearchResults results={results} />
    </div>
  );
}

test('search updates results', async () => {
  // Mock API
  jest.spyOn(api, 'search').mockResolvedValue([
    { id: 1, title: 'React Testing' },
    { id: 2, title: 'Jest Basics' }
  ]);

  render(<SearchPage />);

  // Enter search query
  await userEvent.type(screen.getByRole('searchbox'), 'react');
  await userEvent.click(screen.getByRole('button', { name: /search/i }));

  // Results appear
  expect(await screen.findByText('React Testing')).toBeInTheDocument();
  expect(screen.getByText('Jest Basics')).toBeInTheDocument();

  expect(api.search).toHaveBeenCalledWith('react');
});
```

## API Integration Testing

### Basic API Integration

```javascript
// Component that fetches data
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(r => r.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div role="alert">{error}</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// Integration test with mocked API
test('fetches and displays user data', async () => {
  // Mock fetch
  global.fetch = jest.fn(() =>
    Promise.resolve({
      json: () => Promise.resolve({
        id: 1,
        name: 'John Doe',
        email: 'john@example.com'
      })
    })
  );

  render(<UserProfile userId={1} />);

  // Loading state
  expect(screen.getByText('Loading...')).toBeInTheDocument();

  // Wait for data
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
  expect(screen.getByText('john@example.com')).toBeInTheDocument();

  // Verify fetch was called correctly
  expect(fetch).toHaveBeenCalledWith('/api/users/1');
});
```

### Testing Error States

```javascript
test('displays error message on failed fetch', async () => {
  // Mock failed fetch
  global.fetch = jest.fn(() =>
    Promise.reject(new Error('Network error'))
  );

  render(<UserProfile userId={1} />);

  // Error message appears
  const alert = await screen.findByRole('alert');
  expect(alert).toHaveTextContent('Network error');
});

test('handles 404 error', async () => {
  global.fetch = jest.fn(() =>
    Promise.resolve({
      ok: false,
      status: 404,
      json: () => Promise.resolve({ message: 'User not found' })
    })
  );

  render(<UserProfile userId={999} />);

  expect(await screen.findByRole('alert')).toHaveTextContent('User not found');
});
```

### Testing Multiple API Calls

```javascript
function Dashboard() {
  const [stats, setStats] = useState(null);
  const [notifications, setNotifications] = useState([]);
  const [tasks, setTasks] = useState([]);

  useEffect(() => {
    // Multiple parallel API calls
    Promise.all([
      fetch('/api/stats').then(r => r.json()),
      fetch('/api/notifications').then(r => r.json()),
      fetch('/api/tasks').then(r => r.json())
    ]).then(([statsData, notifData, tasksData]) => {
      setStats(statsData);
      setNotifications(notifData);
      setTasks(tasksData);
    });
  }, []);

  // Render logic...
}

test('loads all dashboard data', async () => {
  // Mock all endpoints
  global.fetch = jest.fn((url) => {
    if (url === '/api/stats') {
      return Promise.resolve({
        json: () => Promise.resolve({ users: 100, posts: 500 })
      });
    }
    if (url === '/api/notifications') {
      return Promise.resolve({
        json: () => Promise.resolve([{ id: 1, message: 'New message' }])
      });
    }
    if (url === '/api/tasks') {
      return Promise.resolve({
        json: () => Promise.resolve([{ id: 1, title: 'Complete tests' }])
      });
    }
  });

  render(<Dashboard />);

  // All data loads
  expect(await screen.findByText('Users: 100')).toBeInTheDocument();
  expect(screen.getByText('New message')).toBeInTheDocument();
  expect(screen.getByText('Complete tests')).toBeInTheDocument();

  expect(fetch).toHaveBeenCalledTimes(3);
});
```

## Mock Service Worker (MSW)

### MSW Setup

```javascript
// src/mocks/handlers.js
import { rest } from 'msw';

export const handlers = [
  // GET request
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json([
        { id: 1, name: 'John Doe' },
        { id: 2, name: 'Jane Smith' }
      ])
    );
  }),

  // POST request
  rest.post('/api/users', (req, res, ctx) => {
    const { name, email } = req.body;

    return res(
      ctx.status(201),
      ctx.json({
        id: 3,
        name,
        email
      })
    );
  }),

  // Error response
  rest.get('/api/error', (req, res, ctx) => {
    return res(
      ctx.status(500),
      ctx.json({ message: 'Internal server error' })
    );
  })
];

// src/mocks/server.js
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// src/setupTests.js
import { server } from './mocks/server';

// Establish API mocking before all tests
beforeAll(() => server.listen());

// Reset any request handlers after each test
afterEach(() => server.resetHandlers());

// Clean up after all tests
afterAll(() => server.close());
```

### Using MSW in Tests

```javascript
import { rest } from 'msw';
import { server } from './mocks/server';

test('fetches users successfully', async () => {
  render(<UserList />);

  // Wait for users to load (uses default handler from handlers.js)
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
  expect(screen.getByText('Jane Smith')).toBeInTheDocument();
});

test('handles server error', async () => {
  // Override handler for this test
  server.use(
    rest.get('/api/users', (req, res, ctx) => {
      return res(ctx.status(500), ctx.json({ message: 'Server error' }));
    })
  );

  render(<UserList />);

  expect(await screen.findByRole('alert')).toHaveTextContent('Server error');
});

test('filters users by query parameter', async () => {
  server.use(
    rest.get('/api/users', (req, res, ctx) => {
      const query = req.url.searchParams.get('search');

      const allUsers = [
        { id: 1, name: 'John Doe' },
        { id: 2, name: 'Jane Smith' },
        { id: 3, name: 'Bob Johnson' }
      ];

      const filteredUsers = query
        ? allUsers.filter(u => u.name.toLowerCase().includes(query.toLowerCase()))
        : allUsers;

      return res(ctx.json(filteredUsers));
    })
  );

  render(<UserList />);

  // Type in search box
  await userEvent.type(screen.getByRole('searchbox'), 'john');

  // Only matching users shown
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
  expect(screen.getByText('Bob Johnson')).toBeInTheDocument();
  expect(screen.queryByText('Jane Smith')).not.toBeInTheDocument();
});
```

### MSW with Delays

```javascript
test('shows loading state while fetching', async () => {
  server.use(
    rest.get('/api/users', (req, res, ctx) => {
      // Simulate network delay
      return res(
        ctx.delay(1000),
        ctx.json([{ id: 1, name: 'John' }])
      );
    })
  );

  render(<UserList />);

  // Loading indicator appears
  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  // Data appears after delay
  expect(await screen.findByText('John')).toBeInTheDocument();

  // Loading indicator gone
  expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
});
```

### MSW with Authentication

```javascript
// Check authorization header
rest.get('/api/profile', (req, res, ctx) => {
  const authHeader = req.headers.get('Authorization');

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res(
      ctx.status(401),
      ctx.json({ message: 'Unauthorized' })
    );
  }

  return res(
    ctx.json({
      id: 1,
      name: 'John Doe',
      email: 'john@example.com'
    })
  );
});

test('sends auth token with request', async () => {
  // Mock localStorage with token
  localStorage.setItem('token', 'fake-jwt-token');

  render(<Profile />);

  expect(await screen.findByText('John Doe')).toBeInTheDocument();
});

test('shows error without auth token', async () => {
  localStorage.clear();

  render(<Profile />);

  expect(await screen.findByRole('alert')).toHaveTextContent('Unauthorized');
});
```

## Testing Data Flow

### Redux Integration

```javascript
// Redux store setup for tests
function renderWithRedux(
  ui,
  {
    initialState,
    store = createStore(rootReducer, initialState),
    ...renderOptions
  } = {}
) {
  function Wrapper({ children }) {
    return <Provider store={store}>{children}</Provider>;
  }

  return render(ui, { wrapper: Wrapper, ...renderOptions });
}

// Component using Redux
function TodoList() {
  const todos = useSelector(state => state.todos);
  const dispatch = useDispatch();

  const addTodo = (text) => {
    dispatch({ type: 'ADD_TODO', payload: { text } });
  };

  return (
    <div>
      <AddTodoForm onAdd={addTodo} />
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}

// Integration test
test('adds todo to store and displays it', async () => {
  renderWithRedux(<TodoList />, {
    initialState: { todos: [] }
  });

  // Add todo
  await userEvent.type(screen.getByRole('textbox'), 'Buy milk');
  await userEvent.click(screen.getByRole('button', { name: /add/i }));

  // Todo appears in list
  expect(screen.getByText('Buy milk')).toBeInTheDocument();
});
```

### Context Integration

```javascript
// Auth context
const AuthContext = createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = async (email, password) => {
    const userData = await api.login(email, password);
    setUser(userData);
  };

  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Component using context
function Dashboard() {
  const { user, logout } = useContext(AuthContext);

  if (!user) {
    return <div>Please log in</div>;
  }

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

// Integration test
test('login flow updates UI', async () => {
  jest.spyOn(api, 'login').mockResolvedValue({
    id: 1,
    name: 'John Doe'
  });

  render(
    <AuthProvider>
      <LoginForm />
      <Dashboard />
    </AuthProvider>
  );

  // Initially not logged in
  expect(screen.getByText('Please log in')).toBeInTheDocument();

  // Login
  await userEvent.type(screen.getByLabelText(/email/i), 'john@example.com');
  await userEvent.type(screen.getByLabelText(/password/i), 'password');
  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  // Dashboard appears
  expect(await screen.findByText('Welcome, John Doe')).toBeInTheDocument();

  // Logout
  await userEvent.click(screen.getByRole('button', { name: /logout/i }));

  // Back to login prompt
  expect(screen.getByText('Please log in')).toBeInTheDocument();
});
```

### React Query Integration

```javascript
import { QueryClient, QueryClientProvider } from 'react-query';

function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        retry: false, // Disable retries in tests
        cacheTime: 0
      }
    }
  });
}

function renderWithQueryClient(ui) {
  const testQueryClient = createTestQueryClient();

  return render(
    <QueryClientProvider client={testQueryClient}>
      {ui}
    </QueryClientProvider>
  );
}

// Component using React Query
function UserProfile({ userId }) {
  const { data, isLoading, error } = useQuery(
    ['user', userId],
    () => fetch(`/api/users/${userId}`).then(r => r.json())
  );

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div role="alert">{error.message}</div>;

  return <div>{data.name}</div>;
}

// Test
test('fetches and caches user data', async () => {
  server.use(
    rest.get('/api/users/1', (req, res, ctx) => {
      return res(ctx.json({ id: 1, name: 'John Doe' }));
    })
  );

  renderWithQueryClient(<UserProfile userId={1} />);

  // Loading state
  expect(screen.getByText('Loading...')).toBeInTheDocument();

  // Data loads
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
});
```

## Testing Forms and Workflows

### Multi-Step Form

```javascript
function RegistrationWizard() {
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    firstName: '',
    lastName: ''
  });

  const handleSubmit = async () => {
    await api.register(formData);
    // Redirect to dashboard
  };

  return (
    <div>
      {step === 1 && (
        <Step1
          data={formData}
          onNext={(data) => {
            setFormData({ ...formData, ...data });
            setStep(2);
          }}
        />
      )}
      {step === 2 && (
        <Step2
          data={formData}
          onNext={(data) => {
            setFormData({ ...formData, ...data });
            setStep(3);
          }}
          onBack={() => setStep(1)}
        />
      )}
      {step === 3 && (
        <Step3
          data={formData}
          onSubmit={handleSubmit}
          onBack={() => setStep(2)}
        />
      )}
    </div>
  );
}

// Integration test for complete workflow
test('completes registration wizard', async () => {
  jest.spyOn(api, 'register').mockResolvedValue({ success: true });

  render(<RegistrationWizard />);

  // Step 1: Email and password
  await userEvent.type(screen.getByLabelText(/email/i), 'john@example.com');
  await userEvent.type(screen.getByLabelText(/password/i), 'SecurePass123!');
  await userEvent.click(screen.getByRole('button', { name: /next/i }));

  // Step 2: Personal info
  await screen.findByLabelText(/first name/i);
  await userEvent.type(screen.getByLabelText(/first name/i), 'John');
  await userEvent.type(screen.getByLabelText(/last name/i), 'Doe');
  await userEvent.click(screen.getByRole('button', { name: /next/i }));

  // Step 3: Review and submit
  await screen.findByText(/review/i);
  expect(screen.getByText('john@example.com')).toBeInTheDocument();
  expect(screen.getByText('John Doe')).toBeInTheDocument();

  await userEvent.click(screen.getByRole('button', { name: /submit/i }));

  // Verify API called with correct data
  expect(api.register).toHaveBeenCalledWith({
    email: 'john@example.com',
    password: 'SecurePass123!',
    firstName: 'John',
    lastName: 'Doe'
  });
});

test('allows going back in wizard', async () => {
  render(<RegistrationWizard />);

  // Fill step 1
  await userEvent.type(screen.getByLabelText(/email/i), 'john@example.com');
  await userEvent.type(screen.getByLabelText(/password/i), 'password');
  await userEvent.click(screen.getByRole('button', { name: /next/i }));

  // On step 2
  await screen.findByLabelText(/first name/i);

  // Go back
  await userEvent.click(screen.getByRole('button', { name: /back/i }));

  // Back on step 1, data preserved
  expect(screen.getByLabelText(/email/i)).toHaveValue('john@example.com');
});
```

### Form Validation

```javascript
function ContactForm({ onSubmit }) {
  const [errors, setErrors] = useState({});

  const validate = (data) => {
    const errors = {};

    if (!data.email) {
      errors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(data.email)) {
      errors.email = 'Email is invalid';
    }

    if (!data.message) {
      errors.message = 'Message is required';
    } else if (data.message.length < 10) {
      errors.message = 'Message must be at least 10 characters';
    }

    return errors;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const data = Object.fromEntries(formData);

    const validationErrors = validate(data);

    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }

    setErrors({});
    onSubmit(data);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" name="email" type="email" />
        {errors.email && <span role="alert">{errors.email}</span>}
      </div>

      <div>
        <label htmlFor="message">Message</label>
        <textarea id="message" name="message" />
        {errors.message && <span role="alert">{errors.message}</span>}
      </div>

      <button type="submit">Send</button>
    </form>
  );
}

// Integration test for validation flow
test('shows validation errors', async () => {
  const handleSubmit = jest.fn();
  render(<ContactForm onSubmit={handleSubmit} />);

  // Submit empty form
  await userEvent.click(screen.getByRole('button', { name: /send/i }));

  // Validation errors appear
  expect(screen.getByText('Email is required')).toBeInTheDocument();
  expect(screen.getByText('Message is required')).toBeInTheDocument();

  // Form not submitted
  expect(handleSubmit).not.toHaveBeenCalled();

  // Fix email but message still invalid
  await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
  await userEvent.type(screen.getByLabelText(/message/i), 'Short');
  await userEvent.click(screen.getByRole('button', { name: /send/i }));

  // Message error updates
  expect(screen.queryByText('Email is required')).not.toBeInTheDocument();
  expect(screen.getByText('Message must be at least 10 characters')).toBeInTheDocument();

  // Fix message
  await userEvent.clear(screen.getByLabelText(/message/i));
  await userEvent.type(screen.getByLabelText(/message/i), 'This is a valid message');
  await userEvent.click(screen.getByRole('button', { name: /send/i }));

  // No errors, form submitted
  expect(screen.queryByRole('alert')).not.toBeInTheDocument();
  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    message: 'This is a valid message'
  });
});
```

## Testing State Management

### Redux Async Actions

```javascript
// Thunk action
function fetchUsers() {
  return async (dispatch) => {
    dispatch({ type: 'FETCH_USERS_START' });

    try {
      const response = await fetch('/api/users');
      const users = await response.json();
      dispatch({ type: 'FETCH_USERS_SUCCESS', payload: users });
    } catch (error) {
      dispatch({ type: 'FETCH_USERS_ERROR', payload: error.message });
    }
  };
}

// Integration test
test('fetches users and updates store', async () => {
  server.use(
    rest.get('/api/users', (req, res, ctx) => {
      return res(ctx.json([{ id: 1, name: 'John' }]));
    })
  );

  const store = createStore(rootReducer);

  render(
    <Provider store={store}>
      <UserList />
    </Provider>
  );

  // Initially loading
  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  // Users load
  expect(await screen.findByText('John')).toBeInTheDocument();

  // Check store state
  const state = store.getState();
  expect(state.users.loading).toBe(false);
  expect(state.users.data).toHaveLength(1);
  expect(state.users.error).toBeNull();
});
```

### Context with Reducer

```javascript
const initialState = {
  cart: [],
  total: 0
};

function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      const newCart = [...state.cart, action.payload];
      return {
        cart: newCart,
        total: newCart.reduce((sum, item) => sum + item.price, 0)
      };
    case 'REMOVE_ITEM':
      const filteredCart = state.cart.filter(item => item.id !== action.payload);
      return {
        cart: filteredCart,
        total: filteredCart.reduce((sum, item) => sum + item.price, 0)
      };
    default:
      return state;
  }
}

const CartContext = createContext();

function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  return (
    <CartContext.Provider value={{ state, dispatch }}>
      {children}
    </CartContext.Provider>
  );
}

// Integration test
test('cart state updates flow', async () => {
  render(
    <CartProvider>
      <ProductList />
      <Cart />
    </CartProvider>
  );

  // Initially empty
  expect(screen.getByText('Cart is empty')).toBeInTheDocument();
  expect(screen.getByText('Total: $0')).toBeInTheDocument();

  // Add item
  await userEvent.click(
    screen.getAllByRole('button', { name: /add to cart/i })[0]
  );

  // Cart updates
  expect(screen.queryByText('Cart is empty')).not.toBeInTheDocument();
  expect(screen.getByText('Total: $99')).toBeInTheDocument();

  // Remove item
  await userEvent.click(screen.getByRole('button', { name: /remove/i }));

  // Back to empty
  expect(screen.getByText('Cart is empty')).toBeInTheDocument();
  expect(screen.getByText('Total: $0')).toBeInTheDocument();
});
```

## Database and API Mocking

### GraphQL Mocking

```javascript
import { graphql } from 'msw';

// GraphQL handlers
const graphqlHandlers = [
  graphql.query('GetUser', (req, res, ctx) => {
    const { id } = req.variables;

    return res(
      ctx.data({
        user: {
          id,
          name: 'John Doe',
          email: 'john@example.com'
        }
      })
    );
  }),

  graphql.mutation('UpdateUser', (req, res, ctx) => {
    const { id, name } = req.variables;

    return res(
      ctx.data({
        updateUser: {
          id,
          name,
          email: 'john@example.com'
        }
      })
    );
  })
];

server.use(...graphqlHandlers);

test('fetches user with GraphQL', async () => {
  render(<UserProfile userId="1" />);

  expect(await screen.findByText('John Doe')).toBeInTheDocument();
});
```

### WebSocket Mocking

```javascript
// Mock WebSocket
class MockWebSocket {
  constructor(url) {
    this.url = url;
    this.readyState = 0;

    setTimeout(() => {
      this.readyState = 1;
      this.onopen?.();
    }, 100);
  }

  send(data) {
    // Simulate receiving message
    setTimeout(() => {
      this.onmessage?.({
        data: JSON.stringify({ type: 'response', payload: 'Server received: ' + data })
      });
    }, 100);
  }

  close() {
    this.readyState = 3;
    this.onclose?.();
  }
}

global.WebSocket = MockWebSocket;

test('WebSocket communication', async () => {
  render(<ChatRoom />);

  // Wait for connection
  await screen.findByText(/connected/i);

  // Send message
  await userEvent.type(screen.getByRole('textbox'), 'Hello!');
  await userEvent.click(screen.getByRole('button', { name: /send/i }));

  // Receive response
  expect(await screen.findByText(/server received: hello/i)).toBeInTheDocument();
});
```

## Integration Test Patterns

### Testing CRUD Operations

```javascript
test('complete CRUD flow', async () => {
  render(<TodoApp />);

  // CREATE
  await userEvent.type(screen.getByRole('textbox'), 'New todo');
  await userEvent.click(screen.getByRole('button', { name: /add/i }));
  expect(await screen.findByText('New todo')).toBeInTheDocument();

  // READ - already verified above

  // UPDATE
  await userEvent.click(screen.getByRole('button', { name: /edit/i }));
  const input = screen.getByDisplayValue('New todo');
  await userEvent.clear(input);
  await userEvent.type(input, 'Updated todo');
  await userEvent.click(screen.getByRole('button', { name: /save/i }));
  expect(screen.getByText('Updated todo')).toBeInTheDocument();

  // DELETE
  await userEvent.click(screen.getByRole('button', { name: /delete/i }));
  await userEvent.click(screen.getByRole('button', { name: /confirm/i }));
  expect(screen.queryByText('Updated todo')).not.toBeInTheDocument();
});
```

### Testing Pagination

```javascript
test('pagination flow', async () => {
  server.use(
    rest.get('/api/items', (req, res, ctx) => {
      const page = req.url.searchParams.get('page') || '1';
      const items = Array.from({ length: 10 }, (_, i) => ({
        id: (parseInt(page) - 1) * 10 + i + 1,
        name: `Item ${(parseInt(page) - 1) * 10 + i + 1}`
      }));

      return res(ctx.json({ items, totalPages: 5 }));
    })
  );

  render(<ItemList />);

  // Page 1 loaded
  expect(await screen.findByText('Item 1')).toBeInTheDocument();
  expect(screen.getByText('Page 1 of 5')).toBeInTheDocument();

  // Go to next page
  await userEvent.click(screen.getByRole('button', { name: /next/i }));

  // Page 2 loaded
  expect(await screen.findByText('Item 11')).toBeInTheDocument();
  expect(screen.getByText('Page 2 of 5')).toBeInTheDocument();

  // Previous button enabled
  expect(screen.getByRole('button', { name: /previous/i })).not.toBeDisabled();
});
```

### Testing Infinite Scroll

```javascript
test('infinite scroll loads more items', async () => {
  let page = 1;

  server.use(
    rest.get('/api/items', (req, res, ctx) => {
      const items = Array.from({ length: 10 }, (_, i) => ({
        id: (page - 1) * 10 + i + 1,
        name: `Item ${(page - 1) * 10 + i + 1}`
      }));

      page++;

      return res(ctx.json({ items, hasMore: page <= 3 }));
    })
  );

  render(<InfiniteScrollList />);

  // Initial items loaded
  expect(await screen.findByText('Item 1')).toBeInTheDocument();

  // Scroll to bottom
  fireEvent.scroll(window, { target: { scrollY: 1000 } });

  // More items loaded
  expect(await screen.findByText('Item 11')).toBeInTheDocument();

  // Scroll again
  fireEvent.scroll(window, { target: { scrollY: 2000 } });

  // Even more items
  expect(await screen.findByText('Item 21')).toBeInTheDocument();
});
```

## Best Practices

### 1. Test User Journeys

```javascript
// ✅ Good - complete user journey
test('user can complete checkout', async () => {
  render(<App />);

  // Browse products
  await userEvent.click(screen.getByText(/products/i));

  // Add to cart
  await userEvent.click(screen.getByRole('button', { name: /add to cart/i }));

  // Go to cart
  await userEvent.click(screen.getByText(/cart/i));

  // Proceed to checkout
  await userEvent.click(screen.getByRole('button', { name: /checkout/i }));

  // Fill shipping info
  await userEvent.type(screen.getByLabelText(/address/i), '123 Main St');

  // Submit order
  await userEvent.click(screen.getByRole('button', { name: /place order/i }));

  // Confirmation
  expect(await screen.findByText(/order confirmed/i)).toBeInTheDocument();
});

// ❌ Bad - testing isolated pieces
test('adds to cart', () => {});
test('shows cart', () => {});
test('checkout form', () => {});
```

### 2. Mock at Network Level

```javascript
// ✅ Good - mock at network boundary (MSW)
server.use(
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json([{ id: 1, name: 'John' }]));
  })
);

// ❌ Bad - mock internal functions
jest.mock('./api', () => ({
  fetchUsers: jest.fn(() => [{ id: 1, name: 'John' }])
}));
```

### 3. Test Data Flow

```javascript
// ✅ Good - test how data flows through app
test('search updates results', async () => {
  render(<App />);

  // Type in search
  await userEvent.type(screen.getByRole('searchbox'), 'react');

  // API called with query
  await waitFor(() => {
    expect(fetch).toHaveBeenCalledWith('/api/search?q=react');
  });

  // Results update
  expect(await screen.findByText('React Testing')).toBeInTheDocument();
});
```

### 4. Test Error Recovery

```javascript
test('retries failed request', async () => {
  let attempt = 0;

  server.use(
    rest.get('/api/data', (req, res, ctx) => {
      attempt++;

      if (attempt < 3) {
        return res(ctx.status(500));
      }

      return res(ctx.json({ data: 'success' }));
    })
  );

  render(<DataComponent />);

  // Eventually succeeds after retries
  expect(await screen.findByText('success')).toBeInTheDocument();
});
```

### 5. Keep Tests Independent

```javascript
// ✅ Good - each test is independent
describe('TodoList', () => {
  test('adds todo', async () => {
    render(<TodoList />);
    // Test logic
  });

  test('removes todo', async () => {
    render(<TodoList />);
    // Different test, fresh component
  });
});

// ❌ Bad - tests depend on each other
describe('TodoList', () => {
  let screen;

  beforeAll(() => {
    screen = render(<TodoList />);
  });

  test('adds todo', async () => {
    // Modifies shared component
  });

  test('removes todo', async () => {
    // Depends on previous test
  });
});
```

## Interview Questions

### Question 1: What is integration testing and how does it differ from unit testing?

**Answer:**

Integration testing verifies that multiple components or systems work together correctly, while unit testing focuses on isolated functions or components.

**Key Differences:**

```javascript
// UNIT TEST - Tests isolated function
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price, 0);
}

test('calculateTotal sums prices', () => {
  const items = [{ price: 10 }, { price: 20 }];
  expect(calculateTotal(items)).toBe(30);
});

// INTEGRATION TEST - Tests components working together
function ShoppingCart() {
  const [items, setItems] = useState([]);

  return (
    <div>
      <ItemList items={items} />
      <AddItemForm onAdd={(item) => setItems([...items, item])} />
      <Total amount={calculateTotal(items)} />
    </div>
  );
}

test('adding item updates total', async () => {
  render(<ShoppingCart />);

  // Initially $0
  expect(screen.getByText('Total: $0')).toBeInTheDocument();

  // Add item through form
  await userEvent.type(screen.getByLabelText(/price/i), '50');
  await userEvent.click(screen.getByRole('button', { name: /add/i }));

  // Total updates
  expect(screen.getByText('Total: $50')).toBeInTheDocument();
});
```

**What Integration Tests Cover:**
- Component communication (parent-child, siblings)
- API calls and data fetching
- State management (Redux, Context)
- Form submissions
- User workflows
- Error handling across components

**When to Use:**
- **Unit tests**: Pure functions, utilities, calculations
- **Integration tests**: User interactions, data flow, API integration
- **E2E tests**: Critical end-to-end user journeys

**Example comparing all three:**
```javascript
// Unit: Test function
test('validateEmail returns true for valid email', () => {
  expect(validateEmail('test@example.com')).toBe(true);
});

// Integration: Test form component with API
test('form submits data to API and shows success', async () => {
  render(<SignupForm />);

  await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
  await userEvent.click(screen.getByRole('button', { name: /submit/i }));

  expect(await screen.findByText(/success/i)).toBeInTheDocument();
});

// E2E: Test complete user flow
test('user can sign up and access dashboard', async ({ page }) => {
  await page.goto('/signup');
  await page.fill('[name="email"]', 'test@example.com');
  await page.click('button:has-text("Sign Up")');
  await expect(page).toHaveURL('/dashboard');
});
```

---

### Question 2: What is Mock Service Worker (MSW) and why use it?

**Answer:**

Mock Service Worker (MSW) intercepts network requests at the service worker/network level, providing realistic API mocking for tests.

**Why MSW is Better:**

```javascript
// ❌ Traditional approach - mock internal functions
jest.mock('./api', () => ({
  fetchUsers: jest.fn(() => Promise.resolve([{ id: 1 }]))
}));

// Problems:
// 1. Tightly coupled to implementation
// 2. Must mock each API function individually
// 3. Doesn't test actual fetch/axios calls
// 4. Breaks if API client changes

// ✅ MSW - mock at network level
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json([{ id: 1, name: 'John' }]));
  })
);

// Benefits:
// 1. Implementation-agnostic (works with fetch, axios, etc.)
// 2. Mock once, use everywhere
// 3. Tests real network calls
// 4. Same mocks work in browser and Node
```

**Setup Example:**
```javascript
// src/mocks/handlers.js
export const handlers = [
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json([
      { id: 1, name: 'John Doe' },
      { id: 2, name: 'Jane Smith' }
    ]));
  }),

  rest.post('/api/users', async (req, res, ctx) => {
    const { name, email } = await req.json();
    return res(
      ctx.status(201),
      ctx.json({ id: 3, name, email })
    );
  })
];

// src/mocks/server.js
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// src/setupTests.js
import { server } from './mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

**Using in Tests:**
```javascript
test('fetches users', async () => {
  render(<UserList />);

  // Uses default handler
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
});

test('handles error', async () => {
  // Override handler for this test
  server.use(
    rest.get('/api/users', (req, res, ctx) => {
      return res(ctx.status(500), ctx.json({ error: 'Server error' }));
    })
  );

  render(<UserList />);

  expect(await screen.findByRole('alert')).toHaveTextContent('Server error');
});
```

**Advanced Features:**
```javascript
// Query parameters
rest.get('/api/search', (req, res, ctx) => {
  const query = req.url.searchParams.get('q');
  // Filter results based on query
  return res(ctx.json(filteredResults));
});

// Request headers
rest.get('/api/profile', (req, res, ctx) => {
  const auth = req.headers.get('Authorization');
  if (!auth) {
    return res(ctx.status(401));
  }
  return res(ctx.json({ user: 'John' }));
});

// Delays (test loading states)
rest.get('/api/slow', (req, res, ctx) => {
  return res(
    ctx.delay(2000),
    ctx.json({ data: 'slow response' })
  );
});
```

---

### Question 3: How do you test components that fetch data from APIs?

**Answer:**

Test API integration by mocking network requests and verifying the component handles loading, success, and error states correctly.

**Complete Example:**
```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(r => {
        if (!r.ok) throw new Error('Failed to fetch');
        return r.json();
      })
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div role="alert">{error}</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

**Test Success State:**
```javascript
test('fetches and displays user data', async () => {
  server.use(
    rest.get('/api/users/1', (req, res, ctx) => {
      return res(ctx.json({
        id: 1,
        name: 'John Doe',
        email: 'john@example.com'
      }));
    })
  );

  render(<UserProfile userId={1} />);

  // Loading state
  expect(screen.getByText('Loading...')).toBeInTheDocument();

  // Data appears
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
  expect(screen.getByText('john@example.com')).toBeInTheDocument();

  // Loading gone
  expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
});
```

**Test Error State:**
```javascript
test('displays error on failed fetch', async () => {
  server.use(
    rest.get('/api/users/1', (req, res, ctx) => {
      return res(ctx.status(500), ctx.json({ message: 'Server error' }));
    })
  );

  render(<UserProfile userId={1} />);

  // Error message appears
  const alert = await screen.findByRole('alert');
  expect(alert).toHaveTextContent('Failed to fetch');
});
```

**Test Loading State:**
```javascript
test('shows loading indicator', async () => {
  server.use(
    rest.get('/api/users/1', (req, res, ctx) => {
      return res(
        ctx.delay(100), // Delay to ensure loading state is visible
        ctx.json({ id: 1, name: 'John' })
      );
    })
  );

  render(<UserProfile userId={1} />);

  // Loading appears immediately
  expect(screen.getByText('Loading...')).toBeInTheDocument();

  // Then data loads
  await screen.findByText('John');
});
```

**Test with React Query:**
```javascript
function UserProfile({ userId }) {
  const { data, isLoading, error } = useQuery(
    ['user', userId],
    () => fetch(`/api/users/${userId}`).then(r => r.json())
  );

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div role="alert">{error.message}</div>;

  return <div>{data.name}</div>;
}

test('fetches user with React Query', async () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } }
  });

  render(
    <QueryClientProvider client={queryClient}>
      <UserProfile userId={1} />
    </QueryClientProvider>
  );

  expect(await screen.findByText('John Doe')).toBeInTheDocument();
});
```

**Test Dynamic Parameters:**
```javascript
test('refetches when userId changes', async () => {
  server.use(
    rest.get('/api/users/:id', (req, res, ctx) => {
      const { id } = req.params;
      return res(ctx.json({ id, name: `User ${id}` }));
    })
  );

  const { rerender } = render(<UserProfile userId={1} />);

  expect(await screen.findByText('User 1')).toBeInTheDocument();

  // Change userId prop
  rerender(<UserProfile userId={2} />);

  expect(await screen.findByText('User 2')).toBeInTheDocument();
});
```

---

### Question 4: How do you test forms with validation and submission?

**Answer:**

Test forms by simulating user input, verifying validation errors, and checking successful submission.

**Form Component:**
```javascript
function LoginForm({ onSubmit }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});
  const [submitting, setSubmitting] = useState(false);

  const validate = () => {
    const errors = {};

    if (!email) {
      errors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(email)) {
      errors.email = 'Email is invalid';
    }

    if (!password) {
      errors.password = 'Password is required';
    } else if (password.length < 6) {
      errors.password = 'Password must be at least 6 characters';
    }

    return errors;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    const validationErrors = validate();

    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }

    setErrors({});
    setSubmitting(true);

    try {
      await onSubmit({ email, password });
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
        {errors.email && <span role="alert">{errors.email}</span>}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
        {errors.password && <span role="alert">{errors.password}</span>}
      </div>

      <button type="submit" disabled={submitting}>
        {submitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

**Test Validation Errors:**
```javascript
test('shows validation errors for empty fields', async () => {
  const handleSubmit = jest.fn();
  render(<LoginForm onSubmit={handleSubmit} />);

  // Submit without filling fields
  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  // Errors appear
  expect(screen.getByText('Email is required')).toBeInTheDocument();
  expect(screen.getByText('Password is required')).toBeInTheDocument();

  // Form not submitted
  expect(handleSubmit).not.toHaveBeenCalled();
});

test('shows error for invalid email', async () => {
  const handleSubmit = jest.fn();
  render(<LoginForm onSubmit={handleSubmit} />);

  await userEvent.type(screen.getByLabelText(/email/i), 'invalid-email');
  await userEvent.type(screen.getByLabelText(/password/i), 'password123');
  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  expect(screen.getByText('Email is invalid')).toBeInTheDocument();
  expect(handleSubmit).not.toHaveBeenCalled();
});

test('shows error for short password', async () => {
  const handleSubmit = jest.fn();
  render(<LoginForm onSubmit={handleSubmit} />);

  await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
  await userEvent.type(screen.getByLabelText(/password/i), '123');
  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  expect(screen.getByText('Password must be at least 6 characters')).toBeInTheDocument();
});
```

**Test Successful Submission:**
```javascript
test('submits form with valid data', async () => {
  const handleSubmit = jest.fn().mockResolvedValue();
  render(<LoginForm onSubmit={handleSubmit} />);

  // Fill valid data
  await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
  await userEvent.type(screen.getByLabelText(/password/i), 'password123');

  // Submit
  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  // No errors
  expect(screen.queryByRole('alert')).not.toBeInTheDocument();

  // Submitted with correct data
  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123'
  });
});
```

**Test Loading State:**
```javascript
test('disables button while submitting', async () => {
  const handleSubmit = jest.fn(() => new Promise(resolve => setTimeout(resolve, 1000)));
  render(<LoginForm onSubmit={handleSubmit} />);

  await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
  await userEvent.type(screen.getByLabelText(/password/i), 'password123');

  const submitButton = screen.getByRole('button', { name: /login/i });
  await userEvent.click(submitButton);

  // Button disabled and text changes
  expect(submitButton).toBeDisabled();
  expect(submitButton).toHaveTextContent('Logging in...');

  // Wait for submission to complete
  await waitFor(() => {
    expect(submitButton).not.toBeDisabled();
    expect(submitButton).toHaveTextContent('Login');
  });
});
```

**Test Field Clearing:**
```javascript
test('clears errors when field is corrected', async () => {
  const handleSubmit = jest.fn();
  render(<LoginForm onSubmit={handleSubmit} />);

  // Submit to show errors
  await userEvent.click(screen.getByRole('button', { name: /login/i }));
  expect(screen.getByText('Email is required')).toBeInTheDocument();

  // Type valid email
  await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
  await userEvent.type(screen.getByLabelText(/password/i), 'password123');
  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  // Errors cleared
  expect(screen.queryByText('Email is required')).not.toBeInTheDocument();
  expect(handleSubmit).toHaveBeenCalled();
});
```

---

### Question 5: How do you test Redux integration in components?

**Answer:**

Test Redux-connected components by providing a real or mock store and verifying state updates flow correctly.

**Redux Setup for Tests:**
```javascript
// test-utils.js
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import rootReducer from './reducers';

function renderWithRedux(
  ui,
  {
    initialState,
    store = createStore(rootReducer, initialState),
    ...renderOptions
  } = {}
) {
  function Wrapper({ children }) {
    return <Provider store={store}>{children}</Provider>;
  }

  return {
    ...render(ui, { wrapper: Wrapper, ...renderOptions }),
    store
  };
}

export * from '@testing-library/react';
export { renderWithRedux as render };
```

**Component Using Redux:**
```javascript
// TodoList component
function TodoList() {
  const todos = useSelector(state => state.todos);
  const dispatch = useDispatch();

  const addTodo = (text) => {
    dispatch({ type: 'ADD_TODO', payload: { id: Date.now(), text } });
  };

  const toggleTodo = (id) => {
    dispatch({ type: 'TOGGLE_TODO', payload: id });
  };

  return (
    <div>
      <AddTodoForm onAdd={addTodo} />
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span>{todo.text}</span>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Integration Tests:**
```javascript
import { render, screen } from './test-utils';

test('adds todo to store and displays it', async () => {
  const { store } = render(<TodoList />, {
    initialState: { todos: [] }
  });

  // Add todo
  await userEvent.type(screen.getByRole('textbox'), 'Buy milk');
  await userEvent.click(screen.getByRole('button', { name: /add/i }));

  // Todo appears
  expect(screen.getByText('Buy milk')).toBeInTheDocument();

  // Store updated
  const state = store.getState();
  expect(state.todos).toHaveLength(1);
  expect(state.todos[0].text).toBe('Buy milk');
});

test('toggles todo completion', async () => {
  const { store } = render(<TodoList />, {
    initialState: {
      todos: [{ id: 1, text: 'Buy milk', completed: false }]
    }
  });

  const checkbox = screen.getByRole('checkbox');

  // Initially unchecked
  expect(checkbox).not.toBeChecked();
  expect(store.getState().todos[0].completed).toBe(false);

  // Toggle
  await userEvent.click(checkbox);

  // Checked
  expect(checkbox).toBeChecked();
  expect(store.getState().todos[0].completed).toBe(true);

  // Toggle again
  await userEvent.click(checkbox);

  // Unchecked
  expect(checkbox).not.toBeChecked();
  expect(store.getState().todos[0].completed).toBe(false);
});
```

**Testing Async Actions (Redux Thunk):**
```javascript
// Async action
function fetchTodos() {
  return async (dispatch) => {
    dispatch({ type: 'FETCH_TODOS_START' });

    try {
      const response = await fetch('/api/todos');
      const todos = await response.json();
      dispatch({ type: 'FETCH_TODOS_SUCCESS', payload: todos });
    } catch (error) {
      dispatch({ type: 'FETCH_TODOS_ERROR', payload: error.message });
    }
  };
}

// Test
test('fetches todos from API', async () => {
  server.use(
    rest.get('/api/todos', (req, res, ctx) => {
      return res(ctx.json([
        { id: 1, text: 'Todo 1' },
        { id: 2, text: 'Todo 2' }
      ]));
    })
  );

  const { store } = render(<TodoList />);

  // Initially loading
  expect(store.getState().loading).toBe(true);

  // Todos load
  expect(await screen.findByText('Todo 1')).toBeInTheDocument();
  expect(screen.getByText('Todo 2')).toBeInTheDocument();

  // Store updated
  const state = store.getState();
  expect(state.loading).toBe(false);
  expect(state.todos).toHaveLength(2);
  expect(state.error).toBeNull();
});
```

**Testing Redux Toolkit:**
```javascript
import { configureStore } from '@reduxjs/toolkit';
import todosReducer from './todosSlice';

function renderWithStore(ui, preloadedState) {
  const store = configureStore({
    reducer: {
      todos: todosReducer
    },
    preloadedState
  });

  return {
    ...render(<Provider store={store}>{ui}</Provider>),
    store
  };
}

test('adds todo with Redux Toolkit', async () => {
  const { store } = renderWithStore(<TodoList />, {
    todos: { items: [], status: 'idle' }
  });

  await userEvent.type(screen.getByRole('textbox'), 'New todo');
  await userEvent.click(screen.getByRole('button', { name: /add/i }));

  expect(screen.getByText('New todo')).toBeInTheDocument();
  expect(store.getState().todos.items).toHaveLength(1);
});
```

---

### Question 6: How do you test Context API integration?

**Answer:**

Test Context by rendering components within the Provider and verifying state changes propagate correctly.

**Context Setup:**
```javascript
const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);

  const login = async (email, password) => {
    setLoading(true);
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify({ email, password })
      });
      const userData = await response.json();
      setUser(userData);
    } finally {
      setLoading(false);
    }
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, loading, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

**Components Using Context:**
```javascript
function LoginForm() {
  const { login, loading } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    login(email, password);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} />
      <button type="submit" disabled={loading}>Login</button>
    </form>
  );
}

function Dashboard() {
  const { user, logout } = useAuth();

  if (!user) {
    return <div>Please log in</div>;
  }

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

**Integration Test:**
```javascript
test('login flow updates entire app', async () => {
  server.use(
    rest.post('/api/login', async (req, res, ctx) => {
      const { email, password } = await req.json();

      if (email === 'test@example.com' && password === 'password') {
        return res(ctx.json({ id: 1, name: 'John Doe', email }));
      }

      return res(ctx.status(401), ctx.json({ error: 'Invalid credentials' }));
    })
  );

  render(
    <AuthProvider>
      <LoginForm />
      <Dashboard />
    </AuthProvider>
  );

  // Initially not logged in
  expect(screen.getByText('Please log in')).toBeInTheDocument();

  // Fill login form
  await userEvent.type(screen.getByRole('textbox'), 'test@example.com');
  await userEvent.type(screen.getByDisplayValue(''), 'password'); // password field
  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  // Dashboard updates with user data
  expect(await screen.findByText('Welcome, John Doe')).toBeInTheDocument();
  expect(screen.queryByText('Please log in')).not.toBeInTheDocument();

  // Logout
  await userEvent.click(screen.getByRole('button', { name: /logout/i }));

  // Back to logged out state
  expect(screen.getByText('Please log in')).toBeInTheDocument();
});
```

**Test with Custom Render:**
```javascript
function renderWithAuth(ui, { user = null } = {}) {
  function MockAuthProvider({ children }) {
    const [currentUser, setCurrentUser] = useState(user);

    const login = jest.fn((email, password) => {
      setCurrentUser({ id: 1, name: 'Test User', email });
      return Promise.resolve();
    });

    const logout = jest.fn(() => {
      setCurrentUser(null);
    });

    return (
      <AuthContext.Provider value={{ user: currentUser, login, logout, loading: false }}>
        {children}
      </AuthContext.Provider>
    );
  }

  return render(ui, { wrapper: MockAuthProvider });
}

test('shows dashboard for logged in user', () => {
  renderWithAuth(<Dashboard />, {
    user: { id: 1, name: 'John Doe', email: 'john@example.com' }
  });

  expect(screen.getByText('Welcome, John Doe')).toBeInTheDocument();
});

test('shows login prompt for guest', () => {
  renderWithAuth(<Dashboard />, { user: null });

  expect(screen.getByText('Please log in')).toBeInTheDocument();
});
```

---

## Summary

Integration testing is crucial for ensuring components work together correctly:

**Key Concepts:**
- **Scope**: Test multiple components/modules together
- **Focus**: User workflows, data flow, API integration
- **Tools**: MSW for API mocking, real stores for state
- **Philosophy**: Test behavior users see, not implementation

**Best Practices:**
- Mock at network boundary (MSW over function mocks)
- Test complete user journeys
- Verify data flows through the app
- Test error states and recovery
- Keep tests independent
- Use realistic test data

**Common Patterns:**
- Parent-child communication
- Form submission with validation
- API data fetching
- State management (Redux, Context)
- Multi-step workflows
- CRUD operations

**Next Steps:**
- Practice writing integration tests
- Learn MSW thoroughly
- Understand when to use integration vs unit tests
- Master testing async behavior
- Build confidence in testing complex flows

Integration tests provide the best return on investment - they catch real bugs while remaining maintainable and resilient to refactoring.

---

**Previous:** [← React Testing Library](./03-react-testing-library.md)

**Next:** [E2E Testing →](./05-e2e-testing.md)

---

[← Back to Testing Interview Prep](./README.md) | [↑ Back to Frontend](../README.md)
