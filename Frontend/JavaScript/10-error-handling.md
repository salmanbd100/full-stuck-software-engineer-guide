# Error Handling

Proper error handling is crucial for building robust applications. Understanding how to handle errors gracefully makes your code more reliable and maintainable.

## 📚 Core Concepts

### 1. Try/Catch/Finally

**Basic Structure**
```javascript
try {
    // Code that might throw an error
    const result = riskyOperation();
    console.log(result);
} catch (error) {
    // Handle the error
    console.error('Something went wrong:', error.message);
} finally {
    // Always executes (optional)
    console.log('Cleanup code here');
}
```

**Practical Example**
```javascript
function parseJSON(jsonString) {
    try {
        const data = JSON.parse(jsonString);
        return { success: true, data };
    } catch (error) {
        return { success: false, error: error.message };
    }
}

const result1 = parseJSON('{"name": "Alice"}');
console.log(result1); // { success: true, data: { name: 'Alice' } }

const result2 = parseJSON('invalid json');
console.log(result2); // { success: false, error: '...' }
```

**Finally Block**
```javascript
function readFile(filename) {
    let file;

    try {
        file = openFile(filename);
        const data = file.read();
        return data;
    } catch (error) {
        console.error('Error reading file:', error);
        return null;
    } finally {
        // Always close file, even if error occurred
        if (file) {
            file.close();
        }
    }
}
```

### 2. Throwing Errors

**throw Statement**
```javascript
function divide(a, b) {
    if (b === 0) {
        throw new Error('Division by zero');
    }
    return a / b;
}

try {
    const result = divide(10, 0);
} catch (error) {
    console.error(error.message); // "Division by zero"
}
```

**Custom Error Messages**
```javascript
function validateAge(age) {
    if (typeof age !== 'number') {
        throw new TypeError('Age must be a number');
    }

    if (age < 0) {
        throw new RangeError('Age cannot be negative');
    }

    if (age < 18) {
        throw new Error('Must be 18 or older');
    }

    return true;
}

try {
    validateAge('25');
} catch (error) {
    console.error(error.name);    // "TypeError"
    console.error(error.message); // "Age must be a number"
}
```

### 3. Error Types

**Built-in Error Types**
```javascript
// Error - Generic error
throw new Error('Something went wrong');

// SyntaxError - Invalid syntax
// eval('const x ='); // SyntaxError

// ReferenceError - Invalid reference
try {
    console.log(undefinedVariable);
} catch (error) {
    console.log(error instanceof ReferenceError); // true
}

// TypeError - Wrong type
try {
    null.toString();
} catch (error) {
    console.log(error instanceof TypeError); // true
}

// RangeError - Number out of range
try {
    const arr = new Array(-1);
} catch (error) {
    console.log(error instanceof RangeError); // true
}

// URIError - URI handling error
try {
    decodeURIComponent('%');
} catch (error) {
    console.log(error instanceof URIError); // true
}
```

### 4. Custom Error Classes

```javascript
// Custom error class
class ValidationError extends Error {
    constructor(message) {
        super(message);
        this.name = 'ValidationError';
    }
}

class NetworkError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.name = 'NetworkError';
        this.statusCode = statusCode;
    }
}

// Usage
function validateUser(user) {
    if (!user.name) {
        throw new ValidationError('Name is required');
    }

    if (!user.email) {
        throw new ValidationError('Email is required');
    }

    return true;
}

try {
    validateUser({ name: 'Alice' });
} catch (error) {
    if (error instanceof ValidationError) {
        console.error('Validation failed:', error.message);
    } else {
        console.error('Unknown error:', error);
    }
}
```

**More Advanced Custom Errors**
```javascript
class APIError extends Error {
    constructor(message, statusCode, endpoint) {
        super(message);
        this.name = 'APIError';
        this.statusCode = statusCode;
        this.endpoint = endpoint;
        this.timestamp = new Date();
    }

    toJSON() {
        return {
            name: this.name,
            message: this.message,
            statusCode: this.statusCode,
            endpoint: this.endpoint,
            timestamp: this.timestamp
        };
    }
}

async function fetchUser(id) {
    const response = await fetch(`/api/users/${id}`);

    if (!response.ok) {
        throw new APIError(
            'Failed to fetch user',
            response.status,
            `/api/users/${id}`
        );
    }

    return response.json();
}

try {
    const user = await fetchUser(123);
} catch (error) {
    if (error instanceof APIError) {
        console.error(error.toJSON());
    }
}
```

### 5. Async Error Handling

