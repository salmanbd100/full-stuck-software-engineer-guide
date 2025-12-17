# Push Notifications API

## Overview

Push Notifications are a critical PWA feature that enables real-time engagement between applications and users. The Push API allows web applications to receive messages from a server even when the application is not active. Combined with the Notification API, this creates a powerful system for delivering timely messages directly to users' devices, similar to native mobile applications.

## Table of Contents
- [Push API Fundamentals](#push-api-fundamentals)
- [Notification API](#notification-api)
- [VAPID Keys](#vapid-keys)
- [Implementation Flow](#implementation-flow)
- [Service Worker Push Events](#service-worker-push-events)
- [Notification Actions](#notification-actions)
- [Permission Handling](#permission-handling)
- [Server-Side Implementation](#server-side-implementation)
- [Client-Side Implementation](#client-side-implementation)
- [Testing Push Notifications](#testing-push-notifications)
- [Complete Examples](#complete-examples)
- [Interview Questions](#interview-questions)

## Push API Fundamentals

### Understanding Push Notifications

Push notifications involve three main components:

```javascript
// Three-party push notification system
const pushSystem = {
  client: {
    role: 'Subscribe to push messages',
    action: 'Request notification permission',
    responsibility: 'Handle incoming push events'
  },

  server: {
    role: 'Send push messages',
    action: 'Use Push Service API with VAPID',
    responsibility: 'Deliver messages to subscription endpoint'
  },

  pushService: {
    role: 'Relay messages from server to client',
    action: 'Managed by browser vendor (Firebase, etc.)',
    responsibility: 'Deliver message reliably to device'
  }
};

// Example flow
const pushFlow = {
  step1: 'Client: Request permission and subscribe',
  step2: 'Browser: Get subscription from Push Service',
  step3: 'Client: Send subscription to server',
  step4: 'Server: Store subscription endpoint',
  step5: 'Server: Send message to Push Service',
  step6: 'Push Service: Relay to device',
  step7: 'Browser: Fire push event in service worker',
  step8: 'Service Worker: Display notification'
};
```

### Push API vs Notification API

```javascript
// Key difference
const difference = {
  pushAPI: {
    purpose: 'Receive messages from server',
    triggered: 'By server sending message',
    works: 'Even when browser is closed',
    requires: 'Server and subscription',
    scope: 'Service Worker scope'
  },

  notificationAPI: {
    purpose: 'Display visual notifications',
    triggered: 'By application code',
    works: 'Only when browser has permission',
    requires: 'User permission',
    scope: 'Can be used in main thread or service worker'
  }
};

// Typical usage: Push API triggers Notification API
// Server sends message -> Push event fires -> Service worker displays notification
```

## Notification API

### Displaying Notifications

```javascript
// Basic notification
function showNotification(title, options) {
  if (!('serviceWorker' in navigator)) {
    console.warn('Service Workers not supported');
    return;
  }

  navigator.serviceWorker.ready.then((registration) => {
    registration.showNotification(title, options);
  });
}

// Notification with options
showNotification('New Message', {
  body: 'You have a new message from Alice',
  icon: '/images/notification-icon.png',
  badge: '/images/badge.png',
  tag: 'message-notification', // Group notifications
  requireInteraction: false, // Auto-close after a bit
  actions: [
    { action: 'open', title: 'Open' },
    { action: 'close', title: 'Close' }
  ],
  data: { userId: 123, messageId: 456 }
});

// Notification with different types
const notificationTypes = {
  info: {
    title: 'Info',
    icon: '/icons/info.png',
    tag: 'info'
  },

  success: {
    title: 'Success!',
    icon: '/icons/success.png',
    tag: 'success'
  },

  warning: {
    title: 'Warning',
    icon: '/icons/warning.png',
    tag: 'warning',
    requireInteraction: true
  },

  error: {
    title: 'Error',
    icon: '/icons/error.png',
    tag: 'error',
    requireInteraction: true
  }
};
```

### Notification Events

```javascript
// Service Worker: Listen to notification events
self.addEventListener('push', (event) => {
  const options = {
    body: event.data ? event.data.text() : 'New message',
    icon: '/icon.png'
  };

  event.waitUntil(
    self.registration.showNotification('App Name', options)
  );
});

// Service Worker: Handle notification clicks
self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  const urlToOpen = '/'; // Default page

  event.waitUntil(
    clients.matchAll({
      type: 'window',
      includeUncontrolled: true
    }).then((clientList) => {
      // Check if app is already open
      for (let i = 0; i < clientList.length; i++) {
        const client = clientList[i];
        if (client.url === urlToOpen && 'focus' in client) {
          return client.focus();
        }
      }

      // App not open, open new window
      if (clients.openWindow) {
        return clients.openWindow(urlToOpen);
      }
    })
  );
});

// Service Worker: Handle notification close/dismiss
self.addEventListener('notificationclose', (event) => {
  console.log('Notification was closed:', event.notification.tag);

  // Track user dismissed notification
  const data = {
    type: 'notification-dismissed',
    tag: event.notification.tag
  };

  // Could send to server for analytics
});
```

## VAPID Keys

VAPID (Voluntary Application Server Identification) keys are essential for authenticated push messaging.

### Generating VAPID Keys

```javascript
// Using web-push library (Node.js)
const webpush = require('web-push');

// Generate keys
const vapidKeys = webpush.generateVAPIDKeys();

console.log('Public Key:', vapidKeys.publicKey);
console.log('Private Key:', vapidKeys.privateKey);

// Store private key securely on server
// Share public key with client
```

### Server Configuration

```javascript
// Server-side: Configure web-push with VAPID keys
const webpush = require('web-push');

webpush.setVapidDetails(
  'mailto:admin@example.com',
  process.env.VAPID_PUBLIC_KEY,
  process.env.VAPID_PRIVATE_KEY
);

// Now ready to send push notifications
```

### Client-Side VAPID Usage

```javascript
// Client-side: Get public key and subscribe
async function subscribeToPush() {
  // Get the service worker registration
  const registration = await navigator.serviceWorker.ready;

  // Get VAPID public key from server
  const response = await fetch('/api/vapid-public-key');
  const { publicKey } = await response.json();

  // Convert key to Uint8Array
  const convertedVapidKey = urlBase64ToUint8Array(publicKey);

  // Subscribe with public key
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: convertedVapidKey
  });

  return subscription;
}

// Helper: Convert base64 to Uint8Array
function urlBase64ToUint8Array(base64String) {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/\-/g, '+')
    .replace(/_/g, '/');

  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);

  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }

  return outputArray;
}
```

### VAPID Key Security

```javascript
// IMPORTANT: Never expose private key to client

// Bad: Private key in client code
const BAD_EXAMPLE = {
  publicKey: 'BPXYZ...',
  privateKey: 'secret_key_here' // NEVER DO THIS
};

// Good: Private key only on server
const GOOD_EXAMPLE = {
  // Client only gets public key
  publicKey: 'BPXYZ...'
  // Server keeps private key in environment variable
};

// Securely manage VAPID keys
const keyManagement = {
  storage: 'Environment variables (.env file)',
  access: 'Only server-side code',
  rotation: 'Every 90 days',
  backup: 'Store securely in vault'
};
```

## Implementation Flow

### Complete Subscription Flow

```javascript
// 1. Request permission
async function requestNotificationPermission() {
  if (!('serviceWorker' in navigator)) {
    console.warn('Service Workers not supported');
    return false;
  }

  if (Notification.permission === 'granted') {
    console.log('Notification permission already granted');
    return true;
  }

  if (Notification.permission === 'denied') {
    console.warn('Notification permission denied');
    return false;
  }

  // Request permission from user
  const permission = await Notification.requestPermission();
  return permission === 'granted';
}

// 2. Subscribe to push
async function subscribeToPush() {
  try {
    // Ensure service worker is ready
    const registration = await navigator.serviceWorker.ready;

    // Check if already subscribed
    let subscription = await registration.pushManager.getSubscription();

    if (!subscription) {
      // Get VAPID public key from server
      const response = await fetch('/api/vapid-public-key');
      const { publicKey } = await response.json();

      // Subscribe
      subscription = await registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: urlBase64ToUint8Array(publicKey)
      });

      console.log('Subscribed to push notifications');
    }

    return subscription;
  } catch (error) {
    console.error('Failed to subscribe:', error);
    throw error;
  }
}

// 3. Send subscription to server
async function sendSubscriptionToServer(subscription) {
  try {
    const response = await fetch('/api/notifications/subscribe', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        subscription: subscription.toJSON()
      })
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    console.log('Subscription sent to server');
  } catch (error) {
    console.error('Failed to send subscription:', error);
    throw error;
  }
}

// 4. Complete flow
async function setupPushNotifications() {
  try {
    // Request permission
    const hasPermission = await requestNotificationPermission();
    if (!hasPermission) return;

    // Subscribe to push
    const subscription = await subscribeToPush();

    // Send to server
    await sendSubscriptionToServer(subscription);

    console.log('Push notifications setup complete');
  } catch (error) {
    console.error('Setup failed:', error);
  }
}

// Call on app startup
document.addEventListener('DOMContentLoaded', setupPushNotifications);
```

## Service Worker Push Events

### Handling Push Events

```javascript
// Service Worker: Handle incoming push messages
self.addEventListener('push', (event) => {
  console.log('Push event received');

  let data = {};

  // Push events can have data
  if (event.data) {
    try {
      data = event.data.json();
    } catch (error) {
      console.error('Failed to parse push data:', error);
      data = { body: event.data.text() };
    }
  }

  const options = {
    body: data.body || 'New notification',
    icon: data.icon || '/icon.png',
    badge: data.badge || '/badge.png',
    tag: data.tag || 'notification',
    requireInteraction: data.requireInteraction || false,
    actions: data.actions || [],
    data: data.data || {}
  };

  // Show notification
  event.waitUntil(
    self.registration.showNotification(
      data.title || 'App Name',
      options
    )
  );
});

// Handle different types of push messages
self.addEventListener('push', (event) => {
  if (!event.data) {
    console.log('Empty push message');
    return;
  }

  const data = event.data.json();

  // Route based on message type
  switch (data.type) {
    case 'chat':
      event.waitUntil(handleChatNotification(data));
      break;

    case 'alert':
      event.waitUntil(handleAlertNotification(data));
      break;

    case 'update':
      event.waitUntil(handleUpdateNotification(data));
      break;

    default:
      event.waitUntil(handleDefaultNotification(data));
  }
});

async function handleChatNotification(data) {
  const options = {
    body: data.message,
    icon: data.senderPhoto,
    tag: `chat-${data.senderId}`, // Group by sender
    requireInteraction: true,
    data: {
      conversationId: data.conversationId
    }
  };

  return self.registration.showNotification(
    `Message from ${data.senderName}`,
    options
  );
}

async function handleAlertNotification(data) {
  const options = {
    body: data.message,
    icon: '/icons/alert.png',
    badge: '/icons/alert-badge.png',
    tag: 'alert',
    requireInteraction: true // Stay visible until user interacts
  };

  return self.registration.showNotification('Alert', options);
}

async function handleUpdateNotification(data) {
  const options = {
    body: data.version ? `Update available: ${data.version}` : 'Update available',
    icon: '/icons/update.png',
    tag: 'update',
    actions: [
      { action: 'update', title: 'Update' },
      { action: 'dismiss', title: 'Dismiss' }
    ]
  };

  return self.registration.showNotification('App Update', options);
}
```

## Notification Actions

### Handling Notification Actions

```javascript
// Service Worker: Handle notification action clicks
self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  if (event.action === 'open') {
    handleOpenAction(event.notification);
  } else if (event.action === 'close') {
    handleCloseAction(event.notification);
  } else if (event.action === 'reply') {
    handleReplyAction(event.notification);
  } else if (event.action === 'update') {
    handleUpdateAction(event.notification);
  } else {
    // Default click (anywhere on notification)
    handleDefaultAction(event.notification);
  }

  event.waitUntil(Promise.resolve());
});

async function handleOpenAction(notification) {
  const data = notification.data;

  const clients = await self.clients.matchAll({
    type: 'window'
  });

  // Focus existing window
  for (const client of clients) {
    if (client.url === '/' && 'focus' in client) {
      client.postMessage({
        type: 'NOTIFICATION_ACTION',
        action: 'open',
        data
      });
      return client.focus();
    }
  }

  // Open new window
  if (self.clients.openWindow) {
    const client = await self.clients.openWindow(
      `/?notification=${data.notificationId}`
    );
    if (client) {
      client.postMessage({
        type: 'NOTIFICATION_ACTION',
        action: 'open',
        data
      });
    }
  }
}

async function handleReplyAction(notification) {
  // For now, send to server
  await fetch('/api/notifications/reply', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      notificationId: notification.data.id,
      action: 'reply'
    })
  });
}

async function handleUpdateAction(notification) {
  // Trigger app update
  const clients = await self.clients.matchAll();

  clients.forEach((client) => {
    client.postMessage({
      type: 'UPDATE_AVAILABLE'
    });
  });
}
```

### Notification Actions Examples

```javascript
// Different notification setups with actions

// Chat notification with reply
const chatNotificationOptions = {
  body: 'Hey, are you there?',
  icon: '/icons/chat.png',
  tag: 'chat',
  requireInteraction: true,
  actions: [
    { action: 'reply', title: 'Reply' },
    { action: 'mute', title: 'Mute' }
  ],
  data: { conversationId: 123 }
};

// Alert notification with acknowledge
const alertNotificationOptions = {
  body: 'Your payment failed. Please retry.',
  icon: '/icons/alert.png',
  tag: 'alert',
  requireInteraction: true,
  actions: [
    { action: 'retry', title: 'Retry Payment' },
    { action: 'dismiss', title: 'Dismiss' }
  ],
  data: { transactionId: 456 }
};

// Update notification with actions
const updateNotificationOptions = {
  body: 'Version 2.0 is available',
  icon: '/icons/update.png',
  tag: 'update',
  actions: [
    { action: 'update', title: 'Update Now' },
    { action: 'later', title: 'Later' }
  ],
  data: { version: '2.0' }
};

// Approval notification
const approvalNotificationOptions = {
  body: 'Review pending request from John',
  icon: '/icons/approval.png',
  tag: 'approval',
  requireInteraction: true,
  actions: [
    { action: 'approve', title: 'Approve' },
    { action: 'reject', title: 'Reject' }
  ],
  data: { requestId: 789 }
};
```

## Permission Handling

### Request Permission Strategies

```javascript
// Different permission request approaches

// 1. Immediate request (may be rejected)
async function requestPermissionImmediately() {
  if (Notification.permission === 'default') {
    const permission = await Notification.requestPermission();
    return permission === 'granted';
  }
  return Notification.permission === 'granted';
}

// 2. Delayed request (better UX)
async function requestPermissionAfterEngagement() {
  // Wait for user interaction
  if (Notification.permission === 'default') {
    const permission = await Notification.requestPermission();
    return permission === 'granted';
  }
  return Notification.permission === 'granted';
}

// 3. Contextual request (best UX)
async function requestPermissionContextually() {
  if (Notification.permission !== 'default') {
    return Notification.permission === 'granted';
  }

  // Show explanation first
  const shouldRequest = await showPermissionExplanation();

  if (shouldRequest) {
    const permission = await Notification.requestPermission();
    return permission === 'granted';
  }

  return false;
}

function showPermissionExplanation() {
  return new Promise((resolve) => {
    const dialog = document.createElement('div');
    dialog.className = 'permission-dialog';
    dialog.innerHTML = `
      <div class="permission-content">
        <h2>Enable Notifications?</h2>
        <p>Get real-time updates about messages and events</p>
        <button class="allow">Allow</button>
        <button class="deny">No Thanks</button>
      </div>
    `;

    dialog.querySelector('.allow').addEventListener('click', () => {
      dialog.remove();
      resolve(true);
    });

    dialog.querySelector('.deny').addEventListener('click', () => {
      dialog.remove();
      resolve(false);
    });

    document.body.appendChild(dialog);
  });
}

// 4. Check permission status
function getPermissionStatus() {
  const status = {
    granted: Notification.permission === 'granted',
    denied: Notification.permission === 'denied',
    default: Notification.permission === 'default'
  };

  return status;
}

// 5. Handle permission denied
function handlePermissionDenied() {
  if (Notification.permission === 'denied') {
    console.warn('User denied notification permission');
    console.log('Show alternative: in-app notifications, email, etc.');
  }
}
```

### Permission UI Best Practices

```javascript
// Notification permission request flow

const permissionFlow = {
  step1: 'App detects permission is "default"',
  step2: 'Wait for user engagement (click, form submission)',
  step3: 'Show contextual explanation',
  step4: 'Request permission',
  step5: 'Handle granted or denied',
  step6: 'Remember user choice'
};

// Example: Request after first feature use
class NotificationPermissionManager {
  constructor() {
    this.featureUsageCount = 0;
  }

  async trackFeatureUsage() {
    this.featureUsageCount++;

    // Request after 3rd feature use
    if (this.featureUsageCount === 3) {
      await this.requestPermission();
    }
  }

  async requestPermission() {
    if (Notification.permission === 'default') {
      const explanation = `
        Get notified about new messages, updates, and events.
      `;

      const shouldRequest = await this.showOptInDialog(explanation);

      if (shouldRequest) {
        const permission = await Notification.requestPermission();
        this.handlePermissionResult(permission);
      }
    }
  }

  async showOptInDialog(message) {
    // Show modal dialog
    return new Promise((resolve) => {
      // ... show dialog and resolve with user choice
      resolve(true);
    });
  }

  handlePermissionResult(permission) {
    if (permission === 'granted') {
      console.log('User granted notification permission');
      this.setupNotifications();
    } else if (permission === 'denied') {
      console.log('User denied notification permission');
      this.setupAlternatives();
    }
  }

  setupNotifications() {
    // Subscribe to push
  }

  setupAlternatives() {
    // Setup email notifications, in-app alerts, etc.
  }
}
```

## Server-Side Implementation

### Node.js with web-push

```javascript
// server.js - Node.js server with Express

const express = require('express');
const webpush = require('web-push');
const bodyParser = require('body-parser');
require('dotenv').config();

const app = express();
app.use(bodyParser.json());

// Configure VAPID details
webpush.setVapidDetails(
  process.env.VAPID_SUBJECT || 'mailto:example@domain.com',
  process.env.VAPID_PUBLIC_KEY,
  process.env.VAPID_PRIVATE_KEY
);

// Database (use real DB in production)
const subscriptions = new Map();

// API: Get VAPID public key
app.get('/api/vapid-public-key', (req, res) => {
  res.json({
    publicKey: process.env.VAPID_PUBLIC_KEY
  });
});

// API: Subscribe to push
app.post('/api/notifications/subscribe', (req, res) => {
  const { subscription } = req.body;

  if (!subscription) {
    return res.status(400).json({ error: 'Missing subscription' });
  }

  try {
    // Store subscription
    const subscriptionId = `sub-${Date.now()}`;
    subscriptions.set(subscriptionId, subscription);

    console.log(`Stored subscription: ${subscriptionId}`);

    res.json({
      success: true,
      subscriptionId
    });
  } catch (error) {
    console.error('Failed to store subscription:', error);
    res.status(500).json({ error: 'Failed to store subscription' });
  }
});

// API: Send push notification
app.post('/api/notifications/send', async (req, res) => {
  const { message } = req.body;

  if (!message) {
    return res.status(400).json({ error: 'Missing message' });
  }

  try {
    const payload = JSON.stringify({
      title: message.title,
      body: message.body,
      icon: message.icon || '/icon.png',
      tag: message.tag,
      data: message.data || {}
    });

    const results = [];

    // Send to all subscribers
    for (const [id, subscription] of subscriptions) {
      try {
        await webpush.sendNotification(subscription, payload);
        results.push({ id, status: 'sent' });
      } catch (error) {
        if (error.statusCode === 410) {
          // Subscription expired, remove it
          subscriptions.delete(id);
          results.push({ id, status: 'removed' });
        } else {
          console.error(`Failed to send to ${id}:`, error);
          results.push({ id, status: 'failed', error: error.message });
        }
      }
    }

    res.json({
      success: true,
      results
    });
  } catch (error) {
    console.error('Send failed:', error);
    res.status(500).json({ error: 'Failed to send notification' });
  }
});

// API: Unsubscribe
app.post('/api/notifications/unsubscribe', (req, res) => {
  const { subscriptionId } = req.body;

  if (subscriptions.has(subscriptionId)) {
    subscriptions.delete(subscriptionId);
    res.json({ success: true });
  } else {
    res.status(404).json({ error: 'Subscription not found' });
  }
});

// Advanced: Send targeted notification
app.post('/api/notifications/send-targeted', async (req, res) => {
  const { userId, message } = req.body;

  // In production, look up user subscriptions from database
  const userSubscriptions = []; // subscriptions.filter(sub => sub.userId === userId)

  const payload = JSON.stringify(message);

  try {
    const results = await Promise.all(
      userSubscriptions.map((sub) => {
        return webpush.sendNotification(sub, payload).catch((error) => {
          if (error.statusCode === 410) {
            // Remove expired subscription
            // db.removeSubscription(sub.id)
          }
          return { error };
        });
      })
    );

    res.json({ success: true, sent: results.length });
  } catch (error) {
    console.error('Targeted send failed:', error);
    res.status(500).json({ error: 'Failed to send notification' });
  }
});

// Advanced: Broadcast notification
async function broadcastNotification(message) {
  const payload = JSON.stringify(message);

  for (const [id, subscription] of subscriptions) {
    try {
      await webpush.sendNotification(subscription, payload);
    } catch (error) {
      if (error.statusCode === 410) {
        subscriptions.delete(id);
      }
    }
  }
}

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Handling Subscription Expiry

```javascript
// Detect and handle expired subscriptions

// Check subscription status
async function isSubscriptionValid(subscription) {
  try {
    // Try to send a test notification
    await webpush.sendNotification(
      subscription,
      JSON.stringify({
        title: 'Test',
        body: 'Connection test'
      })
    );
    return true;
  } catch (error) {
    // 410 Gone = subscription expired
    return error.statusCode !== 410;
  }
}

// Maintain subscription list
const SubscriptionManager = {
  async cleanupExpiredSubscriptions() {
    for (const [id, subscription] of subscriptions) {
      const isValid = await isSubscriptionValid(subscription);

      if (!isValid) {
        console.log(`Removing expired subscription: ${id}`);
        subscriptions.delete(id);
      }
    }
  }
};

// Run cleanup periodically
setInterval(() => {
  SubscriptionManager.cleanupExpiredSubscriptions();
}, 24 * 60 * 60 * 1000); // Daily
```

## Client-Side Implementation

### React Component Example

```jsx
// React component for push notifications

import React, { useEffect, useState } from 'react';

function PushNotificationManager() {
  const [permission, setPermission] = useState(Notification.permission);
  const [subscribed, setSubscribed] = useState(false);

  useEffect(() => {
    checkSubscriptionStatus();
  }, []);

  async function checkSubscriptionStatus() {
    if (!('serviceWorker' in navigator)) return;

    const registration = await navigator.serviceWorker.ready;
    const subscription = await registration.pushManager.getSubscription();

    setSubscribed(!!subscription);
  }

  async function requestPermission() {
    if (Notification.permission === 'granted') {
      await handleSubscribe();
      return;
    }

    const result = await Notification.requestPermission();
    setPermission(result);

    if (result === 'granted') {
      await handleSubscribe();
    }
  }

  async function handleSubscribe() {
    try {
      const registration = await navigator.serviceWorker.ready;

      // Get VAPID key
      const response = await fetch('/api/vapid-public-key');
      const { publicKey } = await response.json();

      // Subscribe
      const subscription = await registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: urlBase64ToUint8Array(publicKey)
      });

      // Send to server
      await fetch('/api/notifications/subscribe', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          subscription: subscription.toJSON()
        })
      });

      setSubscribed(true);
    } catch (error) {
      console.error('Subscription failed:', error);
    }
  }

  async function handleUnsubscribe() {
    try {
      const registration = await navigator.serviceWorker.ready;
      const subscription = await registration.pushManager.getSubscription();

      if (subscription) {
        await subscription.unsubscribe();
        setSubscribed(false);
      }
    } catch (error) {
      console.error('Unsubscribe failed:', error);
    }
  }

  return (
    <div className="push-notification-manager">
      <h2>Notifications</h2>

      {permission === 'default' && (
        <button onClick={requestPermission}>
          Enable Notifications
        </button>
      )}

      {permission === 'granted' && !subscribed && (
        <button onClick={handleSubscribe}>
          Subscribe to Notifications
        </button>
      )}

      {subscribed && (
        <>
          <p>You are subscribed to notifications</p>
          <button onClick={handleUnsubscribe}>
            Unsubscribe
          </button>
        </>
      )}

      {permission === 'denied' && (
        <p>
          Notifications are disabled. Enable them in browser settings.
        </p>
      )}
    </div>
  );
}

function urlBase64ToUint8Array(base64String) {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/\-/g, '+')
    .replace(/_/g, '/');

  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);

  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }

  return outputArray;
}

export default PushNotificationManager;
```

### Vanilla JavaScript Implementation

```javascript
// Vanilla JS implementation without frameworks

class PushNotificationManager {
  constructor() {
    this.permission = Notification.permission;
    this.subscribed = false;
    this.init();
  }

  async init() {
    if (!('serviceWorker' in navigator)) {
      console.warn('Service Workers not supported');
      return;
    }

    await this.registerServiceWorker();
    await this.checkSubscriptionStatus();
    this.setupEventListeners();
  }

  async registerServiceWorker() {
    try {
      await navigator.serviceWorker.register('/service-worker.js');
    } catch (error) {
      console.error('Service Worker registration failed:', error);
    }
  }

  async checkSubscriptionStatus() {
    const registration = await navigator.serviceWorker.ready;
    const subscription = await registration.pushManager.getSubscription();
    this.subscribed = !!subscription;
    this.updateUI();
  }

  setupEventListeners() {
    document.getElementById('enable-btn')?.addEventListener('click', () => {
      this.requestPermission();
    });

    document.getElementById('subscribe-btn')?.addEventListener('click', () => {
      this.subscribe();
    });

    document.getElementById('unsubscribe-btn')?.addEventListener('click', () => {
      this.unsubscribe();
    });
  }

  async requestPermission() {
    if (this.permission === 'granted') {
      await this.subscribe();
      return;
    }

    this.permission = await Notification.requestPermission();

    if (this.permission === 'granted') {
      await this.subscribe();
    }

    this.updateUI();
  }

  async subscribe() {
    try {
      const registration = await navigator.serviceWorker.ready;

      const response = await fetch('/api/vapid-public-key');
      const { publicKey } = await response.json();

      const subscription = await registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: this.urlBase64ToUint8Array(publicKey)
      });

      await this.sendSubscriptionToServer(subscription);

      this.subscribed = true;
      this.updateUI();
    } catch (error) {
      console.error('Subscription failed:', error);
    }
  }

  async unsubscribe() {
    try {
      const registration = await navigator.serviceWorker.ready;
      const subscription = await registration.pushManager.getSubscription();

      if (subscription) {
        await subscription.unsubscribe();
        this.subscribed = false;
        this.updateUI();
      }
    } catch (error) {
      console.error('Unsubscribe failed:', error);
    }
  }

  async sendSubscriptionToServer(subscription) {
    const response = await fetch('/api/notifications/subscribe', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        subscription: subscription.toJSON()
      })
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
  }

  updateUI() {
    const enableBtn = document.getElementById('enable-btn');
    const subscribeBtn = document.getElementById('subscribe-btn');
    const unsubscribeBtn = document.getElementById('unsubscribe-btn');
    const status = document.getElementById('status');

    if (this.permission === 'granted' && this.subscribed) {
      enableBtn?.classList.add('hidden');
      subscribeBtn?.classList.add('hidden');
      unsubscribeBtn?.classList.remove('hidden');
      status.textContent = 'Subscribed to notifications';
    } else if (this.permission === 'granted') {
      enableBtn?.classList.add('hidden');
      subscribeBtn?.classList.remove('hidden');
      unsubscribeBtn?.classList.add('hidden');
      status.textContent = 'Permission granted, click subscribe';
    } else {
      enableBtn?.classList.remove('hidden');
      subscribeBtn?.classList.add('hidden');
      unsubscribeBtn?.classList.add('hidden');
      status.textContent = 'Enable notifications';
    }
  }

  urlBase64ToUint8Array(base64String) {
    const padding = '='.repeat((4 - base64String.length % 4) % 4);
    const base64 = (base64String + padding)
      .replace(/\-/g, '+')
      .replace(/_/g, '/');

    const rawData = window.atob(base64);
    const outputArray = new Uint8Array(rawData.length);

    for (let i = 0; i < rawData.length; ++i) {
      outputArray[i] = rawData.charCodeAt(i);
    }

    return outputArray;
  }
}

// Initialize
const notificationManager = new PushNotificationManager();
```

## Testing Push Notifications

### Manual Testing

```javascript
// Test push notifications in the browser

// 1. Setup
async function setupForTesting() {
  // Get registration
  const registration = await navigator.serviceWorker.ready;

  // Get subscription
  const subscription = await registration.pushManager.getSubscription();

  console.log('Subscription:', subscription);
  return subscription;
}

// 2. Send test notification from server
async function sendTestNotification(subscriptionEndpoint) {
  const response = await fetch('/api/test-notification', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      endpoint: subscriptionEndpoint
    })
  });

  return response.json();
}

