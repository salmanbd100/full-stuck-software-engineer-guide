# Cookies and SameSite Policy

## Overview

**HTTP Cookies** are small pieces of data stored on the client side and sent with every HTTP request to the server. They are essential for implementing stateful authentication, session management, and user tracking. The **SameSite** attribute is a critical security feature introduced to prevent Cross-Site Request Forgery (CSRF) attacks and protect user privacy by controlling when cookies are sent in cross-site requests. Understanding cookies and SameSite policy is essential for building secure web applications.

### Key Characteristics
- **Server-Set**: Sent via `Set-Cookie` HTTP header (can also be set via JavaScript)
- **Automatic Transmission**: Sent with every matching request automatically
- **Flexible Attributes**: Secure, HttpOnly, SameSite, Domain, Path, Expires, Max-Age
- **Session vs Persistent**: Can be temporary (session) or persistent (with expiration)
- **Same-Origin Policy**: Sent based on domain, path, and protocol matching
- **Privacy Implications**: Used for tracking and consent (GDPR/CCPA)
- **Security Critical**: Can compromise authentication if not properly configured

---

## Table of Contents

1. [Cookie Basics](#cookie-basics)
2. [Cookie Attributes](#cookie-attributes)
3. [SameSite Policy](#samesite-policy)
4. [Third-Party Cookies](#third-party-cookies)
5. [Setting Cookies](#setting-cookies)
6. [Reading and Manipulating Cookies](#reading-and-manipulating-cookies)
7. [CSRF Protection](#csrf-protection)
8. [Cookie Security Best Practices](#cookie-security-best-practices)
9. [GDPR/CCPA Compliance](#gdpr-ccpa-compliance)
10. [Implementation Examples](#implementation-examples)
11. [Interview Questions](#interview-questions)
12. [Summary](#summary)

---

## Cookie Basics

### What are Cookies?

```javascript
// Cookies are automatically sent with requests
// Server sends: Set-Cookie: sessionId=abc123; Path=/; HttpOnly
// Browser stores the cookie and sends it with every request to that domain

// Access (if not HttpOnly)
console.log(document.cookie); // "sessionId=abc123; other=value"

// Cookies vs localStorage
// - Cookies: Sent with every request (overhead)
// - localStorage: Stored only, not sent automatically
// - Cookies: Can be HttpOnly (more secure)
// - localStorage: Always accessible via JavaScript (XSS risk)

// Typical uses
const cookieUses = {
  authentication: 'Session tokens, JWT tokens',
  tracking: 'User behavior, analytics',
  preferences: 'User settings, language',
  security: 'CSRF tokens, anti-tracking'
};
```

### Cookie Format

```javascript
// Basic cookie structure
// Set-Cookie: name=value; Path=/; Domain=.example.com; Expires=Wed, 09 Jun 2025 10:18:14 GMT; Secure; HttpOnly; SameSite=Strict

// Parts:
// name=value    - Cookie name and value
// Path          - URL path where cookie is sent
// Domain        - Domain where cookie is accessible
// Expires       - Absolute expiration time (date)
// Max-Age       - Relative expiration time (seconds)
// Secure        - Only sent over HTTPS
// HttpOnly       - Not accessible via JavaScript
// SameSite       - Cross-site request handling (Strict/Lax/None)
```

---

## Cookie Attributes

### 1. Name and Value

```javascript
// Simple cookie
document.cookie = "username=john_doe";

// Value with special characters (must be encoded)
document.cookie = `user=${encodeURIComponent(JSON.stringify({ id: 1, name: 'John' }))}`;

// Reading back
function getCookieValue(name) {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) return decodeURIComponent(parts.pop().split(';').shift());
}

const user = JSON.parse(decodeURIComponent(getCookieValue('user')));
```

### 2. Path Attribute

```javascript
// Path determines which URLs will receive the cookie

// Cookie sent to /admin and /admin/users
document.cookie = "adminToken=xyz; Path=/admin";

// Cookie sent to ALL paths on the domain
document.cookie = "sessionId=abc; Path=/";

// Cookie sent ONLY to /checkout
document.cookie = "cartId=123; Path=/checkout";

//  Best practice: Use / for session cookies
//  Use specific paths for sensitive data
```

### 3. Domain Attribute

```javascript
// Domain determines which domains will receive the cookie

// Accessible on example.com and all subdomains (mail.example.com, api.example.com)
document.cookie = "sessionId=abc; Domain=example.com; Path=/";

// Accessible ONLY on exact subdomain mail.example.com
document.cookie = "token=xyz; Domain=mail.example.com; Path=/";

// If Domain is NOT specified, cookie only applies to current domain
document.cookie = "local=123"; // Only current domain

//   Security: Be careful with broad Domain settings
//  Use specific domains for sensitive cookies
```

### 4. Secure Flag

```javascript
// Secure flag: Cookie only sent over HTTPS, not HTTP

// L Bad: Sent over HTTP (insecure)
document.cookie = "sessionId=abc; Path=/";

//  Good: Only sent over HTTPS
document.cookie = "sessionId=abc; Path=/; Secure";

// In production, ALWAYS use Secure for authentication cookies
// Set-Cookie: authToken=xyz; Secure; HttpOnly; SameSite=Strict

// This prevents man-in-the-middle attacks where attacker intercepts HTTP traffic
```

### 5. HttpOnly Flag

```javascript
// HttpOnly flag: Cookie NOT accessible via JavaScript (document.cookie)
// Only sent with HTTP requests and responses

//  Server sets (prevents XSS token theft)
// Set-Cookie: authToken=secret123; HttpOnly; Secure; SameSite=Strict

// L JavaScript cannot access HttpOnly cookies
console.log(document.cookie); // Won't contain authToken

// HttpOnly cookies are:
// - Most secure for authentication tokens
// - Not accessible to malicious JavaScript
// - Automatically sent with requests
// - Can only be deleted by server

// To delete HttpOnly cookie:
// Set-Cookie: authToken=; Max-Age=0; Path=/;

//  ALWAYS use HttpOnly for authentication tokens
// This is the primary defense against XSS token theft
```

### 6. Expires and Max-Age

```javascript
// Expires: Absolute date/time when cookie expires
const expirationDate = new Date('2025-12-31');
document.cookie = `sessionId=abc; Expires=${expirationDate.toUTCString()}; Path=/`;

// Max-Age: Seconds until cookie expires
document.cookie = "sessionId=abc; Max-Age=3600; Path=/"; // Expires in 1 hour

// If Max-Age is provided, it takes precedence over Expires
document.cookie = "sessionId=abc; Max-Age=3600; Expires=Wed, 09 Jun 2025 10:18:14 GMT; Path=/";

// Session cookie (no expiration): Deleted when browser closes
document.cookie = "tempData=xyz; Path=/"; // No Expires or Max-Age

// Persistent cookie: Persists after browser closes
document.cookie = "preferences=dark; Max-Age=31536000; Path=/"; // 1 year

// Delete cookie: Set Max-Age to 0 or Expires to past date
document.cookie = "sessionId=; Max-Age=0; Path=/";

//  Use Max-Age for modern browsers (clearer than Expires)
//  Session cookies for temporary data
//  Persistent cookies for long-term storage
```

### 7. SameSite Attribute

```javascript
// SameSite: Controls when cookie is sent in cross-site requests
// Values: Strict, Lax, None

// Strict: Cookie ONLY sent in same-site requests
document.cookie = "authToken=xyz; SameSite=Strict; Path=/; Secure";

// Lax: Cookie sent in same-site and top-level navigation
document.cookie = "sessionId=abc; SameSite=Lax; Path=/; Secure";

// None: Cookie sent in all contexts (must use Secure)
document.cookie = "trackingId=123; SameSite=None; Secure; Path=/";

//  Default in modern browsers: Lax (if not specified)
//  Use Strict for sensitive operations
//  Use Lax for user-friendly experience
//  Use None only when necessary (with Secure)
```

---

## SameSite Policy

### Understanding SameSite Values

#### SameSite=Strict

```javascript
// Most restrictive: Cookie ONLY sent in same-site requests
document.cookie = "authToken=xyz; SameSite=Strict; Secure; HttpOnly; Path=/";

// Scenarios:
//  Same-site request: Cookie SENT
//    - Click link within example.com
//    - Form submission to example.com
//    - JavaScript fetch from example.com

// L Cross-site request: Cookie NOT SENT
//    - Click link from google.com to example.com (navigation)
//    - Submit form from google.com to example.com
//    - Fetch from google.com to example.com (AJAX)

// Security: Prevents CSRF attacks completely
// Downside: Lost convenience (e.g., "Stay logged in" across sites doesn't work)

// Use case: High-security operations (admin panels, banking, payments)
// Example:
// POST /transfer/money (requires authentication with sensitive operation)
// SameSite=Strict prevents forged requests from attacker sites
```

#### SameSite=Lax

```javascript
// Balanced: Cookie sent in same-site + top-level navigation
document.cookie = "sessionId=abc; SameSite=Lax; Secure; Path=/";

// Scenarios:
//  Same-site request: Cookie SENT
//    - Click link within example.com
//    - Form submission to example.com
//    - JavaScript fetch from example.com

//  Top-level navigation: Cookie SENT
//    - User clicks link from external site to example.com
//    - User manually types URL
//    - Form with method=GET redirects to example.com

// L Cross-site AJAX: Cookie NOT SENT
//    - Fetch from google.com to example.com
//    - XMLHttpRequest from external site
//    - Form with method=POST from external site (unless top-level)

// Security: Prevents most CSRF attacks
// Convenience: Maintains "Stay logged in" for navigation

// Default in modern browsers if SameSite not specified
// Use case: General-purpose authentication cookies

// Example CSRF prevention:
// Attacker's site: <form method=POST action="https://bank.com/transfer">
// Browser: Does NOT send sessionId cookie (because SameSite=Lax)
// Result: Server doesn't recognize authenticated user, request fails
```

#### SameSite=None

```javascript
// No restriction: Cookie sent in all contexts
// MUST include Secure flag (HTTPS only)
document.cookie = "trackingId=xyz; SameSite=None; Secure; Path=/";

// Scenarios:
//  Same-site request: Cookie SENT
//  Cross-site request: Cookie SENT
//  Any HTTP request: Cookie SENT

// Security: No CSRF protection from SameSite
// Must rely on other CSRF defenses (CSRF tokens, double-submit cookies)

// Privacy: Browser may block this in future versions
// Use cases:
// - Third-party embedding (iframe, pixels, beacons)
// - Cross-site analytics
// - Cross-site payment processing

// Example: Analytics tracking across websites
// Third-party script on multiple sites sends cookie to analytics server
// <img src="https://analytics.com/track?id=user123" style="display:none;">
// Browser sends SameSite=None cookie with the request

//   Avoid using SameSite=None unless absolutely necessary
//   Always require Secure (HTTPS) with SameSite=None
```

### SameSite Comparison Table

| Feature | Strict | Lax | None |
|---------|--------|-----|------|
| **Same-site requests** |  Sent |  Sent |  Sent |
| **Top-level navigation** | L Not sent |  Sent |  Sent |
| **Cross-site AJAX** | L Not sent | L Not sent |  Sent |
| **CSRF Protection** | Very strong | Strong | Weak (need tokens) |
| **User experience** | Strict | Balanced | Permissive |
| **Privacy** | High | High | Low (tracking) |
| **Requires Secure** | No | No | Yes (required) |
| **Use case** | Banking, admin | Auth cookies | Third-party scripts |

---

## Third-Party Cookies

### Understanding Third-Party Cookies

```javascript
// First-party cookie: Set by and sent to the website user visits
// https://example.com sends Set-Cookie: sessionId=abc; Path=/
// Browser stores and sends to example.com only

// Third-party cookie: Set by different domain (ads, analytics)
// https://example.com loads <img src="https://ads.com/track.gif">
// ads.com sends Set-Cookie: userId=xyz; Domain=.ads.com
// Browser stores and sends to ads.com (and ads.com subdomains) from ANY site

// Tracking across websites
// 1. User visits news.com, ads.com sets userId=xyz
// 2. User visits sports.com, ads.com sets same userId=xyz
// 3. ads.com now knows user visited both sites (privacy violation)
```

### Third-Party Cookie Use Cases

```javascript
// Legitimate uses:
const thirdPartyCookies = {
  analytics: 'Track user behavior across sites',
  advertising: 'Serve targeted ads',
  payments: 'Process payments across merchants',
  cdns: 'Optimize content delivery',
  embeds: 'Embedded widgets (YouTube, Twitter, etc.)'
};

// Example: YouTube embedded video
// <iframe src="https://www.youtube.com/embed/dQw4w9WgXcQ"></iframe>
// YouTube iframe might set third-party cookie
// YouTube can then track user across all sites with embedded videos

// Example: Google Analytics
// <script src="https://www.googletagmanager.com/gtag/js"></script>
// Analytics script sets third-party cookie
// Google tracks user across all sites using analytics
```

### Browser Restrictions on Third-Party Cookies

```javascript
// Modern browsers are restricting third-party cookies:
// Chrome: Phasing out by Q1 2025 (Privacy Sandbox alternative)
// Safari: Already blocked by default (Intelligent Tracking Prevention)
// Firefox: Strict tracking protection blocks many third-party cookies
// Edge: Tracks prevention for third-party cookies

// Impact: Privacy improved, but legitimate uses affected
// Alternative approaches:
// - First-party data collection
// - Privacy Sandbox APIs (Topics, Fledge)
// - Server-side tracking

// Example: Analytics migration from third-party to first-party
// Old: External analytics script sets third-party cookie
// New: Your domain sets first-party cookie to analytics backend
// Your backend: Sends analytics data to external service

// Code example (first-party analytics):
async function trackEvent(eventName, eventData) {
  // Your domain sends tracking request
  // Browser includes first-party cookies
  // Your server logs event and forwards to analytics service
  await fetch('/api/analytics', {
    method: 'POST',
    credentials: 'include', // Include first-party cookies
    body: JSON.stringify({ event: eventName, data: eventData })
  });
}
```

---

## Setting Cookies

### Client-Side Cookie Setting (JavaScript)

```javascript
// Simple cookie
document.cookie = "username=john_doe";

// Cookie with attributes
document.cookie = "username=john_doe; Path=/; Max-Age=3600";

// Full cookie with all attributes
document.cookie = "authToken=xyz123; Path=/; Max-Age=604800; Secure; HttpOnly; SameSite=Strict";

// Note: HttpOnly cannot be set from JavaScript (server only)
// JavaScript can only set cookies without HttpOnly flag

// Multiple cookies (separate statements)
document.cookie = "sessionId=abc; Path=/";
document.cookie = "preferences=dark; Path=/; Max-Age=31536000";

// Helper function for easier cookie setting
function setCookie(name, value, options = {}) {
  let cookieString = `${encodeURIComponent(name)}=${encodeURIComponent(value)}`;

  if (options.path) cookieString += `; Path=${options.path}`;
  if (options.domain) cookieString += `; Domain=${options.domain}`;
  if (options.maxAge) cookieString += `; Max-Age=${options.maxAge}`;
  if (options.expires) cookieString += `; Expires=${options.expires.toUTCString()}`;
  if (options.secure) cookieString += '; Secure';
  if (options.sameSite) cookieString += `; SameSite=${options.sameSite}`;

  document.cookie = cookieString;
}

// Usage
setCookie('sessionId', 'abc123', {
  path: '/',
  maxAge: 3600,
  secure: true,
  sameSite: 'Strict'
});
```

### Server-Side Cookie Setting (Node.js/Express)

```javascript
// Express middleware to set cookie
app.get('/login', (req, res) => {
  // Basic cookie
  res.cookie('sessionId', 'xyz123', {
    path: '/',
    maxAge: 1000 * 60 * 60, // 1 hour in milliseconds
    httpOnly: true, // Cannot access via JavaScript
    secure: true,  // HTTPS only
    sameSite: 'Strict' // CSRF protection
  });

  res.send('Cookie set');
});

// Reading cookies
app.get('/profile', (req, res) => {
  const sessionId = req.cookies.sessionId;
  console.log(sessionId); // 'xyz123'
  res.send('User profile');
});

// Deleting cookie
app.get('/logout', (req, res) => {
  res.clearCookie('sessionId', {
    path: '/',
    httpOnly: true,
    secure: true,
    sameSite: 'Strict'
  });

  res.send('Logged out');
});

// Setting multiple cookies
app.get('/multi', (req, res) => {
  res.cookie('preference1', 'value1', { path: '/' });
  res.cookie('preference2', 'value2', { path: '/', maxAge: 86400000 });
  res.send('Multiple cookies set');
});

// Signed cookie (prevents tampering)
app.get('/signed', (req, res) => {
  res.cookie('auth', 'token123', {
    httpOnly: true,
    secure: true,
    signed: true // Express signs the cookie
  });

  res.send('Signed cookie set');
});

// Reading signed cookies
app.get('/verify', (req, res) => {
  const auth = req.signedCookies.auth;
  if (auth) {
    res.send('Valid signed cookie');
  } else {
    res.send('Invalid or tampered cookie');
  }
});
```

### Using js-cookie Library

```javascript
// Installation
// npm install js-cookie

// Simple usage
Cookies.set('username', 'john_doe');
Cookies.get('username'); // 'john_doe'

// With options
Cookies.set('sessionId', 'xyz123', {
  expires: 7, // 7 days
  path: '/',
  secure: true,
  sameSite: 'Strict'
});

// Reading with default
const theme = Cookies.get('theme') || 'light';

// Getting all cookies
const allCookies = Cookies.get(); // Returns object

// Removing cookie
Cookies.remove('sessionId', { path: '/' });

// Working with JSON
Cookies.set('user', JSON.stringify({ id: 1, name: 'John' }));
const user = JSON.parse(Cookies.get('user'));

// Helper for async operations
async function authenticateAndSetCookie(credentials) {
  const response = await fetch('/login', {
    method: 'POST',
    credentials: 'include', // Include cookies
    body: JSON.stringify(credentials)
  });

  if (response.ok) {
    // Server sets HttpOnly cookie automatically
    // JavaScript receives confirmation
    return true;
  }
  return false;
}
```

---

## Reading and Manipulating Cookies

### Getting Cookies

```javascript
// Get all cookies as string
console.log(document.cookie); // "sessionId=abc123; preferences=dark"

// Helper: Get single cookie value
function getCookie(name) {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) return parts.pop().split(';').shift();
  return null;
}

const sessionId = getCookie('sessionId'); // 'abc123'

// Helper: Get all cookies as object
function getAllCookies() {
  return document.cookie
    .split('; ')
    .reduce((acc, cookie) => {
      const [name, value] = cookie.split('=');
      acc[decodeURIComponent(name)] = decodeURIComponent(value);
      return acc;
    }, {});
}

const cookies = getAllCookies();
// { sessionId: 'abc123', preferences: 'dark' }

// With regex (more robust)
function getCookieWithRegex(name) {
  const match = document.cookie.match(new RegExp('(^|; )' + name + '=([^;]*)'));
  return match ? decodeURIComponent(match[2]) : null;
}

// Handle special characters
function getCookieDecoded(name) {
  const value = getCookie(name);
  return value ? decodeURIComponent(value) : null;
}

// Get JSON cookie
function getCookieJSON(name) {
  const value = getCookie(name);
  return value ? JSON.parse(decodeURIComponent(value)) : null;
}
```

### Updating and Deleting Cookies

```javascript
// Update cookie (set with same name)
document.cookie = "sessionId=newValue123; Path=/; Max-Age=3600";

// Delete cookie (set Max-Age to 0)
document.cookie = "sessionId=; Path=/; Max-Age=0";

// Helper: Delete cookie
function deleteCookie(name, path = '/') {
  document.cookie = `${name}=; Path=${path}; Max-Age=0`;
}

deleteCookie('sessionId');

// Delete all cookies
function deleteAllCookies() {
  document.cookie.split(';').forEach(cookie => {
    const eqPos = cookie.indexOf('=');
    const name = eqPos > -1 ? cookie.substr(0, eqPos) : cookie;
    document.cookie = `${name}=; Path=/; Max-Age=0`;
  });
}

// Conditional update
function updateCookieIfExists(name, newValue) {
  if (getCookie(name)) {
    document.cookie = `${name}=${newValue}; Path=/`;
    return true;
  }
  return false;
}

// Update with new expiration
function refreshCookie(name, newMaxAge) {
  const value = getCookie(name);
  if (value) {
    document.cookie = `${name}=${value}; Path=/; Max-Age=${newMaxAge}`;
  }
}
```

---

## CSRF Protection

### Understanding CSRF Attacks

```javascript
// CSRF (Cross-Site Request Forgery) Attack Scenario:

// Step 1: Attacker creates malicious website
// <form method="POST" action="https://bank.com/transfer">
//   <input type="hidden" name="amount" value="1000">
//   <input type="hidden" name="to" value="attacker">
// </form>
// <script>document.forms[0].submit();</script>

// Step 2: Victim visits attacker website while logged in to bank
// Browser has bank sessionId cookie from previous login
// Browser automatically sends sessionId with request to bank.com

// Step 3: Bank server receives request with valid sessionId cookie
// Bank thinks it's the user making the request
// Transfer processed without user's knowledge!

// Root cause: Cookies sent automatically with cross-site requests
```

### SameSite Protection

```javascript
// SameSite=Strict prevents CSRF
document.cookie = "sessionId=xyz; SameSite=Strict; Secure; HttpOnly; Path=/";

// Scenario with SameSite=Strict:
// 1. Attacker creates form on attacker.com
// 2. User visits attacker.com while logged in to bank.com
// 3. Form submits to bank.com
// 4. Browser does NOT send sessionId cookie (different site)
// 5. Bank server gets unauthenticated request
// 6. Request fails - CSRF prevented!

// SameSite=Lax is less strict but still prevents most CSRF
document.cookie = "sessionId=xyz; SameSite=Lax; Secure; HttpOnly; Path=/";

// Scenario with SameSite=Lax:
// 1. Form method=POST from attacker site: Cookie NOT SENT (CSRF prevented)
// 2. Top-level navigation from attacker site: Cookie SENT (user-friendly)
// 3. Same-site request: Cookie SENT (normal)
```

### Additional CSRF Protection: CSRF Tokens

```javascript
// SameSite helps but additional protection recommended:

// Server generates CSRF token
app.get('/form', (req, res) => {
  const token = generateRandomToken();
  // Store token in session
  req.session.csrfToken = token;

  res.send(`
    <form method="POST" action="/submit">
      <input type="hidden" name="csrfToken" value="${token}">
      <input type="text" name="data">
      <button type="submit">Submit</button>
    </form>
  `);
});

// Server validates CSRF token
app.post('/submit', (req, res) => {
  const { csrfToken, data } = req.body;

  // Check token matches session
  if (csrfToken !== req.session.csrfToken) {
    return res.status(403).send('CSRF validation failed');
  }

  // Process request
  res.send('Success');
});

// JavaScript CSRF token usage
async function submitForm(data) {
  const formElement = document.querySelector('form');
  const csrfToken = formElement.querySelector('[name="csrfToken"]').value;

  const response = await fetch('/submit', {
    method: 'POST',
    credentials: 'include', // Include cookies
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': csrfToken // Send in header
    },
    body: JSON.stringify(data)
  });

  return response.json();
}

// Best practice: Combine SameSite + CSRF tokens
// SameSite blocks most attacks
// CSRF tokens provide defense-in-depth
```

---

## Cookie Security Best Practices

### Authentication Cookie Setup

```javascript
//  CORRECT: Secure authentication cookie
document.cookie = "authToken=abc123; Path=/; Max-Age=3600; Secure; HttpOnly; SameSite=Strict";

// Breakdown:
// authToken=abc123 - Token value (generated on server)
// Path=/           - Accessible from all paths
// Max-Age=3600     - Expires in 1 hour (refresh mechanism)
// Secure           - HTTPS only (prevents interception)
// HttpOnly         - JavaScript cannot access (prevents XSS theft)
// SameSite=Strict  - Only same-site requests (prevents CSRF)

// L NEVER do this:
document.cookie = "authToken=secret123"; // Missing all security attributes!

// Server-side (Node.js/Express)
app.post('/login', (req, res) => {
  // Verify credentials
  const user = verifyCredentials(req.body.username, req.body.password);

  if (user) {
    // Generate secure token
    const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET, {
      expiresIn: '1h'
    });

    // Set secure cookie
    res.cookie('authToken', token, {
      httpOnly: true,    // Most important: Prevent XSS theft
      secure: true,      // HTTPS only
      sameSite: 'Strict',// CSRF protection
      maxAge: 1000 * 60 * 60, // 1 hour
      path: '/'
    });

    res.json({ success: true });
  } else {
    res.status(401).json({ error: 'Invalid credentials' });
  }
});
```

### Refresh Token Pattern

```javascript
// Long-lived refresh token (7 days) in HttpOnly cookie
res.cookie('refreshToken', refreshToken, {
  httpOnly: true,
  secure: true,
  sameSite: 'Strict',
  maxAge: 1000 * 60 * 60 * 24 * 7, // 7 days
  path: '/'
});

// Short-lived access token (1 hour) in memory or localStorage
const accessToken = jwt.sign({ userId: user.id }, process.env.JWT_SECRET, {
  expiresIn: '1h'
});

// Return access token to JavaScript
res.json({ accessToken });

// Client code
let accessToken = null;

async function login(username, password) {
  const response = await fetch('/login', {
    method: 'POST',
    credentials: 'include', // Include cookies
    body: JSON.stringify({ username, password })
  });

  const { accessToken: newToken } = await response.json();
  accessToken = newToken; // Store in memory

  return true;
}

// Refresh access token when expired
async function refreshAccessToken() {
  const response = await fetch('/refresh', {
    method: 'POST',
    credentials: 'include' // Include refreshToken cookie
  });

  const { accessToken: newToken } = await response.json();
  accessToken = newToken;

  return newToken;
}

// Use access token in requests
async function apiCall(url, options = {}) {
  const headers = options.headers || {};
  headers['Authorization'] = `Bearer ${accessToken}`;

  const response = await fetch(url, {
    ...options,
    headers,
    credentials: 'include' // Include cookies
  });

  if (response.status === 401) {
    // Token expired, refresh
    await refreshAccessToken();
    // Retry with new token
    return apiCall(url, options);
  }

  return response;
}
```

### Cookie Security Checklist

```javascript
// Security checklist for cookies:

const cookieSecurityChecklist = {
  authentication: {
    ' Use HttpOnly': 'Prevents XSS token theft',
    ' Use Secure': 'HTTPS only, prevents interception',
    ' Use SameSite=Strict': 'Prevents CSRF attacks',
    ' Set Max-Age': 'Auto-expires, limits damage of theft',
    ' Regenerate on login': 'Prevents session fixation',
    ' Never store in localStorage': 'Use HttpOnly cookies instead'
  },

  tracking: {
    ' Inform users': 'Cookie consent banner',
    ' Use first-party': 'Avoid third-party tracking',
    ' Respect preferences': 'DNT header, opt-out'
  },

  general: {
    ' Set Path=/': 'Appropriate scope',
    ' Validate values': 'Prevent injection',
    ' Use sameSite': 'At least Lax',
    ' Monitor expiration': 'Prevent stale cookies',
    ' Log cookie usage': 'Security auditing'
  }
};
```

---

## GDPR/CCPA Compliance

### Cookie Consent Requirements

```javascript
// GDPR (General Data Protection Regulation)
// - Requires explicit consent for non-essential cookies
// - Essential cookies (authentication, security) don't need consent
// - Users must easily withdraw consent

// CCPA (California Consumer Privacy Act)
// - Users have right to know what data is collected
// - Users can opt-out of sale of data
// - Businesses must honor opt-out requests

class CookieConsent {
  constructor() {
    this.consentTypes = {
      essential: true, // Always allowed
      analytics: false,
      marketing: false,
      preferences: false
    };

    this.loadConsent();
    this.renderConsentBanner();
  }

  loadConsent() {
    const stored = localStorage.getItem('cookieConsent');
    if (stored) {
      this.consentTypes = JSON.parse(stored);
    }
  }

  saveConsent(consent) {
    this.consentTypes = consent;
    localStorage.setItem('cookieConsent', JSON.stringify(consent));
  }

  renderConsentBanner() {
    const banner = document.createElement('div');
    banner.innerHTML = `
      <div class="cookie-banner">
        <p>We use cookies to improve your experience.</p>
        <button onclick="acceptAll()">Accept All</button>
        <button onclick="rejectAll()">Reject All</button>
        <a href="#" onclick="openSettings()">Settings</a>
      </div>
    `;
    document.body.insertBefore(banner, document.body.firstChild);
  }

  acceptAll() {
    this.saveConsent({
      essential: true,
      analytics: true,
      marketing: true,
      preferences: true
    });
    this.applyConsent();
  }

  rejectAll() {
    this.saveConsent({
      essential: true,
      analytics: false,
      marketing: false,
      preferences: false
    });
    this.applyConsent();
  }

  applyConsent() {
    if (this.consentTypes.analytics) {
      this.loadAnalytics();
    }
    if (this.consentTypes.marketing) {
      this.loadMarketingScripts();
    }
    if (this.consentTypes.preferences) {
      this.loadPreferenceCookies();
    }

    // Remove banner
    document.querySelector('.cookie-banner')?.remove();
  }

  loadAnalytics() {
    // Load Google Analytics, Mixpanel, etc.
  }

  loadMarketingScripts() {
    // Load marketing tracking scripts
  }

  loadPreferenceCookies() {
    // Set non-essential cookies
  }
}

// Initialization
new CookieConsent();
```

### Implementing Cookie Preference Center

```javascript
// React component for cookie preferences
function CookiePreferenceCenter() {
  const [preferences, setPreferences] = React.useState({
    essential: true,
    analytics: false,
    marketing: false
  });

  const handleChange = (type) => {
    setPreferences(prev => ({
      ...prev,
      [type]: !prev[type]
    }));
  };

  const handleSave = () => {
    // Save preferences to localStorage and cookie
    localStorage.setItem('cookiePreferences', JSON.stringify(preferences));

    // Notify server
    fetch('/api/cookie-preferences', {
      method: 'POST',
      credentials: 'include',
      body: JSON.stringify(preferences)
    });

    // Apply preferences
    applyPreferences(preferences);
  };

  return (
    <div className="cookie-preference-center">
      <h2>Cookie Preferences</h2>

      <div className="cookie-group">
        <label>
          <input
            type="checkbox"
            checked={preferences.essential}
            disabled // Always required
          />
          Essential Cookies (Always Enabled)
        </label>
        <p>Required for site functionality</p>
      </div>

      <div className="cookie-group">
        <label>
          <input
            type="checkbox"
            checked={preferences.analytics}
            onChange={() => handleChange('analytics')}
          />
          Analytics Cookies
        </label>
        <p>Help us understand how you use our site</p>
      </div>

      <div className="cookie-group">
        <label>
          <input
            type="checkbox"
            checked={preferences.marketing}
            onChange={() => handleChange('marketing')}
          />
          Marketing Cookies
        </label>
        <p>Used to show you targeted content</p>
      </div>

      <button onClick={handleSave}>Save Preferences</button>
    </div>
  );
}
```

---

## Implementation Examples

### Complete Authentication Flow with Cookies

```javascript
// ========== SERVER SIDE (Node.js/Express) ==========

const express = require('express');
const jwt = require('jsonwebtoken');
const cookieParser = require('cookie-parser');
const app = express();

app.use(express.json());
app.use(cookieParser());

// User database (simplified)
const users = {
  john: { id: 1, password: 'hashed_password_123' }
};

// Login endpoint
app.post('/login', (req, res) => {
  const { username, password } = req.body;

  // Verify credentials (simplified)
  if (users[username]) {
    // Generate tokens
    const accessToken = jwt.sign(
      { userId: users[username].id, username },
      process.env.ACCESS_TOKEN_SECRET,
      { expiresIn: '1h' }
    );

    const refreshToken = jwt.sign(
      { userId: users[username].id },
      process.env.REFRESH_TOKEN_SECRET,
      { expiresIn: '7d' }
    );

    // Set refreshToken in HttpOnly cookie
    res.cookie('refreshToken', refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'Strict',
      maxAge: 1000 * 60 * 60 * 24 * 7, // 7 days
      path: '/'
    });

    // Return accessToken in response (stored in memory on client)
    res.json({
      success: true,
      accessToken,
      user: { id: users[username].id, username }
    });
  } else {
    res.status(401).json({ error: 'Invalid credentials' });
  }
});

// Protected endpoint
app.get('/profile', authenticateToken, (req, res) => {
  res.json({
    user: req.user,
    data: 'User profile data'
  });
});

// Middleware: Verify access token from Authorization header
function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1]; // "Bearer TOKEN"

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid token' });
    }
    req.user = user;
    next();
  });
}

// Refresh token endpoint
app.post('/refresh', (req, res) => {
  const refreshToken = req.cookies.refreshToken;

  if (!refreshToken) {
    return res.status(401).json({ error: 'No refresh token' });
  }

  jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid refresh token' });
    }

    // Generate new access token
    const newAccessToken = jwt.sign(
      { userId: user.userId },
      process.env.ACCESS_TOKEN_SECRET,
      { expiresIn: '1h' }
    );

    res.json({ accessToken: newAccessToken });
  });
});

// Logout endpoint
app.post('/logout', (req, res) => {
  res.clearCookie('refreshToken', {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'Strict',
    path: '/'
  });

  res.json({ success: true });
});

// ========== CLIENT SIDE (React) ==========

function App() {
  const [accessToken, setAccessToken] = React.useState(null);
  const [user, setUser] = React.useState(null);

  const login = async (username, password) => {
    try {
      const response = await fetch('/login', {
        method: 'POST',
        credentials: 'include', // Include cookies
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, password })
      });

      const data = await response.json();
      setAccessToken(data.accessToken);
      setUser(data.user);
      return true;
    } catch (error) {
      console.error('Login failed:', error);
      return false;
    }
  };

  const apiCall = async (url, options = {}) => {
    const headers = options.headers || {};
    headers['Authorization'] = `Bearer ${accessToken}`;

    const response = await fetch(url, {
      ...options,
      credentials: 'include',
      headers
    });

    if (response.status === 401) {
      // Token expired, refresh
      const refreshResponse = await fetch('/refresh', {
        method: 'POST',
        credentials: 'include'
      });

      const { accessToken: newToken } = await refreshResponse.json();
      setAccessToken(newToken);

      // Retry with new token
      return apiCall(url, options);
    }

    return response;
  };

  const logout = async () => {
    await fetch('/logout', {
      method: 'POST',
      credentials: 'include'
    });

    setAccessToken(null);
    setUser(null);
  };

  return (
    <div>
      {!user ? (
        <LoginForm onLogin={login} />
      ) : (
        <div>
          <p>Welcome, {user.username}</p>
          <button onClick={logout}>Logout</button>
        </div>
      )}
    </div>
  );
}
```

---

## Interview Questions

### 1. Explain the three SameSite values and their security implications.

**Answer:**
- **Strict**: Cookie only sent in same-site requests. Most secure, prevents all CSRF attacks. Downside: breaks "stay logged in" across sites.
- **Lax**: Cookie sent in same-site + top-level navigation. Balanced approach, prevents most CSRF while maintaining UX.
- **None**: Cookie sent in all contexts, requires Secure flag. Necessary for third-party scripts but reduces CSRF protection.

```javascript
// Strict: High security, strict user experience
document.cookie = "authToken=xyz; SameSite=Strict; Secure; HttpOnly";

// Lax: Balanced (default in modern browsers)
document.cookie = "sessionId=abc; SameSite=Lax; Secure; HttpOnly";

// None: Third-party cookies, must use Secure
document.cookie = "tracking=123; SameSite=None; Secure";
```

### 2. Why is HttpOnly more important than Secure for authentication cookies?

**Answer:**
- **HttpOnly**: Prevents JavaScript access, mitigates XSS attacks (most common vulnerability)
- **Secure**: Prevents transmission over HTTP, mitigates man-in-the-middle attacks

```javascript
//  HttpOnly prevents XSS token theft
// Set-Cookie: authToken=xyz; HttpOnly; Secure
// Malicious script cannot access: document.cookie // Won't contain authToken

// L Without HttpOnly (vulnerable to XSS)
localStorage.setItem('authToken', 'xyz');
// Malicious script can steal: localStorage.getItem('authToken')
```

Both are important, but HttpOnly is more critical against the most common attack vector.

### 3. How do you prevent CSRF attacks?

**Answer:**
Multiple defense layers:
1. **SameSite=Strict/Lax**: Primary defense, prevents cross-site cookie sending
2. **CSRF tokens**: Secondary defense, unique token per request
3. **Custom headers**: Verify X-CSRF-Token header (XHR has it, forms don't)
4. **Origin checking**: Verify request origin matches

```javascript
// Layer 1: SameSite
res.cookie('sessionId', token, {
  sameSite: 'Strict',
  httpOnly: true,
  secure: true
});

// Layer 2: CSRF token
const csrfToken = generateToken();
res.render('form', { csrfToken });

// Form includes: <input name="csrfToken" value="xyz">

// Verify on submission
app.post('/submit', (req, res) => {
  if (req.body.csrfToken !== req.session.csrfToken) {
    return res.status(403).send('CSRF validation failed');
  }
  // Process request
});
```

### 4. What's the difference between cookies and localStorage for authentication?

**Answer:**
| Feature | Cookies | localStorage |
|---------|---------|--------------|
| **Sent with requests** |  Automatic | L Manual (via headers) |
| **HttpOnly possible** |  Yes (server-set) | L Always accessible to JS |
| **XSS vulnerability** | L Protected (HttpOnly) |  Vulnerable |
| **CSRF protection** |  SameSite attribute | L Must use CSRF tokens |
| **Setup** |  Simple (server-set) | L Complex (manual handling) |

**Recommendation**: Use HttpOnly cookies for authentication tokens.

### 5. How do you implement secure refresh token rotation?

**Answer:**
```javascript
// Strategy: Long-lived refreshToken in HttpOnly cookie, short-lived accessToken in memory

// 1. Login: Issue both tokens
res.cookie('refreshToken', refreshToken, {
  httpOnly: true,
  secure: true,
  sameSite: 'Strict',
  maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
});

res.json({ accessToken }); // Memory storage

// 2. Access request: Check token expiry
app.get('/api/data', (req, res) => {
  const token = req.headers.authorization?.split(' ')[1];

  try {
    jwt.verify(token, process.env.ACCESS_SECRET);
    // Token valid, return data
  } catch (err) {
    // Token expired, return 401
    res.status(401).json({ error: 'Token expired' });
  }
});

// 3. Client: Refresh when expired
if (response.status === 401) {
  const newAccessToken = await fetch('/refresh', {
    method: 'POST',
    credentials: 'include' // Include refreshToken cookie
  }).then(r => r.json());

  setAccessToken(newAccessToken.accessToken);
  // Retry original request
}

// 4. Rotation: Issue new refreshToken on each refresh
app.post('/refresh', (req, res) => {
  const refreshToken = req.cookies.refreshToken;

  jwt.verify(refreshToken, process.env.REFRESH_SECRET, (err, user) => {
    if (err) return res.status(403).json({ error: 'Invalid refresh token' });

    // Issue new tokens
    const newAccessToken = jwt.sign({ userId: user.userId }, process.env.ACCESS_SECRET, { expiresIn: '1h' });
    const newRefreshToken = jwt.sign({ userId: user.userId }, process.env.REFRESH_SECRET, { expiresIn: '7d' });

    // Set new refresh token (replaces old one)
    res.cookie('refreshToken', newRefreshToken, {
      httpOnly: true,
      secure: true,
      sameSite: 'Strict',
      maxAge: 7 * 24 * 60 * 60 * 1000
    });

    res.json({ accessToken: newAccessToken });
  });
});
```

### 6. Explain third-party cookies and privacy concerns.

**Answer:**
Third-party cookies are set by domains other than the one user visits:
- **Analytics**: Google Analytics sets cookie on all sites using it
- **Advertising**: Ad networks track users across multiple sites
- **Privacy concern**: User behavior tracked across unrelated websites

```javascript
// First-party cookie (normal)
document.cookie = "sessionId=xyz"; // Set by example.com, sent to example.com

// Third-party cookie (tracking)
// example.com includes: <img src="https://ads.com/pixel.gif">
// ads.com sets: Set-Cookie: userId=123; Domain=.ads.com
// Now ads.com can identify user across ANY site they visit
```

Browser restrictions: Safari (ITP), Chrome (Privacy Sandbox), Firefox (ETP) all restricting third-party cookies for privacy.

### 7. How do you implement GDPR-compliant cookie consent?

**Answer:**
```javascript
// 1. Display consent banner before setting non-essential cookies
function showConsentBanner() {
  // Don't set any tracking cookies yet
  // Only essential cookies are set

  const banner = createBanner();
  document.body.appendChild(banner);
}

// 2. Only set cookies after user consent
function acceptCookies(preferences) {
  localStorage.setItem('cookieConsent', JSON.stringify(preferences));

  if (preferences.analytics) {
    // Now load analytics
    loadAnalyticsScript();
  }

  if (preferences.marketing) {
    loadMarketingScripts();
  }
}

// 3. Provide easy access to change preferences
function openPreferenceCenter() {
  // Allow user to modify consent without page reload
}

// 4. Save preferences and communicate to server
async function saveCookiePreferences(preferences) {
  localStorage.setItem('cookieConsent', JSON.stringify(preferences));

  // Notify server for GDPR record-keeping
  await fetch('/api/cookie-preferences', {
    method: 'POST',
    credentials: 'include',
    body: JSON.stringify({
      preferences,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      cookieId: generateOrGetConsentId()
    })
  });
}

// 5. Implement cookie policy link
// - Transparent about what cookies are used
// - How long they're retained
// - How to manage preferences
```

### 8. What security issues can occur with same-site cookies?

**Answer:**
Even with proper SameSite settings, issues can arise:
```javascript
// Issue 1: SameSite=Lax allows GET form submissions
// Attacker: <form method="GET" action="https://bank.com/transfer?amount=1000">
// Browser SENDS cookie with GET navigation (even though it's cross-site)

// Issue 2: SameSite doesn't protect against same-site attacks
// Attacker controls subdomain (attacker.example.com)
// Cookie accessible from subdomain (if Domain=.example.com)

// Issue 3: Legacy browsers (pre-2020) don't support SameSite
// Must use additional CSRF token protection

// Defense: Combine multiple protections
// 1. SameSite=Strict (modern browsers)
// 2. CSRF tokens (legacy browser support, additional defense)
// 3. Origin validation (server-side)
// 4. Custom headers (AJAX requests)

app.post('/transfer', (req, res) => {
  // Verify SameSite (indirectly - check if cookie was sent)
  if (!req.cookies.sessionId) {
    return res.status(403).send('CSRF: No cookie');
  }

  // Verify CSRF token
  if (req.body.csrfToken !== req.session.csrfToken) {
    return res.status(403).send('CSRF: Invalid token');
  }

  // Verify Origin header
  if (req.get('origin') !== 'https://bank.com') {
    return res.status(403).send('CSRF: Invalid origin');
  }

  // All checks passed
  res.send('Transfer processed');
});
```

### 9. How do cookies differ from tokens in terms of security?

**Answer:**
```javascript
// Cookies
// - Automatically sent with requests (convenience)
// - Can be HttpOnly (secure against XSS)
// - Vulnerable to CSRF (needs SameSite/CSRF tokens)
// - Server-side session state required
// - Limited to 4KB per cookie

// Tokens (JWT in Authorization header)
// - Must be manually sent in headers
// - Vulnerable to XSS (JavaScript accessible)
// - Protected from CSRF (custom header not sent cross-site)
// - Stateless (no server session needed)
// - Can store more data

// Best practice: Hybrid approach
// - HttpOnly cookie for refresh token (long-lived, secure)
// - Memory-stored access token (short-lived)
// - Automatic retry on expiration

// Implementation
// 1. Login: Issue refresh token in HttpOnly cookie
res.cookie('refreshToken', token, { httpOnly: true, secure: true });

// 2. Return access token in response (stored in memory)
res.json({ accessToken });

// 3. Requests use access token in Authorization header
const response = await fetch('/api/data', {
  headers: { 'Authorization': `Bearer ${accessToken}` }
});

// 4. Expired access token triggers refresh
// - Client calls /refresh endpoint
// - Server uses refreshToken from cookie
// - Server returns new access token
```

### 10. Explain the Privacy Sandbox and alternatives to third-party cookies.

**Answer:**
Third-party cookies are being phased out. Alternatives:
```javascript
// Privacy Sandbox APIs (Google's proposal)
// 1. Topics API: Browser learns user interests, shares with advertisers
// 2. FLEDGE: On-device ad auction, no data shared
// 3. Aggregation Service: Privacy-preserving reporting

// Server-side solutions
// 1. First-party data collection
//    - Your site collects user data
//    - Your server shares insights (not raw data) with advertisers

// 2. Consent-based sharing
//    - User explicitly consents
//    - Data tied to user (no anonymous tracking)

// Implementation example: First-party analytics
app.get('/page', (req, res) => {
  // Your domain sets first-party cookie
  res.cookie('analytics_id', generateId(), { path: '/', secure: true });

  // Page includes analytics script
  res.send(`
    <html>
      <script src="/analytics.js"></script>
    </html>
  `);
});

// Analytics script sends events to your server
function trackEvent(event) {
  fetch('/api/analytics', {
    method: 'POST',
    credentials: 'include', // First-party cookie
    body: JSON.stringify({ event, timestamp: Date.now() })
  });
}

// Your server: Forward aggregated insights to third-party
app.post('/api/analytics', (req, res) => {
  const analyticsId = req.cookies.analytics_id;

  // Log event (your server stores it)
  logEvent(analyticsId, req.body.event);

  // Later: Send aggregated data to analytics service
  // (Never raw user data, only aggregates)
  sendAggregatedInsights();

  res.json({ success: true });
});

// Benefits:
// - Privacy: No third-party tracking
// - Control: You control data sharing
// - Transparency: Users know what you collect
// - Compliance: GDPR/CCPA compliant
```

---

## Summary

### Key Takeaways

1. **Use HttpOnly + Secure + SameSite=Strict** for authentication cookies
2. **SameSite=Strict** prevents CSRF attacks automatically
3. **HttpOnly is more important than Secure** against XSS (most common)
4. **Combine SameSite + CSRF tokens** for defense-in-depth
5. **Use refresh token rotation** for long-term sessions
6. **Implement cookie consent** for GDPR/CCPA compliance
7. **Avoid third-party cookies** when possible
8. **Set appropriate Domain/Path** to limit cookie scope
9. **Use Max-Age** for clearer expiration than Expires
10. **Validate all cookie data** on the server

### Cookie Security Checklist

**Do's:**
- Set Secure flag in production
- Set HttpOnly for sensitive cookies
- Use SameSite=Strict or Lax
- Regenerate session ID on login
- Implement CSRF tokens as defense-in-depth
- Expire cookies appropriately
- Validate cookie data server-side
- Use HTTPS in production
- Implement cookie consent banners
- Monitor cookie usage and compliance

**Don'ts:**
- Don't store passwords in cookies
- Don't set cookies without SameSite
- Don't skip Secure flag in production
- Don't use SameSite=None without Secure
- Don't trust cookie data without validation
- Don't expose sensitive data in cookie values
- Don't forget to delete cookies on logout
- Don't ignore GDPR/CCPA requirements
- Don't use cookies without user consent (if required)
- Don't mix cookie authentication with localStorage

---

## External Resources

### Official Documentation
- [MDN - HTTP Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [MDN - Set-Cookie Header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [MDN - SameSite Cookie Attribute](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
- [WHATWG - Cookie Living Standard](https://html.spec.whatwg.org/multipage/cookies.html)

### Security Resources
- [OWASP - Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP - CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [web.dev - SameSite Cookies Explained](https://web.dev/samesite-cookies-explained/)

### Privacy and Compliance
- [GDPR Official Text](https://gdpr-info.eu/)
- [CCPA Official Information](https://oag.ca.gov/privacy/ccpa)
- [Privacy Sandbox (Google)](https://privacysandbox.com/)

### Browser APIs
- [MDN - Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [Can I Use - SameSite Cookie Attribute](https://caniuse.com/same-site-cookie-attribute)

### Related Articles
- [JWT vs Cookies: Security](https://blog.bytebytego.com/p/ep91-authentication-session-cookie)
- [Understanding Cookie Security](https://www.troyhunt.com/c/)
- [js-cookie Library](https://github.com/js-cookie/js-cookie)

---

## Navigation

**Previous:** [01-storage-apis.md](./01-storage-apis.md)

**Next:** [03-indexeddb.md](./03-indexeddb.md)

---

**Last Updated**: December 2025
**Difficulty**: Intermediate to Advanced
**Time to Complete**: 5-6 hours
**Repository**: [interview-preparation](https://github.com/salman-rahman/interview-preparation)