**Promises**
```javascript
function fetchData() {
    return fetch('/api/data')
        .then(response => {
            if (!response.ok) {
                throw new Error(`HTTP error: ${response.status}`);
            }
            return response.json();
        })
        .then(data => {
            console.log(data);
            return data;
        })
        .catch(error => {
            console.error('Error fetching data:', error);
            throw error; // Re-throw if needed
        });
}
```

**Async/Await**
```javascript
async function fetchUser(id) {
    try {
        const response = await fetch(`/api/users/${id}`);

        if (!response.ok) {
            throw new Error(`HTTP error: ${response.status}`);
        }

        const user = await response.json();
        return user;

    } catch (error) {
        console.error('Error fetching user:', error);

        // Return default or re-throw
        return null;
    }
}

// Multiple async operations
async function loadDashboard() {
    try {
        const [user, posts, comments] = await Promise.all([
            fetchUser(1),
            fetchPosts(),
            fetchComments()
        ]);

        return { user, posts, comments };

    } catch (error) {
        console.error('Error loading dashboard:', error);
        throw error;
    }
}
```

**Promise.allSettled() - Handle Multiple Promises**
```javascript
async function fetchMultipleUsers(ids) {
    const promises = ids.map(id => fetchUser(id));

    const results = await Promise.allSettled(promises);

    const successful = results
        .filter(r => r.status === 'fulfilled')
        .map(r => r.value);

    const failed = results
        .filter(r => r.status === 'rejected')
        .map(r => r.reason);

    return { successful, failed };
}

const { successful, failed } = await fetchMultipleUsers([1, 2, 3]);
console.log(`Loaded ${successful.length} users`);
console.log(`Failed to load ${failed.length} users`);
```

### 6. Error Boundaries (React)

```javascript
// React Error Boundary Component
class ErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false, error: null };
    }

    static getDerivedStateFromError(error) {
        return { hasError: true, error };
    }

    componentDidCatch(error, errorInfo) {
        console.error('Error caught by boundary:', error, errorInfo);
        // Log to error reporting service
        logErrorToService(error, errorInfo);
    }

    render() {
        if (this.state.hasError) {
            return (
                <div>
                    <h1>Something went wrong</h1>
                    <p>{this.state.error.message}</p>
                </div>
            );
        }

        return this.props.children;
    }
}

// Usage
function App() {
    return (
        <ErrorBoundary>
            <UserProfile />
            <PostsList />
        </ErrorBoundary>
    );
}
```

## 🎯 Common Interview Questions

### Q1: What's the difference between throw and return?

**Answer:**
```javascript
// return - normal flow control
function divide1(a, b) {
    if (b === 0) {
        return null; // Caller needs to check
    }
    return a / b;
}

const result1 = divide1(10, 0);
if (result1 === null) {
    console.log('Error');
}

// throw - exceptional flow control
function divide2(a, b) {
    if (b === 0) {
        throw new Error('Division by zero');
    }
    return a / b;
}

try {
    const result2 = divide2(10, 0);
} catch (error) {
    console.error(error.message);
}
```

### Q2: How do you handle errors in async code?

**Answer: Three approaches**
```javascript
// 1. .catch() with promises
fetch('/api/data')
    .then(res => res.json())
    .catch(error => console.error(error));

// 2. try/catch with async/await
async function fetchData() {
    try {
        const res = await fetch('/api/data');
        const data = await res.json();
        return data;
    } catch (error) {
        console.error(error);
        return null;
    }
}

// 3. Higher-order function wrapper
const asyncHandler = (fn) => {
    return async (...args) => {
        try {
            return await fn(...args);
        } catch (error) {
            console.error(error);
            throw error;
        }
    };
};

const fetchUser = asyncHandler(async (id) => {
    const res = await fetch(`/api/users/${id}`);
    return res.json();
});
```

### Q3: What happens if you don't catch an error?

**Answer:**
```javascript
// Uncaught error crashes the program
function riskyOperation() {
    throw new Error('Oops!');
}

// This will crash
// riskyOperation();

// In async code, uncaught promise rejections
async function asyncRisky() {
    throw new Error('Async error');
}

// Without await/catch, this creates unhandled rejection
asyncRisky();

// Global handlers
process.on('unhandledRejection', (reason, promise) => {
    console.error('Unhandled Rejection:', reason);
});

window.addEventListener('unhandledrejection', (event) => {
    console.error('Unhandled rejection:', event.reason);
});
```

## 💡 Practical Examples

### Example 1: Retry Logic