// 3. Listen for incoming push
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.addEventListener('message', (event) => {
    console.log('Service Worker message:', event.data);
  });
}

// 4. Test notification display
function testNotificationDisplay() {
  navigator.serviceWorker.ready.then((registration) => {
    registration.showNotification('Test Notification', {
      body: 'This is a test',
      icon: '/icon.png',
      badge: '/badge.png',
      tag: 'test',
      requireInteraction: true,
      actions: [
        { action: 'test-action', title: 'Test Action' }
      ]
    });
  });
}

// Chrome DevTools testing
const chromeDevToolsTips = {
  'View push subscriptions': 'DevTools > Application > Service Workers',
  'Check notifications': 'DevTools > Application > Notifications',
  'View SW messages': 'DevTools > Application > Service Workers > Logs',
  'Test offline': 'DevTools > Network > Offline mode'
};
```

### Automated Testing

```javascript
// Jest/Vitest tests for push notifications

describe('Push Notifications', () => {
  let registration;

  beforeEach(async () => {
    // Mock service worker
    registration = {
      pushManager: {
        subscribe: jest.fn(),
        getSubscription: jest.fn()
      },
      showNotification: jest.fn()
    };

    Object.defineProperty(navigator, 'serviceWorker', {
      value: {
        ready: Promise.resolve(registration),
        register: jest.fn().mockResolvedValue(registration)
      },
      configurable: true
    });
  });

  test('requests notification permission', async () => {
    const permissionSpy = jest.spyOn(Notification, 'requestPermission');

    await Notification.requestPermission();

    expect(permissionSpy).toHaveBeenCalled();
  });

  test('subscribes to push notifications', async () => {
    const subscription = { toJSON: () => ({ endpoint: 'test' }) };
    registration.pushManager.subscribe.mockResolvedValue(subscription);

    const result = await registration.pushManager.subscribe({
      userVisibleOnly: true
    });

    expect(result).toBeDefined();
    expect(registration.pushManager.subscribe).toHaveBeenCalled();
  });

  test('handles push events', (done) => {
    const event = new Event('push');
    event.data = {
      text: () => 'Test message',
      json: () => ({ title: 'Test', body: 'Body' })
    };
    event.waitUntil = jest.fn((promise) => promise);

    // Simulate push event handling
    registration.showNotification('Test', { body: 'Body' });

    expect(registration.showNotification).toHaveBeenCalledWith(
      'Test',
      expect.objectContaining({ body: 'Body' })
    );

    done();
  });

  test('handles notification clicks', (done) => {
    const notification = { close: jest.fn() };
    const event = {
      notification,
      action: 'open',
      waitUntil: jest.fn((promise) => promise)
    };

    // Simulate notification click
    notification.close();

    expect(notification.close).toHaveBeenCalled();
    done();
  });
});
```

## Complete Examples

### Example 1: Chat Application with Push

```javascript
// chat-notifications.js

