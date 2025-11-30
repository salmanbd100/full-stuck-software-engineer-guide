# Next.js Pages Router

## Table of Contents
- [Introduction](#introduction)
- [File-Based Routing](#file-based-routing)
- [Dynamic Routes](#dynamic-routes)
- [Navigation](#navigation)
- [Programmatic Routing](#programmatic-routing)
- [Route Parameters](#route-parameters)
- [Interview Questions](#interview-questions)

---

## Introduction

Next.js uses a file-based routing system where the file structure in the `pages` directory automatically becomes the route structure of your application.

### Why Pages Router?
- No routing configuration needed
- Automatic code splitting per page
- Pre-rendering (SSG/SSR) out of the box
- Simple and intuitive
- Hot module replacement

---

## File-Based Routing

### Basic Routes

```
pages/
├── index.js          → /
├── about.js          → /about
├── contact.js        → /contact
└── blog/
    ├── index.js      → /blog
    └── post.js       → /blog/post
```

**Example: pages/index.js**
```jsx
export default function Home() {
  return <h1>Welcome to the Home Page</h1>;
}
```

**Example: pages/about.js**
```jsx
export default function About() {
  return <h1>About Us</h1>;
}
```

### Nested Routes

```
pages/
├── index.js                    → /
└── dashboard/
    ├── index.js                → /dashboard
    ├── settings.js             → /dashboard/settings
    └── profile/
        ├── index.js            → /dashboard/profile
        └── edit.js             → /dashboard/profile/edit
```

**Example: pages/dashboard/settings.js**
```jsx
export default function Settings() {
  return (
    <div>
      <h1>Dashboard Settings</h1>
      <p>Configure your preferences</p>
    </div>
  );
}
```

---

## Dynamic Routes

### Single Dynamic Route

Use square brackets `[param]` for dynamic segments.

**File:** `pages/blog/[slug].js`
```jsx
import { useRouter } from 'next/router';

export default function BlogPost() {
  const router = useRouter();
  const { slug } = router.query;

  return (
    <div>
      <h1>Blog Post: {slug}</h1>
      <p>This is the content for {slug}</p>
    </div>
  );
}
```

**Routes:**
- `/blog/hello-world` → `slug = "hello-world"`
- `/blog/nextjs-guide` → `slug = "nextjs-guide"`

### Multiple Dynamic Segments

```
pages/
└── blog/
    └── [category]/
        └── [slug].js       → /blog/:category/:slug
```

**File:** `pages/blog/[category]/[slug].js`
```jsx
import { useRouter } from 'next/router';

export default function Post() {
  const router = useRouter();
  const { category, slug } = router.query;

  return (
    <div>
      <h2>Category: {category}</h2>
      <h1>Post: {slug}</h1>
    </div>
  );
}
```

**Routes:**
- `/blog/tech/nextjs` → `{ category: "tech", slug: "nextjs" }`
- `/blog/design/ui-patterns` → `{ category: "design", slug: "ui-patterns" }`

### Catch-All Routes

Use `[...param]` to catch all subsequent segments.

**File:** `pages/docs/[...slug].js`
```jsx
import { useRouter } from 'next/router';

export default function Docs() {
  const router = useRouter();
  const { slug } = router.query;

  // slug is an array of path segments
  return (
    <div>
      <h1>Documentation</h1>
      <p>Path segments: {slug?.join(' / ')}</p>
    </div>
  );
}
```

**Routes:**
- `/docs/api` → `slug = ["api"]`
- `/docs/api/users` → `slug = ["api", "users"]`
- `/docs/api/users/create` → `slug = ["api", "users", "create"]`

### Optional Catch-All Routes

Use `[[...param]]` to make catch-all routes optional.

**File:** `pages/blog/[[...slug]].js`
```jsx
import { useRouter } from 'next/router';

export default function Blog() {
  const router = useRouter();
  const { slug } = router.query;

  if (!slug) {
    // Matches /blog
    return <h1>Blog Home</h1>;
  }

  // Matches /blog/*, /blog/*/*, etc.
  return (
    <div>
      <h1>Blog Post</h1>
      <p>Path: {slug.join(' / ')}</p>
    </div>
  );
}
```

**Routes:**
- `/blog` → `slug = undefined`
- `/blog/post-1` → `slug = ["post-1"]`
- `/blog/2024/post-1` → `slug = ["2024", "post-1"]`

---

## Navigation

### Link Component

The `Link` component enables client-side navigation.

```jsx
import Link from 'next/link';

export default function Navigation() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>
      <Link href="/blog/hello-world">Blog Post</Link>
    </nav>
  );
}
```

### Link with Dynamic Routes

```jsx
import Link from 'next/link';

export default function BlogList({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          {/* Method 1: String interpolation */}
          <Link href={`/blog/${post.slug}`}>
            {post.title}
          </Link>

          {/* Method 2: Object syntax */}
          <Link href={{
            pathname: '/blog/[slug]',
            query: { slug: post.slug }
          }}>
            {post.title}
          </Link>
        </li>
      ))}
    </ul>
  );
}
```

### Prefetching

Links are automatically prefetched in production.

```jsx
import Link from 'next/link';

export default function Nav() {
  return (
    <nav>
      {/* Prefetch enabled (default in production) */}
      <Link href="/about">About</Link>

      {/* Disable prefetch */}
      <Link href="/contact" prefetch={false}>
        Contact
      </Link>
    </nav>
  );
}
```

### Replace vs Push

```jsx
import Link from 'next/link';

export default function Links() {
  return (
    <div>
      {/* Adds to history (default) */}
      <Link href="/page1">Page 1</Link>

      {/* Replaces current history entry */}
      <Link href="/page2" replace>
        Page 2 (Replace)
      </Link>
    </div>
  );
}
```

---

## Programmatic Routing

### useRouter Hook

```jsx
import { useRouter } from 'next/router';

export default function Page() {
  const router = useRouter();

  const handleNavigation = () => {
    // Navigate to a page
    router.push('/about');
  };

  const handleDynamicRoute = () => {
    // Navigate with parameters
    router.push({
      pathname: '/blog/[slug]',
      query: { slug: 'hello-world' }
    });

    // Or using template literal
    router.push(`/blog/hello-world`);
  };

  const handleReplace = () => {
    // Replace current history entry
    router.replace('/login');
  };

  const handleBack = () => {
    // Go back in history
    router.back();
  };

  const handleReload = () => {
    // Reload current page
    router.reload();
  };

  return (
    <div>
      <button onClick={handleNavigation}>Go to About</button>
      <button onClick={handleDynamicRoute}>Go to Blog Post</button>
      <button onClick={handleReplace}>Replace with Login</button>
      <button onClick={handleBack}>Go Back</button>
      <button onClick={handleReload}>Reload</button>
    </div>
  );
}
```

### Navigation with State

```jsx
import { useRouter } from 'next/router';

export default function ProductList() {
  const router = useRouter();

  const viewProduct = (productId) => {
    router.push({
      pathname: '/products/[id]',
      query: { id: productId, from: 'list' }
    });
  };

  return (
    <button onClick={() => viewProduct(123)}>
      View Product
    </button>
  );
}
```

### Navigation Events

```jsx
import { useRouter } from 'next/router';
import { useEffect } from 'react';

export default function Page() {
  const router = useRouter();

  useEffect(() => {
    const handleRouteChange = (url) => {
      console.log('App is changing to:', url);
    };

    const handleRouteChangeComplete = (url) => {
      console.log('Route changed to:', url);
      // Analytics, scroll restoration, etc.
    };

    const handleRouteChangeError = (err, url) => {
      console.error('Route change error:', err, url);
    };

    router.events.on('routeChangeStart', handleRouteChange);
    router.events.on('routeChangeComplete', handleRouteChangeComplete);
    router.events.on('routeChangeError', handleRouteChangeError);

    return () => {
      router.events.off('routeChangeStart', handleRouteChange);
      router.events.off('routeChangeComplete', handleRouteChangeComplete);
      router.events.off('routeChangeError', handleRouteChangeError);
    };
  }, [router]);

  return <div>Page with route events</div>;
}
```

---

## Route Parameters

### Query Parameters

```jsx
import { useRouter } from 'next/router';

export default function Search() {
  const router = useRouter();
  const { q, category, page } = router.query;

  // URL: /search?q=nextjs&category=tutorials&page=2
  // q = "nextjs"
  // category = "tutorials"
  // page = "2"

  return (
    <div>
      <h1>Search Results for: {q}</h1>
      <p>Category: {category}</p>
      <p>Page: {page}</p>
    </div>
  );
}
```

### Shallow Routing

Update URL without running data fetching methods.

```jsx
import { useRouter } from 'next/router';

export default function Page() {
  const router = useRouter();

  const updateQuery = () => {
    // Update URL without re-running getServerSideProps
    router.push(
      {
        pathname: '/search',
        query: { q: 'nextjs' }
      },
      undefined,
      { shallow: true }
    );
  };

  return <button onClick={updateQuery}>Update Query</button>;
}
```

### Hash Fragments

```jsx
import { useRouter } from 'next/router';

export default function Docs() {
  const router = useRouter();

  const scrollToSection = () => {
    router.push('/docs#installation');
  };

  return <button onClick={scrollToSection}>Go to Installation</button>;
}
```

---

## Interview Questions

### Q1: What's the difference between `router.push` and `router.replace`?

**Answer:**
- `router.push()`: Adds a new entry to the browser history stack. Users can use the back button to return to the previous page.
- `router.replace()`: Replaces the current history entry. Users cannot use the back button to return to the previous page.

```jsx
// push - adds to history (can go back)
router.push('/dashboard');

// replace - replaces history (cannot go back)
router.replace('/login');
```

**Use cases:**
- `push`: Normal navigation (links, buttons)
- `replace`: After login, form submission, redirects

---

### Q2: How do you create a dynamic route that matches `/blog/2024/12/my-post`?

**Answer:**
```
pages/
└── blog/
    └── [year]/
        └── [month]/
            └── [slug].js
```

**Access parameters:**
```jsx
import { useRouter } from 'next/router';

export default function Post() {
  const router = useRouter();
  const { year, month, slug } = router.query;

  return (
    <div>
      <p>Year: {year}</p>
      <p>Month: {month}</p>
      <p>Slug: {slug}</p>
    </div>
  );
}
```

**Alternative: Catch-all route**
```
pages/
└── blog/
    └── [...slug].js
```

```jsx
const { slug } = router.query;
// slug = ["2024", "12", "my-post"]
const [year, month, postSlug] = slug;
```

---

### Q3: What's the difference between `[...slug]` and `[[...slug]]`?

**Answer:**

**`[...slug]` - Catch-all (required):**
- Matches one or more segments
- Does NOT match the base path

```
pages/docs/[...slug].js

✅ /docs/api          → slug = ["api"]
✅ /docs/api/users    → slug = ["api", "users"]
❌ /docs              → 404 (no match)
```

**`[[...slug]]` - Optional catch-all:**
- Matches zero or more segments
- DOES match the base path

```
pages/docs/[[...slug]].js

✅ /docs              → slug = undefined
✅ /docs/api          → slug = ["api"]
✅ /docs/api/users    → slug = ["api", "users"]
```

---

### Q4: How do you prevent automatic prefetching for a Link?

**Answer:**
```jsx
import Link from 'next/link';

export default function Nav() {
  return (
    <nav>
      {/* Disable prefetch */}
      <Link href="/heavy-page" prefetch={false}>
        Heavy Page (No Prefetch)
      </Link>

      {/* Prefetch enabled by default in production */}
      <Link href="/normal-page">
        Normal Page (Prefetched)
      </Link>
    </nav>
  );
}
```

**When to disable prefetch:**
- Heavy pages that most users won't visit
- Pages behind authentication
- Pages with dynamic data
- To reduce initial bandwidth usage

---

### Q5: How do you navigate programmatically after form submission?

**Answer:**
```jsx
import { useRouter } from 'next/router';
import { useState } from 'react';

export default function CreatePost() {
  const router = useRouter();
  const [title, setTitle] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();

    // Submit form data
    const response = await fetch('/api/posts', {
      method: 'POST',
      body: JSON.stringify({ title }),
      headers: { 'Content-Type': 'application/json' }
    });

    const { id } = await response.json();

    // Navigate to the new post
    router.push(`/posts/${id}`);

    // Or replace (prevent going back to form)
    // router.replace(`/posts/${id}`);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Post title"
      />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

---

### Q6: How do you handle 404 pages in Next.js?

**Answer:**

**Custom 404 page:**
```jsx
// pages/404.js
export default function Custom404() {
  return (
    <div>
      <h1>404 - Page Not Found</h1>
      <p>The page you're looking for doesn't exist.</p>
      <Link href="/">Go back home</Link>
    </div>
  );
}
```

**Programmatic 404:**
```jsx
// pages/posts/[id].js
export async function getStaticProps({ params }) {
  const post = await getPostById(params.id);

  if (!post) {
    return {
      notFound: true // Triggers 404 page
    };
  }

  return {
    props: { post }
  };
}
```

---

## Key Takeaways

1. **File-based routing** - File structure = route structure
2. **Dynamic routes** - Use `[param]` for dynamic segments
3. **Catch-all routes** - Use `[...param]` (required) or `[[...param]]` (optional)
4. **Link component** - Client-side navigation with automatic prefetching
5. **useRouter hook** - Programmatic navigation and route information
6. **Shallow routing** - Update URL without re-fetching data
7. **Navigation events** - Listen to route changes for analytics, loading states

---

## Next Steps

- [App Router](./02-app-router.md)
- [Rendering Strategies](./03-rendering-strategies.md)
- [Data Fetching](./04-data-fetching.md)

---

[← Back to Next.js](./README.md)
