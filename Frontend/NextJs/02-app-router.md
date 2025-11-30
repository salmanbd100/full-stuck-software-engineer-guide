# Next.js App Router

## Table of Contents
- [Introduction](#introduction)
- [Server vs Client Components](#server-vs-client-components)
- [File Conventions](#file-conventions)
- [Layouts and Templates](#layouts-and-templates)
- [Route Groups](#route-groups)
- [Dynamic Routes](#dynamic-routes)
- [Loading and Error States](#loading-and-error-states)
- [Interview Questions](#interview-questions)

---

## Introduction

The App Router is Next.js 13+ routing system built on React Server Components. It provides a more powerful and flexible routing system compared to the Pages Router.

### Key Features
- Server Components by default
- Layouts and nested routing
- Loading and error states
- Parallel and intercepting routes
- Streaming and Suspense
- Improved data fetching

---

## Server vs Client Components

### Server Components (Default)

Components that render on the server.

```jsx
// app/page.js - Server Component by default
export default async function Page() {
  // Can directly access backend resources
  const data = await fetch('https://api.example.com/data');
  const posts = await data.json();

  return (
    <div>
      <h1>Posts</h1>
      {posts.map(post => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  );
}
```

**Benefits:**
- Zero JavaScript bundle
- Direct database/API access
- Better security (API keys stay on server)
- Better performance

**Limitations:**
- No browser APIs (window, localStorage)
- No event handlers (onClick, onChange)
- No hooks (useState, useEffect)
- No browser-only libraries

### Client Components

Use `'use client'` directive at the top of the file.

```jsx
// app/components/Counter.js
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

**When to use Client Components:**
- Interactive UI (onClick, onChange)
- State management (useState, useReducer)
- Effects (useEffect, useLayoutEffect)
- Browser APIs (window, localStorage)
- Custom hooks
- Event listeners

### Composition Pattern

```jsx
// app/page.js - Server Component
import ClientComponent from './ClientComponent';

export default async function Page() {
  const data = await fetchData();

  return (
    <div>
      {/* Server-rendered content */}
      <h1>{data.title}</h1>

      {/* Client-interactive component */}
      <ClientComponent initialData={data} />
    </div>
  );
}
```

---

## File Conventions

```
app/
├── layout.js         # Root layout (required)
├── page.js           # Home page (/)
├── loading.js        # Loading UI
├── error.js          # Error UI
├── not-found.js      # 404 UI
├── template.js       # Re-rendered layout
├── route.js          # API endpoint
└── about/
    ├── page.js       # /about
    └── layout.js     # Layout for /about
```

### page.js
Defines a unique page for a route.

```jsx
// app/about/page.js
export default function AboutPage() {
  return <h1>About Us</h1>;
}
```

### layout.js
Shared UI that wraps child pages.

```jsx
// app/layout.js (Root layout - required)
export const metadata = {
  title: 'My App',
  description: 'My awesome app'
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <header>
          <nav>Navigation</nav>
        </header>
        <main>{children}</main>
        <footer>Footer</footer>
      </body>
    </html>
  );
}
```

**Nested layouts:**
```jsx
// app/dashboard/layout.js
export default function DashboardLayout({ children }) {
  return (
    <div className="dashboard">
      <aside>Sidebar</aside>
      <div>{children}</div>
    </div>
  );
}
```

### loading.js
Instant loading UI with Suspense.

```jsx
// app/dashboard/loading.js
export default function Loading() {
  return (
    <div className="spinner">
      <p>Loading...</p>
    </div>
  );
}
```

### error.js
Error boundary for route segment.

```jsx
// app/dashboard/error.js
'use client'; // Error components must be Client Components

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

### not-found.js
Custom 404 page.

```jsx
// app/not-found.js
import Link from 'next/link';

export default function NotFound() {
  return (
    <div>
      <h2>Not Found</h2>
      <p>Could not find requested resource</p>
      <Link href="/">Return Home</Link>
    </div>
  );
}
```

---

## Layouts and Templates

### Layouts
Preserve state and don't re-render.

```jsx
// app/dashboard/layout.js
export default function DashboardLayout({ children }) {
  return (
    <div>
      <DashboardNav />  {/* Preserves state */}
      {children}
    </div>
  );
}
```

### Templates
Re-render on navigation.

```jsx
// app/dashboard/template.js
export default function DashboardTemplate({ children }) {
  return (
    <div>
      <AnimatedWrapper>  {/* Re-mounts on navigation */}
        {children}
      </AnimatedWrapper>
    </div>
  );
}
```

**When to use templates:**
- Enter/exit animations
- Reset state on navigation
- Analytics page views
- Features that depend on useEffect

---

## Route Groups

Organize routes without affecting URL structure using `(folder)`.

```
app/
├── (marketing)/
│   ├── layout.js       # Marketing layout
│   ├── about/
│   │   └── page.js     # /about
│   └── contact/
│       └── page.js     # /contact
└── (shop)/
    ├── layout.js       # Shop layout
    ├── products/
    │   └── page.js     # /products
    └── cart/
        └── page.js     # /cart
```

**Benefits:**
- Different layouts for different sections
- Organize files logically
- Create multiple root layouts

**Example:**
```jsx
// app/(marketing)/layout.js
export default function MarketingLayout({ children }) {
  return (
    <div>
      <MarketingNav />
      {children}
    </div>
  );
}

// app/(shop)/layout.js
export default function ShopLayout({ children }) {
  return (
    <div>
      <ShopNav />
      {children}
    </div>
  );
}
```

---

## Dynamic Routes

### Single Dynamic Segment

```
app/
└── blog/
    └── [slug]/
        └── page.js     # /blog/:slug
```

```jsx
// app/blog/[slug]/page.js
export default function BlogPost({ params }) {
  return <h1>Post: {params.slug}</h1>;
}

// Generate static params for SSG
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then(res => res.json());

  return posts.map((post) => ({
    slug: post.slug
  }));
}
```

### Multiple Dynamic Segments

```
app/
└── shop/
    └── [category]/
        └── [product]/
            └── page.js  # /shop/:category/:product
```

```jsx
// app/shop/[category]/[product]/page.js
export default function Product({ params }) {
  const { category, product } = params;

  return (
    <div>
      <p>Category: {category}</p>
      <p>Product: {product}</p>
    </div>
  );
}
```

### Catch-All Segments

```
app/
└── docs/
    └── [...slug]/
        └── page.js     # /docs/*, /docs/*/*, etc.
```

```jsx
// app/docs/[...slug]/page.js
export default function Docs({ params }) {
  const { slug } = params;
  // slug is an array: ["getting-started", "installation"]

  return (
    <div>
      <h1>Docs: {slug.join(' / ')}</h1>
    </div>
  );
}
```

### Optional Catch-All

```
app/
└── shop/
    └── [[...slug]]/
        └── page.js     # /shop, /shop/*, /shop/*/*
```

```jsx
// app/shop/[[...slug]]/page.js
export default function Shop({ params }) {
  const { slug } = params;

  if (!slug) {
    return <h1>Shop Home</h1>;
  }

  return <h1>Category: {slug.join(' / ')}</h1>;
}
```

---

## Loading and Error States

### Loading UI

Automatic with `loading.js`:

```jsx
// app/dashboard/loading.js
export default function Loading() {
  return <Skeleton />;
}
```

Manual with Suspense:

```jsx
// app/dashboard/page.js
import { Suspense } from 'react';

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<Spinner />}>
        <DataTable />
      </Suspense>
    </div>
  );
}
```

### Error Handling

```jsx
// app/dashboard/error.js
'use client';

import { useEffect } from 'react';

export default function Error({ error, reset }) {
  useEffect(() => {
    // Log error to service
    console.error(error);
  }, [error]);

  return (
    <div>
      <h2>Dashboard Error</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

### Global Error

```jsx
// app/global-error.js
'use client';

export default function GlobalError({ error, reset }) {
  return (
    <html>
      <body>
        <h2>Something went wrong!</h2>
        <button onClick={reset}>Try again</button>
      </body>
    </html>
  );
}
```

---

## Interview Questions

### Q1: What's the difference between Server and Client Components?

**Answer:**

**Server Components (default):**
- Render on the server
- Zero JavaScript bundle
- Can access backend directly
- Cannot use hooks or event handlers

**Client Components (`'use client'`):**
- Render on client
- Add JavaScript to bundle
- Can use hooks and interactivity
- Cannot directly access backend

```jsx
// Server Component
async function ServerComponent() {
  const data = await db.query(); // Direct DB access
  return <div>{data}</div>;
}

// Client Component
'use client';
function ClientComponent() {
  const [count, setCount] = useState(0); // Hooks allowed
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

---

### Q2: What's the purpose of route groups `(folder)`?

**Answer:**
Route groups organize files without affecting the URL structure.

**Benefits:**
1. Organize routes logically
2. Apply different layouts to different sections
3. Create multiple root layouts

```
app/
├── (marketing)/
│   ├── layout.js       # Marketing layout
│   ├── about/page.js   # /about (not /(marketing)/about)
│   └── contact/page.js # /contact
└── (app)/
    ├── layout.js       # App layout
    ├── dashboard/page.js # /dashboard
    └── settings/page.js  # /settings
```

The `(marketing)` and `(app)` folders don't appear in URLs.

---

### Q3: What's the difference between `layout.js` and `template.js`?

**Answer:**

**layout.js:**
- Preserves state across navigation
- Doesn't re-render
- Use for persistent UI (nav, sidebar)

**template.js:**
- Re-renders on every navigation
- Resets state
- Use for animations, analytics, effects

```jsx
// layout.js - state preserved
export default function Layout({ children }) {
  return <div>{children}</div>; // Same instance persists
}

// template.js - new instance on navigation
export default function Template({ children }) {
  return <div>{children}</div>; // New instance each time
}
```

---

### Q4: How do you handle loading states in App Router?

**Answer:**

**Method 1: loading.js (automatic Suspense)**
```jsx
// app/dashboard/loading.js
export default function Loading() {
  return <Spinner />;
}
```

**Method 2: Manual Suspense**
```jsx
// app/page.js
import { Suspense } from 'react';

export default function Page() {
  return (
    <Suspense fallback={<Spinner />}>
      <AsyncComponent />
    </Suspense>
  );
}
```

**Method 3: Streaming specific components**
```jsx
export default function Page() {
  return (
    <div>
      <Header /> {/* Renders immediately */}
      <Suspense fallback={<Skeleton />}>
        <SlowData /> {/* Streams in when ready */}
      </Suspense>
    </div>
  );
}
```

---

### Q5: How do you create catch-all routes that also match the base path?

**Answer:**
Use **optional catch-all routes** with double square brackets: `[[...slug]]`

```
app/
└── shop/
    └── [[...slug]]/
        └── page.js
```

```jsx
// app/shop/[[...slug]]/page.js
export default function Shop({ params }) {
  const { slug } = params;

  if (!slug) {
    // Matches /shop
    return <ShopHome />;
  }

  // Matches /shop/*, /shop/*/*, etc.
  return <Category path={slug} />;
}
```

**Routes:**
- `/shop` → `slug = undefined`
- `/shop/electronics` → `slug = ["electronics"]`
- `/shop/electronics/phones` → `slug = ["electronics", "phones"]`

---

### Q6: How do you handle errors in App Router?

**Answer:**

**Route-level error:**
```jsx
// app/dashboard/error.js
'use client';

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Error in Dashboard</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Retry</button>
    </div>
  );
}
```

**Global error:**
```jsx
// app/global-error.js
'use client';

export default function GlobalError({ error, reset }) {
  return (
    <html>
      <body>
        <h2>Global Error</h2>
        <button onClick={reset}>Retry</button>
      </body>
    </html>
  );
}
```

**Not found:**
```jsx
// app/not-found.js
export default function NotFound() {
  return <h2>Page Not Found</h2>;
}

// Trigger programmatically
import { notFound } from 'next/navigation';

export default function Page({ params }) {
  const data = await fetchData(params.id);

  if (!data) {
    notFound(); // Shows not-found.js
  }

  return <div>{data.title}</div>;
}
```

---

## Key Takeaways

1. **Server Components are default** - Better performance, zero JS
2. **'use client' for interactivity** - Hooks, events, browser APIs
3. **File conventions** - layout, page, loading, error, not-found
4. **Route groups (folder)** - Organize without affecting URLs
5. **Dynamic routes** - [slug], [...slug], [[...slug]]
6. **Automatic loading states** - loading.js creates Suspense boundary
7. **Error boundaries** - error.js for granular error handling

---

## Next Steps

- [Rendering Strategies](./03-rendering-strategies.md)
- [Data Fetching](./04-data-fetching.md)
- [API Routes](./05-api-routes.md)

---

[← Back to Next.js](./README.md)