class ChatNotificationManager {
  constructor() {
    this.init();
  }

  async init() {
    if (!('serviceWorker' in navigator)) return;

    await this.registerServiceWorker();
    await this.setupPushNotifications();
    this.setupMessageListener();
  }

  async registerServiceWorker() {
    await navigator.serviceWorker.register('/chat-service-worker.js');
  }

  async setupPushNotifications() {
    const hasPermission = await this.requestPermission();
    if (hasPermission) {
      await this.subscribe();
    }
  }

  async requestPermission() {
    if (Notification.permission === 'granted') return true;

    if (Notification.permission === 'denied') return false;

    const permission = await Notification.requestPermission();
    return permission === 'granted';
  }

  async subscribe() {
    const registration = await navigator.serviceWorker.ready;
    const response = await fetch('/api/vapid-public-key');
    const { publicKey } = await response.json();

    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: this.urlBase64ToUint8Array(publicKey)
    });

    // Send subscription to server
    await fetch('/api/chat/subscribe-notifications', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ subscription: subscription.toJSON() })
    });
  }

  setupMessageListener() {
    // Listen for incoming messages and show notification
    document.addEventListener('message-received', (event) => {
      const { from, message } = event.detail;

      navigator.serviceWorker.ready.then((registration) => {
        registration.showNotification(`Message from ${from}`, {
          body: message,
          icon: '/icons/chat.png',
          badge: '/icons/badge.png',
          tag: `chat-${from}`,
          requireInteraction: true,
          data: { fromUser: from }
        });
      });
    });
  }

  urlBase64ToUint8Array(base64String) {
    // ... implementation
  }
}

