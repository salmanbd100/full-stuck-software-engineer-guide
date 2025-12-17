# Browser Permissions

## Overview

**Browser Permissions** are security mechanisms that control access to sensitive device features and user data. Modern web applications can request access to hardware (camera, microphone), user information (location), and system capabilities (notifications, clipboard) through standardized APIs. Permissions protect user privacy by requiring explicit consent before granting access to these features.

### Key Characteristics
- **User-controlled**: Users must explicitly grant or deny permission
- **Per-origin**: Permissions are granted to specific origins (domain + protocol + port)
- **Persistent**: Granted permissions are remembered across sessions
- **Revocable**: Users can revoke permissions at any time through browser settings
- **Secure contexts**: Most permissions require HTTPS (except localhost)
- **Progressive disclosure**: Request permissions when needed, not upfront

---

## Table of Contents

1. [Permissions API Overview](#permissions-api-overview)
2. [Permission Types](#permission-types)
3. [Requesting Permissions](#requesting-permissions)
4. [Permission States](#permission-states)
5. [Geolocation API](#geolocation-api)
6. [Notifications API](#notifications-api)
7. [Camera and Microphone (Media Devices)](#camera-and-microphone-media-devices)
8. [Clipboard API](#clipboard-api)
9. [Other Permissions](#other-permissions)
10. [Best Practices](#best-practices)
11. [Error Handling](#error-handling)
12. [Interview Questions](#interview-questions)
13. [Summary](#summary)
14. [External Resources](#external-resources)

---

## Permissions API Overview

The **Permissions API** provides a unified way to query the status of API permissions. It allows you to check whether permission has been granted, denied, or requires user interaction.

### Checking Permission Status

```javascript
// Query permission status
async function checkPermission(permissionName) {
    try {
        const result = await navigator.permissions.query({ name: permissionName });
        console.log(`${permissionName} permission:`, result.state);
        // result.state can be: 'granted', 'denied', or 'prompt'

        // Listen for permission changes
        result.onchange = () => {
            console.log(`${permissionName} permission changed to:`, result.state);
        };

        return result.state;
    } catch (error) {
        console.error('Error checking permission:', error);
        return null;
    }
}

// Usage
await checkPermission('geolocation');
await checkPermission('notifications');
await checkPermission('camera');
```

### Permission Descriptors

```javascript
// Different permission types require different descriptors

// 1. Simple permissions (name only)
await navigator.permissions.query({ name: 'geolocation' });
await navigator.permissions.query({ name: 'notifications' });

// 2. Push permission (requires userVisibleOnly)
await navigator.permissions.query({
    name: 'push',
    userVisibleOnly: true
});

// 3. MIDI permission (requires sysex)
await navigator.permissions.query({
    name: 'midi',
    sysex: true
});

// 4. Camera/Microphone permissions
await navigator.permissions.query({ name: 'camera' });
await navigator.permissions.query({ name: 'microphone' });
```

### Comprehensive Permission Checker

```javascript
class PermissionManager {
    constructor() {
        this.permissions = new Map();
    }

    async checkPermission(name, descriptor = {}) {
        try {
            const fullDescriptor = { name, ...descriptor };
            const result = await navigator.permissions.query(fullDescriptor);

            // Store permission status
            this.permissions.set(name, result.state);

            // Monitor changes
            result.onchange = () => {
                this.permissions.set(name, result.state);
                this.onPermissionChange(name, result.state);
            };

            return result.state;
        } catch (error) {
            console.error(`Error checking ${name} permission:`, error);
            return null;
        }
    }

    async checkAllPermissions() {
        const permissionNames = [
            'geolocation',
            'notifications',
            'camera',
            'microphone',
            'clipboard-read',
            'clipboard-write'
        ];

        const results = await Promise.allSettled(
            permissionNames.map(name => this.checkPermission(name))
        );

        return results.reduce((acc, result, index) => {
            acc[permissionNames[index]] = result.status === 'fulfilled'
                ? result.value
                : 'unsupported';
            return acc;
        }, {});
    }

    onPermissionChange(name, state) {
        console.log(`Permission ${name} changed to ${state}`);
        // Override this method in subclasses or instances
    }

    getPermissionStatus(name) {
        return this.permissions.get(name) || 'unknown';
    }

    hasPermission(name) {
        return this.permissions.get(name) === 'granted';
    }
}

// Usage
const permissionManager = new PermissionManager();

// Check all permissions
const allPermissions = await permissionManager.checkAllPermissions();
console.log('All permissions:', allPermissions);

// Check specific permission
const geoState = await permissionManager.checkPermission('geolocation');
console.log('Geolocation:', geoState);

// Check if permission is granted
if (permissionManager.hasPermission('notifications')) {
    console.log('Notifications are enabled');
}
```

---

## Permission Types

### Available Permissions

| Permission | API | Description | Browser Support |
|-----------|-----|-------------|-----------------|
| **geolocation** | Geolocation API | User's physical location | All modern browsers |
| **notifications** | Notifications API | Display system notifications | All modern browsers |
| **camera** | MediaDevices | Access camera | All modern browsers |
| **microphone** | MediaDevices | Access microphone | All modern browsers |
| **clipboard-read** | Clipboard API | Read clipboard content | Chrome, Edge, Safari |
| **clipboard-write** | Clipboard API | Write to clipboard | Chrome, Edge, Safari |
| **push** | Push API | Receive push notifications | Chrome, Firefox, Edge |
| **background-sync** | Background Sync | Sync in background | Chrome, Edge |
| **persistent-storage** | Storage API | Prevent data eviction | Chrome, Edge |
| **midi** | Web MIDI API | Access MIDI devices | Chrome, Edge, Opera |
| **wake-lock** | Screen Wake Lock | Prevent screen sleep | Chrome, Edge |

### Permission Support Check

```javascript
// Check if permission is supported
function isPermissionSupported(permissionName) {
    if (!navigator.permissions || !navigator.permissions.query) {
        return false;
    }

    // Try to query the permission
    return navigator.permissions.query({ name: permissionName })
        .then(() => true)
        .catch(() => false);
}

// Check multiple permissions
async function getSupportedPermissions() {
    const allPermissions = [
        'geolocation',
        'notifications',
        'camera',
        'microphone',
        'clipboard-read',
        'clipboard-write',
        'push',
        'background-sync',
        'persistent-storage',
        'midi',
        'wake-lock'
    ];

    const supportedPermissions = [];

    for (const permission of allPermissions) {
        const supported = await isPermissionSupported(permission);
        if (supported) {
            supportedPermissions.push(permission);
        }
    }

    return supportedPermissions;
}

// Usage
const supported = await getSupportedPermissions();
console.log('Supported permissions:', supported);
```

---

## Requesting Permissions

### Best Practices for Requesting Permissions

```javascript
// 1. Request permissions in response to user action
// Bad: Request on page load
window.addEventListener('load', () => {
    Notification.requestPermission(); // Annoying!
});

// Good: Request when user clicks a button
document.getElementById('enableNotifications').addEventListener('click', async () => {
    const permission = await Notification.requestPermission();
    if (permission === 'granted') {
        console.log('Notifications enabled');
    }
});

// 2. Check permission before requesting
async function requestNotificationPermission() {
    // Check current status first
    const result = await navigator.permissions.query({ name: 'notifications' });

    if (result.state === 'granted') {
        console.log('Already granted');
        return true;
    } else if (result.state === 'denied') {
        console.log('Permission denied, cannot request again');
        // Show UI to guide user to browser settings
        showPermissionDeniedMessage();
        return false;
    }

    // Only request if in 'prompt' state
    const permission = await Notification.requestPermission();
    return permission === 'granted';
}

// 3. Provide context before requesting
async function requestPermissionWithContext(feature) {
    // Show modal explaining why permission is needed
    const userConsent = await showPermissionExplanationModal(feature);

    if (!userConsent) {
        console.log('User declined to grant permission');
        return false;
    }

    // User agreed, now request permission
    const granted = await requestFeaturePermission(feature);
    return granted;
}

function showPermissionExplanationModal(feature) {
    return new Promise((resolve) => {
        const modal = document.createElement('div');
        modal.className = 'permission-modal';
        modal.innerHTML = `
            <div class="modal-content">
                <h2>Enable ${feature}</h2>
                <p>We need access to ${feature} to provide you with better experience.</p>
                <button id="allow">Allow</button>
                <button id="deny">Not Now</button>
            </div>
        `;

        document.body.appendChild(modal);

        document.getElementById('allow').onclick = () => {
            modal.remove();
            resolve(true);
        };

        document.getElementById('deny').onclick = () => {
            modal.remove();
            resolve(false);
        };
    });
}
```

### Progressive Permission Requests

```javascript
// Request permissions progressively as features are used
class ProgressivePermissionManager {
    constructor() {
        this.requestedPermissions = new Set();
    }

    async requestWhenNeeded(permission, feature) {
        // Don't ask twice
        if (this.requestedPermissions.has(permission)) {
            const status = await navigator.permissions.query({ name: permission });
            return status.state === 'granted';
        }

        // Show feature benefit first
        const userWantsFeature = await this.showFeatureBenefit(feature);

        if (!userWantsFeature) {
            return false;
        }

        // Request permission
        this.requestedPermissions.add(permission);
        return await this.requestPermission(permission);
    }

    async showFeatureBenefit(feature) {
        // Show UI explaining the feature benefit
        console.log(`Showing benefit for ${feature}`);
        return true; // Implement actual UI
    }

    async requestPermission(permission) {
        // Implement specific permission request logic
        switch (permission) {
            case 'geolocation':
                return await this.requestGeolocation();
            case 'notifications':
                return await this.requestNotifications();
            case 'camera':
                return await this.requestCamera();
            default:
                return false;
        }
    }

    async requestGeolocation() {
        return new Promise((resolve) => {
            navigator.geolocation.getCurrentPosition(
                () => resolve(true),
                () => resolve(false)
            );
        });
    }

    async requestNotifications() {
        const result = await Notification.requestPermission();
        return result === 'granted';
    }

    async requestCamera() {
        try {
            const stream = await navigator.mediaDevices.getUserMedia({ video: true });
            stream.getTracks().forEach(track => track.stop()); // Clean up
            return true;
        } catch {
            return false;
        }
    }
}

// Usage
const permManager = new ProgressivePermissionManager();

// When user wants to share location
document.getElementById('shareLocation').onclick = async () => {
    const granted = await permManager.requestWhenNeeded('geolocation', 'location sharing');
    if (granted) {
        // Use geolocation
        getCurrentLocation();
    }
};
```

---

## Permission States

### Understanding Permission States

```javascript
// Permission states:
// - 'granted': Permission has been granted
// - 'denied': Permission has been denied
// - 'prompt': Permission has not been requested yet

async function handlePermissionState(permissionName) {
    const result = await navigator.permissions.query({ name: permissionName });

    switch (result.state) {
        case 'granted':
            console.log(`${permissionName} is granted`);
            // Enable feature
            enableFeature(permissionName);
            break;

        case 'denied':
            console.log(`${permissionName} is denied`);
            // Show message about enabling in browser settings
            showPermissionDeniedUI(permissionName);
            break;

        case 'prompt':
            console.log(`${permissionName} needs to be requested`);
            // Show button to request permission
            showPermissionRequestUI(permissionName);
            break;
    }
}

function showPermissionDeniedUI(permissionName) {
    const message = document.createElement('div');
    message.className = 'permission-denied';
    message.innerHTML = `
        <p>${permissionName} permission is blocked.</p>
        <p>To enable it, click the lock icon in the address bar.</p>
    `;
    document.body.appendChild(message);
}

function showPermissionRequestUI(permissionName) {
    const button = document.createElement('button');
    button.textContent = `Enable ${permissionName}`;
    button.onclick = () => requestPermission(permissionName);
    document.body.appendChild(button);
}
```

### Monitoring Permission Changes

```javascript
// Listen for permission changes
async function monitorPermission(permissionName, callback) {
    const result = await navigator.permissions.query({ name: permissionName });

    // Initial state
    callback(result.state);

    // Listen for changes
    result.addEventListener('change', () => {
        callback(result.state);
    });

    return result;
}

// Usage
monitorPermission('geolocation', (state) => {
    console.log('Geolocation permission:', state);

    if (state === 'granted') {
        startLocationTracking();
    } else if (state === 'denied') {
        stopLocationTracking();
        showLocationDisabledMessage();
    }
});

// Monitor multiple permissions
class PermissionMonitor {
    constructor() {
        this.monitors = new Map();
    }

    async watch(permissionName, callback) {
        if (this.monitors.has(permissionName)) {
            console.warn(`Already monitoring ${permissionName}`);
            return;
        }

        const result = await navigator.permissions.query({ name: permissionName });

        const handler = () => callback(result.state);
        result.addEventListener('change', handler);

        this.monitors.set(permissionName, { result, handler, callback });

        // Call immediately with current state
        callback(result.state);
    }

    unwatch(permissionName) {
        const monitor = this.monitors.get(permissionName);
        if (monitor) {
            monitor.result.removeEventListener('change', monitor.handler);
            this.monitors.delete(permissionName);
        }
    }

    unwatchAll() {
        for (const [name] of this.monitors) {
            this.unwatch(name);
        }
    }
}

// Usage
const monitor = new PermissionMonitor();

monitor.watch('notifications', (state) => {
    console.log('Notifications:', state);
    updateNotificationUI(state);
});

monitor.watch('geolocation', (state) => {
    console.log('Geolocation:', state);
    updateLocationUI(state);
});

// Clean up when component unmounts
// monitor.unwatchAll();
```

---

## Geolocation API

### Basic Geolocation Usage

```javascript
// Get current position
function getCurrentLocation() {
    if (!navigator.geolocation) {
        console.error('Geolocation not supported');
        return;
    }

    navigator.geolocation.getCurrentPosition(
        (position) => {
            const { latitude, longitude, accuracy, altitude, heading, speed } = position.coords;
            const timestamp = position.timestamp;

            console.log('Location:', {
                latitude,
                longitude,
                accuracy: `${accuracy} meters`,
                timestamp: new Date(timestamp)
            });

            // Use location
            displayLocationOnMap(latitude, longitude);
        },
        (error) => {
            console.error('Geolocation error:', error.message);
            handleGeolocationError(error);
        },
        {
            enableHighAccuracy: true,
            timeout: 10000,
            maximumAge: 0
        }
    );
}

// Handle geolocation errors
function handleGeolocationError(error) {
    switch (error.code) {
        case error.PERMISSION_DENIED:
            console.error('User denied geolocation permission');
            showMessage('Please enable location access in your browser settings');
            break;

        case error.POSITION_UNAVAILABLE:
            console.error('Location information unavailable');
            showMessage('Unable to determine your location');
            break;

        case error.TIMEOUT:
            console.error('Geolocation request timed out');
            showMessage('Location request timed out. Please try again.');
            break;

        default:
            console.error('Unknown geolocation error');
            showMessage('An error occurred while getting your location');
    }
}
```

### Advanced Geolocation Features

```javascript
// Watch position (continuous tracking)
class LocationTracker {
    constructor() {
        this.watchId = null;
        this.currentPosition = null;
    }

    startTracking(onUpdate, onError) {
        if (this.watchId !== null) {
            console.warn('Already tracking location');
            return;
        }

        this.watchId = navigator.geolocation.watchPosition(
            (position) => {
                this.currentPosition = position;
                const { latitude, longitude, accuracy, speed } = position.coords;

                onUpdate({
                    latitude,
                    longitude,
                    accuracy,
                    speed,
                    timestamp: position.timestamp
                });
            },
            (error) => {
                console.error('Location tracking error:', error);
                onError(error);
            },
            {
                enableHighAccuracy: true,
                timeout: 5000,
                maximumAge: 0
            }
        );

        console.log('Started location tracking');
    }

    stopTracking() {
        if (this.watchId !== null) {
            navigator.geolocation.clearWatch(this.watchId);
            this.watchId = null;
            console.log('Stopped location tracking');
        }
    }

    getCurrentPosition() {
        return this.currentPosition;
    }

    calculateDistance(lat1, lon1, lat2, lon2) {
        // Haversine formula
        const R = 6371; // Earth's radius in km
        const dLat = this.toRad(lat2 - lat1);
        const dLon = this.toRad(lon2 - lon1);

        const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
                  Math.cos(this.toRad(lat1)) * Math.cos(this.toRad(lat2)) *
                  Math.sin(dLon / 2) * Math.sin(dLon / 2);

        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
        return R * c; // Distance in km
    }

    toRad(degrees) {
        return degrees * (Math.PI / 180);
    }
}

// Usage
const tracker = new LocationTracker();

document.getElementById('startTracking').onclick = () => {
    tracker.startTracking(
        (location) => {
            console.log('Location update:', location);
            updateMapMarker(location.latitude, location.longitude);

            if (location.speed) {
                console.log(`Speed: ${location.speed} m/s`);
            }
        },
        (error) => {
            console.error('Tracking error:', error);
            tracker.stopTracking();
        }
    );
};

document.getElementById('stopTracking').onclick = () => {
    tracker.stopTracking();
};

// Calculate distance between two points
const distance = tracker.calculateDistance(40.7128, -74.0060, 51.5074, -0.1278);
console.log(`Distance: ${distance.toFixed(2)} km`);
```

### Geolocation with Permission Check

```javascript
// Complete geolocation implementation with permission handling
async function requestLocationAccess() {
    // Check if geolocation is supported
    if (!navigator.geolocation) {
        throw new Error('Geolocation not supported');
    }

    // Check permission status
    try {
        const permissionStatus = await navigator.permissions.query({ name: 'geolocation' });

        if (permissionStatus.state === 'denied') {
            throw new Error('Location permission denied. Please enable it in browser settings.');
        }

        if (permissionStatus.state === 'granted') {
            return await getPosition();
        }

        // Permission is 'prompt' - will trigger permission dialog
        return await getPosition();

    } catch (error) {
        // Fallback if Permissions API not supported
        return await getPosition();
    }
}

function getPosition() {
    return new Promise((resolve, reject) => {
        navigator.geolocation.getCurrentPosition(
            (position) => {
                resolve({
                    latitude: position.coords.latitude,
                    longitude: position.coords.longitude,
                    accuracy: position.coords.accuracy,
                    timestamp: position.timestamp
                });
            },
            (error) => {
                reject(error);
            },
            {
                enableHighAccuracy: true,
                timeout: 10000,
                maximumAge: 300000 // Accept cached position up to 5 minutes old
            }
        );
    });
}

// Usage
async function handleLocationRequest() {
    try {
        const location = await requestLocationAccess();
        console.log('User location:', location);
        displayLocationOnMap(location.latitude, location.longitude);
    } catch (error) {
        console.error('Failed to get location:', error.message);
        showLocationError(error);
    }
}
```

---

## Notifications API

### Basic Notifications

```javascript
// Request notification permission
async function requestNotificationPermission() {
    if (!('Notification' in window)) {
        console.error('Notifications not supported');
        return false;
    }

    // Check current permission
    if (Notification.permission === 'granted') {
        return true;
    }

    if (Notification.permission === 'denied') {
        console.error('Notification permission denied');
        return false;
    }

    // Request permission
    const permission = await Notification.requestPermission();
    return permission === 'granted';
}

// Show a notification
function showNotification(title, options = {}) {
    if (Notification.permission !== 'granted') {
        console.error('Notification permission not granted');
        return;
    }

    const defaultOptions = {
        body: '',
        icon: '/icon.png',
        badge: '/badge.png',
        tag: 'default',
        requireInteraction: false,
        silent: false
    };

    const notification = new Notification(title, { ...defaultOptions, ...options });

    // Event handlers
    notification.onclick = (event) => {
        console.log('Notification clicked:', event);
        window.focus();
        notification.close();
    };

    notification.onclose = () => {
        console.log('Notification closed');
    };

    notification.onerror = (error) => {
        console.error('Notification error:', error);
    };

    return notification;
}

// Usage
document.getElementById('notifyButton').onclick = async () => {
    const granted = await requestNotificationPermission();

    if (granted) {
        showNotification('Hello!', {
            body: 'This is a notification from your web app',
            icon: '/icon.png',
            tag: 'greeting'
        });
    }
};
```

### Advanced Notifications

```javascript
// Notification with actions
class NotificationManager {
    constructor() {
        this.supportedFeatures = this.checkFeatures();
    }

    checkFeatures() {
        return {
            basic: 'Notification' in window,
            actions: 'actions' in Notification.prototype,
            badge: 'badge' in Notification.prototype,
            image: 'image' in Notification.prototype,
            renotify: 'renotify' in Notification.prototype,
            requireInteraction: 'requireInteraction' in Notification.prototype,
            vibrate: 'vibrate' in Notification.prototype
        };
    }

    async requestPermission() {
        if (!this.supportedFeatures.basic) {
            throw new Error('Notifications not supported');
        }

        if (Notification.permission === 'granted') {
            return true;
        }

        if (Notification.permission === 'denied') {
            throw new Error('Notification permission denied');
        }

        const permission = await Notification.requestPermission();

        if (permission !== 'granted') {
            throw new Error('Notification permission not granted');
        }

        return true;
    }

    show(title, options = {}) {
        if (Notification.permission !== 'granted') {
            console.error('Cannot show notification: permission not granted');
            return null;
        }

        const notificationOptions = {
            body: options.body || '',
            icon: options.icon || '/default-icon.png',
            badge: options.badge || '/default-badge.png',
            tag: options.tag || `notification-${Date.now()}`,
            data: options.data || {},
            requireInteraction: options.requireInteraction || false,
            silent: options.silent || false,
            timestamp: options.timestamp || Date.now()
        };

        // Add actions if supported
        if (this.supportedFeatures.actions && options.actions) {
            notificationOptions.actions = options.actions;
        }

        // Add image if supported
        if (this.supportedFeatures.image && options.image) {
            notificationOptions.image = options.image;
        }

        // Add vibration if supported
        if (this.supportedFeatures.vibrate && options.vibrate) {
            notificationOptions.vibrate = options.vibrate;
        }

        const notification = new Notification(title, notificationOptions);

        // Set up event handlers
        if (options.onClick) {
            notification.onclick = options.onClick;
        }

        if (options.onClose) {
            notification.onclose = options.onClose;
        }

        if (options.onError) {
            notification.onerror = options.onError;
        }

        return notification;
    }

    async showWithActions(title, body, actions) {
        await this.requestPermission();

        return this.show(title, {
            body,
            actions,
            tag: 'action-notification',
            requireInteraction: true,
            onClick: (event) => {
                console.log('Notification clicked:', event);
                window.focus();
                event.target.close();
            }
        });
    }

    // Group notifications by tag
    showGrouped(title, body, tag) {
        return this.show(title, {
            body,
            tag,
            renotify: true // Alert user even if same tag
        });
    }

    // Close notification by tag
    closeByTag(tag) {
        // This would need service worker support
        console.log(`Closing notification with tag: ${tag}`);
    }
}

// Usage
const notificationManager = new NotificationManager();

// Basic notification
document.getElementById('basicNotify').onclick = async () => {
    await notificationManager.requestPermission();
    notificationManager.show('New Message', {
        body: 'You have a new message from John',
        icon: '/user-icon.png',
        tag: 'message-notification'
    });
};

// Notification with actions
document.getElementById('actionNotify').onclick = async () => {
    notificationManager.showWithActions(
        'Meeting Reminder',
        'Your meeting starts in 5 minutes',
        [
            { action: 'join', title: 'Join Now' },
            { action: 'snooze', title: 'Snooze' },
            { action: 'dismiss', title: 'Dismiss' }
        ]
    );
};

// Grouped notifications
document.getElementById('groupedNotify').onclick = async () => {
    await notificationManager.requestPermission();

    // These will replace each other due to same tag
    notificationManager.showGrouped('Email', 'New email from Alice', 'email-group');
    setTimeout(() => {
        notificationManager.showGrouped('Email', '2 new emails', 'email-group');
    }, 2000);
};
```

### Service Worker Notifications

```javascript
// Service worker registration and notification
async function registerServiceWorkerAndNotify() {
    if (!('serviceWorker' in navigator)) {
        console.error('Service Worker not supported');
        return;
    }

    try {
        // Register service worker
        const registration = await navigator.serviceWorker.register('/sw.js');
        console.log('Service Worker registered:', registration);

        // Request notification permission
        const permission = await Notification.requestPermission();

        if (permission !== 'granted') {
            console.error('Notification permission denied');
            return;
        }

        // Show notification via service worker
        registration.showNotification('Hello from Service Worker!', {
            body: 'This notification will work even when the page is closed',
            icon: '/icon.png',
            badge: '/badge.png',
            tag: 'sw-notification',
            actions: [
                { action: 'open', title: 'Open App' },
                { action: 'close', title: 'Dismiss' }
            ],
            data: {
                url: '/',
                timestamp: Date.now()
            }
        });

    } catch (error) {
        console.error('Error:', error);
    }
}

// Service worker file (sw.js)
/*
self.addEventListener('notificationclick', (event) => {
    console.log('Notification clicked:', event);

    event.notification.close();

    if (event.action === 'open') {
        // Open the app
        event.waitUntil(
            clients.openWindow(event.notification.data.url)
        );
    }
});

self.addEventListener('notificationclose', (event) => {
    console.log('Notification closed:', event);
});
*/
```

---

## Camera and Microphone (Media Devices)

### Basic Media Access

```javascript
// Request camera access
async function requestCameraAccess() {
    try {
        const stream = await navigator.mediaDevices.getUserMedia({
            video: true
        });

        // Display video stream
        const videoElement = document.getElementById('video');
        videoElement.srcObject = stream;

        console.log('Camera access granted');
        return stream;

    } catch (error) {
        console.error('Camera access denied:', error);
        handleMediaError(error);
        return null;
    }
}

// Request microphone access
async function requestMicrophoneAccess() {
    try {
        const stream = await navigator.mediaDevices.getUserMedia({
            audio: true
        });

        console.log('Microphone access granted');
        return stream;

    } catch (error) {
        console.error('Microphone access denied:', error);
        handleMediaError(error);
        return null;
    }
}

// Request both camera and microphone
async function requestAudioVideoAccess() {
    try {
        const stream = await navigator.mediaDevices.getUserMedia({
            video: true,
            audio: true
        });

        console.log('Audio and video access granted');
        return stream;

    } catch (error) {
        console.error('Media access denied:', error);
        handleMediaError(error);
        return null;
    }
}

function handleMediaError(error) {
    switch (error.name) {
        case 'NotAllowedError':
        case 'PermissionDeniedError':
            console.error('Permission denied by user');
            showMessage('Please allow access to camera/microphone');
            break;

        case 'NotFoundError':
        case 'DevicesNotFoundError':
            console.error('No camera/microphone found');
            showMessage('No camera or microphone detected');
            break;

        case 'NotReadableError':
        case 'TrackStartError':
            console.error('Device already in use');
            showMessage('Camera/microphone is already in use');
            break;

        case 'OverconstrainedError':
        case 'ConstraintNotSatisfiedError':
            console.error('Constraints cannot be satisfied');
            showMessage('Camera/microphone does not meet requirements');
            break;

        default:
            console.error('Unknown error:', error);
            showMessage('An error occurred accessing media devices');
    }
}
```

### Advanced Media Constraints

```javascript
// Advanced camera constraints
class MediaDeviceManager {
    constructor() {
        this.stream = null;
    }

    async getDevices() {
        const devices = await navigator.mediaDevices.enumerateDevices();

        return {
            videoInputs: devices.filter(device => device.kind === 'videoinput'),
            audioInputs: devices.filter(device => device.kind === 'audioinput'),
            audioOutputs: devices.filter(device => device.kind === 'audiooutput')
        };
    }

    async requestCamera(constraints = {}) {
        const defaultConstraints = {
            video: {
                width: { min: 640, ideal: 1280, max: 1920 },
                height: { min: 480, ideal: 720, max: 1080 },
                frameRate: { ideal: 30, max: 60 },
                facingMode: 'user' // 'user' (front) or 'environment' (back)
            }
        };

        const finalConstraints = this.mergeConstraints(defaultConstraints, constraints);

        try {
            this.stream = await navigator.mediaDevices.getUserMedia(finalConstraints);
            return this.stream;
        } catch (error) {
            console.error('Camera access error:', error);
            throw error;
        }
    }

    async requestMicrophone(constraints = {}) {
        const defaultConstraints = {
            audio: {
                echoCancellation: true,
                noiseSuppression: true,
                autoGainControl: true,
                sampleRate: { ideal: 48000 }
            }
        };

        const finalConstraints = this.mergeConstraints(defaultConstraints, constraints);

        try {
            this.stream = await navigator.mediaDevices.getUserMedia(finalConstraints);
            return this.stream;
        } catch (error) {
            console.error('Microphone access error:', error);
            throw error;
        }
    }

    async switchCamera(facingMode = 'environment') {
        this.stopStream();

        return await this.requestCamera({
            video: {
                facingMode: { exact: facingMode }
            }
        });
    }

    async getDisplayMedia(constraints = {}) {
        const defaultConstraints = {
            video: {
                cursor: 'always',
                displaySurface: 'monitor'
            },
            audio: false
        };

        const finalConstraints = this.mergeConstraints(defaultConstraints, constraints);

        try {
            this.stream = await navigator.mediaDevices.getDisplayMedia(finalConstraints);
            return this.stream;
        } catch (error) {
            console.error('Screen share error:', error);
            throw error;
        }
    }

    stopStream() {
        if (this.stream) {
            this.stream.getTracks().forEach(track => {
                track.stop();
            });
            this.stream = null;
        }
    }

    muteAudio(mute = true) {
        if (this.stream) {
            this.stream.getAudioTracks().forEach(track => {
                track.enabled = !mute;
            });
        }
    }

    muteVideo(mute = true) {
        if (this.stream) {
            this.stream.getVideoTracks().forEach(track => {
                track.enabled = !mute;
            });
        }
    }

    getStreamInfo() {
        if (!this.stream) {
            return null;
        }

        return {
            videoTracks: this.stream.getVideoTracks().map(track => ({
                id: track.id,
                label: track.label,
                enabled: track.enabled,
                muted: track.muted,
                settings: track.getSettings()
            })),
            audioTracks: this.stream.getAudioTracks().map(track => ({
                id: track.id,
                label: track.label,
                enabled: track.enabled,
                muted: track.muted,
                settings: track.getSettings()
            }))
        };
    }

    mergeConstraints(defaults, custom) {
        return {
            ...defaults,
            ...custom,
            video: { ...defaults.video, ...custom.video },
            audio: custom.audio !== undefined
                ? (typeof custom.audio === 'object'
                    ? { ...defaults.audio, ...custom.audio }
                    : custom.audio)
                : defaults.audio
        };
    }
}

// Usage
const mediaManager = new MediaDeviceManager();

// Get available devices
const devices = await mediaManager.getDevices();
console.log('Video inputs:', devices.videoInputs);
console.log('Audio inputs:', devices.audioInputs);

// Request camera with specific constraints
const videoStream = await mediaManager.requestCamera({
    video: {
        width: { ideal: 1920 },
        height: { ideal: 1080 },
        frameRate: { ideal: 60 },
        facingMode: 'user'
    }
});

document.getElementById('video').srcObject = videoStream;

// Request microphone
const audioStream = await mediaManager.requestMicrophone({
    audio: {
        echoCancellation: true,
        noiseSuppression: true
    }
});

// Switch to back camera
document.getElementById('switchCamera').onclick = async () => {
    try {
        const stream = await mediaManager.switchCamera('environment');
        document.getElementById('video').srcObject = stream;
    } catch (error) {
        console.error('Failed to switch camera:', error);
    }
};

// Mute/unmute
document.getElementById('muteAudio').onclick = () => {
    mediaManager.muteAudio(true);
};

document.getElementById('unmuteAudio').onclick = () => {
    mediaManager.muteAudio(false);
};

// Screen sharing
document.getElementById('shareScreen').onclick = async () => {
    try {
        const stream = await mediaManager.getDisplayMedia({
            video: true,
            audio: true
        });
        document.getElementById('video').srcObject = stream;
    } catch (error) {
        console.error('Screen share failed:', error);
    }
};

// Stop all streams
document.getElementById('stop').onclick = () => {
    mediaManager.stopStream();
    document.getElementById('video').srcObject = null;
};
```

### Recording Media

```javascript
// Record video/audio
class MediaRecorder {
    constructor() {
        this.recorder = null;
        this.chunks = [];
    }

    startRecording(stream, mimeType = 'video/webm') {
        if (!MediaRecorder.isTypeSupported(mimeType)) {
            console.warn(`${mimeType} not supported, using default`);
            mimeType = '';
        }

        this.recorder = new MediaRecorder(stream, { mimeType });
        this.chunks = [];

        this.recorder.ondataavailable = (event) => {
            if (event.data.size > 0) {
                this.chunks.push(event.data);
            }
        };

        this.recorder.onstop = () => {
            const blob = new Blob(this.chunks, { type: mimeType || 'video/webm' });
            const url = URL.createObjectURL(blob);

            // Create download link
            const a = document.createElement('a');
            a.href = url;
            a.download = `recording-${Date.now()}.webm`;
            a.click();

            URL.revokeObjectURL(url);
        };

        this.recorder.start();
        console.log('Recording started');
    }

    stopRecording() {
        if (this.recorder && this.recorder.state !== 'inactive') {
            this.recorder.stop();
            console.log('Recording stopped');
        }
    }

    pauseRecording() {
        if (this.recorder && this.recorder.state === 'recording') {
            this.recorder.pause();
            console.log('Recording paused');
        }
    }

    resumeRecording() {
        if (this.recorder && this.recorder.state === 'paused') {
            this.recorder.resume();
            console.log('Recording resumed');
        }
    }

    getState() {
        return this.recorder ? this.recorder.state : 'inactive';
    }
}

// Usage
const recorder = new MediaRecorder();
const mediaManager = new MediaDeviceManager();

document.getElementById('startRecording').onclick = async () => {
    const stream = await mediaManager.requestCamera({ video: true, audio: true });
    document.getElementById('video').srcObject = stream;
    recorder.startRecording(stream);
};

document.getElementById('stopRecording').onclick = () => {
    recorder.stopRecording();
    mediaManager.stopStream();
};
```

---

## Clipboard API

### Reading from Clipboard

```javascript
// Read text from clipboard
async function readClipboardText() {
    try {
        const text = await navigator.clipboard.readText();
        console.log('Clipboard text:', text);
        return text;
    } catch (error) {
        console.error('Failed to read clipboard:', error);
        return null;
    }
}

// Read multiple formats from clipboard
async function readClipboard() {
    try {
        const clipboardItems = await navigator.clipboard.read();

        for (const item of clipboardItems) {
            console.log('Available types:', item.types);

            // Read text
            if (item.types.includes('text/plain')) {
                const blob = await item.getType('text/plain');
                const text = await blob.text();
                console.log('Text:', text);
            }

            // Read HTML
            if (item.types.includes('text/html')) {
                const blob = await item.getType('text/html');
                const html = await blob.text();
                console.log('HTML:', html);
            }

            // Read image
            if (item.types.includes('image/png')) {
                const blob = await item.getType('image/png');
                const url = URL.createObjectURL(blob);
                console.log('Image URL:', url);

                // Display image
                const img = document.createElement('img');
                img.src = url;
                document.body.appendChild(img);
            }
        }

        return clipboardItems;
    } catch (error) {
        console.error('Failed to read clipboard:', error);
        return null;
    }
}

// Usage with permission check
async function pasteFromClipboard() {
    // Check permission
    const permissionStatus = await navigator.permissions.query({
        name: 'clipboard-read'
    });

    if (permissionStatus.state === 'denied') {
        console.error('Clipboard read permission denied');
        return;
    }

    const text = await readClipboardText();

    if (text) {
        document.getElementById('pasteArea').value = text;
    }
}

document.getElementById('pasteButton').onclick = pasteFromClipboard;
```

### Writing to Clipboard

```javascript
// Write text to clipboard
async function writeClipboardText(text) {
    try {
        await navigator.clipboard.writeText(text);
        console.log('Text copied to clipboard');
        return true;
    } catch (error) {
        console.error('Failed to write to clipboard:', error);
        return false;
    }
}

// Write multiple formats to clipboard
async function writeToClipboard(data) {
    try {
        const clipboardItems = [];

        // Prepare text
        if (data.text) {
            const textBlob = new Blob([data.text], { type: 'text/plain' });
            clipboardItems.push(new ClipboardItem({
                'text/plain': textBlob
            }));
        }

        // Prepare HTML
        if (data.html) {
            const htmlBlob = new Blob([data.html], { type: 'text/html' });
            clipboardItems.push(new ClipboardItem({
                'text/html': htmlBlob,
                'text/plain': new Blob([data.text || ''], { type: 'text/plain' })
            }));
        }

        // Prepare image
        if (data.image) {
            clipboardItems.push(new ClipboardItem({
                'image/png': data.image
            }));
        }

        await navigator.clipboard.write(clipboardItems);
        console.log('Data copied to clipboard');
        return true;

    } catch (error) {
        console.error('Failed to write to clipboard:', error);
        return false;
    }
}

// Copy image to clipboard
async function copyImageToClipboard(imageUrl) {
    try {
        const response = await fetch(imageUrl);
        const blob = await response.blob();

        await navigator.clipboard.write([
            new ClipboardItem({
                [blob.type]: blob
            })
        ]);

        console.log('Image copied to clipboard');
        return true;

    } catch (error) {
        console.error('Failed to copy image:', error);
        return false;
    }
}

// Copy from canvas
async function copyCanvasToClipboard(canvas) {
    try {
        const blob = await new Promise(resolve => {
            canvas.toBlob(resolve, 'image/png');
        });

        await navigator.clipboard.write([
            new ClipboardItem({
                'image/png': blob
            })
        ]);

        console.log('Canvas content copied to clipboard');
        return true;

    } catch (error) {
        console.error('Failed to copy canvas:', error);
        return false;
    }
}

// Usage
document.getElementById('copyButton').onclick = () => {
    const text = document.getElementById('copyText').value;
    writeClipboardText(text);
};

document.getElementById('copyRichButton').onclick = () => {
    writeToClipboard({
        text: 'Hello, World!',
        html: '<p><strong>Hello</strong>, <em>World</em>!</p>'
    });
};

document.getElementById('copyImageButton').onclick = () => {
    copyImageToClipboard('/path/to/image.png');
};
```

### Clipboard Manager

```javascript
// Complete clipboard manager
class ClipboardManager {
    constructor() {
        this.supportsClipboard = 'clipboard' in navigator;
        this.supportsClipboardRead = this.supportsClipboard && 'read' in navigator.clipboard;
        this.supportsClipboardWrite = this.supportsClipboard && 'write' in navigator.clipboard;
    }

    async checkPermissions() {
        if (!this.supportsClipboard) {
            return { read: 'unsupported', write: 'unsupported' };
        }

        const permissions = {};

        try {
            const readPermission = await navigator.permissions.query({
                name: 'clipboard-read'
            });
            permissions.read = readPermission.state;
        } catch {
            permissions.read = 'unsupported';
        }

        try {
            const writePermission = await navigator.permissions.query({
                name: 'clipboard-write'
            });
            permissions.write = writePermission.state;
        } catch {
            permissions.write = 'unsupported';
        }

        return permissions;
    }

    async readText() {
        if (!this.supportsClipboard) {
            throw new Error('Clipboard API not supported');
        }

        try {
            return await navigator.clipboard.readText();
        } catch (error) {
            console.error('Clipboard read error:', error);
            throw error;
        }
    }

    async writeText(text) {
        if (!this.supportsClipboard) {
            // Fallback to execCommand
            return this.fallbackCopyText(text);
        }

        try {
            await navigator.clipboard.writeText(text);
            return true;
        } catch (error) {
            console.error('Clipboard write error:', error);
            // Try fallback
            return this.fallbackCopyText(text);
        }
    }

    async readData() {
        if (!this.supportsClipboardRead) {
            throw new Error('Clipboard read not supported');
        }

        try {
            const items = await navigator.clipboard.read();
            const result = [];

            for (const item of items) {
                const itemData = {};

                for (const type of item.types) {
                    const blob = await item.getType(type);
                    itemData[type] = blob;

                    // Convert text types to string
                    if (type.startsWith('text/')) {
                        itemData[`${type}_text`] = await blob.text();
                    }
                }

                result.push(itemData);
            }

            return result;
        } catch (error) {
            console.error('Clipboard read error:', error);
            throw error;
        }
    }

    async writeData(items) {
        if (!this.supportsClipboardWrite) {
            throw new Error('Clipboard write not supported');
        }

        try {
            const clipboardItems = items.map(item => {
                return new ClipboardItem(item);
            });

            await navigator.clipboard.write(clipboardItems);
            return true;
        } catch (error) {
            console.error('Clipboard write error:', error);
            throw error;
        }
    }

    // Fallback for older browsers
    fallbackCopyText(text) {
        const textArea = document.createElement('textarea');
        textArea.value = text;
        textArea.style.position = 'fixed';
        textArea.style.left = '-999999px';
        textArea.style.top = '-999999px';
        document.body.appendChild(textArea);

        textArea.focus();
        textArea.select();

        try {
            const successful = document.execCommand('copy');
            document.body.removeChild(textArea);
            return successful;
        } catch (error) {
            console.error('Fallback copy failed:', error);
            document.body.removeChild(textArea);
            return false;
        }
    }

    // Listen for clipboard events
    onCopy(callback) {
        document.addEventListener('copy', (event) => {
            callback(event);
        });
    }

    onCut(callback) {
        document.addEventListener('cut', (event) => {
            callback(event);
        });
    }

    onPaste(callback) {
        document.addEventListener('paste', (event) => {
            callback(event);
        });
    }
}

// Usage
const clipboardManager = new ClipboardManager();

// Check permissions
const permissions = await clipboardManager.checkPermissions();
console.log('Clipboard permissions:', permissions);

// Copy text
document.getElementById('copyButton').onclick = async () => {
    const text = document.getElementById('textInput').value;
    const success = await clipboardManager.writeText(text);

    if (success) {
        showMessage('Copied to clipboard!');
    }
};

// Paste text
document.getElementById('pasteButton').onclick = async () => {
    try {
        const text = await clipboardManager.readText();
        document.getElementById('textInput').value = text;
    } catch (error) {
        showMessage('Failed to read clipboard');
    }
};

// Listen for paste events
clipboardManager.onPaste((event) => {
    console.log('Paste event:', event);

    const items = event.clipboardData.items;
    for (const item of items) {
        console.log('Pasted item type:', item.type);

        if (item.type.startsWith('image/')) {
            const blob = item.getAsFile();
            const url = URL.createObjectURL(blob);

            const img = document.createElement('img');
            img.src = url;
            document.body.appendChild(img);
        }
    }
});
```

---

## Other Permissions

### Wake Lock API (Prevent Screen Sleep)

```javascript
// Request wake lock
async function requestWakeLock() {
    if (!('wakeLock' in navigator)) {
        console.error('Wake Lock API not supported');
        return null;
    }

    try {
        const wakeLock = await navigator.wakeLock.request('screen');
        console.log('Wake lock acquired');

        wakeLock.addEventListener('release', () => {
            console.log('Wake lock released');
        });

        return wakeLock;
    } catch (error) {
        console.error('Wake lock error:', error);
        return null;
    }
}

// Release wake lock
function releaseWakeLock(wakeLock) {
    if (wakeLock) {
        wakeLock.release()
            .then(() => {
                console.log('Wake lock released manually');
            })
            .catch((error) => {
                console.error('Failed to release wake lock:', error);
            });
    }
}

// Manage wake lock
class WakeLockManager {
    constructor() {
        this.wakeLock = null;
        this.handleVisibilityChange = this.handleVisibilityChange.bind(this);
    }

    async request() {
        if (!('wakeLock' in navigator)) {
            throw new Error('Wake Lock not supported');
        }

        try {
            this.wakeLock = await navigator.wakeLock.request('screen');

            this.wakeLock.addEventListener('release', () => {
                console.log('Wake lock released');
                this.wakeLock = null;
            });

            // Re-request wake lock when page becomes visible
            document.addEventListener('visibilitychange', this.handleVisibilityChange);

            console.log('Wake lock acquired');
            return this.wakeLock;
        } catch (error) {
            console.error('Wake lock request failed:', error);
            throw error;
        }
    }

    async release() {
        if (this.wakeLock) {
            await this.wakeLock.release();
            document.removeEventListener('visibilitychange', this.handleVisibilityChange);
            this.wakeLock = null;
        }
    }

    async handleVisibilityChange() {
        if (document.visibilityState === 'visible' && !this.wakeLock) {
            await this.request();
        }
    }

    isActive() {
        return this.wakeLock !== null;
    }
}

// Usage
const wakeLockManager = new WakeLockManager();

document.getElementById('preventSleep').onclick = async () => {
    await wakeLockManager.request();
    showMessage('Screen will stay awake');
};

document.getElementById('allowSleep').onclick = async () => {
    await wakeLockManager.release();
    showMessage('Screen can sleep now');
};
```

### Persistent Storage

```javascript
// Request persistent storage
async function requestPersistentStorage() {
    if (!navigator.storage || !navigator.storage.persist) {
        console.error('Persistent storage not supported');
        return false;
    }

    // Check if already persisted
    const persisted = await navigator.storage.persisted();

    if (persisted) {
        console.log('Storage is already persistent');
        return true;
    }

    // Request persistence
    const granted = await navigator.storage.persist();

    if (granted) {
        console.log('Persistent storage granted');
    } else {
        console.log('Persistent storage denied');
    }

    return granted;
}

// Check storage quota and persistence
async function checkStorage() {
    if (!navigator.storage) {
        console.error('Storage API not supported');
        return null;
    }

    const persisted = await navigator.storage.persisted();
    const estimate = await navigator.storage.estimate();

    return {
        persisted,
        usage: estimate.usage,
        quota: estimate.quota,
        percentUsed: (estimate.usage / estimate.quota) * 100
    };
}

// Usage
const storageInfo = await checkStorage();
console.log('Storage info:', storageInfo);

if (!storageInfo.persisted) {
    const granted = await requestPersistentStorage();
    console.log('Persistent storage:', granted ? 'granted' : 'denied');
}
```

---

## Best Practices

### Permission Request Guidelines

```javascript
// 1. Request permissions in context
class ContextualPermissionManager {
    async requestPermissionInContext(feature, permissionName, requestFn) {
        // Show explanation first
        const userUnderstands = await this.showExplanation(feature);

        if (!userUnderstands) {
            return false;
        }

        // Check current permission state
        const state = await this.checkPermissionState(permissionName);

        if (state === 'denied') {
            this.showSettingsInstructions(feature);
            return false;
        }

        if (state === 'granted') {
            return true;
        }

        // Request permission
        return await requestFn();
    }

    async showExplanation(feature) {
        return new Promise((resolve) => {
            const modal = document.createElement('div');
            modal.innerHTML = `
                <div class="modal">
                    <h2>Enable ${feature}</h2>
                    <p>We need this permission to provide you with ${feature} functionality.</p>
                    <button id="continue">Continue</button>
                    <button id="cancel">Cancel</button>
                </div>
            `;

            document.body.appendChild(modal);

            document.getElementById('continue').onclick = () => {
                modal.remove();
                resolve(true);
            };

            document.getElementById('cancel').onclick = () => {
                modal.remove();
                resolve(false);
            };
        });
    }

    async checkPermissionState(permissionName) {
        try {
            const result = await navigator.permissions.query({ name: permissionName });
            return result.state;
        } catch {
            return 'prompt';
        }
    }

    showSettingsInstructions(feature) {
        alert(`${feature} is blocked. Please enable it in your browser settings.`);
    }
}

// 2. Progressive enhancement
function enableFeatureIfSupported(feature) {
    const button = document.getElementById(`enable-${feature}`);

    if (!isFeatureSupported(feature)) {
        button.disabled = true;
        button.textContent = `${feature} not supported`;
        return;
    }

    button.onclick = async () => {
        const granted = await requestFeaturePermission(feature);

        if (granted) {
            enableFeature(feature);
        }
    };
}

function isFeatureSupported(feature) {
    switch (feature) {
        case 'notifications':
            return 'Notification' in window;
        case 'geolocation':
            return 'geolocation' in navigator;
        case 'camera':
            return 'mediaDevices' in navigator;
        default:
            return false;
    }
}

// 3. Handle permission revocation gracefully
async function monitorPermissionChanges(permissionName, onGranted, onDenied) {
    const result = await navigator.permissions.query({ name: permissionName });

    const handler = () => {
        if (result.state === 'granted') {
            onGranted();
        } else if (result.state === 'denied') {
            onDenied();
        }
    };

    result.addEventListener('change', handler);

    // Call initially
    handler();

    // Return cleanup function
    return () => {
        result.removeEventListener('change', handler);
    };
}

// Usage
const cleanup = await monitorPermissionChanges(
    'notifications',
    () => console.log('Notifications enabled'),
    () => console.log('Notifications disabled')
);

// Clean up when component unmounts
// cleanup();
```

### Privacy and Security Considerations

```javascript
// Privacy-conscious permission handling
class PrivacyAwarePermissionManager {
    constructor() {
        this.permissionExplanations = new Map([
            ['geolocation', 'We use your location to show nearby places'],
            ['notifications', 'We send you important updates'],
            ['camera', 'We need camera access for video calls'],
            ['microphone', 'We need microphone access for voice chat']
        ]);
    }

    async requestWithConsent(permissionName, requestFn) {
        // Show privacy policy link
        const consented = await this.showPrivacyConsent(permissionName);

        if (!consented) {
            console.log('User declined consent');
            return false;
        }

        // Request permission
        const granted = await requestFn();

        // Log permission request (for compliance)
        this.logPermissionRequest(permissionName, granted);

        return granted;
    }

    async showPrivacyConsent(permissionName) {
        const explanation = this.permissionExplanations.get(permissionName);

        return new Promise((resolve) => {
            const modal = document.createElement('div');
            modal.innerHTML = `
                <div class="consent-modal">
                    <h3>Permission Request</h3>
                    <p>${explanation}</p>
                    <p>We respect your privacy. <a href="/privacy">Learn more</a></p>
                    <button id="accept">Accept</button>
                    <button id="decline">Decline</button>
                </div>
            `;

            document.body.appendChild(modal);

            document.getElementById('accept').onclick = () => {
                modal.remove();
                resolve(true);
            };

            document.getElementById('decline').onclick = () => {
                modal.remove();
                resolve(false);
            };
        });
    }

    logPermissionRequest(permissionName, granted) {
        const log = {
            permission: permissionName,
            granted,
            timestamp: new Date().toISOString(),
            userAgent: navigator.userAgent
        };

        console.log('Permission log:', log);

        // Send to analytics/compliance system
        // sendToAnalytics(log);
    }

    // Allow users to revoke permissions from UI
    showPermissionManager() {
        // UI to show all granted permissions and allow revocation
        console.log('Showing permission manager UI');
    }
}
```

---

## Error Handling

### Comprehensive Error Handling

```javascript
// Error handling for all permission types
class PermissionErrorHandler {
    handleGeolocationError(error) {
        const messages = {
            [error.PERMISSION_DENIED]: {
                title: 'Location Access Denied',
                message: 'Please enable location access in your browser settings.',
                action: 'showSettingsGuide'
            },
            [error.POSITION_UNAVAILABLE]: {
                title: 'Location Unavailable',
                message: 'Unable to determine your location. Please try again.',
                action: 'retry'
            },
            [error.TIMEOUT]: {
                title: 'Location Timeout',
                message: 'Location request timed out. Please try again.',
                action: 'retry'
            }
        };

        const errorInfo = messages[error.code] || {
            title: 'Location Error',
            message: 'An unknown error occurred.',
            action: 'none'
        };

        this.showError(errorInfo);
    }

    handleMediaError(error) {
        const errorMap = {
            'NotAllowedError': {
                title: 'Permission Denied',
                message: 'Please allow access to your camera/microphone.',
                action: 'showSettingsGuide'
            },
            'NotFoundError': {
                title: 'Device Not Found',
                message: 'No camera or microphone detected.',
                action: 'none'
            },
            'NotReadableError': {
                title: 'Device In Use',
                message: 'Camera/microphone is already being used by another application.',
                action: 'closeOtherApps'
            },
            'OverconstrainedError': {
                title: 'Requirements Not Met',
                message: 'Your device does not meet the requirements.',
                action: 'lowerRequirements'
            },
            'SecurityError': {
                title: 'Security Error',
                message: 'Media access requires HTTPS.',
                action: 'useHttps'
            }
        };

        const errorInfo = errorMap[error.name] || {
            title: 'Media Error',
            message: error.message || 'An unknown error occurred.',
            action: 'none'
        };

        this.showError(errorInfo);
    }

    handleClipboardError(error) {
        let errorInfo;

        if (error.name === 'NotAllowedError') {
            errorInfo = {
                title: 'Clipboard Access Denied',
                message: 'Please allow clipboard access.',
                action: 'showSettingsGuide'
            };
        } else {
            errorInfo = {
                title: 'Clipboard Error',
                message: 'Failed to access clipboard.',
                action: 'useFallback'
            };
        }

        this.showError(errorInfo);
    }

    showError(errorInfo) {
        console.error(`${errorInfo.title}: ${errorInfo.message}`);

        // Show user-friendly error message
        const errorDiv = document.createElement('div');
        errorDiv.className = 'error-notification';
        errorDiv.innerHTML = `
            <h4>${errorInfo.title}</h4>
            <p>${errorInfo.message}</p>
            ${this.getActionButton(errorInfo.action)}
        `;

        document.body.appendChild(errorDiv);

        // Auto-remove after 5 seconds
        setTimeout(() => errorDiv.remove(), 5000);
    }

    getActionButton(action) {
        const buttons = {
            'showSettingsGuide': '<button onclick="showSettingsGuide()">Show How</button>',
            'retry': '<button onclick="retryPermission()">Retry</button>',
            'closeOtherApps': '<button>Close Other Apps</button>',
            'lowerRequirements': '<button onclick="lowerRequirements()">Use Basic Mode</button>',
            'useHttps': '<button>Learn About HTTPS</button>',
            'useFallback': '<button onclick="useFallback()">Use Alternative</button>',
            'none': ''
        };

        return buttons[action] || '';
    }
}

// Usage
const errorHandler = new PermissionErrorHandler();

// Handle geolocation error
navigator.geolocation.getCurrentPosition(
    (position) => console.log(position),
    (error) => errorHandler.handleGeolocationError(error)
);

// Handle media error
navigator.mediaDevices.getUserMedia({ video: true })
    .then(stream => console.log(stream))
    .catch(error => errorHandler.handleMediaError(error));

// Handle clipboard error
navigator.clipboard.readText()
    .then(text => console.log(text))
    .catch(error => errorHandler.handleClipboardError(error));
```

---

## Interview Questions

### Q1: What is the Permissions API and how does it differ from directly requesting permissions?
**Answer:** The Permissions API provides a unified way to query the status of API permissions without triggering the permission request dialog. It allows you to check if a permission has been granted, denied, or is in the prompt state, and listen for permission changes.

**Key differences:**
- **Query without requesting**: Check permission status without showing dialog to user
- **Unified interface**: Consistent API across different permission types
- **Change notifications**: Listen for permission state changes via `onchange` event
- **Predictable behavior**: Know the permission state before attempting to use a feature

**Example:**
```javascript
// Direct request (shows dialog if not decided)
const permission = await Notification.requestPermission();

// Permissions API (checks without dialog)
const result = await navigator.permissions.query({ name: 'notifications' });
console.log(result.state); // 'granted', 'denied', or 'prompt'

result.onchange = () => {
    console.log('Permission changed:', result.state);
};
```

**Use cases:**
- Check permission before showing feature UI
- Update UI based on permission state
- Monitor permission revocation
- Avoid annoying users with unnecessary permission dialogs

### Q2: What are the three permission states and how should you handle each?
**Answer:** The three permission states are `granted`, `denied`, and `prompt`. Each requires different handling:

**1. 'granted' - Permission has been granted:**
- Enable the feature immediately
- No need to show permission request UI
- Feature can be used without restrictions

**2. 'denied' - Permission has been explicitly denied:**
- Feature cannot be used
- Cannot request permission again (browser blocks it)
- Show instructions to enable in browser settings
- Provide alternative functionality if possible
- Don't repeatedly ask or annoy the user

**3. 'prompt' - Permission has not been requested yet:**
- Permission can be requested
- Show contextual explanation before requesting
- Request only in response to user action
- Provide option to use feature without permission if possible

**Implementation:**
```javascript
const result = await navigator.permissions.query({ name: 'notifications' });

switch (result.state) {
    case 'granted':
        // Enable feature
        showNotifications();
        break;

    case 'denied':
        // Show settings instructions
        showMessage('Notifications are blocked. Enable them in browser settings.');
        hideNotificationButton();
        break;

    case 'prompt':
        // Show request button
        showNotificationButton();
        break;
}
```

### Q3: Why should permission requests be triggered by user actions rather than on page load?
**Answer:** Requesting permissions in response to user actions (not on page load) is a best practice for several reasons:

**User experience:**
- **Context**: Users understand why permission is needed
- **Control**: Users feel in control, not ambushed
- **Trust**: Builds trust by respecting user agency
- **Reduces friction**: Higher acceptance rate when contextual

**Technical reasons:**
- **Browser restrictions**: Some browsers may block requests not tied to user interaction
- **Security**: Prevents malicious sites from spamming permission requests
- **Permission fatigue**: Reduces user annoyance and prompt blindness

**Good vs Bad examples:**
```javascript
// Bad: Request on page load
window.addEventListener('load', () => {
    Notification.requestPermission(); // Annoying!
});

// Good: Request when user clicks button
document.getElementById('enableNotifications').onclick = async () => {
    // Show explanation first
    const explanation = await showWhyWeNeedThis();

    if (explanation.userConsents) {
        const permission = await Notification.requestPermission();

        if (permission === 'granted') {
            showNotifications();
        }
    }
};

// Best: Progressive disclosure
// Show feature benefit í User expresses interest í Request permission
```

**Guidelines:**
1. Request permission when user explicitly enables a feature
2. Explain the benefit before requesting
3. Respect user's decision (don't ask repeatedly)
4. Provide alternative functionality when possible

### Q4: How do you handle permission denial gracefully?
**Answer:** Handling permission denial requires providing helpful feedback and alternatives without annoying the user:

**1. Detect denial state:**
```javascript
const result = await navigator.permissions.query({ name: 'notifications' });

if (result.state === 'denied') {
    handleDenied();
}
```

**2. Provide clear instructions:**
```javascript
function handleDenied() {
    showMessage({
        title: 'Notifications Blocked',
        message: 'To enable notifications:',
        steps: [
            'Click the lock icon in the address bar',
            'Allow notifications for this site',
            'Refresh the page'
        ],
        visual: 'settings-guide.png'
    });
}
```

**3. Offer alternatives:**
```javascript
// Instead of notifications, offer:
- In-app messaging
- Email notifications
- Browser tab title updates
- Visual indicators

if (notificationsBlocked) {
    enableInAppAlerts();
    offerEmailNotifications();
}
```

**4. Don't repeatedly ask:**
```javascript
// Track denial and don't ask again
if (localStorage.getItem('notificationsDenied') === 'true') {
    // Don't show permission request UI
    hideNotificationButton();
    showAlternativeOptions();
}
```

**5. Monitor permission changes:**
```javascript
// If user enables later, re-enable feature
result.onchange = () => {
    if (result.state === 'granted') {
        enableNotifications();
        localStorage.removeItem('notificationsDenied');
    }
};
```

**Best practices:**
- Never shame users for denying
- Don't break core functionality
- Make it easy to change decision later
- Respect user privacy choices

### Q5: What are the security implications of the Geolocation API?
**Answer:** The Geolocation API has significant privacy and security implications:

**Privacy concerns:**
1. **Sensitive data**: Location reveals physical whereabouts, routines, home/work addresses
2. **Tracking**: Can be used to track user movements over time
3. **Inference**: Location data can infer sensitive information (health clinics, religious sites, etc.)

**Security measures:**
1. **HTTPS required**: Geolocation only works on secure contexts (except localhost)
2. **User permission**: Explicit user consent required
3. **Per-origin**: Each origin must request permission separately
4. **No silent tracking**: Cannot access location without permission prompt

**Best practices for developers:**
```javascript
// 1. Request minimal accuracy
navigator.geolocation.getCurrentPosition(
    success,
    error,
    {
        enableHighAccuracy: false, // Lower precision for privacy
        maximumAge: 300000, // Cache for 5 minutes
        timeout: 10000
    }
);

// 2. Explain why you need location
async function requestLocation() {
    showMessage('We need your location to show nearby restaurants');
    const consented = await getUserConsent();

    if (consented) {
        getCurrentPosition();
    }
}

// 3. Don't store location unnecessarily
function processLocation(position) {
    const { latitude, longitude } = position.coords;

    // Use it, don't store it
    fetchNearbyPlaces(latitude, longitude);

    // Don't: saveToDatabase(latitude, longitude)
}

// 4. Provide clear privacy policy
showPrivacyPolicy('/privacy#location-usage');
```

**Attack vectors:**
- **Phishing**: Malicious sites requesting location
- **Data breaches**: Stored location data leaked
- **Third-party tracking**: Location data sold/shared

**User protections:**
- Can deny permission
- Can revoke permission anytime
- Browser shows indicator when location is accessed
- Fake location extensions/settings available

### Q6: How does the MediaDevices API handle camera and microphone permissions?
**Answer:** The MediaDevices API (`navigator.mediaDevices.getUserMedia()`) handles camera and microphone access with these key behaviors:

**Permission request flow:**
```javascript
// Request triggers browser permission dialog
const stream = await navigator.mediaDevices.getUserMedia({
    video: true,
    audio: true
});
```

**Key characteristics:**

1. **Combined prompt**: Camera and microphone requested together in one dialog
2. **Granular control**: User can grant/deny each separately
3. **Active indicator**: Browser shows camera/microphone icon when active
4. **Automatic cleanup**: Tracks should be stopped manually to avoid privacy leaks

**Implementation pattern:**
```javascript
class MediaManager {
    async requestAccess(constraints) {
        try {
            const stream = await navigator.mediaDevices.getUserMedia(constraints);

            // Check what was actually granted
            const videoTracks = stream.getVideoTracks();
            const audioTracks = stream.getAudioTracks();

            console.log('Video:', videoTracks.length > 0);
            console.log('Audio:', audioTracks.length > 0);

            return stream;

        } catch (error) {
            this.handleError(error);
            return null;
        }
    }

    handleError(error) {
        switch (error.name) {
            case 'NotAllowedError':
                console.error('User denied permission');
                break;
            case 'NotFoundError':
                console.error('No device found');
                break;
            case 'NotReadableError':
                console.error('Device in use');
                break;
        }
    }

    stopStream(stream) {
        // Important: Stop tracks to release device
        stream.getTracks().forEach(track => track.stop());
    }
}
```

**Security features:**
- **HTTPS required** (except localhost)
- **Persistent permission**: Remembered across sessions for same origin
- **Active use indicator**: Browser UI shows when camera/mic is active
- **Easy revocation**: User can revoke via browser UI

**Best practices:**
- Request only what you need (`video: true` OR `audio: true`)
- Stop tracks when done (releases device and privacy indicator)
- Handle all error cases gracefully
- Check constraints support with `getSupportedConstraints()`
- Provide fallback UI when permission denied

### Q7: What are the differences between reading and writing clipboard data?
**Answer:** Reading and writing clipboard data have different permission requirements, browser support, and use cases:

**Permission requirements:**

| Operation | Permission | User Interaction | Notes |
|-----------|-----------|------------------|-------|
| `writeText()` | None required | Optional | Usually doesn't need permission |
| `write()` | clipboard-write | Optional | May prompt in some browsers |
| `readText()` | clipboard-read | Required | Always requires permission |
| `read()` | clipboard-read | Required | Always requires permission |

**Writing to clipboard:**
```javascript
// Simple text - usually works without permission
await navigator.clipboard.writeText('Hello World');

// Complex data - may require permission
await navigator.clipboard.write([
    new ClipboardItem({
        'text/plain': new Blob(['Hello'], { type: 'text/plain' }),
        'text/html': new Blob(['<b>Hello</b>'], { type: 'text/html' })
    })
]);
```

**Reading from clipboard:**
```javascript
// Always requires permission
const text = await navigator.clipboard.readText();

// Check permission first
const result = await navigator.permissions.query({ name: 'clipboard-read' });

if (result.state === 'granted') {
    const text = await navigator.clipboard.readText();
} else {
    // Show message or use alternative
}
```

**Key differences:**

1. **Security model:**
   - Write: Low risk (user initiated copy)
   - Read: High risk (exposing private clipboard data)

2. **User interaction:**
   - Write: Can be triggered programmatically
   - Read: Requires user gesture in most browsers

3. **Permission persistence:**
   - Write: Usually no permission needed
   - Read: Permission state persisted

4. **Browser support:**
   - writeText(): Excellent
   - write(): Good
   - readText(): Good but restricted
   - read(): Good but restricted

**Best practices:**
```javascript
// Prefer writeText for simple cases
async function copy(text) {
    try {
        await navigator.clipboard.writeText(text);
        showMessage('Copied!');
    } catch (error) {
        // Fallback to execCommand
        fallbackCopy(text);
    }
}

// Request permission explicitly for read
async function paste() {
    const permission = await navigator.permissions.query({
        name: 'clipboard-read'
    });

    if (permission.state === 'denied') {
        showMessage('Clipboard access denied');
        return;
    }

    try {
        const text = await navigator.clipboard.readText();
        processText(text);
    } catch (error) {
        showMessage('Failed to read clipboard');
    }
}
```

### Q8: How do you implement progressive permission requests?
**Answer:** Progressive permission requests involve requesting permissions gradually as users engage with features, rather than all at once. This improves user experience and acceptance rates.

**Core principles:**

1. **Just-in-time requests**: Ask when feature is used
2. **Contextual explanation**: Explain why before requesting
3. **Feature discovery**: Let users discover features naturally
4. **Graceful degradation**: Work without permissions when possible

**Implementation:**
```javascript
class ProgressivePermissionManager {
    constructor() {
        this.requestedPermissions = new Set();
        this.explainedFeatures = new Set();
    }

    async enableFeature(featureName, permissionName, requestFn) {
        // Step 1: Check if already granted
        if (await this.isGranted(permissionName)) {
            return this.activateFeature(featureName);
        }

        // Step 2: Show feature benefit (first time only)
        if (!this.explainedFeatures.has(featureName)) {
            const interested = await this.showFeatureBenefit(featureName);
            this.explainedFeatures.add(featureName);

            if (!interested) {
                return false;
            }
        }

        // Step 3: Request permission (once per session)
        if (!this.requestedPermissions.has(permissionName)) {
            this.requestedPermissions.add(permissionName);
            const granted = await requestFn();

            if (granted) {
                return this.activateFeature(featureName);
            } else {
                // Store denial, don't ask again this session
                return false;
            }
        }

        return false;
    }

    async showFeatureBenefit(featureName) {
        // Show modal/tooltip explaining feature
        return showModal({
            title: `Enable ${featureName}?`,
            benefit: this.getFeatureBenefit(featureName),
            actions: ['Enable', 'Not Now']
        });
    }

    getFeatureBenefit(featureName) {
        const benefits = {
            'notifications': 'Get instant updates on important events',
            'location': 'Find nearby places and get better recommendations',
            'camera': 'Scan QR codes and take photos',
            'microphone': 'Use voice commands and audio messages'
        };

        return benefits[featureName] || 'Enhance your experience';
    }

    async isGranted(permissionName) {
        try {
            const result = await navigator.permissions.query({ name: permissionName });
            return result.state === 'granted';
        } catch {
            return false;
        }
    }

    activateFeature(featureName) {
        console.log(`Activating feature: ${featureName}`);
        return true;
    }
}

// Usage example
const manager = new ProgressivePermissionManager();

// User clicks "Share Location" button
document.getElementById('shareLocation').onclick = async () => {
    const enabled = await manager.enableFeature(
        'location',
        'geolocation',
        async () => {
            return new Promise((resolve) => {
                navigator.geolocation.getCurrentPosition(
                    () => resolve(true),
                    () => resolve(false)
                );
            });
        }
    );

    if (enabled) {
        showNearbyPlaces();
    }
};

// User enables notifications
document.getElementById('enableNotifications').onclick = async () => {
    const enabled = await manager.enableFeature(
        'notifications',
        'notifications',
        async () => {
            const result = await Notification.requestPermission();
            return result === 'granted';
        }
    );

    if (enabled) {
        subscribeToNotifications();
    }
};
```

**Benefits:**
- Higher acceptance rates (users understand context)
- Better user experience (not overwhelmed)
- Reduced permission fatigue
- Builds trust through transparency

**Timing examples:**
- **Notifications**: When user follows someone or creates first post
- **Location**: When user searches for "nearby" or clicks map
- **Camera**: When user taps "Scan QR Code" or "Take Photo"
- **Microphone**: When user enables voice commands

### Q9: What are common pitfalls when working with browser permissions?
**Answer:** Common pitfalls when working with browser permissions include:

**1. Requesting too early:**
```javascript
// Bad: Request on page load
window.onload = () => {
    Notification.requestPermission(); // User has no context!
};

// Good: Request when user needs it
button.onclick = () => {
    Notification.requestPermission(); // User initiated!
};
```

**2. Not checking permission state:**
```javascript
// Bad: Assume permission is granted
new Notification('Hello'); // May fail silently

// Good: Check first
if (Notification.permission === 'granted') {
    new Notification('Hello');
}
```

**3. Forgetting HTTPS requirement:**
```javascript
// Fails on HTTP (except localhost)
navigator.geolocation.getCurrentPosition(success, error);
```

**4. Not handling denial:**
```javascript
// Bad: No fallback
async function enableNotifications() {
    await Notification.requestPermission();
    // What if denied?
}

// Good: Handle all cases
async function enableNotifications() {
    const permission = await Notification.requestPermission();

    if (permission === 'granted') {
        setupNotifications();
    } else {
        offerEmailAlternative();
    }
}
```

**5. Not releasing media resources:**
```javascript
// Bad: Leaves camera indicator on
const stream = await navigator.mediaDevices.getUserMedia({ video: true });
// ... use stream ...
// Forgot to stop!

// Good: Always stop tracks
const stream = await navigator.mediaDevices.getUserMedia({ video: true });
// ... use stream ...
stream.getTracks().forEach(track => track.stop());
```

**6. Assuming permission persistence:**
```javascript
// Bad: Assume permission from yesterday
function showNotification() {
    new Notification('Hello'); // May fail if revoked
}

// Good: Check permission every time
async function showNotification() {
    if (Notification.permission === 'granted') {
        new Notification('Hello');
    } else if (Notification.permission === 'default') {
        await Notification.requestPermission();
    }
}
```

**7. Not monitoring permission changes:**
```javascript
// Bad: Don't monitor
const result = await navigator.permissions.query({ name: 'notifications' });
// User revokes permission in another tab - app doesn't know

// Good: Monitor changes
result.onchange = () => {
    if (result.state === 'denied') {
        disableNotificationFeature();
    }
};
```

**8. Poor error messages:**
```javascript
// Bad
alert('Error');

// Good
if (error.name === 'NotAllowedError') {
    showMessage('Camera access denied. To enable:  1. Click lock icon 2. Allow camera');
}
```

**9. Not providing alternatives:**
```javascript
// Bad: Feature completely broken without permission
if (!geolocationGranted) {
    throw new Error('Cannot proceed');
}

// Good: Provide fallback
if (!geolocationGranted) {
    showManualLocationInput();
}
```

**10. Requesting multiple permissions at once:**
```javascript
// Bad: Overwhelming
Promise.all([
    Notification.requestPermission(),
    navigator.mediaDevices.getUserMedia({ video: true }),
    navigator.geolocation.getCurrentPosition()
]);

// Good: Request progressively as needed
```

### Q10: How do you handle permissions in a Progressive Web App (PWA)?
**Answer:** Handling permissions in PWAs requires careful consideration of the service worker lifecycle, offline scenarios, and mobile platform integration:

**Key considerations:**

1. **Service Worker Context:**
```javascript
// Service worker can't directly request permissions
// Must be done in page context

// In page (main thread):
async function setupPWAPermissions() {
    // Register service worker first
    const registration = await navigator.serviceWorker.register('/sw.js');

    // Then request permissions
    await Notification.requestPermission();

    // Now service worker can show notifications
    registration.showNotification('PWA Installed!');
}
```

2. **Push Notifications:**
```javascript
// Complete push notification setup
async function setupPushNotifications() {
    // Step 1: Register service worker
    const registration = await navigator.serviceWorker.register('/sw.js');

    // Step 2: Request notification permission
    const permission = await Notification.requestPermission();

    if (permission !== 'granted') {
        throw new Error('Notification permission denied');
    }

    // Step 3: Subscribe to push
    const subscription = await registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: urlBase64ToUint8Array(vapidPublicKey)
    });

    // Step 4: Send subscription to server
    await sendSubscriptionToServer(subscription);
}
```

3. **Install Prompt:**
```javascript
// Handle install prompt
let deferredPrompt;

window.addEventListener('beforeinstallprompt', (e) => {
    // Prevent automatic prompt
    e.preventDefault();
    deferredPrompt = e;

    // Show custom install button
    showInstallButton();
});

async function installPWA() {
    if (!deferredPrompt) {
        return;
    }

    // Show install prompt
    deferredPrompt.prompt();

    // Wait for user response
    const { outcome } = await deferredPrompt.userChoice;

    if (outcome === 'accepted') {
        console.log('PWA installed');
        // Now request other permissions
        await requestPostInstallPermissions();
    }

    deferredPrompt = null;
}
```

4. **Offline Permission Handling:**
```javascript
// Cache permission state
class PWAPermissionManager {
    async checkPermission(name) {
        if (!navigator.onLine) {
            // Use cached state when offline
            return localStorage.getItem(`permission-${name}`) || 'prompt';
        }

        try {
            const result = await navigator.permissions.query({ name });
            // Cache the state
            localStorage.setItem(`permission-${name}`, result.state);
            return result.state;
        } catch {
            return 'prompt';
        }
    }

    async requestPermission(name, requestFn) {
        if (!navigator.onLine) {
            showMessage('Please connect to internet to enable this feature');
            return false;
        }

        const granted = await requestFn();
        localStorage.setItem(`permission-${name}`, granted ? 'granted' : 'denied');
        return granted;
    }
}
```

5. **Background Sync:**
```javascript
// Request background sync permission
async function setupBackgroundSync() {
    const registration = await navigator.serviceWorker.ready;

    try {
        await registration.sync.register('sync-posts');
        console.log('Background sync registered');
    } catch (error) {
        console.error('Background sync failed:', error);
    }
}

// In service worker (sw.js):
/*
self.addEventListener('sync', (event) => {
    if (event.tag === 'sync-posts') {
        event.waitUntil(syncPosts());
    }
});
*/
```

6. **Progressive Enhancement:**
```javascript
// Feature detection and fallbacks
class PWAFeatureManager {
    async enableFeature(feature) {
        const features = {
            notifications: this.setupNotifications.bind(this),
            backgroundSync: this.setupBackgroundSync.bind(this),
            periodicSync: this.setupPeriodicSync.bind(this)
        };

        if (features[feature]) {
            try {
                await features[feature]();
                return true;
            } catch (error) {
                console.error(`${feature} not supported:`, error);
                this.enableFallback(feature);
                return false;
            }
        }
    }

    enableFallback(feature) {
        const fallbacks = {
            notifications: 'in-app-alerts',
            backgroundSync: 'manual-sync',
            periodicSync: 'manual-refresh'
        };

        console.log(`Using fallback: ${fallbacks[feature]}`);
    }
}
```

**Best practices for PWAs:**
- Request permissions after installation prompt acceptance
- Cache permission states for offline use
- Provide clear feedback on permission status
- Use service worker for background features
- Test on multiple platforms (iOS, Android, Desktop)
- Handle permission revocation gracefully
- Provide offline fallbacks
- Monitor permission changes across tabs

---

## Summary

Browser permissions are essential for accessing sensitive device features and user data:

**Key Takeaways:**
- **User-controlled**: Always require explicit user consent
- **Context matters**: Request permissions when users need features, not on page load
- **Handle denial**: Provide alternatives and instructions, don't break functionality
- **Monitor changes**: Listen for permission revocations and update UI accordingly
- **Security first**: Requires HTTPS, follows same-origin policy, protects user privacy

**Common Permissions:**
- **Geolocation**: User's physical location
- **Notifications**: System notifications
- **Camera/Microphone**: Media device access
- **Clipboard**: Read/write clipboard data
- **Other**: Wake lock, persistent storage, background sync

**Best Practices:**
- Progressive disclosure: Explain before requesting
- Graceful degradation: Work without permissions when possible
- Clear communication: Explain why you need permissions
- Respect decisions: Don't repeatedly ask after denial
- Monitor state: React to permission changes
- Provide alternatives: Offer fallback functionality

**Implementation Patterns:**
- Check permission state with Permissions API
- Request in response to user actions
- Handle all error cases
- Provide clear feedback
- Release resources (especially media)
- Cache permission state for offline

Understanding and properly implementing browser permissions is crucial for building trustworthy, user-friendly web applications that respect user privacy while providing powerful functionality.

---

## External Resources

- [MDN: Permissions API](https://developer.mozilla.org/en-US/docs/Web/API/Permissions_API)
- [MDN: Geolocation API](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_API)
- [MDN: Notifications API](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API)
- [MDN: MediaDevices.getUserMedia()](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia)
- [MDN: Clipboard API](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API)
- [Web.dev: Permissions](https://web.dev/permissions/)
- [Requesting Permissions (web.dev)](https://web.dev/permission-ux/)

---

[ê Previous: IndexedDB](./03-indexeddb.md) | [Back to Browser APIs](./README.md)
