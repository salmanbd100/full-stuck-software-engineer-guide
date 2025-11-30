# Promises & Async/Await

## Concept

**Promises** represent the eventual completion (or failure) of an asynchronous operation and its resulting value. **Async/await** is syntactic sugar built on top of promises, making asynchronous code look and behave more like synchronous code.

### Key Points
- Promises have three states: pending, fulfilled, rejected
- Async/await makes asynchronous code easier to read and write
- Error handling uses `.catch()` for promises, `try/catch` for async/await
- Promises are eager (execute immediately), not lazy
- Async functions always return a promise

---

## Example 1: Promise Basics

```javascript
// Creating a promise
const myPromise = new Promise((resolve, reject) => {
    const success = true;

    setTimeout(() => {
        if (success) {
            resolve('Operation successful!');
        } else {
            reject('Operation failed!');
        }
    }, 1000);
});

// Consuming the promise
myPromise
    .then(result => {
        console.log(result); // "Operation successful!"
        return 'Next step';
    })
    .then(result => {
        console.log(result); // "Next step"
    })
    .catch(error => {
        console.error(error);
    })
    .finally(() => {
        console.log('Cleanup or final operations');
    });
```

### Promise States:
1. **Pending**: Initial state, neither fulfilled nor rejected
2. **Fulfilled**: Operation completed successfully
3. **Rejected**: Operation failed

---

## Example 2: Async/Await Syntax

```javascript
// Function returns a promise
function fetchUserData(userId) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (userId) {
                resolve({ id: userId, name: 'John Doe', email: 'john@example.com' });
            } else {
                reject('User ID is required');
            }
        }, 1000);
    });
}

// Using async/await
async function getUserInfo(userId) {
    try {
        console.log('Fetching user...');
        const user = await fetchUserData(userId);
        console.log('User:', user);
        return user;
    } catch (error) {
        console.error('Error:', error);
        throw error;
    } finally {
        console.log('Fetch attempt completed');
    }
}

// Calling async function
getUserInfo(123)
    .then(user => console.log('Got user:', user.name))
    .catch(error => console.error('Failed:', error));

// Key points:
// - async keyword makes function return a promise
// - await pauses execution until promise resolves
// - Can use try/catch for error handling
// - finally block runs regardless of success/failure
```

---

## Example 3: Multiple Promises

```javascript
// Simulated API calls
function fetchUser() {
    return new Promise(resolve => {
        setTimeout(() => resolve({ name: 'Alice', id: 1 }), 1000);
    });
}

function fetchPosts(userId) {
    return new Promise(resolve => {
        setTimeout(() => resolve(['Post 1', 'Post 2', 'Post 3']), 800);
    });
}

function fetchComments(postId) {
    return new Promise(resolve => {
        setTimeout(() => resolve(['Comment 1', 'Comment 2']), 600);
    });
}

// Method 1: Promise.all (parallel execution, all must succeed)
async function loadAllData() {
    try {
        const [user, posts, comments] = await Promise.all([
            fetchUser(),
            fetchPosts(1),
            fetchComments(1)
        ]);

        console.log('User:', user);
        console.log('Posts:', posts);
        console.log('Comments:', comments);
    } catch (error) {
        console.error('One or more promises failed:', error);
    }
}

// Method 2: Promise.allSettled (all complete, regardless of success/failure)
async function loadAllDataSafe() {
    const results = await Promise.allSettled([
        fetchUser(),
        fetchPosts(1),
        fetchComments(1)
    ]);

    results.forEach((result, index) => {
        if (result.status === 'fulfilled') {
            console.log(`Promise ${index} succeeded:`, result.value);
        } else {
            console.log(`Promise ${index} failed:`, result.reason);
        }
    });
}

// Method 3: Promise.race (first to complete wins)
async function loadFirstAvailable() {
    const result = await Promise.race([
        fetchUser(),
        fetchPosts(1),
        fetchComments(1)
    ]);

    console.log('First result:', result); // Comments (fastest)
}

// Method 4: Sequential execution (when order matters)
async function loadSequentially() {
    const user = await fetchUser();
    console.log('Got user:', user);

    const posts = await fetchPosts(user.id);
    console.log('Got posts:', posts);

    const comments = await fetchComments(posts[0]);
    console.log('Got comments:', comments);
}
```

---

## Common Pitfalls

### Pitfall 1: Forgetting to Return Promise

```javascript
// WRONG
function getData() {
    fetch('https://api.example.com/data')
        .then(response => response.json());
    // Doesn't return the promise!
}

const result = getData(); // undefined

// CORRECT
function getData() {
    return fetch('https://api.example.com/data')
        .then(response => response.json());
}

getData().then(data => console.log(data));
```

### Pitfall 2: Not Handling Errors

```javascript
// WRONG - unhandled promise rejection
fetch('https://api.example.com/data')
    .then(response => response.json())
    .then(data => console.log(data));
// If request fails, error is not caught

// CORRECT
fetch('https://api.example.com/data')
    .then(response => {
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        return response.json();
    })
    .then(data => console.log(data))
    .catch(error => console.error('Fetch error:', error));

// BETTER - with async/await
async function fetchData() {
    try {
        const response = await fetch('https://api.example.com/data');
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Fetch error:', error);
        throw error; // Re-throw if caller should handle it
    }
}
```

### Pitfall 3: Sequential Instead of Parallel