// Service Worker: handle push for chat
self.addEventListener('push', (event) => {
  if (!event.data) return;

  const data = event.data.json();

  if (data.type === 'chat_message') {
    const options = {
      body: data.message,
      icon: data.senderAvatar || '/icons/chat.png',
      badge: '/icons/badge.png',
      tag: `chat-${data.senderId}`,
      requireInteraction: true,
      actions: [
        { action: 'reply', title: 'Reply' },
        { action: 'mute', title: 'Mute' }
      ],
      data: {
        conversationId: data.conversationId,
        senderId: data.senderId
      }
    };

    event.waitUntil(
      self.registration.showNotification(
        `Message from ${data.senderName}`,
        options
      )
    );
  }
});

// Handle notification actions
self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  if (event.action === 'reply') {
    // Open reply window
    event.waitUntil(
      clients.openWindow(`/chat/${event.notification.data.conversationId}?reply=true`)
    );
  }
});
```

### Example 2: E-Commerce with Order Updates

```javascript
// ecommerce-notifications.js

class OrderNotificationManager {
  async setupNotifications(userId) {
    const hasPermission = await this.requestPermission();

    if (hasPermission) {
      await this.subscribeToOrderUpdates(userId);
    }
  }

  async requestPermission() {
    if (Notification.permission === 'granted') return true;
    if (Notification.permission === 'denied') return false;

    const permission = await Notification.requestPermission();
    return permission === 'granted';
  }