```javascript
async function fetchWithRetry(url, maxRetries = 3) {
    let lastError;

    for (let i = 0; i < maxRetries; i++) {
        try {
            const response = await fetch(url);

            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }

            return await response.json();

        } catch (error) {
            lastError = error;
            console.log(`Attempt ${i + 1} failed, retrying...`);

            // Wait before retry (exponential backoff)
            await new Promise(resolve =>
                setTimeout(resolve, 1000 * Math.pow(2, i))
            );
        }
    }

    throw new Error(`Failed after ${maxRetries} attempts: ${lastError.message}`);
}

// Usage
try {
    const data = await fetchWithRetry('/api/data');
    console.log(data);
} catch (error) {
    console.error('All retries failed:', error);
}
```

### Example 2: Input Validation

```javascript
class Validator {
    static validateEmail(email) {
        if (!email) {
            throw new ValidationError('Email is required');
        }

        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(email)) {
            throw new ValidationError('Invalid email format');
        }

        return true;
    }

    static validatePassword(password) {
        if (!password) {
            throw new ValidationError('Password is required');
        }

        if (password.length < 8) {
            throw new ValidationError('Password must be at least 8 characters');
        }

        if (!/[A-Z]/.test(password)) {
            throw new ValidationError('Password must contain uppercase letter');
        }

        if (!/[0-9]/.test(password)) {
            throw new ValidationError('Password must contain number');
        }

        return true;
    }
}

// Usage
function registerUser(email, password) {
    try {
        Validator.validateEmail(email);
        Validator.validatePassword(password);

        // Proceed with registration
        console.log('Validation passed');

    } catch (error) {
        if (error instanceof ValidationError) {
            console.error('Validation error:', error.message);
            return { success: false, error: error.message };
        }
        throw error;
    }
}
```

### Example 3: Graceful Degradation

```javascript
class DataService {
    async getData() {
        try {
            // Try primary source
            return await this.fetchFromAPI();
        } catch (apiError) {
            console.warn('API failed, trying cache:', apiError.message);

            try {
                // Fallback to cache
                return await this.fetchFromCache();
            } catch (cacheError) {
                console.warn('Cache failed, using default:', cacheError.message);

                // Final fallback
                return this.getDefaultData();
            }
        }
    }

    async fetchFromAPI() {
        const response = await fetch('/api/data');
        if (!response.ok) throw new Error('API error');
        return response.json();
    }

    async fetchFromCache() {
        const cached = localStorage.getItem('data');
        if (!cached) throw new Error('No cache');
        return JSON.parse(cached);
    }

    getDefaultData() {
        return { message: 'Default data' };
    }
}

const service = new DataService();
const data = await service.getData();
console.log(data);
```

## 🚨 Common Pitfalls

### 1. Swallowing Errors

```javascript
// Bad: Silent failure
try {
    riskyOperation();
} catch (error) {
    // Empty catch - error is lost!
}

// Good: Log or handle
try {
    riskyOperation();
} catch (error) {
    console.error('Error:', error);
    // Or send to logging service
}
```

### 2. Not Re-throwing When Needed

```javascript
// Bad: Error not propagated
async function processData() {
    try {
        const data = await fetchData();
        return data;
    } catch (error) {
        console.error(error);
        // Caller thinks everything is fine!
    }
}

// Good: Re-throw after logging
async function processDataCorrect() {
    try {
        const data = await fetchData();
        return data;
    } catch (error) {
        console.error('Error in processData:', error);
        throw error; // Let caller handle
    }
}
```

### 3. Forgetting Async Error Handling

```javascript
// Bad: Unhandled promise rejection
async function badAsync() {
    throw new Error('Oops');
}

badAsync(); // Unhandled rejection!

// Good: Handle it
badAsync().catch(error => console.error(error));

// Or with try/catch
(async () => {
    try {
        await badAsync();
    } catch (error) {
        console.error(error);
    }
})();
```

## 🎓 Best Practices

1. **Be specific with error types** (use custom errors)
2. **Always handle promise rejections**
3. **Log errors with context** (timestamp, user, action)
4. **Don't swallow errors silently**
5. **Validate input early** (fail fast)
6. **Provide helpful error messages**
7. **Use error boundaries in React**
8. **Implement retry logic for transient failures**
9. **Clean up resources in finally blocks**
10. **Monitor and log errors in production**

## 🔗 Related Topics

- [Promises & Async/Await](./06-promises-async.md)
- [Functions & Scope](./02-functions-scope.md)

---

[← Back to JavaScript](./README.md)
