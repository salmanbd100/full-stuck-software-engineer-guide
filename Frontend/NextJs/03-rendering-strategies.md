# Next.js Rendering Strategies

## Table of Contents
- [Introduction](#introduction)
- [Static Site Generation (SSG)](#static-site-generation-ssg)
- [Server-Side Rendering (SSR)](#server-side-rendering-ssr)
- [Incremental Static Regeneration (ISR)](#incremental-static-regeneration-isr)
- [Client-Side Rendering (CSR)](#client-side-rendering-csr)
- [Choosing a Strategy](#choosing-a-strategy)
- [Interview Questions](#interview-questions)

---

## Introduction

Next.js supports multiple rendering strategies, allowing you to choose the best approach for each page.

### Rendering Methods
1. **SSG** - Pre-render at build time
2. **SSR** - Pre-render on each request
3. **ISR** - Regenerate static pages after deployment
4. **CSR** - Render in the browser

---

## Static Site Generation (SSG)

Pages are generated at **build time** and reused on each request.

### Pages Router

**Basic SSG:**
```jsx
// pages/posts.js
export default function Posts({ posts }) {
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

// Runs at build time
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return {
    props: { posts }
  };
}
```

**Dynamic SSG:**
```jsx
// pages/posts/[id].js
export default function Post({ post }) {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

// Generate paths at build time
export async function getStaticPaths() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  const paths = posts.map(post => ({
    params: { id: post.id.toString() }
  }));

  return {
    paths,
    fallback: false // or 'blocking' or true
  };
}

// Generate props for each path
export async function getStaticProps({ params }) {
  const res = await fetch(`https://api.example.com/posts/${params.id}`);
  const post = await res.json();

  return {
    props: { post }
  };
}
```

**Fallback Options:**

```jsx
export async function getStaticPaths() {
  return {
    paths: [],
    fallback: false  // 404 for non-generated paths
    // OR
    fallback: true   // Generate page on first request (show loading)
    // OR
    fallback: 'blocking' // Generate page on first request (SSR-like)
  };
}
```

**fallback: false**
```jsx
// Only pre-generated paths work
// All other paths return 404
return { paths: [{ params: { id: '1' } }], fallback: false };
// ✅ /posts/1 → works
// ❌ /posts/2 → 404
```

**fallback: true**
```jsx
// Show loading state while generating
export async function getStaticPaths() {
  return {
    paths: [{ params: { id: '1' } }],
    fallback: true
  };
}

export default function Post({ post }) {
  const router = useRouter();

  // Show loading state
  if (router.isFallback) {
    return <div>Loading...</div>;
  }

  return <div>{post.title}</div>;
}
```

**fallback: 'blocking'**
```jsx
// Wait for page generation (no loading state needed)
return {
  paths: [{ params: { id: '1' } }],
  fallback: 'blocking'
};
// First request waits, subsequent requests cached
```

### App Router

```jsx
// app/posts/page.js - SSG by default
export default async function Posts() {
  // Fetch at build time
  const res = await fetch('https://api.example.com/posts', {
    cache: 'force-cache' // Default SSG behavior
  });
  const posts = await res.json();

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

**Dynamic SSG:**
```jsx
// app/posts/[id]/page.js
export default async function Post({ params }) {
  const { id } = params;
  const res = await fetch(`https://api.example.com/posts/${id}`);
  const post = await res.json();

  return <div>{post.title}</div>;
}

// Generate static params
export async function generateStaticParams() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return posts.map(post => ({
    id: post.id.toString()
  }));
}
```

**When to use SSG:**
- Content doesn't change often
- Blog posts, documentation
- Marketing pages
- Product listings
- Best performance (served from CDN)

---

## Server-Side Rendering (SSR)

Pages are generated on **each request**.

### Pages Router

```jsx
// pages/dashboard.js
export default function Dashboard({ data }) {
  return <div>{data.user.name}'s Dashboard</div>;
}

// Runs on every request
export async function getServerSideProps(context) {
  const { req, res, query, params } = context;

  // Access cookies, headers
  const token = req.cookies.token;

  // Fetch user-specific data
  const response = await fetch('https://api.example.com/user', {
    headers: { Authorization: `Bearer ${token}` }
  });
  const data = await response.json();

  // Handle errors
  if (!data.user) {
    return {
      redirect: {
        destination: '/login',
        permanent: false
      }
    };
  }

  return {
    props: { data }
  };
}
```

**Set cache headers:**
```jsx
export async function getServerSideProps({ res }) {
  // Cache for 60 seconds, revalidate in background
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=60, stale-while-revalidate=120'
  );

  const data = await fetchData();

  return { props: { data } };
}
```

### App Router

```jsx
// app/dashboard/page.js
export default async function Dashboard() {
  // Force dynamic rendering
  const res = await fetch('https://api.example.com/user', {
    cache: 'no-store' // SSR - no caching
  });
  const data = await res.json();

  return <div>Welcome, {data.name}</div>;
}

// Or explicitly set dynamic
export const dynamic = 'force-dynamic';
```

**Access request data:**
```jsx
import { headers, cookies } from 'next/headers';

export default async function Page() {
  const headersList = headers();
  const cookieStore = cookies();

  const token = cookieStore.get('token');
  const userAgent = headersList.get('user-agent');

  return <div>User Agent: {userAgent}</div>;
}
```

**When to use SSR:**
- Personalized content
- Real-time data
- Authentication required
- SEO + dynamic content

---

## Incremental Static Regeneration (ISR)

Update static pages **after build** without full rebuild.

### Pages Router

```jsx
// pages/posts/[id].js
export default function Post({ post }) {
  return <div>{post.title}</div>;
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://api.example.com/posts/${params.id}`);
  const post = await res.json();

  return {
    props: { post },
    revalidate: 60 // Regenerate every 60 seconds
  };
}

export async function getStaticPaths() {
  return {
    paths: [],
    fallback: 'blocking'
  };
}
```

**How ISR works:**
1. First request → Static page served
2. After 60 seconds → Next request triggers regeneration
3. Regeneration happens in background
4. Old page served until new one ready
5. New page replaces old one

**On-Demand Revalidation:**
```jsx
// pages/api/revalidate.js
export default async function handler(req, res) {
  // Check secret to validate request
  if (req.query.secret !== process.env.REVALIDATE_SECRET) {
    return res.status(401).json({ message: 'Invalid token' });
  }

  try {
    // Revalidate specific path
    await res.revalidate('/posts/1');
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send('Error revalidating');
  }
}
```

### App Router

```jsx
// app/posts/[id]/page.js
export default async function Post({ params }) {
  const res = await fetch(`https://api.example.com/posts/${params.id}`, {
    next: { revalidate: 60 } // ISR with 60 second revalidation
  });
  const post = await res.json();

  return <div>{post.title}</div>;
}
```

**Route segment config:**
```jsx
// app/posts/[id]/page.js
export const revalidate = 60; // Revalidate every 60 seconds

export default async function Post({ params }) {
  // All fetches inherit this revalidate time
  const post = await fetchPost(params.id);
  return <div>{post.title}</div>;
}
```

**On-Demand Revalidation:**
```jsx
// app/api/revalidate/route.js
import { revalidatePath, revalidateTag } from 'next/cache';

export async function POST(request) {
  const path = request.nextUrl.searchParams.get('path');

  if (path) {
    revalidatePath(path);
    return Response.json({ revalidated: true });
  }

  return Response.json({ revalidated: false });
}
```

**Tag-based revalidation:**
```jsx
// Fetch with cache tags
const res = await fetch('https://api.example.com/posts', {
  next: { tags: ['posts'] }
});

// Revalidate all fetches with 'posts' tag
import { revalidateTag } from 'next/cache';
revalidateTag('posts');
```

**When to use ISR:**
- Content changes periodically
- Product catalogs
- Blog with updates
- News sites
- Balance between SSG and SSR

---

## Client-Side Rendering (CSR)

Render in the browser with JavaScript.

### Pages Router

```jsx
import { useEffect, useState } from 'react';

export default function Dashboard() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading...</div>;

  return <div>Welcome, {data.name}</div>;
}
```

**With SWR:**
```jsx
import useSWR from 'swr';

const fetcher = url => fetch(url).then(res => res.json());

export default function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Failed to load</div>;

  return <div>Hello, {data.name}</div>;
}
```

### App Router

```jsx
// app/dashboard/page.js
'use client';

import { useEffect, useState } from 'react';

export default function Dashboard() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(setData);
  }, []);

  if (!data) return <div>Loading...</div>;

  return <div>Welcome, {data.name}</div>;
}
```

**When to use CSR:**
- Private, user-specific data
- Real-time updates
- Interactive dashboards
- No SEO needed
- Data changes frequently

---

## Choosing a Strategy

### Decision Tree

```
Is SEO important?
├─ No → CSR
└─ Yes
   ├─ Data changes on every request?
   │  └─ Yes → SSR
   └─ No
      ├─ Can you rebuild when data changes?
      │  ├─ Yes → SSG
      │  └─ No → ISR
      └─ Static content? → SSG