  async subscribeToOrderUpdates(userId) {
    const registration = await navigator.serviceWorker.ready;
    const response = await fetch('/api/vapid-public-key');
    const { publicKey } = await response.json();

    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: this.urlBase64ToUint8Array(publicKey)
    });

    await fetch('/api/orders/subscribe-notifications', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        userId,
        subscription: subscription.toJSON()
      })
    });
  }

  urlBase64ToUint8Array(base64String) {
    // ... implementation
  }
}

// Service Worker
self.addEventListener('push', (event) => {
  const data = event.data.json();

  const notificationMap = {
    'order_confirmed': handleOrderConfirmed,
    'order_shipped': handleOrderShipped,
    'order_delivered': handleOrderDelivered,
    'order_cancelled': handleOrderCancelled
  };

  const handler = notificationMap[data.type];

  if (handler) {
    event.waitUntil(handler(data));
  }
});

async function handleOrderConfirmed(data) {
  const options = {
    body: `Order ${data.orderId} confirmed`,
    icon: '/icons/confirmed.png',
    badge: '/icons/badge.png',
    tag: `order-${data.orderId}`,
    data: { orderId: data.orderId }
  };

  return self.registration.showNotification('Order Confirmed', options);
}

async function handleOrderShipped(data) {
  const options = {
    body: `Order ${data.orderId} shipped. Tracking: ${data.trackingNumber}`,
    icon: '/icons/shipped.png',
    badge: '/icons/badge.png',
    tag: `order-${data.orderId}`,
    actions: [
      { action: 'track', title: 'Track' }
    ],
    data: { orderId: data.orderId, trackingNumber: data.trackingNumber }
  };

  return self.registration.showNotification('Order Shipped', options);
}

async function handleOrderDelivered(data) {
  const options = {
    body: `Order ${data.orderId} delivered successfully`,
    icon: '/icons/delivered.png',
    badge: '/icons/badge.png',
    tag: `order-${data.orderId}`,
    actions: [
      { action: 'review', title: 'Leave Review' }
    ],
    data: { orderId: data.orderId }
  };

  return self.registration.showNotification('Order Delivered', options);
}

self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  if (event.action === 'track') {
    event.waitUntil(
      clients.openWindow(`/orders/${event.notification.data.orderId}/tracking`)
    );
  } else if (event.action === 'review') {
    event.waitUntil(
      clients.openWindow(`/orders/${event.notification.data.orderId}/review`)
    );
  }
});
```

## Interview Questions

### Question 1: Explain the three-party system of push notifications.

**Answer:**

Push notifications involve three key components:

```javascript
const threePartySystem = {
  client: {
    role: 'User device running the web app',
    responsibilities: [
      'Request user permission',
      'Subscribe to push messages',
      'Handle incoming push events',
      'Display notifications'
    ],
    technologies: ['Push Manager API', 'Notification API', 'Service Worker']
  },

  server: {
    role: 'Application server',
    responsibilities: [
      'Generate and store VAPID keys',
      'Receive and store subscription endpoints',
      'Send push messages when events occur',
      'Handle subscription expiry'
    ],
    technologies: ['web-push library', 'VAPID', 'HTTP/2']
  },

  pushService: {
    role: 'Managed by browser vendor (FCM, APNs, etc.)',
    responsibilities: [
      'Relay messages from server to clients',
      'Deliver reliably even if client is offline',
      'Handle subscription lifecycle'
    ],
    technologies: ['Firebase Cloud Messaging', 'Apple Push Notification']
  }
};

