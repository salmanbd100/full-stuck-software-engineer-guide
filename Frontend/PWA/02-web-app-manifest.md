# Web App Manifest

## Overview

The Web App Manifest is a JSON file that describes how your PWA should behave when installed on a user's device. It defines the app's name, icons, display mode, theme colors, and more. This file is crucial for making your PWA installable and is a common interview topic.

## Table of Contents
- [What is a Web App Manifest?](#what-is-a-web-app-manifest)
- [Manifest File Location](#manifest-file-location)
- [Core Properties](#core-properties)
- [Display Modes](#display-modes)
- [Icon Configuration](#icon-configuration)
- [Installation Criteria](#installation-criteria)
- [iOS Meta Tags](#ios-meta-tags)
- [Manifest Validation](#manifest-validation)
- [Best Practices](#best-practices)
- [Complete Examples](#complete-examples)
- [Interview Questions](#interview-questions)

---

## What is a Web App Manifest?

### Purpose

The manifest is a JSON file that tells the browser how to display your PWA when installed:

```javascript
// Manifest purposes
const manifestPurposes = {
  installation: 'Define how to install PWA on home screen',
  metadata: 'Provide app name, description, icons',
  display: 'Control how app appears (standalone, fullscreen, etc)',
  colors: 'Set theme and background colors',
  orientation: 'Lock screen orientation if needed',
  scope: 'Define which URLs the manifest applies to',
  shortcuts: 'Add quick action shortcuts',
  categories: 'Categorize app (for app stores)',
  screenshots: 'Show app preview in stores'
};
```

### Why It Matters

```javascript
// Benefits of manifest
const benefits = {
  discoverability: 'Better indexing by search engines',
  installability: 'Install button appears in browser UI',
  engagement: 'Users can add to home screen',
  branding: 'Custom icon and splash screen',
  offline: 'App works like native app when installed',
  appStores: 'Can be listed in PWA app stores'
};
```

---

## Manifest File Location

### Linking the Manifest

```html
<!-- In your index.html head section -->
<!DOCTYPE html>
<html>
<head>
  <!-- Link to manifest file -->
  <link rel="manifest" href="/manifest.json">

  <!-- Alternative path -->
  <link rel="manifest" href="./manifest.json">

  <!-- With absolute URL -->
  <link rel="manifest" href="https://example.com/manifest.json">
</head>
<body>
  <!-- Content -->
</body>
</html>
```

### File Naming Conventions

```
/manifest.json          ê Most common
/app/manifest.json      ê In subdirectory
/static/manifest.json   ê In static assets folder
/public/manifest.json   ê Create React App default
```

### CORS Considerations

```javascript
// Manifest requests don't require CORS if:
// 1. Same-origin request
// 2. Served with proper MIME type: application/json or application/manifest+json

// Server configuration (Node.js/Express)
app.get('/manifest.json', (req, res) => {
  res.setHeader('Content-Type', 'application/manifest+json');
  res.sendFile('manifest.json');
});

// Or for all JSON files
app.set('json spaces', 2);
```

---

## Core Properties

### Essential Properties

```json
{
  "name": "My Awesome Application",
  "short_name": "MyApp",
  "description": "A progressive web app that does awesome things",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "theme_color": "#3367D6",
  "background_color": "#FFFFFF",
  "orientation": "portrait-primary",
  "icons": [
    {
      "src": "/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
```

### Property Reference

#### name
```json
{
  "name": "My Awesome Application"
}
```
- Full app name (up to 45 characters recommended)
- Displayed in app stores, install prompts
- Can be longer than short_name
- Required for PWA to be installable

#### short_name
```json
{
  "short_name": "MyApp"
}
```
- Used on home screen and app switcher
- Keep under 12 characters for best display
- Fallback to name if not provided
- Required for installation

#### description
```json
{
  "description": "A progressive web app that delivers amazing experiences"
}
```
- Brief explanation of app purpose
- Used in app stores and install dialogs
- Recommended: 50-100 characters
- Not required but helps with SEO

#### start_url
```json
{
  "start_url": "/"
}
```
- URL opened when user launches installed app
- Must be within scope
- Can include query parameters: "/app?mode=pwa"
- Recommended: relative URL for flexibility

#### scope
```json
{
  "scope": "/"
}
```
- Defines which URLs are in the PWA
- Service worker can only control pages in scope
- Default: directory of manifest file
- Broader scope = more pages controlled

#### display
```json
{
  "display": "standalone"
}
```
- How app should be displayed when launched
- Options: "fullscreen", "standalone", "minimal-ui", "browser"
- See [Display Modes](#display-modes) for details
- Default: "browser"

#### theme_color
```json
{
  "theme_color": "#3367D6"
}
```
- Color of browser UI elements (address bar, tabs)
- Should match your app's primary color
- Format: hex color code
- Affects user experience and brand

#### background_color
```json
{
  "background_color": "#FFFFFF"
}
```
- Background color during app load
- Shown while assets load
- Create smooth splash screen by matching app colors
- Default: white if not specified

#### orientation
```json
{
  "orientation": "portrait-primary"
}
```
- Lock screen orientation
- Values: "any", "natural", "landscape", "portrait", "portrait-primary", "portrait-secondary", "landscape-primary", "landscape-secondary"
- Default: "any" (device orientation)
- Can be changed by user

---

## Display Modes

### fullscreen
```json
{
  "display": "fullscreen"
}
```
- App takes entire screen
- Hides browser UI completely
- Like playing a game
- Careful: user can't access address bar
- Best for: games, immersive apps

### standalone
```json
{
  "display": "standalone"
}
```
- App looks like native application
- Browser UI hidden
- Has system-like back/forward gestures
- Title bar shows app name
- Most common and recommended

### minimal-ui
```json
{
  "display": "minimal-ui"
}
```
- Like standalone but with minimal browser controls
- Shows back/forward/reload buttons
- Not widely supported
- Good compromise between app and web

### browser
```json
{
  "display": "browser"
}
```
- Traditional web page display
- Full browser chrome visible
- Fallback if other modes not supported
- Default if display not specified

### Fallback
```json
{
  "display": "standalone"
}
```
- Browsers fall back if display mode unsupported
- Order: specified í next fallback í browser
- Recommended: "standalone" as most compatible

---

## Icon Configuration

### Icon Requirements

```javascript
// Icon size recommendations
const iconSizes = {
  minimum: {
    size: '192x192',
    purpose: 'Minimum for PWA installation',
    required: true
  },
  recommended: {
    size: '512x512',
    purpose: 'High-res displays, splash screens',
    required: false
  },
  additional: {
    sizes: ['96x96', '128x128', '256x256', '384x384'],
    purpose: 'Different screen densities',
    required: false
  }
};
```

### Icon Properties

```json
{
  "icons": [
    {
      "src": "/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icon-192-maskable.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable"
    },
    {
      "src": "/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    }
  ]
}
```

#### src
- Path to icon file (relative or absolute)
- Can be PNG, WebP, SVG
- Must be square
- Recommended: PNG for compatibility

#### sizes
```json
{
  "sizes": "192x192"
}
```
- Format: "widthxheight" or multiple: "192x192 512x512"
- Match actual image dimensions
- Multiple sizes help browser choose best one

#### type
```json
{
  "type": "image/png"
}
```
- MIME type: "image/png", "image/jpeg", "image/webp", "image/svg+xml"
- Helps browser determine compatibility
- PNG most compatible

#### purpose
```json
{
  "purpose": "any"
}
```
- **any** - Standard icon, used everywhere
- **maskable** - Icon with safe zone (used on Android, rounded corners)
- **badge** - Monochrome icon for notifications (96x96 minimum)

### Creating Maskable Icons

```javascript
// Maskable icons work with different device shapes
// Safe zone: innermost 40% of icon
// Android might display with rounded corners or circles

// Example creation with canvas
const canvas = document.createElement('canvas');
canvas.width = 192;
canvas.height = 192;

const ctx = canvas.getContext('2d');

// Center 80% of canvas is safe zone
const safeZone = 0.8;
const centerX = 96;
const centerY = 96;
const radius = 96 * safeZone / 2;

// Draw at least something in safe zone
ctx.fillStyle = '#3367D6';
ctx.fillRect(0, 0, 192, 192);

ctx.fillStyle = '#FFF';
ctx.beginPath();
ctx.arc(centerX, centerY, radius, 0, Math.PI * 2);
ctx.fill();

// Export as PNG
const imageData = canvas.toDataURL('image/png');
```

---

## Installation Criteria

### Minimum Requirements

For a PWA to show the install prompt, it must meet ALL criteria:

```javascript
const installationCriteria = {
  manifest: {
    required: 'manifest.json file',
    must_include: ['name', 'short_name', 'start_url', 'display', 'icons'],
    icons: 'At least 192x192 PNG icon'
  },
  https: {
    required: 'Served over HTTPS',
    exception: 'localhost for development'
  },
  serviceWorker: {
    required: 'Registered service worker',
    must_have: ['install', 'activate', 'fetch event']
  },
  other: {
    description: 'At least 2 screens',
    engagement: 'Some user engagement',
    mobile: 'Responsive design'
  }
};
```

### Checking Installation Eligibility

```javascript
// Check if PWA is installable
async function checkInstallability() {
  // Check manifest link
  const manifestLink = document.querySelector('link[rel="manifest"]');
  console.log('Manifest linked:', !!manifestLink);

  // Check HTTPS
  console.log('HTTPS:', window.location.protocol === 'https:');

  // Check service worker
  const swReg = await navigator.serviceWorker.getRegistration();
  console.log('SW registered:', !!swReg);

  // Check manifest content
  const manifest = await fetch(manifestLink.href).then(r => r.json());
  console.log('Has name:', !!manifest.name);
  console.log('Has short_name:', !!manifest.short_name);
  console.log('Has start_url:', !!manifest.start_url);
  console.log('Has icons:', !!manifest.icons?.length);
  console.log('Icon 192x192:', manifest.icons?.some(i => i.sizes.includes('192')));
}

checkInstallability();
```

### Install Prompt

```javascript
// Capture install prompt
let deferredPrompt;

window.addEventListener('beforeinstallprompt', event => {
  console.log('PWA is installable!');

  // Prevent auto-prompt
  event.preventDefault();

  // Save event for later
  deferredPrompt = event;

  // Show custom install button
  document.getElementById('install-button').style.display = 'block';
});

document.getElementById('install-button').addEventListener('click', async () => {
  if (!deferredPrompt) return;

  // Show install prompt
  deferredPrompt.prompt();

  // Wait for user response
  const { outcome } = await deferredPrompt.userChoice;
  console.log(`User response: ${outcome}`);

  deferredPrompt = null;
});

// Listen for successful installation
window.addEventListener('appinstalled', () => {
  console.log('PWA installed!');
  // Hide install button
  document.getElementById('install-button').style.display = 'none';
});

// Check if app is installed
navigator.getInstalledRelatedApps?.().then(apps => {
  console.log('Installed apps:', apps);
});
```

---

## iOS Meta Tags

### iOS Support

iOS has limited PWA support. Use meta tags as fallback:

```html
<!DOCTYPE html>
<html>
<head>
  <!-- PWA Manifest (all browsers) -->
  <link rel="manifest" href="/manifest.json">

  <!-- iOS specific fallbacks -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black">
  <meta name="apple-mobile-web-app-title" content="MyApp">

  <!-- Icon for iOS home screen -->
  <link rel="apple-touch-icon" href="/icon-180.png">
  <!-- 180x180 is Apple's standard -->
  <!-- 152x152, 144x144 also supported -->

  <!-- Splash screen (iOS only, for single icon) -->
  <link rel="apple-touch-startup-image" href="/splash.png">

  <!-- For light/dark mode (iOS 13+) -->
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
</head>
<body>
  <!-- Content -->
</body>
</html>
```

### Why iOS Fallbacks?

```javascript
// iOS limitations (as of Safari 16+):
const iosLimitations = {
  serviceWorkers: 'Full support (16+), limited (15)',
  manifest: 'Not fully supported',
  installPrompt: 'Add to Home Screen manually',
  pushNotifications: 'Not supported',
  backgroundSync: 'Not supported',
  offline: 'Service worker works, but limited'
};

// Use meta tags to provide fallback experience
// manifest.json + meta tags = best compatibility
```

### Complete iOS Setup

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- Manifest for other browsers -->
  <link rel="manifest" href="/manifest.json">

  <!-- iOS specific -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="MyApp">

  <!-- Icons for iOS (different sizes for different devices) -->
  <link rel="apple-touch-icon" href="/icon-180.png">
  <link rel="apple-touch-icon" sizes="152x152" href="/icon-152.png">
  <link rel="apple-touch-icon" sizes="144x144" href="/icon-144.png">
  <link rel="apple-touch-icon" sizes="120x120" href="/icon-120.png">

  <!-- Startup image (splash screen on iOS) -->
  <link rel="apple-touch-startup-image" href="/splash-1024x768.png" media="(device-width: 1024px)">
  <link rel="apple-touch-startup-image" href="/splash-750x1334.png" media="(device-width: 375px) and (device-height: 667px)">

  <!-- Other standard meta tags -->
  <meta name="theme-color" content="#3367D6">
  <meta name="description" content="My awesome PWA">
</head>
<body>
  <!-- Content -->
</body>
</html>
```

---

## Manifest Validation

### Using Lighthouse

```javascript
// Lighthouse is built into Chrome DevTools
// Steps:
// 1. Open DevTools (F12)
// 2. Go to Lighthouse tab
// 3. Run PWA Audit
// 4. Check Installable section
// 5. Fix issues found

// Lighthouse checks for:
//  HTTPS
//  Service worker
//  Manifest with required properties
//  Icons sizes
//  Colors
//  Viewport meta tag
```

### Manual Validation

```javascript
// Manual validation script
async function validateManifest() {
  const errors = [];
  const warnings = [];

  // Fetch manifest
  const manifestLink = document.querySelector('link[rel="manifest"]');
  if (!manifestLink) {
    errors.push('No manifest link found');
    return { errors, warnings };
  }

  const manifest = await fetch(manifestLink.href).then(r => r.json());

  // Check required fields
  const required = ['name', 'short_name', 'start_url', 'display', 'icons'];
  required.forEach(field => {
    if (!manifest[field]) {
      errors.push(`Missing required field: ${field}`);
    }
  });

  // Check icons
  if (!manifest.icons?.some(i => i.sizes.includes('192'))) {
    errors.push('Missing 192x192 icon (required for installation)');
  }

  if (!manifest.icons?.some(i => i.sizes.includes('512'))) {
    warnings.push('Missing 512x512 icon (recommended for splash screens)');
  }

  // Check colors
  if (!manifest.theme_color) {
    warnings.push('Missing theme_color');
  }

  if (!manifest.background_color) {
    warnings.push('Missing background_color');
  }

  // Check HTTPS
  if (window.location.protocol !== 'https:' && window.location.hostname !== 'localhost') {
    errors.push('Not served over HTTPS');
  }

  return { errors, warnings };
}

// Run validation
validateManifest().then(({ errors, warnings }) => {
  console.log('Errors:', errors);
  console.log('Warnings:', warnings);
});
```

### Online Tools

```javascript
// Free PWA manifest validators:
const validators = [
  'https://www.pwabuilder.com/',
  'https://web.dev/measure/',
  'https://manifest-validator.appspot.com/',
  'https://tools.enquos.ch/pwa/'
];

// Also check:
// - Chrome DevTools > Application > Manifest
// - Firefox > about:debugging
// - Edge > Edge://extensions
```

---

## Best Practices

### Do's

```json
{
  "name": "Complete descriptive name for your application",
  "short_name": "Short Name",
  "description": "Clear description of what app does and benefits",
  "start_url": "/app/",
  "scope": "/app/",
  "display": "standalone",
  "orientation": "portrait-primary",
  "theme_color": "#3367D6",
  "background_color": "#FFFFFF",
  "icons": [
    {
      "src": "/images/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/images/icon-192-maskable.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable"
    },
    {
      "src": "/images/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    }
  ],
  "categories": ["productivity"],
  "screenshots": [
    {
      "src": "/images/screenshot-1.png",
      "sizes": "540x720"
    }
  ]
}
```

### Don'ts

```javascript
// L Don't use single display mode without fallback thought
"display": "fullscreen" // No fallback option for user needs

// L Don't use very long names
"name": "This is my super long application name that nobody needs" // Too long

// L Don't use relative icons paths incorrectly
"icons": [{
  "src": "icon.png" // Works but confusing, use absolute
}]

// L Don't forget iOS fallback
// Only manifest, no meta tags = no iOS support

// L Don't use non-square icons
// Icons must be square for proper display

// L Don't ignore maskable icons
// Better Android experience with maskable purpose
```

---

## Complete Examples

### Minimal Manifest (Bare Minimum)

```json
{
  "name": "My App",
  "short_name": "App",
  "start_url": "/",
  "display": "standalone",
  "icons": [
    {
      "src": "/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
```

### Standard Manifest (Recommended)

```json
{
  "name": "Awesome Todo Application",
  "short_name": "TodoApp",
  "description": "A simple yet powerful todo app that works offline",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "orientation": "portrait-primary",
  "theme_color": "#6200EE",
  "background_color": "#FFFFFF",
  "icons": [
    {
      "src": "/images/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/images/icon-192-maskable.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable"
    },
    {
      "src": "/images/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    }
  ],
  "categories": ["productivity"],
  "screenshots": [
    {
      "src": "/images/screenshot-1.png",
      "sizes": "540x720",
      "type": "image/png",
      "form_factor": "narrow"
    },
    {
      "src": "/images/screenshot-2.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide"
    }
  ]
}
```

### Advanced Manifest (Complete Feature Set)

```json
{
  "name": "Enterprise Todo Application",
  "short_name": "Todos",
  "description": "Enterprise-grade todo management with offline support and team collaboration",
  "start_url": "/app/todos",
  "scope": "/app/",
  "display": "standalone",
  "orientation": "any",
  "theme_color": "#1976D2",
  "background_color": "#FFFFFF",
  "dir": "ltr",
  "lang": "en-US",
  "categories": ["productivity", "utilities"],
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-192x192-maskable.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    }
  ],
  "screenshots": [
    {
      "src": "/screenshots/screenshot-1-narrow.png",
      "sizes": "540x720",
      "type": "image/png",
      "form_factor": "narrow"
    },
    {
      "src": "/screenshots/screenshot-2-narrow.png",
      "sizes": "540x720",
      "type": "image/png",
      "form_factor": "narrow"
    },
    {
      "src": "/screenshots/screenshot-1-wide.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide"
    }
  ],
  "shortcuts": [
    {
      "name": "Create New Todo",
      "short_name": "New Todo",
      "description": "Quickly create a new todo item",
      "url": "/app/todos/new",
      "icons": [
        {
          "src": "/icons/new-todo-96.png",
          "sizes": "96x96"
        }
      ]
    },
    {
      "name": "View Today",
      "short_name": "Today",
      "description": "See todos for today",
      "url": "/app/todos/today",
      "icons": [
        {
          "src": "/icons/today-96.png",
          "sizes": "96x96"
        }
      ]
    }
  ],
  "prefer_related_applications": false,
  "related_applications": [
    {
      "platform": "play",
      "url": "https://play.google.com/store/apps/details?id=com.example.todos",
      "id": "com.example.todos"
    }
  ]
}
```

---

## Interview Questions

### Question 1: What is a Web App Manifest and why is it important?

**Answer:**

A Web App Manifest is a JSON file that describes your PWA's metadata and behavior when installed. It's important because:

1. **Installation** - Required for the install prompt to appear
2. **Metadata** - Defines app name, description, icons
3. **Display Control** - Controls how app appears (standalone, fullscreen, etc.)
4. **Branding** - Customizes colors, icons, splash screens
5. **Discoverability** - Helps search engines and app stores discover your PWA
6. **User Experience** - Creates native app-like appearance

Without a valid manifest, users can't install the PWA, and the browser has no guidance on how to display it.

---

### Question 2: What properties must a manifest include to be installable?

**Answer:**

Minimum required properties for PWA installation:

1. **name** or **short_name** - App name for display
2. **start_url** - URL to load when app launches
3. **display** - How app should be displayed
4. **icons** - At least one icon (192x192 minimum)
5. **theme_color** - Browser UI color (though not strictly required, highly recommended)

Additionally required by browser:
- **HTTPS** - Must be served over HTTPS (or localhost)
- **Service Worker** - Must be registered and active
- **Proper linking** - `<link rel="manifest" href="/manifest.json">`

```json
{
  "name": "My App",
  "short_name": "App",
  "start_url": "/",
  "display": "standalone",
  "icons": [
    {
      "src": "/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
```

---

### Question 3: Explain the different display modes and when to use each.

**Answer:**

**fullscreen**
- Hides all browser UI
- Uses entire screen
- Best for: games, immersive apps
- Caution: users can't see address bar

**standalone** (most common)
- App-like appearance
- No address bar or browser controls
- Has system back/forward gestures
- Best for: most PWAs

**minimal-ui**
- Like standalone with minimal controls
- Back/forward/reload buttons visible
- Limited browser support
- Good compromise between app and web

**browser** (default)
- Traditional web page display
- Full browser chrome visible
- Used if other modes unsupported

**Recommendation:** Start with "standalone" as it's well-supported and provides good app-like experience while allowing user to navigate.

---

### Question 4: How do you support iOS devices with PWA fallbacks?

**Answer:**

iOS has limited PWA support, so use meta tags as fallbacks:

```html
<head>
  <!-- Standard manifest for all browsers -->
  <link rel="manifest" href="/manifest.json">

  <!-- iOS fallbacks -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-title" content="MyApp">
  <meta name="apple-mobile-web-app-status-bar-style" content="black">

  <!-- iOS home screen icon (180x180 standard) -->
  <link rel="apple-touch-icon" href="/icon-180.png">

  <!-- Optional: different sizes for different devices -->
  <link rel="apple-touch-icon" sizes="152x152" href="/icon-152.png">
  <link rel="apple-touch-icon" sizes="120x120" href="/icon-120.png">

  <!-- Optional: splash screen -->
  <link rel="apple-touch-startup-image" href="/splash-750x1334.png">
</head>
```

**Why both?**
- Manifest: For modern browsers (Chrome, Firefox, Edge)
- Meta tags: For iOS Safari (which ignores manifest)

**Result:** Users on iOS can add to home screen manually, with proper icon and fullscreen behavior.

---

### Question 5: What is a maskable icon and why is it important?

**Answer:**

A **maskable icon** is designed with a safe zone (innermost 40% of icon) where all important content fits. It's used by Android to display icons with different shapes (circles, rounded squares, etc.).

**Why important:**
1. **Android Adaptive Icons** - Icons appear with different shapes on different launchers
2. **Better UX** - No cropping of important logo elements
3. **Modern Standard** - Expected on modern Android devices

**Creating maskable icons:**

```javascript
// Safe zone illustration
// 192x192 icon
// Safe zone: center 80% = 77x77 starting at (58, 58)

// Best practices:
// 1. Keep logo/essential element in center 40% (H77x77)
// 2. Use full space for background/secondary elements
// 3. No important details should be cut off if icon is circled

// In manifest
"icons": [
  {
    "src": "/icon-192.png",
    "sizes": "192x192",
    "type": "image/png",
    "purpose": "any"  // Regular icon
  },
  {
    "src": "/icon-192-maskable.png",
    "sizes": "192x192",
    "type": "image/png",
    "purpose": "maskable"  // For adaptive icons
  }
]
```

---

### Question 6: How do you validate a manifest file?

**Answer:**

Multiple validation approaches:

**1. Lighthouse (Easiest)**
```
DevTools > Lighthouse > Run PWA Audit
Checks all manifest requirements and shows issues
```

**2. Chrome DevTools**
```
Application tab > Manifest
Shows loaded manifest and validation errors
```

**3. Manual Validation**
```javascript
async function validateManifest() {
  const manifestLink = document.querySelector('link[rel="manifest"]');
  const manifest = await fetch(manifestLink.href).then(r => r.json());

  const checks = {
    hasName: !!manifest.name,
    hasShortName: !!manifest.short_name,
    hasStartUrl: !!manifest.start_url,
    hasDisplay: !!manifest.display,
    has192Icon: manifest.icons?.some(i => i.sizes.includes('192')),
    has512Icon: manifest.icons?.some(i => i.sizes.includes('512')),
    hasThemeColor: !!manifest.theme_color,
    hasBackgroundColor: !!manifest.background_color
  };

  console.table(checks);
}
```

**4. Online Tools**
- PWA Builder (pwabuilder.com)
- Web.dev Measure (web.dev/measure)
- Manifest Validator

---

### Question 7: What does start_url do and why is it important?

**Answer:**

**start_url** defines which page opens when user launches the installed PWA.

```json
{
  "start_url": "/"
}
```

**Why important:**

1. **Entry Point** - User expects app to open at correct location
2. **Onboarding** - Can point to onboarding screen vs home
3. **Context** - Can include parameters: "/app?mode=pwa"
4. **Analytics** - Can track PWA launches differently

**Best Practices:**

```json
{
  "start_url": "/"  // Simple, relative, flexible
}
```

**vs**

```json
{
  "start_url": "https://example.com/"  // Absolute, less flexible
}
```

**vs**

```json
{
  "start_url": "/app/"  // Specific app section
}
```

**Important:** start_url must be within scope. If scope is "/app/", start_url can be "/app/" or "/app/page", but not "/".

---

### Question 8: What is the difference between scope and start_url?

**Answer:**

| Property | Purpose | Scope |
|----------|---------|-------|
| **start_url** | URL opened when app launches | Single URL |
| **scope** | Which URLs are part of the PWA | Directory/path range |

**Example:**

```json
{
  "start_url": "/app/todos",
  "scope": "/app/"
}
```

- User launches app í opens `/app/todos`
- Within `/app/*` í controlled by service worker
- Outside `/app/*` í open in browser normally
- Service worker can't control URLs outside scope

**Real-world usage:**

```json
// E-commerce site with PWA for catalog
{
  "start_url": "/products",
  "scope": "/products/"
}

// Blog with PWA for articles
{
  "start_url": "/blog",
  "scope": "/blog/"
}

// Full site PWA
{
  "start_url": "/",
  "scope": "/"
}
```

---

### Question 9: How do you handle multiple languages in manifest?

**Answer:**

Manifest itself doesn't support multiple languages natively. Use these approaches:

**1. Serve Different Manifest Files**

```html
<!-- Detect language and link accordingly -->
<script>
  const lang = navigator.language.split('-')[0];
  const link = document.createElement('link');
  link.rel = 'manifest';
  link.href = `/manifest-${lang}.json`;
  document.head.appendChild(link);
</script>
```

**2. Use Absolute English + Language Meta Tags**

```json
{
  "name": "My App",
  "description": "My App Description"
}
```

```html
<!-- Language specified in HTML -->
<html lang="en">
  <head>
    <link rel="manifest" href="/manifest.json">
  </head>
</html>
```

**3. Progressive Enhancement with Meta Tags**

```html
<!-- English manifest + local meta tags -->
<link rel="manifest" href="/manifest.json">
<meta name="description" content="English description">

<script>
  if (navigator.language.startsWith('es')) {
    document.querySelector('meta[name="description"]')
      .content = 'DescripciÛn en espaÒol';
  }
</script>
```

**Best practice:** Use English in manifest (most compatible) + language-specific meta tags.

---

### Question 10: What's the difference between theme_color and background_color?

**Answer:**

**theme_color**
- Colors the browser UI elements (address bar, tabs)
- Visible while app is running
- Makes app feel more integrated
- Should match app's primary color

```json
{
  "theme_color": "#3367D6"  // Blue address bar
}
```

**background_color**
- Background shown while app resources load
- Visible during startup
- Creates smooth transition from launch to loaded app
- Should match app's background

```json
{
  "background_color": "#FFFFFF"  // White startup screen
}
```

**Visual Example:**

```
1. User taps PWA icon
   ì
2. System shows [background_color] splash screen
   ì
3. App loads resources
   ì
4. Real app appears (user sees [theme_color] address bar)
```

**Best practice:**

```json
{
  "theme_color": "#1976D2",      // Primary brand color
  "background_color": "#FFFFFF"   // App background color
}
```

When colors match app design, users see smooth loading experience instead of jarring color change.

---

## Summary

The Web App Manifest is essential for:
1. Making PWAs installable
2. Controlling app appearance and behavior
3. Providing metadata to browsers and app stores
4. Creating native app-like experience
5. Supporting different devices and orientations

Key properties to master: name, short_name, start_url, scope, display, icons, theme_color, background_color.

Remember: Manifest + Service Worker + HTTPS = Installable PWA!

---

[ê Back to PWA Guide](./README.md)
