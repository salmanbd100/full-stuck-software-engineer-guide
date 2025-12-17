# Background Sync API

## Overview

The Background Sync API enables web applications to defer actions until the user has a stable internet connection. This is a critical PWA feature that allows applications to reliably sync data even when users are temporarily offline. Unlike push notifications that rely on external servers, background sync allows the application to schedule synchronization tasks that persist across browser sessions.

## Table of Contents
- [What is Background Sync](#what-is-background-sync)
- [One-Time Sync](#one-time-sync)
- [Periodic Background Sync](#periodic-background-sync)
- [Use Cases](#use-cases)
- [Implementation](#implementation)
- [Tag-Based Sync](#tag-based-sync)
- [Testing Background Sync](#testing-background-sync)
- [Browser Support](#browser-support)
- [Workbox Integration](#workbox-integration)
- [Complete Examples](#complete-examples)
- [Interview Questions](#interview-questions)

## What is Background Sync

Background Sync is a Web API that allows service workers to schedule background synchronization tasks. When a sync event is triggered (either by the application or the system), the service worker wakes up and attempts to complete pending tasks.

### Key Characteristics

```javascript
// Background Sync characteristics
const backgroundSyncFeatures = {
  reliability: 'Tasks persist across browser restarts',
  offline: 'Works even when user is offline',
  automatic: 'Retries automatically when connection returns',
  deferred: 'Defers work until conditions are met',
  efficient: 'System manages retry and timing',
  reliable: 'Guaranteed delivery when online'
};

// Typical use case flow
const userFlow = {
  step1: 'User performs action (submit form, send message)',
  step2: 'Browser checks internet connection',
  step3_online: 'If online, send immediately',
  step3_offline: 'If offline, register for background sync',
  step4: 'Service worker queues the task',
  step5: 'System monitors connection',
  step6: 'When online, service worker fires sync event',
  step7: 'Task is retried automatically'
};
```

### Comparison with Other Approaches

```javascript
// Different data sync strategies
const syncStrategies = {
  // L Traditional polling
  polling: {
    approach: 'Request server every N seconds',
    reliability: 'Low - misses connection changes',
    battery: 'Poor - constant requests',
    userExperience: 'Laggy - delayed updates'
  },

  // † WebSockets
  websockets: {
    approach: 'Maintain persistent connection',
    reliability: 'Medium - connection can drop',
    battery: 'Poor - always connected',
    userExperience: 'Good - real-time updates',
    limitation: 'Only works while app is open'
  },

  //  Background Sync (Best for PWA)
  backgroundSync: {
    approach: 'Sync when connection available',
    reliability: 'Very High - OS managed',
    battery: 'Good - no polling',
    userExperience: 'Excellent - silent sync',
    benefit: 'Works across browser restarts'
  }
};
```

## One-Time Sync

One-time sync registers a single synchronization task that fires once when connectivity is restored.

### Basic Implementation

```javascript
// Client-side: Register for background sync
async function submitFormOffline(formData) {
  try {
    // Try to send immediately
    const response = await fetch('/api/submit', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(formData)
    });

    if (response.ok) {
      return response.json();
    }
  } catch (error) {
    // Network error - register for background sync
    console.log('Network error, registering for background sync');

    // Save form data for later
    const db = await openIndexedDB();
    await db.store('pending-forms', {
      id: Date.now(),
      data: formData,
      timestamp: new Date()
    });

    // Register sync tag
    if ('serviceWorker' in navigator && 'SyncManager' in window) {
      const registration = await navigator.serviceWorker.ready;
      await registration.sync.register('sync-forms');
      console.log('Background sync registered for sync-forms');
      return { queued: true, message: 'Will send when online' };
    }
  }
}

// Service Worker: Listen for sync events
self.addEventListener('sync', (event) => {
  console.log('Sync event received:', event.tag);

  if (event.tag === 'sync-forms') {
    // Keep the service worker alive until sync completes
    event.waitUntil(syncPendingForms());
  }
});

// Service Worker: Perform the actual sync
async function syncPendingForms() {
  const db = await openIndexedDB();
  const forms = await db.getAll('pending-forms');

  for (const form of forms) {
    try {
      const response = await fetch('/api/submit', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(form.data)
      });

      if (response.ok) {
        // Remove from pending
        await db.delete('pending-forms', form.id);
        console.log('Form synced successfully:', form.id);
      }
    } catch (error) {
      console.error('Sync failed, will retry:', error);
      // Don't remove from queue - will retry on next sync event
      throw error; // Rethrow to signal sync failed
    }
  }
}
```

### Better Error Handling

```javascript
// Service Worker: Advanced sync with retry logic
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-forms') {
    event.waitUntil(
      syncWithRetry('sync-forms', 3, 1000)
        .catch((error) => {
          console.error('Sync failed after retries:', error);
          // Log for analytics
          logSyncFailure('sync-forms', error);
        })
    );
  }
});

async function syncWithRetry(tag, maxRetries = 3, delayMs = 1000) {
  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      console.log(`Attempt ${attempt}/${maxRetries} for ${tag}`);

      switch (tag) {
        case 'sync-forms':
          await syncPendingForms();
          break;
        case 'sync-messages':
          await syncPendingMessages();
          break;
        default:
          throw new Error(`Unknown sync tag: ${tag}`);
      }

      // Success
      return;
    } catch (error) {
      lastError = error;
      console.warn(`Attempt ${attempt} failed:`, error.message);

      if (attempt < maxRetries) {
        // Wait before retrying
        await new Promise((resolve) => setTimeout(resolve, delayMs));
      }
    }
  }

  // All retries failed
  throw new Error(
    `Sync failed after ${maxRetries} attempts: ${lastError.message}`
  );
}
```

## Periodic Background Sync

Periodic sync allows tasks to run periodically (e.g., every 24 hours) even when the app is not open.

### Implementation

```javascript
// Client-side: Register periodic sync
async function enablePeriodicSync() {
  if (!('serviceWorker' in navigator) || !('SyncManager' in window)) {
    console.warn('Periodic background sync not supported');
    return;
  }

  try {
    const registration = await navigator.serviceWorker.ready;

    // Register periodic sync to run every day
    await registration.periodicSync.register('daily-analytics', {
      minInterval: 24 * 60 * 60 * 1000 // 24 hours
    });

    console.log('Periodic sync registered for daily-analytics');
  } catch (error) {
    if (error.name === 'NotAllowedError') {
      console.warn('Permission denied for periodic sync');
    } else {
      console.error('Failed to register periodic sync:', error);
    }
  }
}

// Service Worker: Listen for periodic sync
self.addEventListener('periodicsync', (event) => {
  console.log('Periodic sync event received:', event.tag);

  if (event.tag === 'daily-analytics') {
    event.waitUntil(syncAnalytics());
  } else if (event.tag === 'sync-cache') {
    event.waitUntil(updateCache());
  }
});

// Service Worker: Send analytics data
async function syncAnalytics() {
  try {
    const analytics = await getStoredAnalytics();

    if (analytics.length === 0) {
      console.log('No analytics to sync');
      return;
    }

    const response = await fetch('/api/analytics/batch', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ events: analytics })
    });

    if (response.ok) {
      // Clear synced analytics
      await clearStoredAnalytics();
      console.log('Analytics synced successfully');
    } else {
      throw new Error(`HTTP ${response.status}`);
    }
  } catch (error) {
    console.error('Analytics sync failed:', error);
    throw error; // Will trigger automatic retry
  }
}

// Service Worker: Update cache periodically
async function updateCache() {
  const cacheName = 'app-cache-v1';
  const urlsToCache = [
    '/api/config',
    '/api/initial-data',
    '/images/logo.png'
  ];

  try {
    const cache = await caches.open(cacheName);

    for (const url of urlsToCache) {
      const response = await fetch(url);
      if (response.ok) {
        await cache.put(url, response);
      }
    }

    console.log('Cache updated successfully');
  } catch (error) {
    console.error('Cache update failed:', error);
    throw error;
  }
}
```

## Use Cases

### 1. Form Submissions

```javascript
// Online or offline form submission
class FormManager {
  async submitForm(formData) {
    // Store in IndexedDB for recovery
    await this.saveForm(formData);

    try {
      // Try immediate submission
      const response = await this.sendToServer(formData);
      await this.removeForm(formData.id);
      return response;
    } catch (error) {
      // Register background sync
      if (navigator.serviceWorker && navigator.serviceWorker.controller) {
        const registration = await navigator.serviceWorker.ready;
        await registration.sync.register('sync-forms');
        return { queued: true };
      }
      throw error;
    }
  }

  async sendToServer(formData) {
    const response = await fetch('/api/forms', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(formData)
    });

    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  }
}
```

### 2. Chat Messages

```javascript
// Send messages offline, sync when online
class ChatManager {
  async sendMessage(conversationId, text) {
    const message = {
      id: this.generateId(),
      conversationId,
      text,
      timestamp: new Date().toISOString(),
      status: 'pending' // pending, sent, failed
    };

    // Save locally
    await this.saveMessage(message);

    try {
      // Try sending immediately
      const response = await fetch('/api/messages', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(message)
      });

      if (response.ok) {
        // Update local status
        message.status = 'sent';
        await this.updateMessage(message);
      }
    } catch (error) {
      // Will sync when online
      console.log('Message queued for sync');
      const registration = await navigator.serviceWorker.ready;
      await registration.sync.register('sync-messages');
    }

    return message;
  }
}

// Service Worker: Sync messages
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-messages') {
    event.waitUntil(
      (async () => {
        const db = await openDB();
        const messages = await db.getAll('messages', 'pending');

        for (const message of messages) {
          try {
            const response = await fetch('/api/messages', {
              method: 'POST',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify(message)
            });

            if (response.ok) {
              message.status = 'sent';
              await db.put('messages', message);
            }
          } catch (error) {
            throw error; // Retry
          }
        }
      })()
    );
  }
});
```

### 3. Analytics Events

```javascript
// Queue analytics events offline
class AnalyticsManager {
  async trackEvent(eventName, data) {
    const event = {
      name: eventName,
      data,
      timestamp: new Date().toISOString()
    };

    // Queue in IndexedDB
    await this.queueEvent(event);

    // Try sending immediately (non-blocking)
    this.sendBatch().catch(() => {
      // Ignore errors, will sync in background
    });
  }

  async sendBatch() {
    const events = await this.getQueuedEvents();
    if (events.length === 0) return;

    const response = await fetch('/api/analytics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ events })
    });

    if (response.ok) {
      await this.clearEvents(events.map((e) => e.id));
    }
  }
}

// Service Worker: Periodic analytics sync
self.addEventListener('periodicsync', (event) => {
  if (event.tag === 'sync-analytics') {
    event.waitUntil(
      (async () => {
        const db = await openDB();
        const events = await db.getAll('analytics-queue');

        if (events.length > 0) {
          const response = await fetch('/api/analytics/batch', {
            method: 'POST',
            body: JSON.stringify({ events })
          });

          if (response.ok) {
            await db.clear('analytics-queue');
          }
        }
      })()
    );
  }
});
```

## Implementation

### Step 1: Set Up Service Worker

```javascript
// service-worker.js
const CACHE_VERSION = 'app-v1';

self.addEventListener('install', (event) => {
  console.log('Service Worker installed');
  self.skipWaiting();
});

self.addEventListener('activate', (event) => {
  console.log('Service Worker activated');
  event.waitUntil(clients.claim());
});

// Handle sync events
self.addEventListener('sync', (event) => {
  console.log('Sync event:', event.tag);

  const syncHandlers = {
    'sync-forms': syncForms,
    'sync-messages': syncMessages,
    'sync-data': syncData
  };

  if (event.tag in syncHandlers) {
    event.waitUntil(syncHandlers[event.tag]());
  }
});

async function syncForms() {
  // Implementation
}

async function syncMessages() {
  // Implementation
}

async function syncData() {
  // Implementation
}
```

### Step 2: Register Service Worker

```javascript
// app.js
async function registerServiceWorker() {
  if (!('serviceWorker' in navigator)) {
    console.warn('Service Workers not supported');
    return;
  }

  try {
    const registration = await navigator.serviceWorker.register(
      '/service-worker.js'
    );
    console.log('Service Worker registered:', registration);
  } catch (error) {
    console.error('Service Worker registration failed:', error);
  }
}

document.addEventListener('DOMContentLoaded', registerServiceWorker);
```

### Step 3: Handle Offline Actions

```javascript
// offline-handler.js
class OfflineHandler {
  constructor() {
    this.online = navigator.onLine;
    window.addEventListener('online', () => this.handleOnline());
    window.addEventListener('offline', () => this.handleOffline());
  }

  handleOffline() {
    this.online = false;
    console.log('Application went offline');
    this.showOfflineNotification();
  }

  handleOnline() {
    this.online = true;
    console.log('Application came online');
    this.hideOfflineNotification();
    this.triggerSync();
  }

  async triggerSync() {
    if ('serviceWorker' in navigator && 'SyncManager' in window) {
      const registration = await navigator.serviceWorker.ready;
      try {
        await registration.sync.register('sync-all');
        console.log('Background sync triggered');
      } catch (error) {
        console.error('Failed to trigger sync:', error);
      }
    }
  }

  showOfflineNotification() {
    const notification = document.createElement('div');
    notification.className = 'offline-banner';
    notification.textContent = 'You are offline. Changes will sync when online.';
    document.body.appendChild(notification);
  }

  hideOfflineNotification() {
    const notification = document.querySelector('.offline-banner');
    if (notification) {
      notification.remove();
    }
  }
}

const handler = new OfflineHandler();
```

## Tag-Based Sync

Using sync tags allows you to organize and manage different types of background sync tasks.

```javascript
// Multiple sync tasks with tags
async function registerSyncTasks() {
  const registration = await navigator.serviceWorker.ready;

  // Different sync tags for different purposes
  const syncTasks = {
    'sync-forms': 'Form submissions',
    'sync-messages': 'Chat messages',
    'sync-photos': 'Photo uploads',
    'sync-analytics': 'Analytics events'
  };

  for (const [tag, description] of Object.entries(syncTasks)) {
    try {
      await registration.sync.register(tag);
      console.log(`Registered: ${description}`);
    } catch (error) {
      console.error(`Failed to register ${tag}:`, error);
    }
  }
}

// Service Worker: Handle multiple sync tags
self.addEventListener('sync', (event) => {
  const handlers = {
    'sync-forms': async () => {
      console.log('Syncing forms...');
      const db = await openDB();
      const forms = await db.getAll('pending-forms');
      // Process forms
    },

    'sync-messages': async () => {
      console.log('Syncing messages...');
      const db = await openDB();
      const messages = await db.getAll('pending-messages');
      // Process messages
    },

    'sync-photos': async () => {
      console.log('Syncing photos...');
      const db = await openDB();
      const photos = await db.getAll('pending-photos');
      // Process photos
    },

    'sync-analytics': async () => {
      console.log('Syncing analytics...');
      const db = await openDB();
      const events = await db.getAll('analytics-events');
      // Process events
    }
  };

  if (event.tag in handlers) {
    event.waitUntil(handlers[event.tag]());
  }
});

// Get pending sync tags
async function getPendingSyncTags() {
  const registration = await navigator.serviceWorker.ready;
  const tags = await registration.sync.getTags();
  console.log('Pending sync tags:', tags);
  return tags;
}
```

## Testing Background Sync

### Manual Testing

```javascript
// Test: Submit form while offline
async function testOfflineFormSubmission() {
  // 1. Go offline: DevTools > Network > Offline
  console.log('Step 1: Set DevTools to Offline mode');

  // 2. Submit form
  const form = document.querySelector('form');
  form.dispatchEvent(new Event('submit'));

  // 3. Check IndexedDB for queued data
  console.log('Step 2: Check IndexedDB Application > Storage');

  // 4. Go online
  console.log('Step 3: Set DevTools to Online mode');

  // 5. Observe sync event in Service Worker logs
  console.log('Step 4: Check Service Worker logs for sync event');
}

// Test: Check pending sync tags
async function testPendingSyncTags() {
  const registration = await navigator.serviceWorker.ready;
  const tags = await registration.sync.getTags();
  console.log('Pending sync tags:', tags);
}

// Test: Manually trigger sync
async function testManualSync() {
  const registration = await navigator.serviceWorker.ready;
  try {
    await registration.sync.register('test-sync');
    console.log('Sync tag registered for testing');
  } catch (error) {
    console.error('Failed to register sync:', error);
  }
}
```

### Automated Testing

```javascript
// Vitest/Jest testing
describe('Background Sync', () => {
  let serviceWorkerContainer;

  beforeEach(() => {
    // Mock service worker
    serviceWorkerContainer = {
      ready: Promise.resolve({
        sync: {
          register: jest.fn(),
          getTags: jest.fn()
        }
      })
    };

    Object.defineProperty(navigator, 'serviceWorker', {
      value: serviceWorkerContainer,
      configurable: true
    });
  });

  test('registers sync when form submission fails offline', async () => {
    // Mock fetch to fail
    global.fetch = jest.fn().mockRejectedValue(new Error('Network error'));

    const registration = await navigator.serviceWorker.ready;

    // Attempt submission
    const formData = { email: 'test@example.com' };
    try {
      await submitForm(formData);
    } catch (error) {
      // Expected to fail
    }

    // Verify sync was registered
    expect(registration.sync.register).toHaveBeenCalledWith('sync-forms');
  });

  test('processes pending items on sync event', async () => {
    const mockSync = jest.fn();
    const event = new Event('sync');
    event.tag = 'sync-forms';
    event.waitUntil = jest.fn((promise) => promise);

    // Simulate sync event
    self.dispatchEvent(event);

    // Verify handler was called
    // (Implementation depends on your sync handler)
  });

  test('retries failed syncs', async () => {
    const registration = await navigator.serviceWorker.ready;

    // First attempt fails
    registration.sync.register.mockRejectedValueOnce(new Error('Network error'));

    // Second attempt succeeds
    registration.sync.register.mockResolvedValueOnce(undefined);

    // This would be part of your retry logic
  });
});
```

## Browser Support

### Current Support Status (2024)

```javascript
// Check browser support
const backgroundSyncSupported = () => {
  return (
    'serviceWorker' in navigator &&
    'SyncManager' in window &&
    'ServiceWorkerRegistration' in window
  );
};

const periodicSyncSupported = () => {
  return (
    'serviceWorker' in navigator &&
    'periodicSync' in ServiceWorkerRegistration.prototype
  );
};

// Graceful degradation
async function initializeSync() {
  if (backgroundSyncSupported()) {
    console.log(' One-time sync supported');
  } else {
    console.log('† One-time sync not supported, using fallback');
    // Use IndexedDB queue + polling as fallback
  }

  if (periodicSyncSupported()) {
    console.log(' Periodic sync supported');
  } else {
    console.log('† Periodic sync not supported, using fallback');
    // Use web worker timer as fallback
  }
}
```

### Browser Compatibility Matrix

| Browser | One-Time Sync | Periodic Sync | Notes |
|---------|---------------|---------------|-------|
| Chrome 49+ |  Yes |  Yes (71+) | Full support |
| Firefox | † Behind flag | † Behind flag | Not enabled by default |
| Safari | L No | L No | No support |
| Edge |  Yes |  Yes | Same as Chrome |
| Opera |  Yes |  Yes | Same as Chrome |

## Workbox Integration

### Using Workbox for Background Sync

```javascript
// Using Workbox library (simplifies background sync)
import { BackgroundSyncPlugin } from 'workbox-background-sync';
import { registerRoute } from 'workbox-routing';
import { NetworkFirst } from 'workbox-strategies';

// Wrap API calls with background sync
const bgSyncPlugin = new BackgroundSyncPlugin('api-queue', {
  maxRetentionTime: 24 * 60 // 24 hours
});

// Sync form submissions
registerRoute(
  ({ url }) => url.pathname === '/api/forms',
  new NetworkFirst({
    cacheName: 'forms-cache',
    plugins: [bgSyncPlugin]
  }),
  'POST'
);

// Sync messages
registerRoute(
  ({ url }) => url.pathname === '/api/messages',
  new NetworkFirst({
    cacheName: 'messages-cache',
    plugins: [bgSyncPlugin]
  }),
  'POST'
);
```

### Workbox Configuration in Next.js

```javascript
// next.config.js with workbox
const withPWA = require('next-pwa')({
  dest: 'public',
  register: true,
  skipWaiting: true,
  buildExcludes: [/middleware-manifest.json$/],
  publicExcludes: ['!noprecache/**/*'],
  ext: '.js',
  // Configure background sync
  sw: 'service-worker.js'
});

module.exports = withPWA({
  // Next.js config
});

// Service worker with Workbox
import { precacheAndRoute } from 'workbox-precaching';
import { registerRoute } from 'workbox-routing';
import { NetworkFirst } from 'workbox-strategies';
import { BackgroundSyncPlugin } from 'workbox-background-sync';

// Precache files
precacheAndRoute(self.__WB_MANIFEST || []);

// Background sync queue
const bgSyncPlugin = new BackgroundSyncPlugin('sync-queue', {
  maxRetentionTime: 24 * 60
});

// API routes with background sync
registerRoute(
  ({ url }) => url.origin === self.location.origin && url.pathname.startsWith('/api/'),
  new NetworkFirst({
    plugins: [bgSyncPlugin]
  }),
  'POST'
);
```

## Complete Examples

### Example 1: Todo App with Offline Support

```javascript
// Main app.js
class TodoApp {
  constructor() {
    this.registerServiceWorker();
    this.setupEventListeners();
    this.loadTodos();
  }

  async registerServiceWorker() {
    if (!('serviceWorker' in navigator)) return;

    try {
      await navigator.serviceWorker.register('/sw.js');
      console.log('Service Worker registered');
    } catch (error) {
      console.error('Service Worker registration failed:', error);
    }
  }

  setupEventListeners() {
    document.getElementById('todo-form').addEventListener('submit', (e) => {
      e.preventDefault();
      this.addTodo();
    });

    // Handle online/offline
    window.addEventListener('online', () => this.handleOnline());
    window.addEventListener('offline', () => this.handleOffline());
  }

  async addTodo() {
    const input = document.getElementById('todo-input');
    const text = input.value.trim();

    if (!text) return;

    const todo = {
      id: Date.now(),
      text,
      completed: false,
      createdAt: new Date().toISOString(),
      synced: false
    };

    // Save locally
    await this.saveTodoLocal(todo);

    // Display immediately
    this.renderTodo(todo);

    // Try to sync
    await this.syncTodo(todo);

    input.value = '';
  }

  async syncTodo(todo) {
    try {
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(todo)
      });

      if (response.ok) {
        todo.synced = true;
        await this.saveTodoLocal(todo);
        this.updateTodoUI(todo);
      }
    } catch (error) {
      console.log('Sync failed, will retry in background');

      // Register for background sync
      const registration = await navigator.serviceWorker.ready;
      await registration.sync.register('sync-todos');
    }
  }

  async handleOnline() {
    console.log('App came online');
    // Trigger background sync
    const registration = await navigator.serviceWorker.ready;
    await registration.sync.register('sync-todos');
  }

  handleOffline() {
    console.log('App went offline');
    // Show offline indicator
    document.body.classList.add('offline');
  }

  // IndexedDB operations
  async saveTodoLocal(todo) {
    // Implementation
  }

  async loadTodos() {
    // Implementation
  }

  renderTodo(todo) {
    // Implementation
  }

  updateTodoUI(todo) {
    // Implementation
  }
}

// Service Worker: sw.js
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-todos') {
    event.waitUntil(syncTodos());
  }
});

async function syncTodos() {
  const db = await openDB();
  const todos = await db.getAll('todos');
  const unsynced = todos.filter((t) => !t.synced);

  for (const todo of unsynced) {
    try {
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(todo)
      });

      if (response.ok) {
        todo.synced = true;
        await db.put('todos', todo);
      }
    } catch (error) {
      throw error; // Retry
    }
  }
}
```

### Example 2: Chat App with Background Message Sync

```javascript
// chat.js
class ChatApp {
  async sendMessage(conversationId, text) {
    const message = {
      id: this.generateId(),
      conversationId,
      text,
      timestamp: new Date().toISOString(),
      status: 'pending',
      synced: false
    };

    // Save locally and display
    await this.saveMessage(message);
    this.displayMessage(message);

    // Try to send
    try {
      const response = await fetch('/api/messages', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(message)
      });

      if (response.ok) {
        message.status = 'sent';
        message.synced = true;
        await this.updateMessage(message);
      }
    } catch (error) {
      console.log('Message queued for sync');
      message.status = 'queued';
      await this.updateMessage(message);

      // Register for background sync
      const registration = await navigator.serviceWorker.ready;
      await registration.sync.register('sync-messages');
    }
  }

  async saveMessage(message) {
    const db = await this.getDB();
    await db.add('messages', message);
  }

  displayMessage(message) {
    const el = document.createElement('div');
    el.className = `message ${message.status}`;
    el.textContent = message.text;
    document.querySelector('.chat').appendChild(el);
  }

  generateId() {
    return `${Date.now()}-${Math.random()}`;
  }

  async getDB() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open('chat-app', 1);
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }

  async updateMessage(message) {
    const db = await this.getDB();
    const tx = db.transaction('messages', 'readwrite');
    const store = tx.objectStore('messages');
    store.put(message);
  }
}

// Service Worker: chat-sw.js
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-messages') {
    event.waitUntil(syncMessages());
  }
});

async function syncMessages() {
  const db = await openDB();
  const tx = db.transaction('messages', 'readonly');
  const store = tx.objectStore('messages');
  const messages = await store.getAll();

  const unsynced = messages.filter((m) => !m.synced);

  for (const message of unsynced) {
    try {
      const response = await fetch('/api/messages', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(message)
      });

      if (response.ok) {
        message.synced = true;
        const updateTx = db.transaction('messages', 'readwrite');
        updateTx.objectStore('messages').put(message);
      }
    } catch (error) {
      throw error;
    }
  }
}
```

## Interview Questions

### Question 1: What is the Background Sync API and why is it important for PWAs?

**Answer:**

The Background Sync API is a Web API that allows service workers to schedule background synchronization tasks. It enables applications to defer actions (like form submissions) until the user has a stable internet connection, providing reliable data synchronization across browser sessions.

**Why it's important:**

```javascript
const importance = {
  reliability: 'Guaranteed delivery when online, even after browser restarts',
  userExperience: 'Users can work offline without worrying about losing data',
  efficiency: 'System manages retry timing, no need for polling',
  scope: 'Works even when tab/browser is closed',
  offline: 'Handles connectivity changes transparently'
};

// Problem it solves
const withoutBackgroundSync = {
  issue: 'User fills form offline, loses data if browser closes',
  solution: 'Data lost, bad UX'
};

const withBackgroundSync = {
  issue: 'User fills form offline',
  solution: 'Data queued, synced automatically when online'
};
```

**Code Example:**

```javascript
// Without Background Sync (problematic)
async function submitForm(data) {
  try {
    const response = await fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify(data)
    });
    // L If browser closes before response, data is lost
  } catch (error) {
    // L Error handling is application's responsibility
  }
}

// With Background Sync (reliable)
async function submitFormReliably(data) {
  // Save locally first
  await db.save('pending-forms', data);

  // Try to send
  try {
    const response = await fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify(data)
    });
  } catch (error) {
    // Register for background sync
    const registration = await navigator.serviceWorker.ready;
    await registration.sync.register('sync-forms');
    //  Data will sync automatically when online
  }
}
```

---

### Question 2: What's the difference between one-time sync and periodic sync?

**Answer:**

**One-time Sync:**
- Triggered manually or by user action
- Fires once when connectivity is restored
- Use for immediate actions (form submissions, messages)
- More reliable as it responds to actual connectivity

```javascript
// One-time sync example
async function submitForm(data) {
  await db.save('pending', data);
  const registration = await navigator.serviceWorker.ready;
  // Register once, fires when online
  await registration.sync.register('sync-once');
}
```

**Periodic Sync:**
- Triggered automatically by the system
- Fires repeatedly at specified intervals
- Use for background tasks (analytics, cache updates)
- Requires user permission
- System optimizes timing based on device state

```javascript
// Periodic sync example
async function enablePeriodicSync() {
  const registration = await navigator.serviceWorker.ready;
  // Fires every 24 hours
  await registration.periodicSync.register('sync-daily', {
    minInterval: 24 * 60 * 60 * 1000
  });
}
```

**Comparison:**

| Aspect | One-Time | Periodic |
|--------|----------|----------|
| **Trigger** | User action / connectivity | Time interval |
| **Frequency** | Once per registration | Repeating |
| **Use Case** | Forms, messages | Analytics, cache |
| **Permission** | No extra permission | Requires permission |
| **Reliability** | Very high | Medium (system-managed) |

---

### Question 3: How do you handle offline-first architecture with Background Sync?

**Answer:**

Offline-first means the app works completely offline first, then syncs when possible.

```javascript
// Complete offline-first implementation
class OfflineFirstApp {
  // Step 1: Always save locally first
  async addItem(item) {
    // 1. Save to IndexedDB immediately
    await this.db.add('items', item);

    // 2. Update UI immediately
    this.renderItem(item);

    // 3. Try to sync in background
    this.syncInBackground(item);
  }

  // Step 2: Non-blocking sync attempt
  async syncInBackground(item) {
    try {
      const response = await fetch('/api/items', {
        method: 'POST',
        body: JSON.stringify(item)
      });

      if (response.ok) {
        // Mark as synced
        item.synced = true;
        await this.db.update('items', item);
      }
    } catch (error) {
      // No network - will sync in background
      const registration = await navigator.serviceWorker.ready;
      await registration.sync.register('sync-items');
    }
  }

  // Step 3: Service Worker handles background sync
  setupSync() {
    self.addEventListener('sync', (event) => {
      if (event.tag === 'sync-items') {
        event.waitUntil(this.syncAllItems());
      }
    });
  }

  async syncAllItems() {
    const items = await this.db.getAll('items');
    const unsynced = items.filter((i) => !i.synced);

    for (const item of unsynced) {
      const response = await fetch('/api/items', {
        method: 'POST',
        body: JSON.stringify(item)
      });

      if (response.ok) {
        item.synced = true;
        await this.db.update('items', item);
      }
    }
  }
}
```

---

### Question 4: How do you test Background Sync functionality?

**Answer:**

Testing background sync involves both manual and automated approaches:

```javascript
// Manual Testing
async function testBackgroundSync() {
  // 1. Open DevTools > Network tab
  // 2. Set to "Offline" mode
  // 3. Perform action (submit form)
  // 4. Verify data saved in IndexedDB
  // 5. Set back to "Online"
  // 6. Check Service Worker console for sync event
  // 7. Verify data sent to server

  // Programmatic check
  const registration = await navigator.serviceWorker.ready;
  const tags = await registration.sync.getTags();
  console.log('Pending sync tags:', tags);
}

// Automated Testing
describe('Background Sync', () => {
  test('queues data when offline', async () => {
    // Mock offline
    Object.defineProperty(navigator, 'onLine', {
      writable: true,
      value: false
    });

    // Mock fetch to fail
    global.fetch = jest.fn().mockRejectedValue(new Error('Network'));

    // Try action
    await submitForm({ name: 'test' });

    // Verify queued in IndexedDB
    const db = await openDB();
    const pending = await db.getAll('pending-forms');
    expect(pending).toHaveLength(1);

    // Verify sync registered
    const registration = await navigator.serviceWorker.ready;
    expect(registration.sync.register).toHaveBeenCalledWith('sync-forms');
  });

  test('syncs data when online', async () => {
    // Setup pending data
    const db = await openDB();
    await db.add('pending-forms', { id: 1, name: 'test' });

    // Mock fetch to succeed
    global.fetch = jest.fn().mockResolvedValue({
      ok: true,
      json: async () => ({ success: true })
    });

    // Trigger sync event
    const syncEvent = new Event('sync');
    syncEvent.tag = 'sync-forms';
    syncEvent.waitUntil = jest.fn();

    self.dispatchEvent(syncEvent);

    // Verify fetch was called
    expect(global.fetch).toHaveBeenCalledWith(
      '/api/forms',
      expect.objectContaining({
        method: 'POST'
      })
    );
  });
});
```

---

### Question 5: How do you manage different sync tags effectively?

**Answer:**

Sync tags organize different types of background sync tasks:

```javascript
// Well-organized sync tags
const SYNC_TAGS = {
  FORMS: 'sync-forms',           // Form submissions
  MESSAGES: 'sync-messages',     // Chat messages
  PHOTOS: 'sync-photos',         // Photo uploads
  ANALYTICS: 'sync-analytics'    // Analytics events
};

// Register tags
async function registerSyncTags() {
  const registration = await navigator.serviceWorker.ready;

  for (const [key, tag] of Object.entries(SYNC_TAGS)) {
    try {
      await registration.sync.register(tag);
      console.log(`Registered: ${tag}`);
    } catch (error) {
      console.error(`Failed to register ${tag}:`, error);
    }
  }
}

// Service Worker: Handle tags efficiently
self.addEventListener('sync', (event) => {
  const handlers = {
    [SYNC_TAGS.FORMS]: () => syncForms(),
    [SYNC_TAGS.MESSAGES]: () => syncMessages(),
    [SYNC_TAGS.PHOTOS]: () => syncPhotos(),
    [SYNC_TAGS.ANALYTICS]: () => syncAnalytics()
  };

  if (event.tag in handlers) {
    event.waitUntil(handlers[event.tag]());
  }
});

// Check pending tags
async function getPendingWork() {
  const registration = await navigator.serviceWorker.ready;
  const tags = await registration.sync.getTags();

  const pending = {};
  for (const tag of tags) {
    pending[tag] = true;
  }

  return pending;
}

// UI: Show pending work
async function showPendingWork() {
  const pending = await getPendingWork();

  for (const [tag, isPending] of Object.entries(pending)) {
    const element = document.querySelector(`[data-sync-tag="${tag}"]`);
    if (element) {
      element.classList.toggle('syncing', isPending);
    }
  }
}
```

---

### Question 6: What are the limitations of Background Sync?

**Answer:**

Key limitations developers should understand:

```javascript
const limitations = {
  browserSupport: {
    issue: 'Limited browser support (Chrome, Edge only)',
    solution: 'Provide fallback sync mechanism'
  },

  noGuarantee: {
    issue: 'No guarantee task will execute (system dependent)',
    solution: 'Don\'t rely on Background Sync alone for critical data'
  },

  timeLimit: {
    issue: 'Service worker has limited execution time (~5 minutes)',
    solution: 'Keep sync handlers fast, break large tasks into batches'
  },

  userControl: {
    issue: 'User can disable service workers/sync',
    solution: 'Provide manual sync option'
  },

  noReturnValue: {
    issue: 'Can\'t return data back to main thread after sync',
    solution: 'Use postMessage or update local state for UI updates'
  }
};

// Example: Handling limitations
class RobustBackgroundSync {
  // Fallback for unsupported browsers
  async submitForm(data) {
    await this.saveLocal(data);

    if (this.supportsBackgroundSync()) {
      await this.registerSync('sync-forms');
    } else {
      // Fallback: use polling or periodic storage check
      await this.setupPollingSync();
    }
  }

  // Handle time limits
  async syncInBatches(items, batchSize = 50) {
    for (let i = 0; i < items.length; i += batchSize) {
      const batch = items.slice(i, i + batchSize);

      try {
        await this.syncBatch(batch);
      } catch (error) {
        // Stop processing if error occurs
        throw error; // Let sync handler retry
      }

      // Yield to prevent timeout
      await new Promise((resolve) => setTimeout(resolve, 100));
    }
  }

  // Provide manual sync
  async manualSync() {
    const registration = await navigator.serviceWorker.ready;
    await registration.sync.register('manual-sync');
  }

  supportsBackgroundSync() {
    return (
      'serviceWorker' in navigator &&
      'SyncManager' in window
    );
  }
}
```

---

### Question 7: How do you communicate sync status to the user?

**Answer:**

Keeping users informed about sync status improves UX:

```javascript
// Sync status communication
class SyncStatusManager {
  constructor() {
    this.syncStatus = new Map();
    this.setupListeners();
  }

  // Listen for online/offline events
  setupListeners() {
    window.addEventListener('online', () => this.handleOnline());
    window.addEventListener('offline', () => this.handleOffline());

    // Listen to sync events from Service Worker
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.addEventListener('message', (event) => {
        if (event.data.type === 'SYNC_STATUS') {
          this.updateStatus(event.data.tag, event.data.status);
        }
      });
    }
  }

  handleOffline() {
    this.showNotification('You are offline. Data will sync when online.');
    this.updateUI('offline');
  }

  handleOnline() {
    this.showNotification('Back online! Syncing data...');
    this.updateUI('syncing');
  }

  // Update sync status
  updateStatus(tag, status) {
    this.syncStatus.set(tag, status);

    switch (status) {
      case 'pending':
        this.showStatusBadge(tag, 'Pending');
        break;
      case 'syncing':
        this.showStatusBadge(tag, 'Syncing');
        break;
      case 'success':
        this.showStatusBadge(tag, 'Synced');
        break;
      case 'error':
        this.showStatusBadge(tag, 'Error');
        break;
    }
  }

  // UI Updates
  showNotification(message) {
    const notification = document.createElement('div');
    notification.className = 'sync-notification';
    notification.textContent = message;
    document.body.appendChild(notification);

    setTimeout(() => notification.remove(), 3000);
  }

  showStatusBadge(tag, statusText) {
    const element = document.querySelector(`[data-sync-tag="${tag}"]`);
    if (element) {
      const badge = element.querySelector('.sync-status') ||
        document.createElement('span');
      badge.className = 'sync-status';
      badge.textContent = statusText;
      element.appendChild(badge);
    }
  }

  updateUI(state) {
    document.body.classList.remove('online', 'offline', 'syncing');
    document.body.classList.add(state);
  }
}

// Service Worker: Send sync status updates
self.addEventListener('sync', async (event) => {
  const clients = await self.clients.matchAll();

  // Notify start
  clients.forEach((client) => {
    client.postMessage({
      type: 'SYNC_STATUS',
      tag: event.tag,
      status: 'syncing'
    });
  });

  try {
    await handleSync(event.tag);

    // Notify success
    clients.forEach((client) => {
      client.postMessage({
        type: 'SYNC_STATUS',
        tag: event.tag,
        status: 'success'
      });
    });
  } catch (error) {
    // Notify error
    clients.forEach((client) => {
      client.postMessage({
        type: 'SYNC_STATUS',
        tag: event.tag,
        status: 'error',
        error: error.message
      });
    });
  }
});
```

---

### Question 8: How do you handle sync failures and retries?

**Answer:**

Robust sync requires sophisticated error handling:

```javascript
// Advanced retry strategy
class SyncWithRetry {
  async syncWithExponentialBackoff(task, maxAttempts = 5) {
    let attempt = 0;
    let delay = 1000; // Start with 1 second

    while (attempt < maxAttempts) {
      try {
        attempt++;
        console.log(`Attempt ${attempt}/${maxAttempts}`);

        await task();
        return; // Success
      } catch (error) {
        if (attempt >= maxAttempts) {
          throw new Error(`Failed after ${maxAttempts} attempts: ${error.message}`);
        }

        console.warn(`Attempt ${attempt} failed, waiting ${delay}ms`);

        // Wait before retry
        await new Promise((resolve) => setTimeout(resolve, delay));

        // Exponential backoff: 1s, 2s, 4s, 8s, 16s
        delay *= 2;
      }
    }
  }

  // Service Worker: Retry logic
  async handleSyncWithRetry(tag, maxRetries = 3) {
    try {
      await this.syncWithExponentialBackoff(
        async () => {
          const data = await this.getQueuedData(tag);
          const response = await fetch('/api/sync', {
            method: 'POST',
            body: JSON.stringify(data)
          });

          if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
          }
        },
        maxRetries
      );
    } catch (error) {
      console.error('Sync permanently failed:', error);

      // Notify user
      const clients = await self.clients.matchAll();
      clients.forEach((client) => {
        client.postMessage({
          type: 'SYNC_ERROR',
          tag,
          message: error.message
        });
      });

      throw error; // Let browser retry
    }
  }

  async getQueuedData(tag) {
    const db = await this.openDB();
    return db.getAll(`queue-${tag}`);
  }

  async openDB() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open('sync-db', 1);
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
}

// Classify errors
function classifyError(error) {
  if (error.name === 'NetworkError') {
    return 'NETWORK_ERROR'; // Retry-able
  }

  if (error.response?.status === 429) {
    return 'RATE_LIMITED'; // Retry with backoff
  }

  if (error.response?.status === 400) {
    return 'BAD_REQUEST'; // Don't retry
  }

  if (error.response?.status === 500) {
    return 'SERVER_ERROR'; // Retry
  }

  return 'UNKNOWN';
}
```

---

### Question 9: How do you ensure data consistency with Background Sync?

**Answer:**

Data consistency is critical when syncing:

```javascript
// Ensure data consistency
class ConsistentSync {
  // 1. Version tracking
  async saveWithVersion(data) {
    data.version = this.generateVersion();
    data.lastModified = new Date().toISOString();
    await this.saveLocal(data);
    return data;
  }

  // 2. Conflict resolution
  async resolveConflict(local, remote) {
    // Last-write-wins strategy
    if (new Date(local.lastModified) > new Date(remote.lastModified)) {
      return local;
    }
    return remote;
  }

  // 3. Idempotent operations
  async syncWithIdempotency(data) {
    // Include unique sync ID
    const syncId = `${data.id}-${data.version}`;

    const response = await fetch('/api/sync', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-Idempotency-Key': syncId // Server prevents duplicates
      },
      body: JSON.stringify(data)
    });

    return response.json();
  }

  // 4. Checksum validation
  async syncWithChecksum(data) {
    const checksum = this.calculateChecksum(data);

    const response = await fetch('/api/sync', {
      method: 'POST',
      body: JSON.stringify({
        data,
        checksum
      })
    });

    const result = await response.json();

    // Verify server received correctly
    if (this.calculateChecksum(result.data) !== result.checksum) {
      throw new Error('Data corruption detected');
    }

    return result;
  }

  calculateChecksum(data) {
    // Simple checksum (use crypto for production)
    return JSON.stringify(data)
      .split('')
      .reduce((sum, char) => sum + char.charCodeAt(0), 0);
  }

  generateVersion() {
    return `${Date.now()}-${Math.random()}`;
  }

  async saveLocal(data) {
    // Implementation
  }
}
```

---

### Question 10: What's the relationship between Background Sync and Web Workers?

**Answer:**

Background Sync and Web Workers are complementary technologies:

```javascript
// Relationship between Background Sync and Web Workers

const relationship = {
  backgroundSync: {
    purpose: 'Schedule sync tasks to run even when app is closed',
    runsIn: 'Service Worker',
    triggeredBy: 'Network connectivity changes, time intervals',
    scope: 'Requires service worker registration'
  },

  webWorkers: {
    purpose: 'Run heavy computations without blocking UI',
    runsIn: 'Separate thread',
    triggeredBy: 'Application code',
    scope: 'Only while application is open'
  }
};

// Combined usage example
class OptimizedSyncApp {
  // Offload heavy processing to Web Worker
  processLargeDataset(data) {
    return new Promise((resolve, reject) => {
      const worker = new Worker('/workers/process.js');

      worker.postMessage({ data });

      worker.onmessage = (event) => {
        resolve(event.data);
        worker.terminate();
      };

      worker.onerror = reject;
    });
  }

  // Use Background Sync for offline queueing
  async handleOfflineAction(data) {
    // Save locally
    await this.saveLocal(data);

    // Process in background
    const processed = await this.processLargeDataset(data);

    // Queue for sync
    await this.queueForSync(processed);

    // Register background sync
    const registration = await navigator.serviceWorker.ready;
    await registration.sync.register('sync-processed-data');
  }

  // Service Worker can also use workers for heavy processing
  // but Web Workers are typically used for foreground work
  async saveLocal(data) {
    // Implementation
  }

  async queueForSync(data) {
    // Implementation
  }
}

// Main difference in practice
const practicalDifference = {
  useWebWorker: 'When you need to process data while user is using the app',
  useBackgroundSync: 'When you need to sync data after app closes',
  useBoth: 'Process data in Web Worker, then sync via Background Sync'
};
```

---

## Summary

Background Sync is a powerful PWA feature that ensures reliable data synchronization across connectivity changes. Key takeaways:

**Core Concepts:**
- One-time sync for immediate actions (forms, messages)
- Periodic sync for recurring tasks (analytics, cache updates)
- Tag-based organization for multiple sync tasks
- Graceful degradation for unsupported browsers

**Implementation:**
- Save data locally first (offline-first architecture)
- Try to sync immediately
- Fall back to Background Sync if offline
- Handle retries and failures robustly

**Best Practices:**
- Always save locally before attempting sync
- Use appropriate sync tags for organization
- Implement proper error handling and retries
- Communicate sync status to users
- Ensure data consistency with versioning and checksums
- Test thoroughly in offline scenarios

**Limitations:**
- Limited browser support (Chrome/Edge)
- System-managed timing (no guarantees)
- Service worker execution time limits
- Can't return data to main thread

Remember: Background Sync makes PWAs truly reliable by ensuring data is never lost, even in unreliable network conditions.

---

**Next:** [Push Notifications í](./05-push-notifications.md)

**Previous:** [ê Offline Patterns](./03-offline-patterns.md)

---

[ê Back to PWA](./README.md) | [ë Back to Frontend](../README.md)