// Flow demonstration
const flowSteps = [
  '1. Client: Request permission and subscribe',
  '2. Browser: Connect to Push Service',
  '3. Push Service: Generate endpoint and return',
  '4. Client: Send subscription endpoint to server',
  '5. Server: Store endpoint for later use',
  '6. Server (later): Send message to Push Service via endpoint',
  '7. Push Service: Deliver to client',
  '8. Browser: Fire push event in service worker',
  '9. Service Worker: Show notification'
];
```

---

### Question 2: What's the difference between Push API and Notification API?

**Answer:**

```javascript
// Key differences
const comparison = {
  pushAPI: {
    purpose: 'Receive messages from server',
    triggeredBy: 'Server sends message via Push Service',
    persistence: 'Works when browser/app is closed',
    requires: 'Server-side implementation',
    scope: 'Must run in Service Worker',
    example: 'Server sends "You have a message"'
  },

  notificationAPI: {
    purpose: 'Display visual notifications',
    triggeredBy: 'Application code (after push event)',
    persistence: 'Works in context where permission granted',
    requires: 'User permission only',
    scope: 'Can run in main thread or Service Worker',
    example: 'Show popup with message content'
  }
};

// Typical usage: They work together
const typicalFlow = {
  step1: 'Server sends message via Push API',
  step2: 'Service Worker receives push event',
  step3: 'Service Worker uses Notification API to display',
  step4: 'User sees notification on desktop'
};

// Code example
// Without Push (app must be open)
navigator.serviceWorker.ready.then((registration) => {
  registration.showNotification('Message', { body: 'Only if app is open' });
});

// With Push (works even when closed)
self.addEventListener('push', (event) => {
  // Browser wakes up service worker when message arrives
  event.waitUntil(
    self.registration.showNotification('Message', {
      body: 'Delivered even when app is closed'
    })
  );
});
```

---

### Question 3: Why are VAPID keys important?

**Answer:**

VAPID (Voluntary Application Server Identification) keys provide authentication and security:

```javascript
const vapidImportance = {
  authentication: {
    issue: 'Without VAPID, any server could send messages to any client',
    solution: 'VAPID proves the server is who it claims to be'
  },

  security: {
    issue: 'Push Services need to verify message legitimacy',
    solution: 'VAPID private key signs messages, public key on client verifies'
  },

  encryption: {
    issue: 'Messages travel through untrusted networks',
    solution: 'VAPID enables encrypted message delivery'
  },

  rateLimit: {
    issue: 'Push Services prevent spam and abuse',
    solution: 'VAPID identifies sender for rate limiting'
  }
};

// How VAPID works
const vapidFlow = [
  '1. Server generates public/private key pair',
  '2. Client receives public key',
  '3. Server uses private key to sign messages',
  '4. Push Service verifies signature with public key',
  '5. Push Service routes message to client',
  '6. Client receives authenticated message'
];

// Implementation impact
const implementation = {
  withoutVAPID: 'Message rejected by Push Service',
  withVAPID: 'Message accepted and delivered'
};
```

---

### Question 4: How do you handle the subscription lifecycle?

**Answer:**

Subscriptions can expire and need maintenance:

```javascript
const subscriptionLifecycle = {
  creation: {
    action: 'User subscribes',
    code: 'registration.pushManager.subscribe({...})',
    result: 'Get subscription endpoint'
  },

  storage: {
    action: 'Send to server',
    code: 'fetch("/api/subscribe", { body: subscription })',
    result: 'Server stores endpoint'
  },

  usage: {
    action: 'Server sends messages',
    code: 'webpush.sendNotification(subscription, payload)',
    result: 'Messages delivered'
  },

  expiry: {
    action: 'Browser may invalidate',
    trigger: 'User clears data, browser updates, etc',
    detection: 'HTTP 410 response from Push Service'
  },

  removal: {
    action: 'Clean up expired subscriptions',
    code: 'subscriptions.delete(id)',
    result: 'No more failed delivery attempts'
  }
};

// Handling expiry
const handleExpiry = {
  detection: 'Catch HTTP 410 error',
  response: 'Remove subscription from database',
  notification: 'Prompt user to resubscribe'
};

// Complete lifecycle example
async function manageSubscriptionLifecycle() {
  const registration = await navigator.serviceWorker.ready;

  // Subscribe
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: vapidKey
  });

  // Send to server
  await fetch('/api/subscribe', {
    method: 'POST',
    body: JSON.stringify({ subscription: subscription.toJSON() })
  });

  // Later: handle expiry
  navigator.serviceWorker.ready.then(async (reg) => {
    const sub = await reg.pushManager.getSubscription();

    if (!sub) {
      // Subscription expired or was removed
      console.log('Subscription expired, need to resubscribe');
      await resubscribe();
    }
  });
}

// Server-side expiry handling
async function sendNotificationWithExpiry(subscription, message) {
  try {
    await webpush.sendNotification(subscription, message);
  } catch (error) {
    if (error.statusCode === 410) {
      // Gone - subscription expired
      await removeSubscription(subscription.endpoint);
      console.log('Subscription expired and removed');
    } else {
      throw error; // Other error
    }
  }
}
```

---

### Question 5: What are best practices for permission handling?

**Answer:**

Permission requests need careful UX consideration:

```javascript
const bestPractices = {
  timing: {
    bad: 'Request immediately on page load',
    good: 'Request after user engagement (click, form submission)',
    reason: 'Better permission grant rates and UX'
  },

  context: {
    bad: 'Generic "Enable notifications?" prompt',
    good: 'Explain benefit: "Stay updated on new messages"',
    reason: 'Users understand why they need to allow'
  },

  frequency: {
    bad: 'Ask every page load',
    good: 'Ask once, remember user choice',
    reason: 'Avoid annoying user with repeated prompts'
  },

  graceful: {
    bad: 'Show errors if permission denied',
    good: 'Offer alternatives (email, in-app alerts)',
    reason: 'Maintain engagement even without push'
  }
};

// Implementation example
class OptimalPermissionFlow {
  async requestPermissionContextually() {
    // Step 1: Understand current permission
    const permission = Notification.permission;

    if (permission === 'granted') {
      // Already have permission
      return await this.subscribe();
    }

    if (permission === 'denied') {
      // User already denied, don't ask again
      return this.showAlternatives();
    }

    // permission === 'default' - ask now but with context

    // Step 2: Show explanation first
    const userWantsNotifications = await this.showExplanationDialog();

    if (!userWantsNotifications) {
      return; // User not interested
    }

    // Step 3: Request permission
    const result = await Notification.requestPermission();

    if (result === 'granted') {
      return await this.subscribe();
    } else {
      // User denied
      return this.showAlternatives();
    }
  }

  async showExplanationDialog() {
    // Show dialog explaining benefits
    return new Promise((resolve) => {
      const modal = document.createElement('div');
      modal.innerHTML = `
        <div class="permission-modal">
          <h2>Stay Connected</h2>
          <p>Get real-time updates about new messages and events</p>
          <button class="allow">Allow Notifications</button>
          <button class="skip">Not Now</button>
        </div>
      `;

      modal.querySelector('.allow').addEventListener('click', () => {
        modal.remove();
        resolve(true);
      });

      modal.querySelector('.skip').addEventListener('click', () => {
        modal.remove();
        resolve(false);
      });

      document.body.appendChild(modal);
    });
  }

  async subscribe() {
    // Subscribe implementation
  }

  showAlternatives() {
    // Show email signup form, in-app alerts, etc.
  }
}
```

---

### Question 6: How do you avoid pushing irrelevant notifications?

**Answer:**

Managing notification relevance improves user experience:

```javascript
const strategies = {
  targeting: {
    approach: 'Send only to interested users',
    implementation: 'Store user preferences with subscription',
    example: 'User opted in for "chat" but not "marketing"'
  },

  frequency: {
    approach: 'Rate limit notifications',
    implementation: 'Track last notification time',
    example: 'Maximum 1 notification per minute'
  },

  batching: {
    approach: 'Combine multiple events',
    implementation: 'Wait a bit before sending',
    example: '3 new messages, send 1 notification'
  },

  quietHours: {
    approach: 'Don\'t send at certain times',
    implementation: 'Respect user timezone',
    example: 'No notifications between 11 PM and 7 AM'
  },

  relevance: {
    approach: 'Personalize content',
    implementation: 'Use user preferences and history',
    example: 'Only notify about relevant categories'
  }
};