```

### Comparison Table

| Strategy | Build Time | Request Time | SEO | Use Case |
|----------|-----------|-------------|-----|----------|
| **SSG** | ✅ Generated | ⚡ Instant | ✅ | Blog, docs, marketing |
| **SSR** | ❌ None | 🐌 Slow | ✅ | Personalized, auth |
| **ISR** | ✅ Generated | ⚡ Instant | ✅ | Products, news |
| **CSR** | ❌ None | 🐌 Slow | ❌ | Dashboards, private |

### Hybrid Approach

```jsx
// pages/product/[id].js
export default function Product({ product, relatedProducts }) {
  const [reviews, setReviews] = useState([]);

  // SSG for product data
  // CSR for reviews

  useEffect(() => {
    fetch(`/api/reviews/${product.id}`)
      .then(res => res.json())
      .then(setReviews);
  }, [product.id]);

  return (
    <div>
      <h1>{product.name}</h1> {/* SSG */}
      <Reviews data={reviews} /> {/* CSR */}
    </div>
  );
}

export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  return {
    props: { product },
    revalidate: 3600 // ISR - update hourly
  };
}

export async function getStaticPaths() {
  return { paths: [], fallback: 'blocking' };
}
```

---

## Interview Questions

### Q1: What's the difference between SSG and SSR?

**Answer:**

**SSG (Static Site Generation):**
- Generates HTML at **build time**
- Same HTML served to all users
- Cached on CDN - extremely fast
- Best for content that doesn't change often

**SSR (Server-Side Rendering):**
- Generates HTML on **each request**
- Can be personalized per user
- Slower than SSG (server processing)
- Best for dynamic, personalized content

```jsx
// SSG - build time
export async function getStaticProps() {
  const posts = await fetchPosts();
  return { props: { posts } };
}

