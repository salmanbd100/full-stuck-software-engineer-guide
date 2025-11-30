# Next.js Data Fetching

## Table of Contents
- [Pages Router Data Fetching](#pages-router-data-fetching)
- [App Router Data Fetching](#app-router-data-fetching)
- [Client-Side Data Fetching](#client-side-data-fetching)
- [Parallel and Sequential Fetching](#parallel-and-sequential-fetching)
- [Error Handling](#error-handling)
- [Interview Questions](#interview-questions)

---

## Pages Router Data Fetching

### getStaticProps

Fetch data at build time for SSG.

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

export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return {
    props: { posts },
    revalidate: 3600 // ISR - revalidate every hour
  };
}
```

**Context object:**
```jsx
export async function getStaticProps(context) {
  const {
    params,      // Route parameters for dynamic routes
    preview,     // Preview mode
    previewData, // Preview data
    locale,      // Active locale
    locales,     // All locales
    defaultLocale // Default locale
  } = context;

  return { props: {} };
}
```

**Return options:**
```jsx
export async function getStaticProps() {
  // Success
  return {
    props: { data },
    revalidate: 60,
    notFound: false,
    redirect: undefined
  };

  // Not found
  return {
    notFound: true
  };

  // Redirect
  return {
    redirect: {
      destination: '/other-page',
      permanent: false
    }
  };
}
```

### getStaticPaths

Define dynamic routes to pre-render.

```jsx
// pages/posts/[id].js
export default function Post({ post }) {
  return <div>{post.title}</div>;
}

export async function getStaticPaths() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  const paths = posts.map(post => ({
    params: { id: post.id.toString() }
  }));

  return {
    paths,
    fallback: 'blocking'
  };
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://api.example.com/posts/${params.id}`);
  const post = await res.json();

  if (!post) {
    return { notFound: true };
  }

  return {
    props: { post },
    revalidate: 60
  };
}
```

### getServerSideProps

Fetch data on each request for SSR.

```jsx
// pages/dashboard.js
export default function Dashboard({ user, data }) {
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <Data items={data} />
    </div>
  );
}

export async function getServerSideProps(context) {
  const {
    req,         // HTTP request
    res,         // HTTP response
    params,      // Route parameters
    query,       // Query string
    preview,
    previewData,
    resolvedUrl,
    locale,
    locales,
    defaultLocale
  } = context;

  // Access cookies
  const token = req.cookies.token;

  // Fetch user data
  const userRes = await fetch('https://api.example.com/user', {
    headers: { Authorization: `Bearer ${token}` }
  });
  const user = await userRes.json();

  if (!user) {
    return {
      redirect: {
        destination: '/login',
        permanent: false
      }
    };
  }

  // Fetch dashboard data
  const dataRes = await fetch('https://api.example.com/dashboard');
  const data = await dataRes.json();

  // Set cache headers
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=10, stale-while-revalidate=59'
  );

  return {
    props: { user, data }
  };
}
```

---

## App Router Data Fetching

### Server Components (Default)

```jsx
// app/posts/page.js
export default async function Posts() {
  // Fetch with caching
  const res = await fetch('https://api.example.com/posts', {
    cache: 'force-cache' // SSG (default)
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

### Cache Options

```jsx
// SSG - cache indefinitely
const res = await fetch('https://api.example.com/data', {
  cache: 'force-cache' // Default
});

// SSR - no caching
const res = await fetch('https://api.example.com/data', {
  cache: 'no-store'
});

// ISR - revalidate after time
const res = await fetch('https://api.example.com/data', {
  next: { revalidate: 60 } // Revalidate every 60 seconds
});

// ISR - tag-based revalidation
const res = await fetch('https://api.example.com/data', {
  next: { tags: ['posts'] }
});
```

### Route Segment Config

```jsx
// app/posts/page.js

// Force static rendering
export const dynamic = 'force-static';

// Force dynamic rendering
export const dynamic = 'force-dynamic';

// Set revalidation time for all fetches
export const revalidate = 60;

// Opt into dynamic rendering
export const fetchCache = 'force-no-store';

export default async function Page() {
  const data = await fetchData();
  return <div>{data}</div>;
}
```

### generateStaticParams

```jsx
// app/posts/[id]/page.js
export default async function Post({ params }) {
  const { id } = params;
  const post = await fetchPost(id);

  return <div>{post.title}</div>;
}

// Generate static params at build time
export async function generateStaticParams() {
  const posts = await fetchPosts();

  return posts.map(post => ({
    id: post.id.toString()
  }));
}
```

### Parallel Data Fetching

```jsx
// app/page.js
export default async function Page() {
  // Fetch in parallel
  const [posts, users] = await Promise.all([
    fetchPosts(),
    fetchUsers()
  ]);

  return (
    <div>
      <Posts data={posts} />
      <Users data={users} />
    </div>
  );
}
```

---

## Client-Side Data Fetching

### useEffect

```jsx
'use client';

import { useEffect, useState } from 'react';

export default function Posts() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/posts')
      .then(res => res.json())
      .then(data => {
        setPosts(data);
        setLoading(false);
      })
      .catch(err => {
        console.error(err);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading...</div>;

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

### SWR (Recommended)

```jsx
'use client';

import useSWR from 'swr';

const fetcher = url => fetch(url).then(res => res.json());

export default function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Failed to load</div>;

  return <div>Hello, {data.name}</div>;
}
```

**With options:**
```jsx
const { data, error } = useSWR('/api/user', fetcher, {
  refreshInterval: 3000,      // Polling
  revalidateOnFocus: true,    // Revalidate on window focus
  revalidateOnReconnect: true // Revalidate on reconnect
});
```

### React Query

```jsx
'use client';

import { useQuery } from '@tanstack/react-query';

export default function Posts() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['posts'],
    queryFn: async () => {
      const res = await fetch('/api/posts');
      return res.json();
    }
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {data.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

---

## Parallel and Sequential Fetching

### Parallel Fetching

Fetch multiple resources simultaneously.

```jsx
// app/page.js
async function getData() {
  const [posts, users, comments] = await Promise.all([
    fetch('https://api.example.com/posts').then(res => res.json()),
    fetch('https://api.example.com/users').then(res => res.json()),
    fetch('https://api.example.com/comments').then(res => res.json())
  ]);

  return { posts, users, comments };
}

export default async function Page() {
  const { posts, users, comments } = await getData();

  return <div>...</div>;
}
```

### Sequential Fetching

Fetch resources one after another.

```jsx
export default async function Page() {
  // Fetch user first
  const user = await fetchUser();

  // Then fetch user's posts
  const posts = await fetchUserPosts(user.id);

  // Then fetch post details
  const postDetails = await fetchPostDetails(posts[0].id);

  return <div>...</div>;
}
```

### Streaming with Suspense

```jsx
import { Suspense } from 'react';

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>

      {/* Stream components independently */}
      <Suspense fallback={<Skeleton />}>
        <Posts />
      </Suspense>

      <Suspense fallback={<Skeleton />}>
        <Users />
      </Suspense>
    </div>
  );
}

async function Posts() {
  const posts = await fetchPosts(); // Slow fetch
  return <div>{posts.map(p => p.title)}</div>;
}

async function Users() {
  const users = await fetchUsers(); // Fast fetch
  return <div>{users.map(u => u.name)}</div>;
}
```

---

## Error Handling

### Pages Router

```jsx
export async function getServerSideProps() {
  try {
    const res = await fetch('https://api.example.com/data');

    if (!res.ok) {
      throw new Error('Failed to fetch data');
    }

    const data = await res.json();

    return { props: { data } };
  } catch (error) {
    return {
      props: { error: error.message }
    };
  }
}

export default function Page({ data, error }) {
  if (error) {
    return <div>Error: {error}</div>;
  }

  return <div>{data.title}</div>;
}
```

### App Router

```jsx
// app/posts/page.js
export default async function Posts() {
  try {
    const res = await fetch('https://api.example.com/posts');

    if (!res.ok) {
      throw new Error('Failed to fetch');
    }

    const posts = await res.json();

    return <div>{posts.map(p => p.title)}</div>;
  } catch (error) {
    return <div>Error: {error.message}</div>;
  }
}
```

**With error.js:**
```jsx
// app/posts/error.js
'use client';

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Error loading posts</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

---

## Interview Questions

### Q1: What's the difference between getStaticProps and getServerSideProps?

**Answer:**

**getStaticProps:**
- Runs at build time
- Generates static HTML
- Can use ISR with revalidate
- Best for pages that can be pre-rendered

**getServerSideProps:**
- Runs on every request
- Generates HTML per request
- Can access request/response
- Best for personalized or real-time data

```jsx
// Static - build time
export async function getStaticProps() {
  const posts = await fetchPosts();
  return { props: { posts }, revalidate: 60 };
}

// Dynamic - request time
export async function getServerSideProps({ req }) {
  const user = await fetchUser(req.cookies.token);
  return { props: { user } };
}
```

---

### Q2: How do you implement ISR in App Router?

**Answer:**

```jsx
// Method 1: Fetch with revalidate
const res = await fetch('https://api.example.com/data', {
  next: { revalidate: 60 }
});

// Method 2: Route segment config
export const revalidate = 60;

export default async function Page() {
  const data = await fetchData();
  return <div>{data}</div>;
}

// Method 3: Tag-based revalidation
const res = await fetch('https://api.example.com/data', {
  next: { tags: ['posts'] }
});

// Revalidate on-demand
import { revalidateTag } from 'next/cache';
revalidateTag('posts');
```

---

### Q3: How do you fetch data in parallel vs sequential?

**Answer:**

**Parallel (faster):**
```jsx
export default async function Page() {
  const [posts, users] = await Promise.all([
    fetchPosts(),
    fetchUsers()
  ]);

  return <div>...</div>;
}
```

**Sequential (when dependent):**
```jsx
export default async function Page() {
  const user = await fetchUser();
  const posts = await fetchUserPosts(user.id); // Depends on user

  return <div>...</div>;
}
```

---

### Q4: What's the difference between SWR and React Query?

**Answer:**

Both are client-side data fetching libraries with similar features.

**SWR:**
- Lighter weight
- Built by Vercel
- Simple API
- Great for Next.js

**React Query:**
- More features
- Better DevTools
- More configuration options
- Framework agnostic

```jsx
// SWR
const { data } = useSWR('/api/user', fetcher);

// React Query
const { data } = useQuery({
  queryKey: ['user'],
  queryFn: fetchUser
});
```

---

### Q5: How do you handle loading states with Suspense?

**Answer:**

```jsx
import { Suspense } from 'react';

export default function Page() {
  return (
    <div>
      {/* Instant loading state */}
      <Suspense fallback={<Skeleton />}>
        <AsyncComponent />
      </Suspense>

      {/* Independent loading states */}
      <Suspense fallback={<PostsSkeleton />}>
        <Posts />
      </Suspense>

      <Suspense fallback={<UsersSkeleton />}>
        <Users />
      </Suspense>
    </div>
  );
}

async function AsyncComponent() {
  const data = await fetchData();
  return <div>{data}</div>;
}
```

---

## Key Takeaways

1. **Pages Router:** getStaticProps (build), getServerSideProps (request)
2. **App Router:** fetch with cache options, Server Components by default
3. **Cache options:** force-cache (SSG), no-store (SSR), revalidate (ISR)
4. **Client-side:** useEffect, SWR, React Query for dynamic data
5. **Parallel fetching:** Promise.all for independent data
6. **Sequential fetching:** await for dependent data
7. **Suspense:** Stream components independently with loading states

---

[← Back to Next.js](./README.md)