// Implementation example
class RelevantNotificationManager {
  async sendNotification(userId, event) {
    // 1. Check user preferences
    const preferences = await this.getUserPreferences(userId);

    if (!preferences[event.category]) {
      console.log('User not interested in this category');
      return; // Don't send
    }

    // 2. Check frequency limits
    const lastNotification = await this.getLastNotificationTime(userId);
    const timeSinceLastNotification = Date.now() - lastNotification;

    if (timeSinceLastNotification < 60 * 1000) {
      console.log('Too soon since last notification');
      return; // Don't send
    }

    // 3. Check quiet hours
    const userTimezone = await this.getUserTimezone(userId);
    const currentHour = new Date().toLocaleString('en-US', {
      timeZone: userTimezone,
      hour: '2-digit'
    });

    if (currentHour >= 23 || currentHour < 7) {
      console.log('During quiet hours');
      return; // Don't send
    }

    // 4. Send notification
    await this.sendPushNotification(userId, event);
  }

  async getUserPreferences(userId) {
    const subscription = await this.getSubscription(userId);
    return {
      chat: true,
      marketing: false,
      updates: true
    };
  }

  async getLastNotificationTime(userId) {
    // Retrieve from database
    return 0;
  }

  async getUserTimezone(userId) {
    // Get from user profile
    return 'America/New_York';
  }

  async sendPushNotification(userId, event) {
    const subscription = await this.getSubscription(userId);
    await webpush.sendNotification(
      subscription,
      JSON.stringify(event)
    );
  }
}
```

---

### Question 7: How do you handle security concerns with push notifications?

**Answer:**

Security is critical for push notifications:

```javascript
const securityConcerns = {
  vapidKeyExposure: {
    risk: 'If private key exposed, anyone can send messages',
    mitigation: 'Store private key in secure environment variable',
    never: 'Don\'t commit to git, don\'t send to client'
  },

  subscriptionDataLeak: {
    risk: 'Subscription endpoint is like a token',
    mitigation: 'Treat like auth token, use HTTPS only',
    never: 'Don\'t log publicly, don\'t store unencrypted'
  },

  messageContent: {
    risk: 'Push message could expose sensitive data',
    mitigation: 'Encrypt message payload, use generic content',
    example: 'Send "You have a message" instead of message content'
  },

  spoofing: {
    risk: 'Malicious site could send notifications',
    mitigation: 'Use VAPID for authentication',
    never: 'Always verify message origin'
  },

  csrfAttacks: {
    risk: 'Cross-site requests could trigger subscriptions',
    mitigation: 'Use CSRF tokens for subscribe endpoints',
    never: 'Don\'t allow anonymous subscriptions'
  }
};

// Secure implementation
class SecurePushNotificationManager {
  async securelySubscribe() {
    // 1. Verify user is authenticated
    if (!this.isAuthenticated()) {
      throw new Error('User not authenticated');
    }

    // 2. Get CSRF token
    const csrfToken = this.getCsrfToken();

    // 3. Subscribe with security headers
    const response = await fetch('/api/notifications/subscribe', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': csrfToken,
        'X-Requested-With': 'XMLHttpRequest'
      },
      body: JSON.stringify({
        subscription: subscription.toJSON()
      }),
      credentials: 'same-origin' // Include cookies
    });

    if (!response.ok) {
      throw new Error('Subscription failed');
    }
  }

  async securelyUnsubscribe(subscriptionId) {
    // 1. Verify ownership
    const owns = await this.verifySubscriptionOwnership(subscriptionId);
    if (!owns) {
      throw new Error('Not authorized');
    }

    // 2. Unsubscribe with verification
    const response = await fetch('/api/notifications/unsubscribe', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': this.getCsrfToken()
      },
      body: JSON.stringify({ subscriptionId })
    });

    return response.ok;
  }

  isAuthenticated() {
    return !!localStorage.getItem('authToken');
  }

  getCsrfToken() {
    return document.querySelector('meta[name="csrf-token"]').content;
  }

  async verifySubscriptionOwnership(subscriptionId) {
    const response = await fetch(`/api/subscriptions/${subscriptionId}/verify`);
    const data = await response.json();
    return data.owned;
  }
}

// Server-side security
class SecurePushServer {
  async handleSubscribe(req, res) {
    // 1. Verify authentication
    const user = this.authenticateRequest(req);
    if (!user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }

    // 2. Verify CSRF
    if (!this.verifyCsrfToken(req)) {
      return res.status(403).json({ error: 'CSRF token invalid' });
    }

    // 3. Validate subscription format
    const { subscription } = req.body;
    if (!this.isValidSubscription(subscription)) {
      return res.status(400).json({ error: 'Invalid subscription' });
    }

    // 4. Store securely
    await this.storeSubscriptionSecurely(user.id, subscription);

    res.json({ success: true });
  }

  async sendSecureNotification(user, message) {
    // 1. Get subscription (don't expose endpoint)
    const subscription = await this.getSubscription(user.id);

    // 2. Validate before sending
    if (!subscription) {
      console.log('No valid subscription');
      return;
    }

    // 3. Encrypt sensitive data
    const encryptedPayload = this.encryptPayload(message);

    // 4. Send with VAPID
    try {
      await webpush.sendNotification(subscription, encryptedPayload);
    } catch (error) {
      if (error.statusCode === 410) {
        // Remove expired
        await this.removeSubscription(user.id);
      }
    }
  }
}
```

---

### Question 8: How do you test push notifications in development?

**Answer:**

Testing requires specific tools and approaches:

```javascript
const testingStrategies = {
  unitTesting: {
    test: 'Push API subscribe function',
    tool: 'Jest/Vitest with mocks',
    example: 'Mock navigator.serviceWorker'
  },

  integrationTesting: {
    test: 'Full subscription flow',
    tool: 'Puppeteer, Playwright',
    example: 'Test subscription -> storage -> retrieval'
  },

  manualTesting: {
    test: 'Real push notifications',
    tool: 'DevTools, test server',
    example: 'Send actual push from server'
  },

  e2eTesting: {
    test: 'Full user journey',
    tool: 'Cypress, Playwright',
    example: 'Permission -> subscribe -> receive notification'
  }
};

// Manual testing checklist
const manualTestingChecklist = {
  setup: [
    'Register service worker',
    'Request notification permission',
    'Subscribe to push'
  ],

  deviceTests: [
    'Test on mobile',
    'Test in offline mode',
    'Test with app closed'
  ],

  notificationBehavior: [
    'Notification displays correctly',
    'Actions work as expected',
    'Click handlers fire correctly'
  ],

  edgeCases: [
    'Permission denied then granted',
    'Subscription expires',
    'Multiple notifications',
    'Notification during active app'
  ]
};

// Testing with mock server
class PushNotificationTestServer {
  async testNotificationFlow() {
    // 1. Get subscription
    const subscriptions = await this.getSubscriptions();

    // 2. Send test notification
    for (const subscription of subscriptions) {
      try {
        await webpush.sendNotification(
          subscription,
          JSON.stringify({
            title: 'Test Notification',
            body: 'This is a test',
            timestamp: new Date().toISOString()
          })
        );
      } catch (error) {
        console.error('Test failed:', error);
      }
    }
  }
}

