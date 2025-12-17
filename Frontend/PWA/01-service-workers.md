# Service Workers

## Overview

Service Workers are powerful browser APIs that act as intermediaries between web applications and the network. They enable offline functionality, background synchronization, and push notifications. Understanding service workers is fundamental to building Progressive Web Apps and is a critical interview topic.

## Table of Contents
- [What are Service Workers?](#what-are-service-workers)
- [Service Worker Lifecycle](#service-worker-lifecycle)
- [Registration Patterns](#registration-patterns)
- [Scope and Lifecycle Events](#scope-and-lifecycle-events)
- [Fetch Events & Request Interception](#fetch-events--request-interception)
- [Caching Strategies Overview](#caching-strategies-overview)
- [Service Worker Updates](#service-worker-updates)
- [Skip Waiting & Clients Claim](#skip-waiting--clients-claim)
- [Message Passing](#message-passing)
- [Service Worker Debugging](#service-worker-debugging)
- [Common Patterns](#common-patterns)
- [Interview Questions](#interview-questions)

---

## What are Service Workers?

### Key Concepts

Service Workers are JavaScript files that run in the background, separate from the main thread, acting as a proxy between the web app and the network.

```javascript
// Service workers are NOT:
// - DOM accessible (no document, window in global scope)
// - Available in private browsing in some browsers
// - Available on insecure connections (HTTPS required)

// Service workers ARE:
// - Network proxies
// - Offline-capable
// - Persistently cached
// - Able to handle push notifications
// - Able to synchronize in background

const serviceWorkerCapabilities = {
  proxyNetwork: 'Intercept all fetch requests',
  offline: 'Work without network connection',
  cache: 'Store assets for instant loading',
  push: 'Receive push notifications',
  background: 'Sync data in background',
  notification: 'Display notifications to users'
};
```

### Why Service Workers Matter

```javascript
// Before Service Workers
// - No offline support
// - Network requests must succeed for app to work
// - Slow repeat visits (no caching)
// - No background operations

// With Service Workers
// - Full offline support
// - Works on slow networks (graceful degradation)
// - Lightning-fast repeat visits (cached)
// - Background sync and push notifications
// - Native app-like experience
```

---

## Service Worker Lifecycle

### The Four States

Service workers have a clear lifecycle with distinct states and events:

```javascript
// 1. REGISTRATION (in main.js)
navigator.serviceWorker.register('/sw.js')
  // “
  // 2. INSTALLATION (in sw.js)
  // “
  // 3. ACTIVATION (in sw.js)
  // “
  // 4. FETCH HANDLING & MESSAGE EVENTS
```

### Detailed Lifecycle Flow

```javascript
// MAIN THREAD (index.html or main.js)
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(registration => {
      console.log('SW registered:', registration);
      // registration.installing
      // registration.waiting
      // registration.active
    })
    .catch(error => {
      console.error('SW registration failed:', error);
    });
}

// SERVICE WORKER FILE (sw.js)

// STAGE 1: INSTALL EVENT
// Fires once when SW is first registered
// Perfect for precaching assets
self.addEventListener('install', event => {
  console.log('Service worker installing...');

  event.waitUntil(
    caches.open('v1').then(cache => {
      // Cache critical assets
      return cache.addAll([
        '/',
        '/index.html',
        '/style.css',
        '/script.js'
      ]);
    })
  );

  // Force activate immediately (skip waiting)
  // Without this, activation waits for old SW to be unused
  self.skipWaiting();
});

// STAGE 2: ACTIVATE EVENT
// Fires when SW becomes active
// Perfect for cleanup (delete old caches, etc.)
self.addEventListener('activate', event => {
  console.log('Service worker activating...');

  event.waitUntil(
    caches.keys().then(cacheNames => {
      // Delete old cache versions
      return Promise.all(
        cacheNames.map(cacheName => {
          if (cacheName !== 'v1') {
            console.log('Deleting old cache:', cacheName);
            return caches.delete(cacheName);
          }
        })
      );
    })
  );

  // Take control of all clients immediately
  return self.clients.claim();
});

// STAGE 3: FETCH EVENT (Continuous)
// Fires for every network request from the app
// Where caching strategies are implemented
self.addEventListener('fetch', event => {
  console.log('Fetch event:', event.request.url);

  event.respondWith(
    caches.match(event.request)
      .then(response => {
        // Serve from cache if available
        return response || fetch(event.request);
      })
      .catch(error => {
        // Offline fallback
        return new Response('Offline - Service Worker', {
          status: 503,
          statusText: 'Service Unavailable',
          headers: new Headers({
            'Content-Type': 'text/plain'
          })
        });
      })
  );
});

// STAGE 4: MESSAGE EVENT (Optional)
// Two-way communication with main thread
self.addEventListener('message', event => {
  if (event.data.type === 'SKIP_WAITING') {
    self.skipWaiting();
  }
});
```

---

## Registration Patterns

### Basic Registration

```javascript
// Simple registration
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');
}
```

### Registration with Error Handling

```javascript
// Production-ready registration
async function registerServiceWorker() {
  if (!('serviceWorker' in navigator)) {
    console.log('Service Workers not supported');
    return;
  }

  try {
    const registration = await navigator.serviceWorker.register('/sw.js');
    console.log('SW registered successfully:', registration);
    return registration;
  } catch (error) {
    console.error('SW registration failed:', error);
  }
}

registerServiceWorker();
```

### Registration with Scope

```javascript
// Scope limits which pages the SW controls
// Only pages under /app/ will be controlled
navigator.serviceWorker.register('/sw.js', {
  scope: '/app/'
});

// Multiple SWs with different scopes
navigator.serviceWorker.register('/sw-checkout.js', {
  scope: '/checkout/'
});

navigator.serviceWorker.register('/sw-admin.js', {
  scope: '/admin/'
});
```

### Checking Registration State

```javascript
// Check if already registered
async function checkServiceWorker() {
  if (!('serviceWorker' in navigator)) return;

  const registration = await navigator.serviceWorker.getRegistration();

  if (registration) {
    console.log('Already registered:', registration);

    // Check different states
    console.log('Installing:', registration.installing);
    console.log('Waiting:', registration.waiting);
    console.log('Active:', registration.active);
  } else {
    console.log('Not registered yet');
  }
}
```

---

## Scope and Lifecycle Events

### Understanding Scope

```javascript
// Scope determines which URLs the SW controls
// Rule: SW can only control pages under its own directory

// /app/sw.js can control:
//   /app/
//   /app/page.html
//   /app/components/
//   /app/components/comp.html
// But NOT:
//   /
//   /other/page.html
//   /otherapp/

// Override with explicit scope:
navigator.serviceWorker.register('/app/sw.js', {
  scope: '/' // Now controls whole site
});

// Multiple SWs cannot overlap scopes
// This will error:
navigator.serviceWorker.register('/app/sw1.js', { scope: '/app/' });
navigator.serviceWorker.register('/app/sw2.js', { scope: '/app/' }); // Error!
```

### Lifecycle Events (In-Depth)

```javascript
// INSTALL EVENT - Runs once per new SW version
self.addEventListener('install', event => {
  console.log('Installing SW...');

  event.waitUntil(
    // Wait for promise to settle before considering install complete
    new Promise((resolve, reject) => {
      // Precache critical assets
      caches.open('app-v1')
        .then(cache => cache.addAll(['/index.html', '/app.js']))
        .then(resolve)
        .catch(reject);
    })
  );

  // Optional: Skip waiting to activate immediately
  self.skipWaiting();
});

// ACTIVATE EVENT - Runs when SW becomes active
self.addEventListener('activate', event => {
  console.log('Activating SW...');

  event.waitUntil(
    // Cleanup old caches
    caches.keys().then(names => {
      return Promise.all(
        names
          .filter(name => name !== 'app-v1')
          .map(name => caches.delete(name))
      );
    })
  );

  // Take control immediately of all pages in scope
  return self.clients.claim();
});

// FETCH EVENT - Intercept all requests
self.addEventListener('fetch', event => {
  // Not all requests go through fetch event
  // Excluded: main_frame, sub_frame navigations (sometimes)

  if (event.request.method !== 'GET') {
    return; // Only handle GET requests
  }

  event.respondWith(/* ... */);
});

// MESSAGE EVENT - Receive messages from pages
self.addEventListener('message', event => {
  // event.data = whatever was sent from page
  // event.ports = communication channel back

  if (event.data.type === 'SKIP_WAITING') {
    self.skipWaiting();
  }
});

// SYNC EVENT - Background Sync (advanced)
self.addEventListener('sync', event => {
  if (event.tag === 'sync-comments') {
    event.waitUntil(syncComments());
  }
});

// PUSH EVENT - Push Notifications
self.addEventListener('push', event => {
  const data = event.data.json();
  self.registration.showNotification(data.title, {
    body: data.body,
    icon: '/icon.png'
  });
});
```

---

## Fetch Events & Request Interception

### Intercepting Requests

```javascript
// All fetch requests in controlled pages trigger this event
self.addEventListener('fetch', event => {
  const url = new URL(event.request.url);

  // Log all requests
  console.log('Fetch:', url.pathname);

  // Different handling for different types
  if (url.pathname.startsWith('/api/')) {
    // API requests - network first
    event.respondWith(networkFirst(event.request));
  } else if (event.request.destination === 'image') {
    // Images - cache first
    event.respondWith(cacheFirst(event.request));
  } else if (event.request.destination === 'font') {
    // Fonts - cache first, long term
    event.respondWith(cacheLongTerm(event.request));
  } else {
    // HTML pages - stale-while-revalidate
    event.respondWith(staleWhileRevalidate(event.request));
  }
});

// Helper functions
async function cacheFirst(request) {
  const cache = await caches.open('cache-v1');
  const cached = await cache.match(request);
  if (cached) return cached;

  try {
    const response = await fetch(request);
    cache.put(request, response.clone());
    return response;
  } catch (error) {
    return new Response('Offline', { status: 503 });
  }
}

async function networkFirst(request) {
  try {
    const response = await fetch(request);
    const cache = await caches.open('cache-v1');
    cache.put(request, response.clone());
    return response;
  } catch (error) {
    const cached = await caches.match(request);
    if (cached) return cached;
    return new Response('Offline', { status: 503 });
  }
}

async function staleWhileRevalidate(request) {
  const cache = await caches.open('cache-v1');
  const cached = await cache.match(request);

  const fetchPromise = fetch(request).then(response => {
    cache.put(request, response.clone());
    return response;
  });

  return cached || fetchPromise;
}

async function cacheLongTerm(request) {
  const cache = await caches.open('cache-long-v1');
  const cached = await cache.match(request);
  if (cached) return cached;

  const response = await fetch(request);
  cache.put(request, response.clone());
  return response;
}
```

### Modifying Requests

```javascript
// Intercept and modify requests
self.addEventListener('fetch', event => {
  // Add authentication header
  const request = event.request;
  const modifiedRequest = new Request(request, {
    headers: new Headers(request.headers),
    // Add custom headers
    headers: {
      ...request.headers,
      'X-SW-Request': 'true'
    }
  });

  event.respondWith(fetch(modifiedRequest));
});
```

---

## Caching Strategies Overview

### Five Major Strategies

```javascript
// 1. CACHE FIRST
// Use cache, fallback to network
// Best for: Static assets, images, fonts
self.addEventListener('fetch', event => {
  if (event.request.destination === 'style' ||
      event.request.destination === 'script') {
    event.respondWith(
      caches.match(event.request)
        .then(response => response || fetch(event.request))
    );
  }
});

// 2. NETWORK FIRST
// Try network, fall back to cache
// Best for: API calls, fresh content
self.addEventListener('fetch', event => {
  if (event.request.url.includes('/api/')) {
    event.respondWith(
      fetch(event.request)
        .then(response => {
          caches.open('api-v1').then(cache => {
            cache.put(event.request, response.clone());
          });
          return response;
        })
        .catch(() => caches.match(event.request))
    );
  }
});

// 3. STALE-WHILE-REVALIDATE
// Serve from cache, update in background
// Best for: Images, non-critical data
self.addEventListener('fetch', event => {
  if (event.request.destination === 'image') {
    event.respondWith(
      caches.match(event.request)
        .then(cached => {
          const fetchPromise = fetch(event.request)
            .then(response => {
              caches.open('images-v1').then(cache => {
                cache.put(event.request, response.clone());
              });
              return response;
            });
          return cached || fetchPromise;
        })
    );
  }
});

// 4. CACHE ONLY
// Never go to network
// Best for: Offline-first apps with no updates
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => response || new Response('Not cached', { status: 404 }))
  );
});

// 5. NETWORK ONLY
// Never use cache
// Best for: Real-time data
self.addEventListener('fetch', event => {
  event.respondWith(fetch(event.request));
});
```

---

## Service Worker Updates

### The Update Flow

```javascript
// How service workers check for updates
// 1. Browser compares new SW with registered SW
// 2. If different, downloads and installs new version
// 3. New SW enters "waiting" state if old SW still has clients
// 4. Activate when old SW clients are gone

// In main.js - Check for updates periodically
navigator.serviceWorker.addEventListener('controllerchange', () => {
  // New SW took control - reload page
  console.log('New SW taking control');
  window.location.reload();
});

// In sw.js - Notify users of update
self.addEventListener('activate', event => {
  event.waitUntil(
    self.clients.matchAll().then(clients => {
      clients.forEach(client => {
        client.postMessage({
          type: 'SW_UPDATED',
          message: 'New version available!'
        });
      });
    })
  );
});
```

### Update Strategies

```javascript
// Strategy 1: Immediate Update (aggressive)
self.addEventListener('install', event => {
  self.skipWaiting(); // Don't wait for clients to close
});

self.addEventListener('activate', event => {
  return self.clients.claim(); // Take control immediately
});

// In main.js
navigator.serviceWorker.addEventListener('controllerchange', () => {
  window.location.reload();
});

// Strategy 2: User-Initiated Update
// In main.js - notify user of update
navigator.serviceWorker.addEventListener('message', event => {
  if (event.data.type === 'UPDATE_AVAILABLE') {
    const updateBtn = document.getElementById('update-button');
    updateBtn.style.display = 'block';
    updateBtn.addEventListener('click', () => {
      // Tell SW to activate
      navigator.serviceWorker.controller.postMessage({
        type: 'SKIP_WAITING'
      });
    });
  }
});

// In sw.js
self.addEventListener('install', event => {
  self.skipWaiting(); // Don't wait
});

self.addEventListener('message', event => {
  if (event.data.type === 'SKIP_WAITING') {
    self.skipWaiting();
  }
});

// Notify about update
self.addEventListener('activate', event => {
  event.waitUntil(
    self.clients.matchAll().then(clients => {
      clients.forEach(client => {
        client.postMessage({ type: 'UPDATE_AVAILABLE' });
      });
    })
  );
});

// Strategy 3: Delayed Update (conservative)
// Don't skip waiting, let old SW finish before updating
self.addEventListener('install', event => {
  // Don't call skipWaiting()
  // Just install and wait
});

// Users can manually trigger update
const updateServiceWorker = async () => {
  const reg = await navigator.serviceWorker.getRegistration();
  await reg.update();
};
```

---

## Skip Waiting & Clients Claim

### skipWaiting()

```javascript
// skipWaiting tells browser: "Activate me immediately, don't wait"
self.addEventListener('install', event => {
  // Without skipWaiting:
  // New SW waits in "waiting" state until user closes all tabs

  // With skipWaiting:
  // New SW activates immediately
  self.skipWaiting();
});

// Use case: Aggressive updates
// Downside: Users with old SW might see inconsistencies
```

### clients.claim()

```javascript
// clients.claim tells browser: "Take control of all existing pages"
self.addEventListener('activate', event => {
  event.waitUntil(
    self.clients.claim()
    // Now this new SW is the controller
    // Pages don't need to reload to be controlled
  );
});

// Without claim:
// New page loads = controlled by new SW
// Old pages still = controlled by old SW (until reload)

// With claim:
// All pages immediately = controlled by new SW
```

### Combined Pattern

```javascript
// The "immediate activation" pattern
self.addEventListener('install', event => {
  console.log('SW installing...');
  self.skipWaiting(); // Don't wait for old SW
});

self.addEventListener('activate', event => {
  console.log('SW activating...');
  event.waitUntil(
    caches.keys().then(names => {
      return Promise.all(
        names.map(name => {
          if (name !== 'cache-v2') {
            return caches.delete(name);
          }
        })
      );
    })
  );

  return self.clients.claim(); // Take control now
});

// In main.js - detect controller change
navigator.serviceWorker.addEventListener('controllerchange', () => {
  console.log('New SW took control');
  // Page is now controlled by new SW
  // Can reload if needed
});
```

---

## Message Passing

### Two-Way Communication

```javascript
// IN MAIN THREAD (main.js)
// Send message to Service Worker
function sendMessageToSW(message) {
  if (navigator.serviceWorker.controller) {
    navigator.serviceWorker.controller.postMessage(message);
  }
}

// Receive messages from Service Worker
navigator.serviceWorker.addEventListener('message', event => {
  console.log('Message from SW:', event.data);

  if (event.data.type === 'UPDATE_AVAILABLE') {
    console.log('Update available!');
    showUpdateNotification();
  }
});

// Send with reply
async function askServiceWorker(message) {
  if (!navigator.serviceWorker.controller) return;

  // Create message channel
  const messageChannel = new MessageChannel();

  navigator.serviceWorker.controller.postMessage(
    message,
    [messageChannel.port2]
  );

  return new Promise(resolve => {
    messageChannel.port1.onmessage = event => {
      resolve(event.data);
    };
  });
}

// Use it
const cacheSize = await askServiceWorker({
  type: 'GET_CACHE_SIZE'
});
console.log('Cache size:', cacheSize);

// IN SERVICE WORKER (sw.js)
self.addEventListener('message', event => {
  if (event.data.type === 'SKIP_WAITING') {
    self.skipWaiting();
  }

  if (event.data.type === 'GET_CACHE_SIZE') {
    // Reply with MessagePort
    caches.open('v1').then(cache => {
      cache.keys().then(requests => {
        event.ports[0].postMessage({
          size: requests.length
        });
      });
    });
  }
});
```

### Notifying Clients of Updates

```javascript
// sw.js - When new SW activates
self.addEventListener('activate', event => {
  event.waitUntil(
    self.clients.matchAll().then(clients => {
      clients.forEach(client => {
        client.postMessage({
          type: 'SW_UPDATED',
          version: '2.0.0'
        });
      });
    })
  );
});

// main.js - Listen for updates
navigator.serviceWorker.addEventListener('message', event => {
  if (event.data.type === 'SW_UPDATED') {
    showUpdateBanner(`New version ${event.data.version} available`);
  }
});
```

---

## Service Worker Debugging

### DevTools

```javascript
// Chrome DevTools > Application tab
// 1. Service Workers section
//    - Shows registered SWs and their status
//    - Can unregister, update, skip waiting
// 2. Cache Storage
//    - View cached responses
//    - Delete individual caches
// 3. Network tab
//    - Shows "(from ServiceWorker)" for cached responses
// 4. Console
//    - Logs from SW appear here
```

### Debugging Techniques

```javascript
// 1. Console logging
self.addEventListener('fetch', event => {
  console.log('Fetch:', event.request.url); // Appears in DevTools
});

// 2. Check if SW is registered
navigator.serviceWorker.getRegistration().then(reg => {
  console.log('Registered:', !!reg);
  console.log('Active:', !!reg?.active);
  console.log('Waiting:', !!reg?.waiting);
});

// 3. Get all registered SWs
navigator.serviceWorker.getRegistrations().then(regs => {
  console.log('All registrations:', regs);
});

// 4. Check controller
console.log('Controlled by SW:', !!navigator.serviceWorker.controller);

// 5. Add error event listener
navigator.serviceWorker.addEventListener('error', event => {
  console.error('SW error:', event.error);
});

// 6. Add controllerchange listener
navigator.serviceWorker.addEventListener('controllerchange', () => {
  console.log('Controller changed');
});

// 7. Debug cache operations
self.addEventListener('fetch', event => {
  caches.match(event.request).then(response => {
    console.log('Cache hit:', event.request.url, !!response);
  });
});

// 8. Monitor cache size
async function getCacheStats() {
  const cacheNames = await caches.keys();
  const stats = {};

  for (const name of cacheNames) {
    const cache = await caches.open(name);
    const requests = await cache.keys();
    stats[name] = requests.length;
  }

  console.table(stats);
}
```

---

## Common Patterns

### Offline Page Pattern

```javascript
// Register offline page during install
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('pages-v1').then(cache => {
      return cache.add('/offline.html');
    })
  );
});

// Serve offline page for failed navigation
self.addEventListener('fetch', event => {
  if (event.request.mode === 'navigate') { // Navigation request
    event.respondWith(
      fetch(event.request)
        .catch(() => caches.match('/offline.html'))
    );
  }
});
```

### Cache with Version Management

```javascript
const CACHE_VERSION = 'app-v1';

self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_VERSION).then(cache => {
      return cache.addAll([
        '/',
        '/index.html',
        '/styles.css',
        '/app.js'
      ]);
    })
  );
  self.skipWaiting();
});

self.addEventListener('activate', event => {
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(cacheName => {
          if (cacheName !== CACHE_VERSION) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
  return self.clients.claim();
});
```

### Precaching Pattern

```javascript
// Use importScripts to inject manifest
// (Usually done by build tools like Workbox)
const PRECACHE_MANIFEST = [
  { url: '/', revision: 'abc123' },
  { url: '/app.js', revision: 'def456' },
  { url: '/style.css', revision: 'ghi789' }
];

self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('precache-v1').then(cache => {
      return Promise.all(
        PRECACHE_MANIFEST.map(item => {
          const url = new URL(item.url, self.location);
          const request = new Request(url, { cache: 'reload' });
          return fetch(request).then(response => {
            return cache.put(item.url, response);
          });
        })
      );
    })
  );
});
```

---

## Interview Questions

### Question 1: What is a Service Worker and what are its main purposes?

**Answer:**

A Service Worker is a JavaScript file that runs in the background, separate from the main thread, acting as a proxy between the web app and the network. Its main purposes are:

1. **Enable Offline Functionality** - Cache assets and API responses for offline access
2. **Improve Performance** - Serve cached content instantly on repeat visits
3. **Background Sync** - Queue requests when offline and sync when online
4. **Push Notifications** - Receive and display push notifications
5. **Network Resilience** - Provide graceful degradation on slow networks

Key characteristics:
- Runs in separate thread (doesn't block main thread)
- Requires HTTPS (or localhost for development)
- Persistent (cached in browser, survives page reloads)
- Not DOM-accessible (no document, window)
- Can't access main thread directly

---

### Question 2: Explain the Service Worker lifecycle in detail.

**Answer:**

The Service Worker lifecycle has three main stages:

**1. Registration**
```javascript
navigator.serviceWorker.register('/sw.js');
```
Initiates the SW lifecycle.

**2. Installation (install event)**
```javascript
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('v1').then(cache => {
      return cache.addAll(['/index.html', '/style.css']);
    })
  );
});
```
Fires once per new SW version. Perfect for precaching assets. SW enters "installing" state, then "installed".

**3. Activation (activate event)**
```javascript
self.addEventListener('activate', event => {
  event.waitUntil(
    caches.keys().then(names => {
      return Promise.all(
        names.map(name => {
          if (name !== 'v1') return caches.delete(name);
        })
      );
    })
  );
  return self.clients.claim();
});
```
Fires when SW becomes active (usually when old SW is no longer controlling pages). Good for cleanup. SW enters "active" state.

**4. Fetch Handling (fetch event)**
```javascript
self.addEventListener('fetch', event => {
  event.respondWith(caches.match(event.request));
});
```
Fires for every network request from controlled pages. Where caching strategies run.

**State Flow:**
- Registered ’ Installing ’ Installed ’ Waiting (or Active)
- With skipWaiting: Registered ’ Installing ’ Activated
- With clients.claim: All pages controlled immediately

---

### Question 3: What's the difference between skipWaiting() and clients.claim()?

**Answer:**

**skipWaiting():**
- Prevents new SW from waiting in "waiting" state
- Activates immediately even if old SW still has clients
- Skips the period where new and old SW coexist
- Used in `install` event
- Aggressive update strategy

**clients.claim():**
- Takes control of existing pages without them reloading
- Used in `activate` event
- Without claim: new pages use new SW, old pages still use old SW
- With claim: all pages immediately controlled by new SW
- Used to sync state with old pages

Example showing the difference:
```javascript
// WITHOUT skipWaiting and claim
// User has old page open
// Update deployed
// New page load ’ uses new SW
// Old page still ’ uses old SW
// User refreshes old page ’ now uses new SW

// WITH skipWaiting and claim
// User has old page open
// Update deployed
// New SW activates immediately (skipWaiting)
// Old page now controlled by new SW (claim)
// Everything in sync, might reload page

// Combined: gives immediate update with immediate control
```

---

### Question 4: How do you implement and use the Cache First strategy?

**Answer:**

Cache First strategy: Check cache first, use network only if not cached. Best for static assets.

```javascript
// During install - precache assets
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('static-v1').then(cache => {
      return cache.addAll([
        '/index.html',
        '/style.css',
        '/app.js',
        '/image.png'
      ]);
    })
  );
});

// During fetch - serve from cache
self.addEventListener('fetch', event => {
  // Only cache GET requests
  if (event.request.method !== 'GET') {
    return;
  }

  // Cache first strategy
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        if (response) {
          console.log('Serving from cache:', event.request.url);
          return response;
        }

        // Not in cache, fetch from network
        return fetch(event.request).then(response => {
          // Don't cache error responses
          if (!response || response.status !== 200) {
            return response;
          }

          // Cache successful response
          const responseToCache = response.clone();
          caches.open('static-v1').then(cache => {
            cache.put(event.request, responseToCache);
          });

          return response;
        });
      })
      .catch(() => {
        // Offline and not in cache
        return new Response('Offline', {
          status: 503,
          statusText: 'Service Unavailable'
        });
      })
  );
});

// Update cache strategy
self.addEventListener('activate', event => {
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(cacheName => {
          if (cacheName !== 'static-v1') {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

**When to use:**
- Static assets (CSS, JS, images)
- Fonts
- Rarely-changing content
- Offline-first applications

---

### Question 5: How do you handle Service Worker updates and notify users?

**Answer:**

Service Workers update when the file changes. Here's how to handle it:

```javascript
// In main.js - Check for updates and notify user
let refreshing = false;

if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');

  // Listen for controller change (new SW took over)
  navigator.serviceWorker.addEventListener('controllerchange', () => {
    if (!refreshing) {
      refreshing = true;
      window.location.reload();
    }
  });

  // Listen for messages from SW
  navigator.serviceWorker.addEventListener('message', event => {
    if (event.data.type === 'UPDATE_AVAILABLE') {
      showUpdatePrompt();
    }
  });
}

function showUpdatePrompt() {
  const updateBanner = document.createElement('div');
  updateBanner.className = 'update-banner';
  updateBanner.innerHTML = `
    <p>New version available!</p>
    <button id="update-btn">Update Now</button>
  `;
  document.body.appendChild(updateBanner);

  document.getElementById('update-btn').addEventListener('click', () => {
    // Tell SW to skip waiting
    navigator.serviceWorker.controller.postMessage({
      type: 'SKIP_WAITING'
    });
  });
}

// In sw.js - Notify of updates
self.addEventListener('install', event => {
  console.log('New SW installing...');
  // Don't auto-skip, let user decide
});

self.addEventListener('message', event => {
  if (event.data.type === 'SKIP_WAITING') {
    self.skipWaiting();
  }
});

self.addEventListener('activate', event => {
  console.log('New SW activating...');

  // Tell all clients about update
  event.waitUntil(
    self.clients.matchAll().then(clients => {
      clients.forEach(client => {
        client.postMessage({
          type: 'UPDATE_AVAILABLE'
        });
      });
    })
  );

  return self.clients.claim();
});

// Or automatic update with minimal disruption
self.addEventListener('install', event => {
  self.skipWaiting(); // Auto-activate
});

self.addEventListener('activate', event => {
  return self.clients.claim(); // Take control immediately
});

navigator.serviceWorker.addEventListener('controllerchange', () => {
  window.location.reload(); // Transparent reload for user
});
```

---

### Question 6: How do you implement message passing between the page and Service Worker?

**Answer:**

Message passing enables two-way communication between main thread and Service Worker.

```javascript
// IN MAIN THREAD (main.js)
// Send message to SW
function sendMessageToSW(data) {
  if (navigator.serviceWorker.controller) {
    navigator.serviceWorker.controller.postMessage(data);
  }
}

// Usage
sendMessageToSW({
  type: 'GET_CACHE_SIZE',
  url: '/api/data'
});

// Receive replies
navigator.serviceWorker.addEventListener('message', event => {
  console.log('From SW:', event.data);
  if (event.data.type === 'CACHE_SIZE_RESPONSE') {
    console.log('Cache size:', event.data.size);
  }
});

// Two-way communication with reply
async function getDataFromSW(request) {
  return new Promise(resolve => {
    const channel = new MessageChannel();

    navigator.serviceWorker.controller.postMessage(
      { type: 'GET_DATA', url: '/api/data' },
      [channel.port2]
    );

    channel.port1.onmessage = event => {
      resolve(event.data);
    };
  });
}

const data = await getDataFromSW('/api/users');
console.log(data);

// IN SERVICE WORKER (sw.js)
self.addEventListener('message', event => {
  // Receive message from page
  if (event.data.type === 'GET_CACHE_SIZE') {
    // Calculate cache size
    caches.open('v1').then(cache => {
      cache.keys().then(requests => {
        // Send back with postMessage
        self.clients.matchAll().then(clients => {
          clients.forEach(client => {
            client.postMessage({
              type: 'CACHE_SIZE_RESPONSE',
              size: requests.length
            });
          });
        });
      });
    });
  }

  // Handle with MessagePort reply
  if (event.data.type === 'GET_DATA') {
    fetch(event.data.url)
      .then(res => res.json())
      .then(data => {
        // Reply through port
        event.ports[0].postMessage({
          success: true,
          data: data
        });
      })
      .catch(error => {
        event.ports[0].postMessage({
          success: false,
          error: error.message
        });
      });
  }
});
```

---

### Question 7: What happens if a Service Worker throws an error during install?

**Answer:**

If a SW throws an error during the `install` event:

1. The `install` event fails
2. The SW enters the "redundant" state
3. The old SW (if any) remains active and controlling pages
4. Next page load triggers another registration attempt
5. The failing SW is never activated

```javascript
// Problematic code - throws error
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('v1').then(cache => {
      // This might fail and cause entire install to fail
      return cache.addAll([
        '/non-existent-file.js' // Error!
      ]);
    })
  );
});

// Better approach - handle errors gracefully
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('v1').then(cache => {
      return cache.addAll([
        '/index.html',
        '/style.css'
      ]).catch(error => {
        console.error('Failed to cache:', error);
        // Don't throw - let install continue
      });
    })
  );

  self.skipWaiting(); // Still activate even if some caching failed
});

// Defensive pattern
const CRITICAL_ASSETS = ['/index.html', '/offline.html'];
const OPTIONAL_ASSETS = ['/image1.png', '/image2.png'];

self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('v1')
      .then(cache => cache.addAll(CRITICAL_ASSETS)) // Must succeed
      .then(cache => {
        // Optional assets
        return caches.open('v1')
          .then(cache => cache.addAll(OPTIONAL_ASSETS))
          .catch(() => console.log('Optional assets failed'));
      })
  );
});
```

---

### Question 8: How do you prevent a Service Worker from hijacking all requests?

**Answer:**

Service Workers intercept all requests by default. Here's how to be selective:

```javascript
self.addEventListener('fetch', event => {
  const url = new URL(event.request.url);
  const method = event.request.method;
  const mode = event.request.mode;

  // Skip non-GET requests
  if (method !== 'GET') {
    return; // Let network handle it
  }

  // Skip cross-origin requests (optional)
  if (url.origin !== self.location.origin) {
    return; // Let network handle it
  }

  // Skip navigation requests for different page
  if (mode === 'navigate' && !url.pathname.startsWith('/app')) {
    return; // Let network handle it
  }

  // Skip requests to specific APIs
  if (url.pathname.startsWith('/api/real-time')) {
    return; // Always fresh
  }

  // Skip large file downloads
  if (url.pathname.endsWith('.zip') || url.pathname.endsWith('.iso')) {
    return;
  }

  // Only handle requests we care about
  event.respondWith(handleRequest(event.request));
});

function handleRequest(request) {
  // Implement caching strategy
  if (request.destination === 'image') {
    return cacheFirst(request);
  } else if (request.url.includes('/api/')) {
    return networkFirst(request);
  } else {
    return staleWhileRevalidate(request);
  }
}
```

---

### Question 9: How do you test a Service Worker?

**Answer:**

Service Workers require testing in specific ways:

```javascript
// Unit testing with Jest
describe('Service Worker', () => {
  let scope;

  beforeEach(() => {
    // Global scope for SW
    scope = {
      addEventListener: jest.fn(),
      skipWaiting: jest.fn(),
      clients: {
        matchAll: jest.fn().mockResolvedValue([])
      },
      caches: {
        open: jest.fn().mockResolvedValue({
          addAll: jest.fn().mockResolvedValue(undefined)
        })
      }
    };

    global.self = scope;
  });

  test('installs and opens cache', async () => {
    require('./sw.js');

    const installHandler = scope.addEventListener.mock.calls
      .find(call => call[0] === 'install')[1];

    const event = {
      waitUntil: jest.fn(promise => promise)
    };

    await installHandler(event);

    expect(scope.caches.open).toHaveBeenCalledWith('v1');
    expect(scope.skipWaiting).toHaveBeenCalled();
  });
});

// Integration testing
// Use workbox-test-helpers or register real SW
describe('Service Worker Integration', () => {
  it('caches and serves files', async () => {
    // Use real SW file, not mocked
    const registration = await navigator.serviceWorker.register('/sw.js');

    // Wait for activation
    await new Promise(resolve => {
      registration.onupdatefound = () => {
        const installingWorker = registration.installing;
        installingWorker.onstatechange = () => {
          if (installingWorker.state === 'activated') {
            resolve();
          }
        };
      };
    });

    // Test caching
    const response = await fetch('/index.html');
    expect(response.ok).toBe(true);

    // Verify cached
    const cache = await caches.open('v1');
    const cached = await cache.match('/index.html');
    expect(cached).toBeDefined();
  });
});
```

---

### Question 10: What are the security considerations for Service Workers?

**Answer:**

Security is critical with Service Workers since they can intercept all requests:

```javascript
// 1. HTTPS Only
// Service Workers only work on HTTPS (or localhost)
// This prevents man-in-the-middle attacks

// 2. Same-Origin Policy
// SW can only be registered from same origin
// This prevents cross-site injection

navigator.serviceWorker.register('/sw.js');
// Works: same origin
// Error: navigator.serviceWorker.register('https://malicious.com/sw.js');

// 3. Scope Limitation
// SW can only control pages within its scope
navigator.serviceWorker.register('/sw.js', {
  scope: '/app/' // Only controls /app/
});

// 4. Validate Cache Content
// Don't blindly cache user-provided data
self.addEventListener('fetch', event => {
  const url = new URL(event.request.url);

  // Only cache same-origin requests
  if (url.origin !== self.location.origin) {
    return;
  }

  // Don't cache sensitive endpoints
  if (url.pathname.includes('/auth') || url.pathname.includes('/payment')) {
    return; // Always fetch fresh
  }

  event.respondWith(caches.match(event.request));
});

// 5. Validate and Sanitize Messages
self.addEventListener('message', event => {
  // Validate message origin
  if (event.origin !== 'https://example.com') {
    console.warn('Ignoring message from untrusted origin');
    return;
  }

  // Validate message format
  if (!event.data || !event.data.type) {
    console.warn('Invalid message format');
    return;
  }

  // Handle known message types only
  if (event.data.type === 'SKIP_WAITING') {
    self.skipWaiting();
  }
});

// 6. Cache Versioning
// Always version caches to ensure cleanup
const CACHE_VERSION = 'app-v1';
self.addEventListener('activate', event => {
  event.waitUntil(
    caches.keys().then(names => {
      return Promise.all(
        names.map(name => {
          if (name !== CACHE_VERSION) {
            return caches.delete(name); // Clean old caches
          }
        })
      );
    })
  );
});

// 7. Monitor Caching Size
// Prevent storage quota abuse
async function cleanupOldCache() {
  const cacheNames = await caches.keys();

  for (const name of cacheNames) {
    const cache = await caches.open(name);
    const requests = await cache.keys();

    // Keep only recent 100 items
    if (requests.length > 100) {
      for (let i = 0; i < requests.length - 100; i++) {
        await cache.delete(requests[i]);
      }
    }
  }
}
```

---

## Summary

Service Workers are the cornerstone of PWAs, enabling offline functionality, performance improvements, and app-like experiences. Key takeaways:

1. **Lifecycle** - Install ’ Activate ’ Fetch handling ’ Message events
2. **Caching Strategies** - Choose based on content type and freshness needs
3. **Updates** - Use skipWaiting and clients.claim for immediate updates
4. **Communication** - Message passing for two-way communication
5. **Security** - HTTPS required, same-origin policy, scope limitations

Master service workers to excel in PWA interviews and build modern web applications!

---

[ Back to PWA Guide](./README.md)