// SSR - request time
export async function getServerSideProps() {
  const user = await fetchUser();
  return { props: { user } };
}
```

---

### Q2: Explain the `fallback` options in `getStaticPaths`.

**Answer:**

**fallback: false**
- Only pre-generated paths work
- All others return 404
- Use when all paths are known

**fallback: true**
- Show loading state while generating
- Need to handle `router.isFallback`
- Use for large number of pages

**fallback: 'blocking'**
- Wait for generation (SSR-like)
- No loading state needed
- First request slower, then cached

```jsx
export async function getStaticPaths() {
  return {
    paths: [{ params: { id: '1' } }],
    fallback: 'blocking' // Best for SEO
  };
}
```

---

### Q3: What is ISR and when should you use it?

**Answer:**

**ISR (Incremental Static Regeneration):**
- Combines benefits of SSG and SSR
- Static pages that update after deployment
- Set revalidation period

```jsx
export async function getStaticProps() {
  const data = await fetchData();

  return {
    props: { data },
    revalidate: 60 // Regenerate every 60 seconds
  };
}
```

**How it works:**
1. First request → Cached page served (fast)
2. After 60s → Next request triggers regeneration
3. Old page served while new one generates
4. New page replaces old one

**Use cases:**
- Product catalogs (prices change)
- News sites (new articles)
- Blog with comments
- Any content that updates periodically

---

### Q4: How do you implement on-demand revalidation?

**Answer:**

**Pages Router:**
```jsx
// pages/api/revalidate.js
export default async function handler(req, res) {
  // Verify secret
  if (req.query.secret !== process.env.SECRET) {
    return res.status(401).json({ message: 'Invalid' });
  }

  try {
    await res.revalidate('/posts/1');
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send('Error');
  }
}
```

**App Router:**
```jsx
// app/api/revalidate/route.js
import { revalidatePath, revalidateTag } from 'next/cache';

export async function POST() {
  revalidatePath('/posts');
  revalidateTag('posts');
  return Response.json({ revalidated: true });
}
```

**Trigger from CMS webhook:**
```bash
curl -X POST https://yoursite.com/api/revalidate?secret=TOKEN
```

---

### Q5: How do you mix SSG and CSR in a single page?

**Answer:**

```jsx
export default function Product({ product }) {
  const [reviews, setReviews] = useState([]);

  // SSG content
  const staticContent = (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );

  // CSR content
  useEffect(() => {
    fetch(`/api/reviews/${product.id}`)
      .then(res => res.json())
      .then(setReviews);
  }, [product.id]);

  return (
    <div>
      {staticContent}
      <Reviews data={reviews} /> {/* Client-side */}
    </div>
  );
}

// SSG for product
export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  return { props: { product } };
}
```

**Benefits:**
- SEO for static content
- Real-time for dynamic content
- Best of both worlds

---

## Key Takeaways

1. **SSG** - Build time, fastest, best for static content
2. **SSR** - Request time, personalized, best for dynamic content
3. **ISR** - SSG + periodic updates, best for semi-static content
4. **CSR** - Browser rendering, best for private/real-time data
5. **fallback** - Controls behavior for non-pre-rendered paths
6. **revalidate** - Enables ISR with time-based or on-demand updates
7. **Hybrid approach** - Mix strategies in single page for optimal performance

---

## Next Steps

- [Data Fetching](./04-data-fetching.md)
- [API Routes](./05-api-routes.md)
- [Image Optimization](./06-image-optimization.md)

---

[← Back to Next.js](./README.md)