```javascript
// WRONG - Sequential (slow)
async function loadData() {
    const user = await fetchUser();      // Waits 1s
    const posts = await fetchPosts();    // Then waits 1s
    const comments = await fetchComments(); // Then waits 1s
    // Total: ~3 seconds
    return { user, posts, comments };
}

// CORRECT - Parallel (fast)
async function loadDataFast() {
    const [user, posts, comments] = await Promise.all([
        fetchUser(),      // All start together
        fetchPosts(),     //
        fetchComments()   //
    ]);
    // Total: ~1 second (time of slowest)
    return { user, posts, comments };
}
```

### Pitfall 4: Mixing Promises and Async/Await

```javascript
// CONFUSING - Mixing styles
async function mixedStyle() {
    const user = await fetchUser();
    return fetchPosts(user.id)
        .then(posts => {
            return posts.map(post => post.title);
        }); // Unnecessary .then(), could use await
}

// BETTER - Consistent style
async function consistentStyle() {
    const user = await fetchUser();
    const posts = await fetchPosts(user.id);
    return posts.map(post => post.title);
}
```

---

## Best Practices

### 1. Always Handle Errors

```javascript
// Good error handling pattern
async function robustFetch(url) {
    try {
        const response = await fetch(url);

        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }

        const data = await response.json();
        return { success: true, data };

    } catch (error) {
        console.error('Fetch failed:', error);
        return { success: false, error: error.message };
    }
}

// Usage
const result = await robustFetch('https://api.example.com/data');
if (result.success) {
    console.log(result.data);
} else {
    console.error(result.error);
}
```

### 2. Use Promise.all for Independent Operations

```javascript
// Good: Parallel execution
async function loadDashboard() {
    const start = Date.now();

    const [userData, notifications, analytics] = await Promise.all([
        fetchUserData(),
        fetchNotifications(),
        fetchAnalytics()
    ]);

    console.log(`Loaded in ${Date.now() - start}ms`);

    return { userData, notifications, analytics };
}
```

### 3. Retry Logic for Failed Requests

```javascript
async function fetchWithRetry(url, options = {}, retries = 3) {
    for (let i = 0; i < retries; i++) {
        try {
            const response = await fetch(url, options);
            if (response.ok) {
                return await response.json();
            }

            // If not last retry, wait before retrying
            if (i < retries - 1) {
                await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
            }
        } catch (error) {
            if (i === retries - 1) {
                throw error;
            }
            console.log(`Retry ${i + 1}/${retries}...`);
        }
    }

    throw new Error(`Failed after ${retries} retries`);
}

// Usage
try {
    const data = await fetchWithRetry('https://api.example.com/data');
    console.log(data);
} catch (error) {
    console.error('All retries failed:', error);
}
```

---

## Real-world Scenarios

### Scenario 1: Fetch with Timeout

```javascript
function fetchWithTimeout(url, timeout = 5000) {
    return Promise.race([
        fetch(url),
        new Promise((_, reject) =>
            setTimeout(() => reject(new Error('Request timeout')), timeout)
        )
    ]);
}

// Usage
async function loadData() {
    try {
        const response = await fetchWithTimeout('https://api.example.com/data', 3000);
        const data = await response.json();
        return data;
    } catch (error) {
        if (error.message === 'Request timeout') {
            console.error('Request took too long');
        } else {
            console.error('Request failed:', error);
        }
    }
}
```

### Scenario 2: Batching Requests

```javascript
async function batchFetch(urls, batchSize = 3) {
    const results = [];

    // Process URLs in batches
    for (let i = 0; i < urls.length; i += batchSize) {
        const batch = urls.slice(i, i + batchSize);
        const batchResults = await Promise.all(
            batch.map(url => fetch(url).then(r => r.json()))
        );
        results.push(...batchResults);

        console.log(`Processed batch ${Math.floor(i / batchSize) + 1}`);
    }

    return results;
}

// Usage: Fetch 10 URLs, 3 at a time
const urls = Array.from({ length: 10 }, (_, i) =>
    `https://api.example.com/item/${i}`
);

const data = await batchFetch(urls, 3);
```

### Scenario 3: Promise-based Event Emitter

```javascript
class AsyncEventEmitter {
    constructor() {
        this.listeners = {};
    }

    on(event, callback) {
        if (!this.listeners[event]) {
            this.listeners[event] = [];
        }
        this.listeners[event].push(callback);
    }

    async emit(event, data) {
        if (!this.listeners[event]) return;

        const promises = this.listeners[event].map(callback =>
            Promise.resolve(callback(data))
        );

        return await Promise.all(promises);
    }
}

// Usage
const emitter = new AsyncEventEmitter();

emitter.on('data', async (data) => {
    await new Promise(resolve => setTimeout(resolve, 100));
    console.log('Handler 1:', data);
});

emitter.on('data', async (data) => {
    await new Promise(resolve => setTimeout(resolve, 50));
    console.log('Handler 2:', data);
});

await emitter.emit('data', { message: 'Hello' });
console.log('All handlers completed');
```

---

## External Resources

- [MDN: Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [MDN: async/await](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Async_await)
- [JavaScript.info: Promises](https://javascript.info/promise-basics)
- [JavaScript.info: Async/await](https://javascript.info/async-await)

---

[← Back to JavaScript](./README.md) | [Next: Event Loop →](./07-event-loop.md)
