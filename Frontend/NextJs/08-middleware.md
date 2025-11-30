# Next.js Middleware

## Table of Contents
- [What is Middleware?](#what-is-middleware)
- [Basic Middleware](#basic-middleware)
- [Authentication](#authentication)
- [Redirects and Rewrites](#redirects-and-rewrites)
- [Headers and Cookies](#headers-and-cookies)
- [Advanced Patterns](#advanced-patterns)
- [Interview Questions](#interview-questions)

---

## What is Middleware?

Middleware runs before a request is completed, allowing you to modify the response by rewriting, redirecting, modifying headers, or responding directly.

**Use cases:**
- Authentication/Authorization
- Redirects and rewrites
- A/B testing
- Geolocation
- Bot protection
- Feature flags

---

## Basic Middleware

### Creating Middleware

```js
// middleware.js (root level)
import { NextResponse } from 'next/server';

export function middleware(request) {
  console.log('Middleware running for:', request.url);

  // Continue to the next middleware or route
  return NextResponse.next();
}
```

### Matcher Configuration

```js
// middleware.js
export function middleware(request) {
  // Your logic
}

// Match specific paths
export const config = {
  matcher: [
    '/dashboard/:path*',
    '/admin/:path*',
    '/api/:path*'
  ]
};
```

**Matcher patterns:**
```js
export const config = {
  // Single path
  matcher: '/about',

  // Multiple paths
  matcher: ['/about', '/dashboard'],

  // Dynamic routes
  matcher: '/dashboard/:path*',

  // Regex patterns
  matcher: '/((?!api|_next/static|_next/image|favicon.ico).*)',

  // Conditional matching
  matcher: [
    '/((?!_next|api|favicon.ico).*)',
  ],
};
```

---

## Authentication

### Protected Routes

```js
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  // Check for auth token
  const token = request.cookies.get('token')?.value;

  // Redirect to login if not authenticated
  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Allow request to continue
  return NextResponse.next();
}

// Protect dashboard routes
export const config = {
  matcher: ['/dashboard/:path*', '/profile/:path*']
};
```

### JWT Verification

```js
// middleware.js
import { NextResponse } from 'next/server';
import { jwtVerify } from 'jose';

export async function middleware(request) {
  const token = request.cookies.get('token')?.value;

  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  try {
    // Verify JWT
    const secret = new TextEncoder().encode(process.env.JWT_SECRET);
    const { payload } = await jwtVerify(token, secret);

    // Add user info to header
    const response = NextResponse.next();
    response.headers.set('x-user-id', payload.userId);

    return response;
  } catch (error) {
    // Invalid token - redirect to login
    return NextResponse.redirect(new URL('/login', request.url));
  }
}

export const config = {
  matcher: '/dashboard/:path*'
};
```

### Role-Based Access

```js
// middleware.js
import { NextResponse } from 'next/server';

export async function middleware(request) {
  const token = request.cookies.get('token')?.value;

  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  const user = await verifyToken(token);

  // Check if accessing admin route
  if (request.nextUrl.pathname.startsWith('/admin')) {
    if (user.role !== 'admin') {
      return NextResponse.redirect(new URL('/unauthorized', request.url));
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/admin/:path*']
};
```

---

## Redirects and Rewrites

### Redirects

```js
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  const { pathname } = request.nextUrl;

  // Redirect old URLs
  if (pathname === '/old-page') {
    return NextResponse.redirect(new URL('/new-page', request.url));
  }

  // Redirect with status code
  if (pathname === '/temp-moved') {
    return NextResponse.redirect(new URL('/new-location', request.url), 307);
  }

  // Permanent redirect (301)
  if (pathname.startsWith('/blog-old/')) {
    const slug = pathname.replace('/blog-old/', '');
    return NextResponse.redirect(
      new URL(`/blog/${slug}`, request.url),
      301
    );
  }

  return NextResponse.next();
}
```

### Rewrites

```js
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  // Rewrite to different page (URL stays the same)
  if (request.nextUrl.pathname === '/about') {
    return NextResponse.rewrite(new URL('/company/about', request.url));
  }

  // Proxy to external API
  if (request.nextUrl.pathname.startsWith('/api/proxy')) {
    return NextResponse.rewrite('https://external-api.com/data');
  }

  return NextResponse.next();
}
```

---

## Headers and Cookies

### Setting Headers

```js
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  const response = NextResponse.next();

  // Set custom headers
  response.headers.set('x-custom-header', 'value');
  response.headers.set('x-request-id', crypto.randomUUID());

  // CORS headers
  response.headers.set('Access-Control-Allow-Origin', '*');
  response.headers.set('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE');

  // Security headers
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-XSS-Protection', '1; mode=block');

  return response;
}
```

### Managing Cookies

```js
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  const response = NextResponse.next();

  // Read cookie
  const sessionId = request.cookies.get('session')?.value;

  // Set cookie
  response.cookies.set('visited', 'true', {
    path: '/',
    maxAge: 60 * 60 * 24 * 7, // 7 days
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
  });

  // Delete cookie
  response.cookies.delete('old-cookie');

  return response;
}
```

---

## Advanced Patterns

### A/B Testing

```js
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  // Check if user already has variant
  let variant = request.cookies.get('variant')?.value;

  if (!variant) {
    // Assign random variant
    variant = Math.random() < 0.5 ? 'A' : 'B';
  }

  const response = variant === 'A'
    ? NextResponse.rewrite(new URL('/variant-a', request.url))
    : NextResponse.rewrite(new URL('/variant-b', request.url));

  // Save variant in cookie
  response.cookies.set('variant', variant, {
    path: '/',
    maxAge: 60 * 60 * 24 * 30, // 30 days
  });

  return response;
}

export const config = {
  matcher: '/landing-page'
};
```

### Geolocation

```js
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  // Vercel provides geo data
  const country = request.geo?.country || 'US';
  const city = request.geo?.city || '';

  // Redirect based on location
  if (country === 'GB') {
    return NextResponse.rewrite(new URL('/gb', request.url));
  }

  if (country === 'DE') {
    return NextResponse.rewrite(new URL('/de', request.url));
  }

  // Add geo info to headers
  const response = NextResponse.next();
  response.headers.set('x-geo-country', country);
  response.headers.set('x-geo-city', city);

  return response;
}
```

### Feature Flags

```js
// middleware.js
import { NextResponse } from 'next/server';

const FEATURE_FLAGS = {
  newUI: process.env.FEATURE_NEW_UI === 'true',
  betaFeatures: process.env.FEATURE_BETA === 'true',
};

export function middleware(request) {
  const response = NextResponse.next();

  // Add feature flags to headers
  response.headers.set('x-features', JSON.stringify(FEATURE_FLAGS));

  // Redirect based on feature flag
  if (FEATURE_FLAGS.newUI && request.nextUrl.pathname === '/') {
    return NextResponse.rewrite(new URL('/new-home', request.url));
  }

  return response;
}
```

### Rate Limiting

```js
// middleware.js
import { NextResponse } from 'next/server';

const rateLimit = new Map();

export function middleware(request) {
  const ip = request.ip || 'unknown';
  const now = Date.now();
  const windowMs = 60 * 1000; // 1 minute
  const maxRequests = 10;

  // Get or create rate limit entry
  if (!rateLimit.has(ip)) {
    rateLimit.set(ip, { count: 0, resetTime: now + windowMs });
  }

  const limit = rateLimit.get(ip);

  // Reset if window expired
  if (now > limit.resetTime) {
    limit.count = 0;
    limit.resetTime = now + windowMs;
  }

  // Check limit
  limit.count++;

  if (limit.count > maxRequests) {
    return new NextResponse('Too Many Requests', { status: 429 });
  }

  return NextResponse.next();
}

export const config = {
  matcher: '/api/:path*'
};
```

---

## Interview Questions

### Q1: What is middleware in Next.js and when should you use it?

**Answer:**

Middleware runs before a request is completed, allowing you to modify responses.

**Use cases:**
- Authentication/authorization
- Redirects based on conditions
- A/B testing
- Geolocation routing
- Bot protection
- Header modification

```js
export function middleware(request) {
  const token = request.cookies.get('token');

  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}
```

---

### Q2: How do you protect routes with middleware?

**Answer:**

```js
import { NextResponse } from 'next/server';

export function middleware(request) {
  const token = request.cookies.get('token')?.value;

  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/profile/:path*']
};
```

---

### Q3: What's the difference between redirect and rewrite?

**Answer:**

**Redirect:**
- Changes the URL in browser
- Client sees new URL
- Can be temporary (307) or permanent (301)

**Rewrite:**
- URL stays the same in browser
- Serves content from different path
- Transparent to user

```js
// Redirect - URL changes to /new-page
return NextResponse.redirect(new URL('/new-page', request.url));

// Rewrite - URL stays /old-page, content from /new-page
return NextResponse.rewrite(new URL('/new-page', request.url));
```

---

### Q4: How do you implement A/B testing with middleware?

**Answer:**

```js
export function middleware(request) {
  let variant = request.cookies.get('variant')?.value;

  if (!variant) {
    variant = Math.random() < 0.5 ? 'A' : 'B';
  }

  const url = variant === 'A'
    ? new URL('/variant-a', request.url)
    : new URL('/variant-b', request.url);

  const response = NextResponse.rewrite(url);
  response.cookies.set('variant', variant, { maxAge: 2592000 });

  return response;
}
```

---

### Q5: How do you set custom headers in middleware?

**Answer:**

```js
export function middleware(request) {
  const response = NextResponse.next();

  // Custom headers
  response.headers.set('x-custom-header', 'value');
  response.headers.set('x-request-id', crypto.randomUUID());

  // Security headers
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-Frame-Options', 'DENY');

  return response;
}
```

---

## Key Takeaways

1. **Middleware runs before request completion** - modify responses
2. **Use matcher** to target specific paths
3. **Authentication** - protect routes, verify tokens
4. **Redirects** - change URL permanently or temporarily
5. **Rewrites** - serve different content, same URL
6. **Headers/Cookies** - modify request/response data
7. **Advanced patterns** - A/B testing, geolocation, rate limiting

---

[← Back to Next.js](./README.md)