// Local testing with service worker
const localTestSetup = {
  requirements: [
    'HTTPS or localhost',
    'Service worker registered',
    'VAPID keys configured',
    'Test server running'
  ],

  steps: [
    '1. Start local server with HTTPS',
    '2. Open app in browser',
    '3. Grant notification permission',
    '4. Open DevTools',
    '5. Send test notification from server',
    '6. Verify notification appears',
    '7. Check Service Worker logs'
  ]
};
```

---

### Question 9: How do you scale push notifications to millions of users?

**Answer:**

At scale, new considerations emerge:

```javascript
const scalingChallenges = {
  storage: {
    issue: 'Store millions of subscription endpoints',
    solution: 'Use database with good indexing',
    database: 'MongoDB, PostgreSQL with replication'
  },

  delivery: {
    issue: 'Send millions of notifications reliably',
    solution: 'Use queue system, batch sending',
    tools: 'Redis queue, Bull, RabbitMQ'
  },

  performance: {
    issue: 'Server can\'t send all notifications synchronously',
    solution: 'Async processing, worker processes',
    approach: 'Background jobs, message queues'
  },

  failureHandling: {
    issue: 'Some deliveries will fail',
    solution: 'Retry logic, dead letter queues',
    monitoring: 'Track success/failure rates'
  },

  costOptimization: {
    issue: 'Push services charge per message',
    solution: 'Batch messages, smart timing',
    example: 'Combine multiple events into one notification'
  }
};

// Scaled architecture
const scaledArchitecture = {
  phase1: {
    users: '1K',
    approach: 'Direct sending',
    implementation: 'Send immediately on event'
  },

  phase2: {
    users: '100K',
    approach: 'Basic queuing',
    implementation: 'Redis queue + worker process'
  },

  phase3: {
    users: '1M+',
    approach: 'Advanced queuing',
    implementation: [
      'Multiple worker processes',
      'Database connection pooling',
      'Batch sending',
      'Rate limiting',
      'Monitoring and alerting'
    ]
  }
};

// Scaled implementation example
class ScaledPushNotificationSystem {
  async sendNotificationAtScale(userId, message) {
    // 1. Add to queue instead of sending directly
    await this.notificationQueue.add({
      userId,
      message,
      timestamp: new Date()
    });
  }

  async processNotificationQueue() {
    // 1. Get batch from queue
    const batch = await this.notificationQueue.getNext(1000);

    // 2. Group by user preferences for optimization
    const grouped = this.groupByPreferences(batch);

    // 3. Send in parallel batches
    const results = await Promise.all(
      grouped.map((group) => this.sendBatch(group))
    );

    // 4. Track metrics
    this.trackMetrics(results);
  }

  async sendBatch(batch) {
    // Send multiple notifications in one request
    const subscriptions = await this.getSubscriptions(
      batch.map((b) => b.userId)
    );

    return Promise.all(
      subscriptions.map((sub) =>
        webpush.sendNotification(sub, JSON.stringify(batch[0].message))
      )
    );
  }

  async getSubscriptions(userIds) {
    // Query from cache first (Redis)
    // Fall back to database
    return this.cache.get(userIds) ||
      (await this.database.findSubscriptions(userIds));
  }

  groupByPreferences(batch) {
    // Group notifications by user preferences
    // Send "priority" notifications immediately
    // Batch "marketing" notifications
    return [];
  }

  trackMetrics(results) {
    // Track delivery success/failure rates
    // Send to monitoring system
    // Alert on failures
  }
}

// Database schema for scale
const databaseSchema = {
  subscriptions: {
    id: 'UUID',
    userId: 'UUID', // Indexed
    endpoint: 'URL', // Indexed
    expirationTime: 'Timestamp', // Indexed for cleanup
    keys: 'JSON',
    createdAt: 'Timestamp',
    updatedAt: 'Timestamp'
  },

  subscriptionPreferences: {
    subscriptionId: 'UUID', // Foreign key
    category: 'String', // Indexed
    enabled: 'Boolean',
    createdAt: 'Timestamp'
  },

  notificationLog: {
    subscriptionId: 'UUID', // Indexed
    messageId: 'UUID',
    sentAt: 'Timestamp', // Indexed
    status: 'String' // sent, failed, expired
  }
};
```

---

### Question 10: What happens when a subscription endpoint expires?

**Answer:**

Handling expired subscriptions is crucial:

```javascript
const subscriptionExpiry = {
  causes: [
    'User clears browser data',
    'Browser uninstalls',
    'User explicitly unsubscribes',
    'Push service invalidates',
    'Device loses all notifications'
  ],

  detection: {
    how: 'HTTP 410 Gone response from Push Service',
    when: 'When server tries to send notification',
    handler: 'Catch error and remove subscription'
  },

  clientSideDetection: {
    how: 'Check registration.pushManager.getSubscription()',
    when: 'On app startup',
    response: 'Prompt to resubscribe'
  }
};

// Server-side handling
class ExpiryHandler {
  async sendNotificationWithExpiry(subscription, message) {
    try {
      await webpush.sendNotification(subscription, message);
    } catch (error) {
      // Check for expiration
      if (error.statusCode === 410) {
        // Subscription expired
        await this.handleExpiredSubscription(subscription);
      } else {
        // Other error, may retry
        throw error;
      }
    }
  }

  async handleExpiredSubscription(subscription) {
    // 1. Remove from database
    await this.database.removeSubscription(subscription.endpoint);

    // 2. Log for analytics
    console.log('Subscription expired:', subscription.endpoint);

    // 3. Update metrics
    this.metrics.incrementExpiredCount();

    // 4. Optionally notify user
    // Send email: "Notifications disabled, reactivate here"
  }
}

// Client-side detection
class ClientSideExpiry {
  async checkSubscriptionStatus() {
    const registration = await navigator.serviceWorker.ready;
    const subscription = await registration.pushManager.getSubscription();

    if (!subscription) {
      // Subscription expired or removed
      console.log('No subscription found');

      // Determine if user wants to resubscribe
      const shouldResubscribe = await this.promptResubscribe();

      if (shouldResubscribe) {
        await this.resubscribe();
      }
    }

    return !!subscription;
  }

  async promptResubscribe() {
    return new Promise((resolve) => {
      const dialog = confirm(
        'Notifications are disabled. Enable them again?'
      );
      resolve(dialog);
    });
  }

  async resubscribe() {
    // Get VAPID key
    const response = await fetch('/api/vapid-public-key');
    const { publicKey } = await response.json();

    // Subscribe
    const registration = await navigator.serviceWorker.ready;
    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: this.urlBase64ToUint8Array(publicKey)
    });

    // Send to server
    await fetch('/api/notifications/subscribe', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        subscription: subscription.toJSON()
      })
    });
  }
}

// Proactive cleanup
class ProactiveCleanup {
  async cleanupExpiredSubscriptions() {
    const allSubscriptions = await this.database.getAllSubscriptions();

    for (const subscription of allSubscriptions) {
      const isValid = await this.verifySubscription(subscription);

      if (!isValid) {
        console.log('Removing invalid subscription');
        await this.database.removeSubscription(subscription.id);
      }
    }
  }

  async verifySubscription(subscription) {
    try {
      // Send a minimal test message
      await webpush.sendNotification(
        subscription,
        JSON.stringify({ type: 'ping' })
      );
      return true;
    } catch (error) {
      return error.statusCode !== 410;
    }
  }
}
```

---

## Summary

Push Notifications are essential for modern PWAs. Key takeaways:

**Core Concepts:**
- Three-party system: Client, Server, Push Service
- Push API for receiving messages from server
- Notification API for displaying notifications
- VAPID keys for authentication

**Implementation:**
- Request user permission (contextually)
- Subscribe to push notifications
- Send subscription endpoint to server
- Handle push events in service worker
- Show notifications with relevant content

**Best Practices:**
- Use HTTPS only
- Store VAPID private key securely
- Handle subscription expiry
- Rate limit notifications
- Respect quiet hours
- Provide alternatives for denied permissions

**Scaling:**
- Use job queues for async processing
- Batch messages together
- Monitor delivery success rates
- Clean up expired subscriptions regularly
- Implement proper error handling

**Security:**
- Never expose private VAPID key
- Validate subscription ownership
- Use CSRF protection
- Encrypt sensitive data
- Authenticate requests

Remember: Push notifications should enhance user experience, not annoy. Every notification should be valuable and timely.

---

**Previous:** [ê Background Sync](./04-background-sync.md)

---

[ê Back to PWA](./README.md) | [ë Back to Frontend](../README.md)
