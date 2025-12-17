# IndexedDB

## Overview

**IndexedDB** is a low-level API for client-side storage of significant amounts of structured data, including files and blobs. It's a NoSQL database built into modern browsers that allows you to store large amounts of data with high-performance indexed lookups. Unlike localStorage (5-10MB limit), IndexedDB can store hundreds of megabytes or even gigabytes of data, making it ideal for offline-first applications, caching strategies, and complex data management.

### Key Characteristics
- **Asynchronous API**: Non-blocking operations that don't freeze the UI
- **Transactional Database**: All operations happen within transactions (ACID properties)
- **Object-Oriented**: Stores JavaScript objects directly (with some limitations)
- **Indexed**: Fast lookups using indexes on object properties
- **Large Storage**: Can store hundreds of MB to GB of data
- **Same-Origin Policy**: Each origin has its own set of databases
- **Persistent**: Data persists until explicitly deleted by user or application

---

## Table of Contents

1. [Key Concepts](#key-concepts)
2. [Creating and Opening Databases](#creating-and-opening-databases)
3. [Object Stores and Schemas](#object-stores-and-schemas)
4. [CRUD Operations](#crud-operations)
5. [Transactions](#transactions)
6. [Indexes and Queries](#indexes-and-queries)
7. [Cursors for Iteration](#cursors-for-iteration)
8. [Using idb Library](#using-idb-library)
9. [Performance Optimization](#performance-optimization)
10. [Version Management and Migrations](#version-management-and-migrations)
11. [IndexedDB vs Other Storage](#indexeddb-vs-other-storage)
12. [Interview Questions](#interview-questions)
13. [Summary](#summary)
14. [External Resources](#external-resources)

---

## Key Concepts

### 1. Database
A container for object stores. Each origin can have multiple databases.

### 2. Object Store
Similar to a table in SQL databases. Stores JavaScript objects with a key.

### 3. Index
A specialized object store for fast lookups on specific properties.

### 4. Transaction
A wrapper around operations ensuring data integrity. Operations must happen within transactions.

### 5. Cursor
An iterator for traversing multiple records in an object store or index.

### 6. Key
A unique identifier for each record. Can be auto-incremented or manually specified.

### 7. Key Path
The property of an object to be used as the key (e.g., "id", "email").

### 8. Version
Each database has a version number. Schema changes require version upgrades.

---

## Creating and Opening Databases

### Opening a Database (Vanilla API)

```javascript
// Open (or create) a database
const request = indexedDB.open('MyDatabase', 1); // name, version

// Success callback
request.onsuccess = (event) => {
    const db = event.target.result;
    console.log('Database opened successfully', db);
};

// Error callback
request.onerror = (event) => {
    console.error('Database error:', event.target.error);
};

// Upgrade callback (runs when version changes)
request.onupgradeneeded = (event) => {
    const db = event.target.result;
    console.log('Database upgrade needed', db);

    // Create object stores here
    if (!db.objectStoreNames.contains('users')) {
        const objectStore = db.createObjectStore('users', {
            keyPath: 'id',
            autoIncrement: true
        });

        // Create indexes
        objectStore.createIndex('email', 'email', { unique: true });
        objectStore.createIndex('name', 'name', { unique: false });
    }
};

// Blocked callback (another tab has the database open)
request.onblocked = (event) => {
    console.warn('Database blocked. Please close other tabs.');
};
```

### Complete Database Setup Example

```javascript
class DatabaseManager {
    constructor(dbName, version) {
        this.dbName = dbName;
        this.version = version;
        this.db = null;
    }

    open() {
        return new Promise((resolve, reject) => {
            const request = indexedDB.open(this.dbName, this.version);

            request.onsuccess = (event) => {
                this.db = event.target.result;

                // Handle version change in other tabs
                this.db.onversionchange = () => {
                    this.db.close();
                    alert('Database is outdated. Please reload the page.');
                };

                resolve(this.db);
            };

            request.onerror = (event) => {
                reject(new Error(`Database error: ${event.target.error}`));
            };

            request.onupgradeneeded = (event) => {
                const db = event.target.result;
                this.handleUpgrade(db, event.oldVersion, event.newVersion);
            };

            request.onblocked = () => {
                console.warn('Database opening blocked by another tab');
            };
        });
    }

    handleUpgrade(db, oldVersion, newVersion) {
        console.log(`Upgrading from version ${oldVersion} to ${newVersion}`);

        // Version 1: Initial setup
        if (oldVersion < 1) {
            // Users store
            const usersStore = db.createObjectStore('users', {
                keyPath: 'id',
                autoIncrement: true
            });
            usersStore.createIndex('email', 'email', { unique: true });
            usersStore.createIndex('name', 'name', { unique: false });

            // Posts store
            const postsStore = db.createObjectStore('posts', {
                keyPath: 'id',
                autoIncrement: true
            });
            postsStore.createIndex('userId', 'userId', { unique: false });
            postsStore.createIndex('createdAt', 'createdAt', { unique: false });
        }

        // Version 2: Add new indexes
        if (oldVersion < 2) {
            const transaction = event.target.transaction;
            const usersStore = transaction.objectStore('users');
            usersStore.createIndex('age', 'age', { unique: false });
        }
    }

    close() {
        if (this.db) {
            this.db.close();
            this.db = null;
        }
    }

    deleteDatabase() {
        this.close();
        return new Promise((resolve, reject) => {
            const request = indexedDB.deleteDatabase(this.dbName);
            request.onsuccess = () => resolve();
            request.onerror = (event) => reject(event.target.error);
        });
    }
}

// Usage
const dbManager = new DatabaseManager('MyApp', 1);
dbManager.open()
    .then(db => {
        console.log('Database ready', db);
    })
    .catch(error => {
        console.error('Failed to open database', error);
    });
```

---

## Object Stores and Schemas

### Creating Object Stores

```javascript
// In onupgradeneeded callback
request.onupgradeneeded = (event) => {
    const db = event.target.result;

    // Auto-incrementing key
    const store1 = db.createObjectStore('products', {
        autoIncrement: true
    });

    // Using a key path (object property)
    const store2 = db.createObjectStore('users', {
        keyPath: 'id'
    });

    // Using a compound key path
    const store3 = db.createObjectStore('orders', {
        keyPath: ['userId', 'orderId']
    });

    // Out-of-line key (specify key separately when adding)
    const store4 = db.createObjectStore('logs', {
        autoIncrement: false
    });
};
```

### Key Types and Strategies

```javascript
// 1. Auto-increment (database generates keys)
const autoStore = db.createObjectStore('items', { autoIncrement: true });

// Adding items (no key needed)
const transaction = db.transaction(['items'], 'readwrite');
const store = transaction.objectStore('items');
store.add({ name: 'Item 1', price: 10 }); // Key will be 1
store.add({ name: 'Item 2', price: 20 }); // Key will be 2

// 2. Key path (use object property as key)
const keyPathStore = db.createObjectStore('users', { keyPath: 'email' });

// Adding items (must have email property)
store.add({ email: 'user@example.com', name: 'John' }); // Key is 'user@example.com'

// 3. Out-of-line key (specify key separately)
const outOfLineStore = db.createObjectStore('logs');

// Adding items (specify key explicitly)
store.add({ message: 'Error occurred', timestamp: Date.now() }, 'log-123');

// 4. Compound key path
const compoundStore = db.createObjectStore('scores', {
    keyPath: ['gameId', 'userId']
});

// Adding items (must have both properties)
store.add({ gameId: 'game-1', userId: 'user-123', score: 9500 });
```

### Schema Design Best Practices

```javascript
// Good schema design example
request.onupgradeneeded = (event) => {
    const db = event.target.result;

    // Users store with proper indexes
    const usersStore = db.createObjectStore('users', {
        keyPath: 'id',
        autoIncrement: true
    });

    // Create indexes for common queries
    usersStore.createIndex('email', 'email', { unique: true });
    usersStore.createIndex('username', 'username', { unique: true });
    usersStore.createIndex('createdAt', 'createdAt', { unique: false });
    usersStore.createIndex('role', 'role', { unique: false });

    // Compound index for complex queries
    usersStore.createIndex('role_createdAt', ['role', 'createdAt'], {
        unique: false
    });

    // Posts store with foreign key simulation
    const postsStore = db.createObjectStore('posts', {
        keyPath: 'id',
        autoIncrement: true
    });

    postsStore.createIndex('authorId', 'authorId', { unique: false });
    postsStore.createIndex('category', 'category', { unique: false });
    postsStore.createIndex('publishedAt', 'publishedAt', { unique: false });

    // Compound index for filtering by author and date
    postsStore.createIndex('authorId_publishedAt', ['authorId', 'publishedAt'], {
        unique: false
    });
};
```

---

## CRUD Operations

### Create (Add/Put)

```javascript
// Add vs Put:
// - add() fails if key already exists
// - put() overwrites existing records

async function addUser(db, user) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');
        const request = store.add(user);

        request.onsuccess = () => {
            resolve(request.result); // Returns the key
        };

        request.onerror = () => {
            reject(request.error);
        };

        transaction.oncomplete = () => {
            console.log('Transaction completed');
        };

        transaction.onerror = () => {
            console.error('Transaction failed');
        };
    });
}

async function updateUser(db, user) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');
        const request = store.put(user); // Overwrites if exists

        request.onsuccess = () => resolve(request.result);
        request.onerror = () => reject(request.error);
    });
}

// Usage
const newUser = {
    email: 'john@example.com',
    name: 'John Doe',
    age: 30,
    role: 'admin',
    createdAt: new Date()
};

addUser(db, newUser)
    .then(key => console.log('User added with key:', key))
    .catch(error => console.error('Error adding user:', error));
```

### Read (Get/GetAll)

```javascript
// Get single record by key
async function getUser(db, key) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        const request = store.get(key);

        request.onsuccess = () => {
            resolve(request.result); // undefined if not found
        };

        request.onerror = () => {
            reject(request.error);
        };
    });
}

// Get all records
async function getAllUsers(db) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        const request = store.getAll();

        request.onsuccess = () => {
            resolve(request.result); // Array of all records
        };

        request.onerror = () => {
            reject(request.error);
        };
    });
}

// Get with limit
async function getRecentUsers(db, limit = 10) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        const request = store.getAll(null, limit); // null = no filter, limit = max count

        request.onsuccess = () => resolve(request.result);
        request.onerror = () => reject(request.error);
    });
}

// Get by index
async function getUserByEmail(db, email) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        const index = store.index('email');
        const request = index.get(email);

        request.onsuccess = () => resolve(request.result);
        request.onerror = () => reject(request.error);
    });
}

// Usage
getUser(db, 1).then(user => console.log(user));
getUserByEmail(db, 'john@example.com').then(user => console.log(user));
getAllUsers(db).then(users => console.log('All users:', users));
```

### Update

```javascript
// Update using get + put pattern
async function updateUserEmail(db, userId, newEmail) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');

        // First, get the existing record
        const getRequest = store.get(userId);

        getRequest.onsuccess = () => {
            const user = getRequest.result;

            if (!user) {
                reject(new Error('User not found'));
                return;
            }

            // Update the email
            user.email = newEmail;
            user.updatedAt = new Date();

            // Put it back
            const putRequest = store.put(user);

            putRequest.onsuccess = () => {
                resolve(user);
            };

            putRequest.onerror = () => {
                reject(putRequest.error);
            };
        };

        getRequest.onerror = () => {
            reject(getRequest.error);
        };
    });
}

// Partial update helper
async function partialUpdateUser(db, userId, updates) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');

        const getRequest = store.get(userId);

        getRequest.onsuccess = () => {
            const user = getRequest.result;

            if (!user) {
                reject(new Error('User not found'));
                return;
            }

            // Merge updates
            const updatedUser = {
                ...user,
                ...updates,
                updatedAt: new Date()
            };

            const putRequest = store.put(updatedUser);

            putRequest.onsuccess = () => resolve(updatedUser);
            putRequest.onerror = () => reject(putRequest.error);
        };

        getRequest.onerror = () => reject(getRequest.error);
    });
}

// Usage
partialUpdateUser(db, 1, { age: 31, city: 'New York' })
    .then(user => console.log('Updated:', user))
    .catch(error => console.error('Update failed:', error));
```

### Delete

```javascript
// Delete single record
async function deleteUser(db, key) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');
        const request = store.delete(key);

        request.onsuccess = () => {
            resolve(); // Returns undefined
        };

        request.onerror = () => {
            reject(request.error);
        };
    });
}

// Delete all records in store
async function clearUsers(db) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');
        const request = store.clear();

        request.onsuccess = () => resolve();
        request.onerror = () => reject(request.error);
    });
}

// Delete with condition
async function deleteInactiveUsers(db, daysInactive = 90) {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - daysInactive);

    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');
        const index = store.index('lastLoginAt');

        const range = IDBKeyRange.upperBound(cutoffDate);
        const request = index.openCursor(range);

        const deletedKeys = [];

        request.onsuccess = (event) => {
            const cursor = event.target.result;

            if (cursor) {
                deletedKeys.push(cursor.primaryKey);
                cursor.delete(); // Delete current record
                cursor.continue(); // Move to next
            } else {
                resolve(deletedKeys); // All done
            }
        };

        request.onerror = () => reject(request.error);
    });
}

// Usage
deleteUser(db, 1)
    .then(() => console.log('User deleted'))
    .catch(error => console.error('Delete failed:', error));
```

---

## Transactions

### Transaction Basics

```javascript
// Transaction modes:
// - 'readonly': Default mode, allows reading only
// - 'readwrite': Allows reading and writing
// - 'versionchange': Only available in onupgradeneeded

// Single store transaction
const transaction = db.transaction('users', 'readonly');
const store = transaction.objectStore('users');

// Multiple stores transaction
const transaction2 = db.transaction(['users', 'posts'], 'readwrite');
const usersStore = transaction2.objectStore('users');
const postsStore = transaction2.objectStore('posts');

// Transaction lifecycle events
transaction.oncomplete = () => {
    console.log('Transaction completed successfully');
};

transaction.onerror = (event) => {
    console.error('Transaction error:', event.target.error);
};

transaction.onabort = (event) => {
    console.warn('Transaction aborted:', event.target.error);
};
```

### Atomic Operations with Transactions

```javascript
// Multiple operations in a single transaction (atomic)
async function transferPostOwnership(db, postId, fromUserId, toUserId) {
    return new Promise((resolve, reject) => {
        // Both stores in same transaction ensures atomicity
        const transaction = db.transaction(['posts', 'users'], 'readwrite');
        const postsStore = transaction.objectStore('posts');
        const usersStore = transaction.objectStore('users');

        let post, fromUser, toUser;

        // Get the post
        const getPostRequest = postsStore.get(postId);

        getPostRequest.onsuccess = () => {
            post = getPostRequest.result;

            if (!post || post.authorId !== fromUserId) {
                transaction.abort();
                reject(new Error('Post not found or unauthorized'));
                return;
            }

            // Get both users
            const getFromUserRequest = usersStore.get(fromUserId);

            getFromUserRequest.onsuccess = () => {
                fromUser = getFromUserRequest.result;

                const getToUserRequest = usersStore.get(toUserId);

                getToUserRequest.onsuccess = () => {
                    toUser = getToUserRequest.result;

                    if (!toUser) {
                        transaction.abort();
                        reject(new Error('Target user not found'));
                        return;
                    }

                    // Update post ownership
                    post.authorId = toUserId;
                    post.updatedAt = new Date();
                    postsStore.put(post);

                    // Update user post counts
                    fromUser.postCount = (fromUser.postCount || 0) - 1;
                    toUser.postCount = (toUser.postCount || 0) + 1;

                    usersStore.put(fromUser);
                    usersStore.put(toUser);
                };
            };
        };

        transaction.oncomplete = () => {
            resolve({ post, fromUser, toUser });
        };

        transaction.onerror = () => {
            reject(transaction.error);
        };

        transaction.onabort = () => {
            reject(new Error('Transaction aborted'));
        };
    });
}

// Usage
transferPostOwnership(db, 'post-123', 'user-1', 'user-2')
    .then(result => console.log('Ownership transferred:', result))
    .catch(error => console.error('Transfer failed:', error));
```

### Transaction Durability

```javascript
// Transactions commit automatically when all requests complete
// No explicit commit needed

async function batchAddUsers(db, users) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');

        const addedKeys = [];

        // Add all users
        users.forEach(user => {
            const request = store.add(user);

            request.onsuccess = () => {
                addedKeys.push(request.result);
            };
        });

        // Transaction commits when all operations complete
        transaction.oncomplete = () => {
            resolve(addedKeys);
        };

        // If any operation fails, entire transaction rolls back
        transaction.onerror = () => {
            reject(transaction.error);
        };
    });
}

// Explicit abort
async function conditionalUpdate(db, userId, updates) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');

        const getRequest = store.get(userId);

        getRequest.onsuccess = () => {
            const user = getRequest.result;

            // Business logic check
            if (user.role === 'admin' && updates.role !== 'admin') {
                // Abort transaction - changes won't be saved
                transaction.abort();
                reject(new Error('Cannot change admin role'));
                return;
            }

            const updatedUser = { ...user, ...updates };
            store.put(updatedUser);
        };

        transaction.oncomplete = () => resolve();
        transaction.onabort = () => reject(new Error('Transaction aborted'));
        transaction.onerror = () => reject(transaction.error);
    });
}
```

---

## Indexes and Queries

### Creating and Using Indexes

```javascript
// Create indexes during database upgrade
request.onupgradeneeded = (event) => {
    const db = event.target.result;
    const store = db.createObjectStore('products', {
        keyPath: 'id',
        autoIncrement: true
    });

    // Simple index
    store.createIndex('name', 'name', { unique: false });

    // Unique index
    store.createIndex('sku', 'sku', { unique: true });

    // Index on nested property
    store.createIndex('categoryName', 'category.name', { unique: false });

    // Compound index (multiple properties)
    store.createIndex('category_price', ['category.name', 'price'], {
        unique: false
    });

    // Multi-entry index (for arrays)
    store.createIndex('tags', 'tags', {
        unique: false,
        multiEntry: true // Each array element becomes a separate index entry
    });
};
```

### Query Patterns with Indexes

```javascript
// 1. Exact match query
async function getProductBySku(db, sku) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['products'], 'readonly');
        const store = transaction.objectStore('products');
        const index = store.index('sku');
        const request = index.get(sku);

        request.onsuccess = () => resolve(request.result);
        request.onerror = () => reject(request.error);
    });
}

// 2. Range queries
async function getProductsByPriceRange(db, minPrice, maxPrice) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['products'], 'readonly');
        const store = transaction.objectStore('products');
        const index = store.index('price');

        // Create a range
        const range = IDBKeyRange.bound(minPrice, maxPrice);
        const request = index.getAll(range);

        request.onsuccess = () => resolve(request.result);
        request.onerror = () => reject(request.error);
    });
}

// 3. Prefix search (for strings)
async function searchProductsByNamePrefix(db, prefix) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['products'], 'readonly');
        const store = transaction.objectStore('products');
        const index = store.index('name');

        // Range that starts with prefix
        const range = IDBKeyRange.bound(
            prefix,
            prefix + '\uffff', // Unicode max character
            false, // lower bound inclusive
            false  // upper bound inclusive
        );

        const request = index.getAll(range);

        request.onsuccess = () => resolve(request.result);
        request.onerror = () => reject(request.error);
    });
}

// 4. Multi-entry index query (tags)
async function getProductsByTag(db, tag) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['products'], 'readonly');
        const store = transaction.objectStore('products');
        const index = store.index('tags');
        const request = index.getAll(tag); // Finds all products with this tag

        request.onsuccess = () => resolve(request.result);
        request.onerror = () => reject(request.error);
    });
}

// 5. Compound index query
async function getProductsByCategoryAndPrice(db, category, maxPrice) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['products'], 'readonly');
        const store = transaction.objectStore('products');
        const index = store.index('category_price');

        // Range with compound key
        const range = IDBKeyRange.bound(
            [category, 0],         // Lower bound: category + min price
            [category, maxPrice],  // Upper bound: category + max price
            false,
            false
        );

        const request = index.getAll(range);

        request.onsuccess = () => resolve(request.result);
        request.onerror = () => reject(request.error);
    });
}
```

### Key Range Patterns

```javascript
// IDBKeyRange static methods for different range types

// 1. Exact match
const exactRange = IDBKeyRange.only(42);

// 2. Lower bound (>= value)
const lowerRange = IDBKeyRange.lowerBound(100); // >= 100
const lowerRangeExclusive = IDBKeyRange.lowerBound(100, true); // > 100

// 3. Upper bound (<= value)
const upperRange = IDBKeyRange.upperBound(1000); // <= 1000
const upperRangeExclusive = IDBKeyRange.upperBound(1000, true); // < 1000

// 4. Bounded range
const boundedRange = IDBKeyRange.bound(100, 1000); // >= 100 AND <= 1000
const boundedRangeExclusive = IDBKeyRange.bound(100, 1000, true, true); // > 100 AND < 1000

// 5. Date ranges
const lastWeek = new Date();
lastWeek.setDate(lastWeek.getDate() - 7);
const dateRange = IDBKeyRange.lowerBound(lastWeek); // All records after last week

// 6. Complex queries with cursors
async function getProductsByComplexCriteria(db, category, minPrice, maxPrice) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['products'], 'readonly');
        const store = transaction.objectStore('products');
        const index = store.index('price');

        const range = IDBKeyRange.bound(minPrice, maxPrice);
        const request = index.openCursor(range);

        const results = [];

        request.onsuccess = (event) => {
            const cursor = event.target.result;

            if (cursor) {
                const product = cursor.value;

                // Additional filtering (can't be done with index alone)
                if (product.category.name === category && product.inStock) {
                    results.push(product);
                }

                cursor.continue();
            } else {
                resolve(results); // Done
            }
        };

        request.onerror = () => reject(request.error);
    });
}
```

---

## Cursors for Iteration

### Basic Cursor Usage

```javascript
// Forward cursor
async function iterateAllUsers(db) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        const request = store.openCursor(); // Forward direction by default

        const users = [];

        request.onsuccess = (event) => {
            const cursor = event.target.result;

            if (cursor) {
                // cursor.key - the key
                // cursor.primaryKey - the primary key
                // cursor.value - the actual object

                users.push(cursor.value);
                cursor.continue(); // Move to next record
            } else {
                // No more records
                resolve(users);
            }
        };

        request.onerror = () => reject(request.error);
    });
}

// Reverse cursor
async function getLatestUsers(db, limit = 10) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        const index = store.index('createdAt');

        // Open cursor in reverse order
        const request = index.openCursor(null, 'prev');

        const users = [];

        request.onsuccess = (event) => {
            const cursor = event.target.result;

            if (cursor && users.length < limit) {
                users.push(cursor.value);
                cursor.continue();
            } else {
                resolve(users);
            }
        };

        request.onerror = () => reject(request.error);
    });
}
```

### Cursor Directions

```javascript
// Cursor direction options:
// - 'next': Forward, from lowest to highest key
// - 'nextunique': Forward, skip duplicates
// - 'prev': Backward, from highest to lowest key
// - 'prevunique': Backward, skip duplicates

async function demonstrateCursorDirections(db) {
    const transaction = db.transaction(['users'], 'readonly');
    const store = transaction.objectStore('users');
    const index = store.index('role');

    // Get all users by role (forward)
    const forwardCursor = index.openCursor(null, 'next');

    // Get unique roles (forward, skip duplicate keys)
    const uniqueCursor = index.openCursor(null, 'nextunique');

    // Get all users by role (backward)
    const backwardCursor = index.openCursor(null, 'prev');

    // Get unique roles (backward, skip duplicate keys)
    const uniqueBackwardCursor = index.openCursor(null, 'prevunique');
}
```

### Advanced Cursor Operations

```javascript
// Update records with cursor
async function incrementAllUserAges(db) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');
        const request = store.openCursor();

        let count = 0;

        request.onsuccess = (event) => {
            const cursor = event.target.result;

            if (cursor) {
                const user = cursor.value;
                user.age += 1;

                // Update current record
                cursor.update(user);
                count++;

                cursor.continue();
            } else {
                resolve(count);
            }
        };

        request.onerror = () => reject(request.error);
    });
}

// Delete records with cursor
async function deleteOldLogs(db, daysOld = 30) {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - daysOld);

    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['logs'], 'readwrite');
        const store = transaction.objectStore('logs');
        const index = store.index('timestamp');

        const range = IDBKeyRange.upperBound(cutoffDate);
        const request = index.openCursor(range);

        let deletedCount = 0;

        request.onsuccess = (event) => {
            const cursor = event.target.result;

            if (cursor) {
                cursor.delete(); // Delete current record
                deletedCount++;
                cursor.continue();
            } else {
                resolve(deletedCount);
            }
        };

        request.onerror = () => reject(request.error);
    });
}

// Skip records with advance()
async function getPaginatedUsers(db, page = 0, pageSize = 20) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        const request = store.openCursor();

        const offset = page * pageSize;
        let count = 0;
        const users = [];

        request.onsuccess = (event) => {
            const cursor = event.target.result;

            if (cursor) {
                if (count < offset) {
                    // Skip to the starting position
                    count = offset;
                    cursor.advance(offset); // Skip multiple records at once
                } else if (users.length < pageSize) {
                    users.push(cursor.value);
                    count++;
                    cursor.continue();
                } else {
                    resolve(users);
                }
            } else {
                resolve(users);
            }
        };

        request.onerror = () => reject(request.error);
    });
}

// Key cursor (only keys, not values - more efficient)
async function getAllUserIds(db) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        const request = store.openKeyCursor(); // Returns only keys

        const ids = [];

        request.onsuccess = (event) => {
            const cursor = event.target.result;

            if (cursor) {
                ids.push(cursor.key);
                cursor.continue();
            } else {
                resolve(ids);
            }
        };

        request.onerror = () => reject(request.error);
    });
}
```

---

## Using idb Library

### Why idb?

Jake Archibald's `idb` library provides a Promise-based wrapper around IndexedDB, making it much easier to use with async/await syntax.

**Installation:**
```bash
npm install idb
```

### Basic Operations with idb

```javascript
import { openDB } from 'idb';

// Open database with idb
async function initDB() {
    const db = await openDB('MyDatabase', 1, {
        upgrade(db, oldVersion, newVersion, transaction) {
            // Create object stores
            if (!db.objectStoreNames.contains('users')) {
                const usersStore = db.createObjectStore('users', {
                    keyPath: 'id',
                    autoIncrement: true
                });

                usersStore.createIndex('email', 'email', { unique: true });
                usersStore.createIndex('name', 'name');
            }

            if (!db.objectStoreNames.contains('posts')) {
                const postsStore = db.createObjectStore('posts', {
                    keyPath: 'id',
                    autoIncrement: true
                });

                postsStore.createIndex('authorId', 'authorId');
                postsStore.createIndex('createdAt', 'createdAt');
            }
        },
        blocked() {
            console.warn('Database upgrade blocked');
        },
        blocking() {
            console.warn('This connection is blocking a newer version');
        },
        terminated() {
            console.warn('Database connection terminated unexpectedly');
        }
    });

    return db;
}

// CRUD operations with idb (much cleaner!)
class UserRepository {
    constructor(db) {
        this.db = db;
    }

    // Create
    async addUser(user) {
        return await this.db.add('users', user);
    }

    // Read
    async getUser(id) {
        return await this.db.get('users', id);
    }

    async getAllUsers() {
        return await this.db.getAll('users');
    }

    async getUserByEmail(email) {
        return await this.db.getFromIndex('users', 'email', email);
    }

    // Update
    async updateUser(user) {
        return await this.db.put('users', user);
    }

    // Delete
    async deleteUser(id) {
        return await this.db.delete('users', id);
    }

    async clearAll() {
        return await this.db.clear('users');
    }

    // Count
    async getUserCount() {
        return await this.db.count('users');
    }

    // Get all keys
    async getAllUserIds() {
        return await this.db.getAllKeys('users');
    }
}

// Usage
const db = await initDB();
const userRepo = new UserRepository(db);

// Clean async/await syntax
const userId = await userRepo.addUser({
    name: 'John Doe',
    email: 'john@example.com',
    age: 30
});

const user = await userRepo.getUser(userId);
console.log(user);

const allUsers = await userRepo.getAllUsers();
console.log('All users:', allUsers);
```

### Advanced Patterns with idb

```javascript
// Transactions with idb
async function transferPostOwnership(db, postId, fromUserId, toUserId) {
    const tx = db.transaction(['posts', 'users'], 'readwrite');

    try {
        // Get stores from transaction
        const postsStore = tx.objectStore('posts');
        const usersStore = tx.objectStore('users');

        // Get records
        const post = await postsStore.get(postId);
        const fromUser = await usersStore.get(fromUserId);
        const toUser = await usersStore.get(toUserId);

        if (!post || post.authorId !== fromUserId) {
            throw new Error('Post not found or unauthorized');
        }

        if (!toUser) {
            throw new Error('Target user not found');
        }

        // Update records
        post.authorId = toUserId;
        post.updatedAt = new Date();

        fromUser.postCount = (fromUser.postCount || 0) - 1;
        toUser.postCount = (toUser.postCount || 0) + 1;

        await postsStore.put(post);
        await usersStore.put(fromUser);
        await usersStore.put(toUser);

        // Wait for transaction to complete
        await tx.done;

        return { post, fromUser, toUser };
    } catch (error) {
        // Transaction automatically aborts on error
        throw error;
    }
}

// Cursor iteration with idb
async function getActiveUsers(db) {
    const tx = db.transaction('users', 'readonly');
    const store = tx.objectStore('users');
    const index = store.index('lastLoginAt');

    const activeUsers = [];
    const sevenDaysAgo = new Date();
    sevenDaysAgo.setDate(sevenDaysAgo.getDate() - 7);

    // Iterate with cursor
    for await (const cursor of index.iterate(IDBKeyRange.lowerBound(sevenDaysAgo))) {
        activeUsers.push(cursor.value);
    }

    return activeUsers;
}

// Batch operations with idb
async function batchAddUsers(db, users) {
    const tx = db.transaction('users', 'readwrite');
    const store = tx.objectStore('users');

    const promises = users.map(user => store.add(user));

    // Wait for all operations to complete
    await Promise.all(promises);

    // Wait for transaction to complete
    await tx.done;
}

// Query with range
async function getProductsByPriceRange(db, minPrice, maxPrice) {
    const tx = db.transaction('products', 'readonly');
    const index = tx.objectStore('products').index('price');

    const range = IDBKeyRange.bound(minPrice, maxPrice);
    return await index.getAll(range);
}

// Pagination with idb
async function getPaginatedUsers(db, page = 0, pageSize = 20) {
    const tx = db.transaction('users', 'readonly');
    const store = tx.objectStore('users');

    const allKeys = await store.getAllKeys();
    const startIndex = page * pageSize;
    const endIndex = startIndex + pageSize;
    const pageKeys = allKeys.slice(startIndex, endIndex);

    const users = await Promise.all(
        pageKeys.map(key => store.get(key))
    );

    return {
        users,
        totalCount: allKeys.length,
        totalPages: Math.ceil(allKeys.length / pageSize),
        currentPage: page
    };
}

// Search with multiple conditions
async function searchProducts(db, filters) {
    const tx = db.transaction('products', 'readonly');
    const store = tx.objectStore('products');

    let results = [];

    if (filters.category) {
        // Use index for initial filtering
        const index = store.index('category');
        results = await index.getAll(filters.category);
    } else {
        results = await store.getAll();
    }

    // Additional filtering in memory
    return results.filter(product => {
        if (filters.minPrice && product.price < filters.minPrice) return false;
        if (filters.maxPrice && product.price > filters.maxPrice) return false;
        if (filters.inStock && !product.inStock) return false;
        if (filters.search) {
            const searchLower = filters.search.toLowerCase();
            return product.name.toLowerCase().includes(searchLower) ||
                   product.description.toLowerCase().includes(searchLower);
        }
        return true;
    });
}
```

---

## Performance Optimization

### Best Practices

```javascript
// 1. Use indexes for frequent queries
// Bad: Iterate through all records
async function findUserByEmailSlow(db) {
    const users = await db.getAll('users');
    return users.find(u => u.email === 'john@example.com');
}

// Good: Use index
async function findUserByEmailFast(db) {
    return await db.getFromIndex('users', 'email', 'john@example.com');
}

// 2. Batch operations in single transaction
// Bad: Multiple transactions
async function addUsersSlow(db, users) {
    for (const user of users) {
        await db.add('users', user); // New transaction each time!
    }
}

// Good: Single transaction
async function addUsersFast(db, users) {
    const tx = db.transaction('users', 'readwrite');
    await Promise.all(users.map(user => tx.store.add(user)));
    await tx.done;
}

// 3. Use getAll() with limit instead of cursors for simple cases
// Slower: Cursor for simple retrieval
async function getFirstTenUsersSlow(db) {
    const tx = db.transaction('users', 'readonly');
    const users = [];
    let count = 0;

    for await (const cursor of tx.store) {
        if (count++ >= 10) break;
        users.push(cursor.value);
    }

    return users;
}

// Faster: getAll with limit
async function getFirstTenUsersFast(db) {
    return await db.getAll('users', null, 10);
}

// 4. Use compound indexes for multi-field queries
// Instead of filtering in memory:
async function getAdminUsersSlow(db) {
    const users = await db.getAll('users');
    return users.filter(u => u.role === 'admin' && u.isActive);
}

// Use compound index 'role_isActive' created during upgrade:
async function getAdminUsersFast(db) {
    return await db.getAllFromIndex('users', 'role_isActive', ['admin', true]);
}

// 5. Use key-only cursors when you only need keys
// Slower: Full cursor with values
async function countUsersByRole(db) {
    const tx = db.transaction('users', 'readonly');
    const index = tx.objectStore('users').index('role');
    const counts = {};

    for await (const cursor of index) {
        const role = cursor.value.role;
        counts[role] = (counts[role] || 0) + 1;
    }

    return counts;
}

// Faster: Key cursor (doesn't load values)
async function countUsersByRoleFast(db) {
    const tx = db.transaction('users', 'readonly');
    const index = tx.objectStore('users').index('role');
    const counts = {};

    let cursor = await index.openKeyCursor();
    while (cursor) {
        const role = cursor.key;
        counts[role] = (counts[role] || 0) + 1;
        cursor = await cursor.continue();
    }

    return counts;
}
```

### Caching Strategies

```javascript
// In-memory cache with IndexedDB persistence
class CachedUserRepository {
    constructor(db) {
        this.db = db;
        this.cache = new Map();
        this.cacheExpiry = 5 * 60 * 1000; // 5 minutes
    }

    async getUser(id) {
        // Check cache first
        const cached = this.cache.get(id);
        if (cached && Date.now() - cached.timestamp < this.cacheExpiry) {
            return cached.user;
        }

        // Load from IndexedDB
        const user = await this.db.get('users', id);

        if (user) {
            this.cache.set(id, {
                user,
                timestamp: Date.now()
            });
        }

        return user;
    }

    async updateUser(user) {
        await this.db.put('users', user);

        // Invalidate cache
        this.cache.delete(user.id);
    }

    clearCache() {
        this.cache.clear();
    }
}

// Lazy loading with pagination
class PaginatedDataLoader {
    constructor(db, storeName, pageSize = 50) {
        this.db = db;
        this.storeName = storeName;
        this.pageSize = pageSize;
        this.loadedPages = new Set();
        this.data = new Map();
    }

    async loadPage(page) {
        if (this.loadedPages.has(page)) {
            return this.getPageData(page);
        }

        const tx = this.db.transaction(this.storeName, 'readonly');
        const store = tx.objectStore(this.storeName);

        const offset = page * this.pageSize;
        const allKeys = await store.getAllKeys();
        const pageKeys = allKeys.slice(offset, offset + this.pageSize);

        const items = await Promise.all(
            pageKeys.map(key => store.get(key))
        );

        items.forEach((item, index) => {
            const globalIndex = offset + index;
            this.data.set(globalIndex, item);
        });

        this.loadedPages.add(page);

        return items;
    }

    getPageData(page) {
        const offset = page * this.pageSize;
        const result = [];

        for (let i = 0; i < this.pageSize; i++) {
            const item = this.data.get(offset + i);
            if (item) result.push(item);
        }

        return result;
    }

    clearCache() {
        this.data.clear();
        this.loadedPages.clear();
    }
}
```

### Storage Quota Management

```javascript
// Check storage quota
async function checkStorageQuota() {
    if (navigator.storage && navigator.storage.estimate) {
        const estimate = await navigator.storage.estimate();
        const percentUsed = (estimate.usage / estimate.quota) * 100;

        console.log(`Using ${estimate.usage} of ${estimate.quota} bytes (${percentUsed.toFixed(2)}%)`);

        return {
            usage: estimate.usage,
            quota: estimate.quota,
            percentUsed,
            available: estimate.quota - estimate.usage
        };
    }

    return null;
}

// Request persistent storage
async function requestPersistentStorage() {
    if (navigator.storage && navigator.storage.persist) {
        const isPersisted = await navigator.storage.persist();
        console.log(`Persistent storage ${isPersisted ? 'granted' : 'denied'}`);
        return isPersisted;
    }

    return false;
}

// Check if storage is persisted
async function isStoragePersisted() {
    if (navigator.storage && navigator.storage.persisted) {
        return await navigator.storage.persisted();
    }

    return false;
}

// Cleanup old data based on storage pressure
async function cleanupOldData(db) {
    const quota = await checkStorageQuota();

    if (quota && quota.percentUsed > 80) {
        console.log('Storage pressure detected, cleaning up...');

        // Delete old logs
        const thirtyDaysAgo = new Date();
        thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

        const tx = db.transaction('logs', 'readwrite');
        const store = tx.objectStore('logs');
        const index = store.index('timestamp');

        for await (const cursor of index.iterate(IDBKeyRange.upperBound(thirtyDaysAgo))) {
            await cursor.delete();
        }

        await tx.done;

        console.log('Cleanup completed');
    }
}
```

---

## Version Management and Migrations

### Version Upgrade Strategies

```javascript
import { openDB } from 'idb';

async function openDatabase() {
    return await openDB('MyApp', 3, {
        upgrade(db, oldVersion, newVersion, transaction) {
            console.log(`Upgrading from version ${oldVersion} to ${newVersion}`);

            // Version 1: Initial schema
            if (oldVersion < 1) {
                const usersStore = db.createObjectStore('users', {
                    keyPath: 'id',
                    autoIncrement: true
                });

                usersStore.createIndex('email', 'email', { unique: true });
                usersStore.createIndex('createdAt', 'createdAt');
            }

            // Version 2: Add new indexes
            if (oldVersion < 2) {
                const usersStore = transaction.objectStore('users');
                usersStore.createIndex('role', 'role');
                usersStore.createIndex('lastLoginAt', 'lastLoginAt');

                // Add new object store
                const postsStore = db.createObjectStore('posts', {
                    keyPath: 'id',
                    autoIncrement: true
                });

                postsStore.createIndex('authorId', 'authorId');
                postsStore.createIndex('publishedAt', 'publishedAt');
            }

            // Version 3: Data migration
            if (oldVersion < 3) {
                const usersStore = transaction.objectStore('users');

                // Migrate data: Add default role to existing users
                const cursor = await usersStore.openCursor();

                while (cursor) {
                    const user = cursor.value;

                    if (!user.role) {
                        user.role = 'user'; // Default role
                        await cursor.update(user);
                    }

                    cursor = await cursor.continue();
                }
            }
        }
    });
}

// Complex migration with data transformation
async function migrateUserSchema(db, oldVersion) {
    if (oldVersion < 4) {
        const tx = db.transaction(['users'], 'readwrite');
        const store = tx.objectStore('users');

        // Migrate from flat structure to nested
        for await (const cursor of store) {
            const user = cursor.value;

            // Old structure: { id, firstName, lastName, email, phone, city, country }
            // New structure: { id, name: { first, last }, email, contact: { phone }, address: { city, country } }

            const updatedUser = {
                id: user.id,
                name: {
                    first: user.firstName || '',
                    last: user.lastName || ''
                },
                email: user.email,
                contact: {
                    phone: user.phone || ''
                },
                address: {
                    city: user.city || '',
                    country: user.country || ''
                },
                createdAt: user.createdAt,
                updatedAt: new Date()
            };

            await cursor.update(updatedUser);
        }

        await tx.done;
    }
}

// Rename object store (requires recreating)
async function renameObjectStore(db, oldName, newName, oldVersion) {
    if (oldVersion < 5) {
        // Get all data from old store
        const tx = db.transaction([oldName], 'readonly');
        const oldStore = tx.objectStore(oldName);
        const allData = await oldStore.getAll();

        // Create new store
        const newStore = db.createObjectStore(newName, {
            keyPath: oldStore.keyPath,
            autoIncrement: oldStore.autoIncrement
        });

        // Copy indexes
        for (const indexName of oldStore.indexNames) {
            const index = oldStore.index(indexName);
            newStore.createIndex(indexName, index.keyPath, {
                unique: index.unique,
                multiEntry: index.multiEntry
            });
        }

        // Delete old store
        db.deleteObjectStore(oldName);

        // Copy data to new store
        const writeTx = db.transaction([newName], 'readwrite');
        const writeStore = writeTx.objectStore(newName);

        for (const item of allData) {
            await writeStore.add(item);
        }

        await writeTx.done;
    }
}
```

### Backward Compatibility

```javascript
// Handle missing features gracefully
class DatabaseWrapper {
    constructor(dbName, version) {
        this.dbName = dbName;
        this.version = version;
        this.db = null;
    }

    async init() {
        if (!window.indexedDB) {
            throw new Error('IndexedDB not supported');
        }

        try {
            this.db = await openDB(this.dbName, this.version, {
                upgrade: this.handleUpgrade.bind(this)
            });
        } catch (error) {
            console.error('Failed to open database:', error);

            // Fallback: Try opening without version (latest version)
            try {
                this.db = await openDB(this.dbName);
            } catch (fallbackError) {
                throw new Error('Cannot open database');
            }
        }
    }

    handleUpgrade(db, oldVersion, newVersion, transaction) {
        // Safe upgrade with error handling
        try {
            // Version upgrades here
        } catch (error) {
            console.error('Upgrade error:', error);
            // Log error but don't throw (might be recoverable)
        }
    }

    async safeGet(storeName, key) {
        try {
            return await this.db.get(storeName, key);
        } catch (error) {
            console.error(`Error reading from ${storeName}:`, error);
            return null;
        }
    }

    async safeAdd(storeName, value) {
        try {
            return await this.db.add(storeName, value);
        } catch (error) {
            console.error(`Error adding to ${storeName}:`, error);
            return null;
        }
    }
}
```

---

## IndexedDB vs Other Storage

### Comparison Table

| Feature | localStorage | sessionStorage | IndexedDB | Cookies |
|---------|-------------|----------------|-----------|---------|
| **Capacity** | 5-10 MB | 5-10 MB | 100s of MB to GB | 4 KB per cookie |
| **Async** | No (blocking) | No (blocking) | Yes | No |
| **Data Types** | Strings only | Strings only | Objects, Blobs, Files | Strings only |
| **Indexes** | No | No | Yes | No |
| **Transactions** | No | No | Yes | No |
| **Expiration** | Never | Tab close | Never | Configurable |
| **Server Access** | No | No | No | Yes (sent with requests) |
| **Performance** | Fast for small data | Fast for small data | Fast for large data | Slow |

### When to Use Each

```javascript
// 1. localStorage - Simple key-value, small data
// Good for: User preferences, theme settings, simple flags
localStorage.setItem('theme', 'dark');
localStorage.setItem('language', 'en');

// 2. sessionStorage - Temporary data per tab
// Good for: Form drafts, wizard progress, temporary state
sessionStorage.setItem('formDraft', JSON.stringify(formData));

// 3. IndexedDB - Large structured data, offline support
// Good for: Offline data sync, caching API responses, large datasets
const db = await openDB('AppData', 1);
await db.add('products', { id: 1, name: 'Product', price: 100 });

// 4. Cookies - Server communication
// Good for: Authentication tokens, tracking
document.cookie = 'sessionId=abc123; Secure; SameSite=Strict';

// Practical decision tree
function chooseStorageType(requirements) {
    if (requirements.serverAccess) {
        return 'cookies';
    }

    if (requirements.dataSize > 10 * 1024 * 1024) { // > 10 MB
        return 'IndexedDB';
    }

    if (requirements.complexQueries || requirements.transactions) {
        return 'IndexedDB';
    }

    if (requirements.temporary) {
        return 'sessionStorage';
    }

    return 'localStorage';
}
```

### Migration from localStorage to IndexedDB

```javascript
// Migrate simple key-value data from localStorage to IndexedDB
async function migrateFromLocalStorage(db) {
    const tx = db.transaction('settings', 'readwrite');
    const store = tx.objectStore('settings');

    // Migrate all localStorage items
    for (let i = 0; i < localStorage.length; i++) {
        const key = localStorage.key(i);
        const value = localStorage.getItem(key);

        try {
            // Try to parse as JSON
            const parsedValue = JSON.parse(value);
            await store.put({ key, value: parsedValue });
        } catch {
            // Store as string if not JSON
            await store.put({ key, value });
        }
    }

    await tx.done;

    console.log('Migration from localStorage completed');
}

// Hybrid approach: Use both
class HybridStorage {
    constructor(db) {
        this.db = db;
    }

    async set(key, value) {
        // Store in both for redundancy
        localStorage.setItem(key, JSON.stringify(value));
        await this.db.put('cache', { key, value, timestamp: Date.now() });
    }

    async get(key) {
        // Try localStorage first (faster)
        const localValue = localStorage.getItem(key);
        if (localValue) {
            try {
                return JSON.parse(localValue);
            } catch {
                return localValue;
            }
        }

        // Fallback to IndexedDB
        const record = await this.db.get('cache', key);
        return record?.value;
    }
}
```

---

## Interview Questions

### Q1: What is IndexedDB and when would you use it over localStorage?
**Answer:** IndexedDB is a low-level, asynchronous, transactional database API built into browsers for storing large amounts of structured data. Unlike localStorage (5-10MB limit, synchronous), IndexedDB can store hundreds of MB to GB of data asynchronously without blocking the UI. It supports complex queries through indexes, transactions for data integrity, and can store various data types including objects, blobs, and files.

**Use IndexedDB when:**
- Storing large amounts of data (> 10MB)
- Building offline-first applications
- Need complex queries and indexing
- Require transaction support
- Working with structured data (e.g., user-generated content, cached API responses)

**Use localStorage when:**
- Small, simple key-value data (< 5MB)
- Synchronous access is acceptable
- Simple user preferences or settings
- No complex querying needed

### Q2: Explain the concept of transactions in IndexedDB and why they're important.
**Answer:** Transactions in IndexedDB are wrappers around one or more database operations that ensure data integrity through ACID properties (Atomicity, Consistency, Isolation, Durability). All read and write operations must occur within a transaction.

**Key aspects:**
- **Atomicity**: All operations in a transaction either succeed together or fail together (all-or-nothing)
- **Consistency**: Database moves from one valid state to another
- **Isolation**: Concurrent transactions don't interfere with each other
- **Durability**: Committed changes persist even after browser crashes

**Transaction modes:**
- `readonly`: Only read operations (default)
- `readwrite`: Read and write operations
- `versionchange`: Only available during database upgrades

**Example:**
```javascript
const tx = db.transaction(['users', 'posts'], 'readwrite');
// Multiple operations within same transaction
await tx.objectStore('users').put(user);
await tx.objectStore('posts').put(post);
await tx.done; // All succeed or all fail
```

Transactions automatically commit when all requests complete and auto-abort on errors, ensuring data consistency.

### Q3: How do indexes work in IndexedDB? What are their benefits and limitations?
**Answer:** Indexes in IndexedDB are specialized data structures that enable fast lookups on specific object properties without scanning all records. They're created during database upgrades and maintained automatically.

**Benefits:**
- **Fast queries**: O(log n) lookups instead of O(n) scans
- **Range queries**: Support for bound ranges (between, greater than, less than)
- **Sorting**: Records can be retrieved in sorted order
- **Multiple indexes**: Query the same data in different ways

**Types:**
- **Simple index**: Single property (e.g., `email`)
- **Compound index**: Multiple properties (e.g., `['category', 'price']`)
- **Multi-entry index**: Array properties where each element is indexed separately
- **Unique index**: Enforces uniqueness constraint

**Limitations:**
- Must be created during `onupgradeneeded` (can't add dynamically)
- Can't index all property types (no functions, symbols)
- No full-text search (substring matching requires workarounds)
- Storage overhead (indexes take up space)
- Can't do complex queries like SQL JOINs

**Example:**
```javascript
// Create compound index for category + price queries
store.createIndex('category_price', ['category', 'price']);

// Query: Get electronics under $500
const range = IDBKeyRange.bound(['electronics', 0], ['electronics', 500]);
const results = await index.getAll(range);
```

### Q4: What's the difference between `add()` and `put()` methods?
**Answer:** Both methods insert records into an object store, but handle existing keys differently:

**`add()`:**
- Fails if a record with the same key already exists
- Throws `ConstraintError` on duplicate key
- Use for "create only" operations
- Guarantees no overwriting of existing data

**`put()`:**
- Overwrites existing records with the same key
- Creates new record if key doesn't exist
- Use for "create or update" (upsert) operations
- More flexible but can accidentally overwrite data

**Example:**
```javascript
// Add - fails if user with id=1 exists
try {
    await store.add({ id: 1, name: 'John' });
} catch (error) {
    console.error('User already exists:', error);
}

// Put - overwrites if exists, creates if not
await store.put({ id: 1, name: 'John Updated' }); // Always succeeds
```

**Best practice:** Use `add()` for insertions to catch duplicate keys, and `put()` for updates where you explicitly want to overwrite.

### Q5: How do you handle database versioning and schema migrations in IndexedDB?
**Answer:** Database versioning in IndexedDB manages schema changes through the `onupgradeneeded` callback, which runs when the requested version is higher than the current version.

**Key concepts:**
- Each database has a version number (integer)
- Schema changes (object stores, indexes) only possible during upgrade
- `onupgradeneeded` receives `oldVersion` and `newVersion`
- Migrations should be cumulative (handle all versions)

**Best practices:**
```javascript
const db = await openDB('MyApp', 3, {
    upgrade(db, oldVersion, newVersion, transaction) {
        // Version 1: Initial schema
        if (oldVersion < 1) {
            const store = db.createObjectStore('users', { keyPath: 'id' });
            store.createIndex('email', 'email', { unique: true });
        }

        // Version 2: Add new index
        if (oldVersion < 2) {
            const store = transaction.objectStore('users');
            store.createIndex('role', 'role');
        }

        // Version 3: Data migration
        if (oldVersion < 3) {
            const store = transaction.objectStore('users');
            for await (const cursor of store) {
                const user = cursor.value;
                if (!user.role) {
                    user.role = 'user'; // Default role
                    await cursor.update(user);
                }
            }
        }
    }
});
```

**Common pitfalls:**
- Can't create stores/indexes outside `onupgradeneeded`
- Must handle all version increments (can't skip versions)
- Changing versions requires closing all connections
- Should maintain backward compatibility when possible

### Q6: What are cursors and when should you use them?
**Answer:** Cursors are iterators for traversing multiple records in an object store or index. They're essential for operations that need to examine or modify many records.

**When to use cursors:**
- **Complex filtering**: Conditions that can't be expressed with indexes
- **Batch updates**: Modifying multiple records
- **Pagination**: Implementing infinite scroll or page-based navigation
- **Aggregations**: Counting, summing, or grouping data
- **Conditional deletion**: Removing records based on complex criteria

**When NOT to use cursors:**
- Simple retrieval of all records (use `getAll()` instead)
- Single record lookup (use `get()` or index queries)
- Small datasets that fit in memory

**Cursor directions:**
- `next`: Forward, ascending keys
- `nextunique`: Forward, skip duplicate keys
- `prev`: Backward, descending keys
- `prevunique`: Backward, skip duplicate keys

**Example - Update with cursor:**
```javascript
// Increase prices by 10% for specific category
const tx = db.transaction('products', 'readwrite');
const store = tx.objectStore('products');
const index = store.index('category');

for await (const cursor of index.iterate('electronics')) {
    const product = cursor.value;
    product.price *= 1.1;
    await cursor.update(product);
}

await tx.done;
```

**Performance tip:** Use `openKeyCursor()` when you only need keys (faster than loading full values).

### Q7: How does the idb library improve upon the native IndexedDB API?
**Answer:** The `idb` library by Jake Archibald provides a Promise-based wrapper around IndexedDB, making it compatible with async/await and significantly improving developer experience.

**Key improvements:**
1. **Promise-based**: All operations return Promises (native API uses callbacks)
2. **Async/await support**: Clean, readable code without callback hell
3. **Simpler syntax**: Reduces boilerplate code
4. **Better error handling**: Errors propagate naturally through Promise chains
5. **Async iterators**: Use `for await...of` with cursors
6. **Transaction helpers**: Automatic transaction management

**Comparison:**
```javascript
// Native API (callback-based)
const request = db.transaction(['users'], 'readwrite')
    .objectStore('users')
    .add(user);

request.onsuccess = () => console.log('Added');
request.onerror = (e) => console.error('Error:', e);

// idb library (Promise-based)
try {
    await db.add('users', user);
    console.log('Added');
} catch (error) {
    console.error('Error:', error);
}
```

**With transactions:**
```javascript
// idb makes transactions cleaner
const tx = db.transaction('users', 'readwrite');
await tx.store.add(user);
await tx.done; // Wait for commit
```

**Note:** idb is a thin wrapper (3KB), adds minimal overhead, and is production-ready. It's highly recommended for new projects.

### Q8: How do you implement pagination with IndexedDB?
**Answer:** Pagination in IndexedDB can be implemented using cursors with `advance()` or by using `getAll()` with limits. Choice depends on use case.

**Method 1: Cursor with advance() (efficient for large datasets):**
```javascript
async function getPaginatedData(db, page, pageSize) {
    const tx = db.transaction('products', 'readonly');
    const store = tx.objectStore('products');

    const offset = page * pageSize;
    const results = [];

    let cursor = await store.openCursor();

    if (cursor && offset > 0) {
        cursor = await cursor.advance(offset); // Skip records
    }

    while (cursor && results.length < pageSize) {
        results.push(cursor.value);
        cursor = await cursor.continue();
    }

    return results;
}
```

**Method 2: getAllKeys + get (better for random access):**
```javascript
async function getPaginatedData(db, page, pageSize) {
    const tx = db.transaction('products', 'readonly');
    const store = tx.objectStore('products');

    const allKeys = await store.getAllKeys();
    const startIndex = page * pageSize;
    const endIndex = startIndex + pageSize;
    const pageKeys = allKeys.slice(startIndex, endIndex);

    const results = await Promise.all(
        pageKeys.map(key => store.get(key))
    );

    return {
        data: results,
        totalPages: Math.ceil(allKeys.length / pageSize),
        currentPage: page,
        totalItems: allKeys.length
    };
}
```

**Method 3: getAll with limit (simplest, but loads all into memory):**
```javascript
async function getFirstPage(db, pageSize) {
    return await db.getAll('products', null, pageSize);
}
```

**Best practice:** Cache total count separately to avoid counting records on each page load.

### Q9: What are the security implications of IndexedDB?
**Answer:** IndexedDB follows the same-origin policy and has several security considerations:

**Key security aspects:**

1. **Same-origin policy**: Each origin has isolated database storage
   - `https://example.com` can't access `https://other.com`'s data
   - Subdomains are different origins
   - Ports matter: `http://localhost:3000` ` `http://localhost:4000`

2. **No encryption at rest**: Data stored in plaintext on disk
   - Accessible if device is compromised
   - Browser/OS security is the only protection
   - Solution: Encrypt sensitive data before storing

3. **XSS vulnerabilities**: Malicious scripts can access IndexedDB
   - Any JavaScript on the page can read/write
   - Sanitize user input to prevent XSS
   - Use Content Security Policy (CSP)

4. **No built-in authentication**: Anyone with page access can access database
   - Implement application-layer access control
   - Don't store sensitive data without encryption

5. **Private browsing mode**: Data deleted when session ends
   - Reduced quota in private mode
   - Detect and handle appropriately

**Security best practices:**
```javascript
// 1. Encrypt sensitive data
import { encrypt, decrypt } from './crypto';

await db.add('users', {
    id: 1,
    username: 'john',
    email: encrypt('john@example.com'), // Encrypt PII
    ssn: encrypt('123-45-6789')         // Encrypt sensitive data
});

// 2. Validate data on retrieval
const user = await db.get('users', userId);
if (!isValidUser(user)) {
    throw new Error('Data integrity check failed');
}

// 3. Check for private browsing
async function isPrivateMode() {
    if (navigator.storage && navigator.storage.persisted) {
        return !(await navigator.storage.persisted());
    }
    return false;
}
```

**Never store:** Unencrypted passwords, tokens, credit cards, or PII without encryption.

### Q10: How do you handle IndexedDB quota exceeded errors?
**Answer:** Quota exceeded errors occur when storage limit is reached. Handling requires detecting the error, informing users, and implementing cleanup strategies.

**Detection and handling:**
```javascript
async function addDataWithQuotaHandling(db, storeName, data) {
    try {
        return await db.add(storeName, data);
    } catch (error) {
        if (error.name === 'QuotaExceededError') {
            console.error('Storage quota exceeded');

            // Strategy 1: Delete old data
            await cleanupOldData(db, storeName);

            // Retry
            try {
                return await db.add(storeName, data);
            } catch (retryError) {
                // Strategy 2: Ask user to free space
                notifyUser('Storage full. Please free up space.');
                throw retryError;
            }
        }
        throw error;
    }
}

// Cleanup old data
async function cleanupOldData(db, storeName, daysToKeep = 30) {
    const cutoff = new Date();
    cutoff.setDate(cutoff.getDate() - daysToKeep);

    const tx = db.transaction(storeName, 'readwrite');
    const store = tx.objectStore(storeName);
    const index = store.index('createdAt');

    const range = IDBKeyRange.upperBound(cutoff);

    for await (const cursor of index.iterate(range)) {
        await cursor.delete();
    }

    await tx.done;
}
```

**Preventive measures:**
```javascript
// Check quota before large operations
async function checkQuotaBefore(requiredSpace) {
    if (navigator.storage && navigator.storage.estimate) {
        const estimate = await navigator.storage.estimate();
        const available = estimate.quota - estimate.usage;

        if (available < requiredSpace) {
            throw new Error('Insufficient storage space');
        }
    }
}

// Request persistent storage (prevents eviction)
async function requestPersistentStorage() {
    if (navigator.storage && navigator.storage.persist) {
        const isPersisted = await navigator.storage.persist();
        console.log(`Persistent storage: ${isPersisted}`);
        return isPersisted;
    }
    return false;
}

// Monitor storage usage
async function monitorStorageUsage() {
    const estimate = await navigator.storage.estimate();
    const percentUsed = (estimate.usage / estimate.quota) * 100;

    if (percentUsed > 80) {
        console.warn('Storage usage high:', percentUsed.toFixed(2) + '%');
        // Trigger cleanup
        await cleanupOldData(db, 'cache', 7); // Keep only 7 days
    }
}
```

**Quota limits** vary by browser:
- **Chrome/Edge**: ~60% of disk space
- **Firefox**: ~50% of disk space (max 2GB per origin)
- **Safari**: 1GB per origin

**Best practices:**
- Regularly cleanup old/unused data
- Request persistent storage for critical apps
- Monitor quota usage
- Provide user controls to manage storage
- Implement graceful degradation when quota exceeded

---

## Summary

IndexedDB is a powerful client-side database for modern web applications, offering:

**Key Strengths:**
- Large storage capacity (hundreds of MB to GB)
- Asynchronous, non-blocking API
- Transaction support for data integrity
- Fast indexed queries
- Support for complex data structures

**Core Concepts:**
- **Object stores**: Similar to tables in SQL
- **Indexes**: Fast lookups on properties
- **Transactions**: ACID-compliant operations
- **Cursors**: Iteration over multiple records
- **Versions**: Schema management and migrations

**Best Practices:**
- Use the `idb` library for cleaner Promise-based code
- Create indexes for frequently queried properties
- Batch operations in single transactions
- Handle quota exceeded errors gracefully
- Encrypt sensitive data before storage
- Monitor storage usage and implement cleanup
- Request persistent storage for critical data

**When to Use:**
- Offline-first applications
- Caching large API responses
- Storing user-generated content
- Complex data with relationships
- High-performance data access

IndexedDB is essential for building modern, performant web applications that work seamlessly offline and provide excellent user experiences.

---

## External Resources

- [MDN: IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [idb Library by Jake Archibald](https://github.com/jakearchibald/idb)
- [Working with IndexedDB](https://web.dev/indexeddb/)
- [IndexedDB Best Practices](https://developers.google.com/web/ilt/pwa/working-with-indexeddb)
- [Storage for the Web](https://web.dev/storage-for-the-web/)
- [Can I use IndexedDB?](https://caniuse.com/indexeddb)

---

[ê Back to Browser APIs](./README.md) | [Next: Browser Permissions í](./04-browser-permissions.md)
