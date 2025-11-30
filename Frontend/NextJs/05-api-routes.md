# Next.js API Routes

## Table of Contents
- [Pages API Routes](#pages-api-routes)
- [App Router Route Handlers](#app-router-route-handlers)
- [HTTP Methods](#http-methods)
- [Request and Response](#request-and-response)
- [Middleware](#middleware)
- [Error Handling](#error-handling)
- [Interview Questions](#interview-questions)

---

## Pages API Routes

API routes provide a solution to build your API with Next.js.

### Basic API Route

```jsx
// pages/api/hello.js
export default function handler(req, res) {
  res.status(200).json({ message: 'Hello World' });
}
```

**File structure:**
```
pages/api/
├── hello.js          → /api/hello
├── user.js           → /api/user
└── posts/
    ├── index.js      → /api/posts
    └── [id].js       → /api/posts/:id
```

### Dynamic API Routes

```jsx
// pages/api/posts/[id].js
export default function handler(req, res) {
  const { id } = req.query;

  // Fetch post by ID
  const post = getPostById(id);

  if (!post) {
    return res.status(404).json({ error: 'Post not found' });
  }

  res.status(200).json(post);
}
```

### Catch-All API Routes

```jsx
// pages/api/[...params].js
export default function handler(req, res) {
  const { params } = req.query;
  // params = ["user", "123", "posts"]

  res.status(200).json({ params });
}
```

---

## App Router Route Handlers

Route handlers are the App Router equivalent of API routes.

### Basic Route Handler

```jsx
// app/api/hello/route.js
export async function GET(request) {
  return Response.json({ message: 'Hello World' });
}
```

**File structure:**
```
app/api/
├── hello/
│   └── route.js      → /api/hello
├── users/
│   └── route.js      → /api/users
└── posts/
    └── [id]/
        └── route.js  → /api/posts/:id
```

### Dynamic Routes

```jsx
// app/api/posts/[id]/route.js
export async function GET(request, { params }) {
  const { id } = params;

  const post = await getPostById(id);

  if (!post) {
    return Response.json({ error: 'Not found' }, { status: 404 });
  }

  return Response.json(post);
}
```

---

## HTTP Methods

### Pages Router

```jsx
// pages/api/posts.js
export default function handler(req, res) {
  const { method } = req;

  switch (method) {
    case 'GET':
      return handleGet(req, res);
    case 'POST':
      return handlePost(req, res);
    case 'PUT':
      return handlePut(req, res);
    case 'DELETE':
      return handleDelete(req, res);
    default:
      res.setHeader('Allow', ['GET', 'POST', 'PUT', 'DELETE']);
      return res.status(405).end(`Method ${method} Not Allowed`);
  }
}

function handleGet(req, res) {
  const posts = getAllPosts();
  res.status(200).json(posts);
}

function handlePost(req, res) {
  const { title, content } = req.body;
  const newPost = createPost({ title, content });
  res.status(201).json(newPost);
}

function handlePut(req, res) {
  const { id } = req.query;
  const updatedPost = updatePost(id, req.body);
  res.status(200).json(updatedPost);
}

function handleDelete(req, res) {
  const { id } = req.query;
  deletePost(id);
  res.status(204).end();
}
```

### App Router

```jsx
// app/api/posts/route.js
export async function GET(request) {
  const posts = await getAllPosts();
  return Response.json(posts);
}

export async function POST(request) {
  const body = await request.json();
  const newPost = await createPost(body);
  return Response.json(newPost, { status: 201 });
}

export async function PUT(request) {
  const body = await request.json();
  const updated = await updatePost(body);
  return Response.json(updated);
}

export async function DELETE(request) {
  const { searchParams } = new URL(request.url);
  const id = searchParams.get('id');
  await deletePost(id);
  return new Response(null, { status: 204 });
}
```

---

## Request and Response

### Pages Router - Request

```jsx
export default function handler(req, res) {
  // HTTP method
  const { method } = req;

  // Query parameters: /api/posts?page=1&limit=10
  const { page, limit } = req.query;

  // Request body
  const { title, content } = req.body;

  // Headers
  const userAgent = req.headers['user-agent'];
  const contentType = req.headers['content-type'];

  // Cookies
  const token = req.cookies.token;

  res.status(200).json({ method, page, limit });
}
```

### Pages Router - Response

```jsx
export default function handler(req, res) {
  // JSON response
  res.status(200).json({ message: 'Success' });

  // Text response
  res.status(200).send('Hello');

  // Set headers
  res.setHeader('Content-Type', 'application/json');
  res.setHeader('Cache-Control', 's-maxage=86400');

  // Set cookies
  res.setHeader('Set-Cookie', 'token=abc123; Path=/; HttpOnly');

  // Redirect
  res.redirect(307, '/login');

  // Status codes
  res.status(201).json({ created: true }); // Created
  res.status(400).json({ error: 'Bad Request' });
  res.status(401).json({ error: 'Unauthorized' });
  res.status(404).json({ error: 'Not Found' });
  res.status(500).json({ error: 'Server Error' });
}
```

### App Router - Request

```jsx
// app/api/route.js
export async function POST(request) {
  // URL and query params
  const url = new URL(request.url);
  const searchParams = url.searchParams;
  const page = searchParams.get('page');

  // Headers
  const headers = request.headers;
  const userAgent = headers.get('user-agent');
  const authorization = headers.get('authorization');

  // Cookies
  const cookieStore = cookies();
  const token = cookieStore.get('token');

  // Request body
  const body = await request.json();
  const { title, content } = body;

  // Form data
  const formData = await request.formData();
  const name = formData.get('name');

  return Response.json({ page, title });
}
```

### App Router - Response

```jsx
export async function GET() {
  // JSON response
  return Response.json({ message: 'Success' });

  // With status and headers
  return Response.json(
    { data: [] },
    {
      status: 200,
      headers: {
        'Cache-Control': 's-maxage=86400',
        'Content-Type': 'application/json'
      }
    }
  );

  // Redirect
  return Response.redirect('https://example.com/login', 307);

  // Text response
  return new Response('Hello World');

  // Error responses
  return Response.json({ error: 'Not found' }, { status: 404 });
}
```

---

## Middleware

### Pages Router - CORS

```jsx
// pages/api/posts.js
function cors(req, res) {
  res.setHeader('Access-Control-Allow-Credentials', true);
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE');
  res.setHeader(
    'Access-Control-Allow-Headers',
    'X-CSRF-Token, X-Requested-With, Accept, Content-Type, Authorization'
  );

  if (req.method === 'OPTIONS') {
    res.status(200).end();
    return true;
  }

  return false;
}

export default function handler(req, res) {
  if (cors(req, res)) return;

  res.status(200).json({ message: 'Hello' });
}
```

### Authentication Middleware

```jsx
// lib/auth.js
export function withAuth(handler) {
  return async (req, res) => {
    const token = req.cookies.token;

    if (!token) {
      return res.status(401).json({ error: 'Unauthorized' });
    }

    try {
      const user = await verifyToken(token);
      req.user = user;
      return handler(req, res);
    } catch (error) {
      return res.status(401).json({ error: 'Invalid token' });
    }
  };
}

// pages/api/protected.js
import { withAuth } from '@/lib/auth';

function handler(req, res) {
  // req.user is available
  res.status(200).json({ user: req.user });
}

export default withAuth(handler);
```

### App Router Middleware

```jsx
// middleware.js (root level)
import { NextResponse } from 'next/server';

export function middleware(request) {
  // Check auth
  const token = request.cookies.get('token');

  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Add custom header
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'value');

  return response;
}

// Match specific paths
export const config = {
  matcher: ['/api/protected/:path*', '/dashboard/:path*']
};
```

---

## Error Handling

### Pages Router

```jsx
export default async function handler(req, res) {
  try {
    const data = await fetchData();

    if (!data) {
      return res.status(404).json({ error: 'Not found' });
    }

    res.status(200).json(data);
  } catch (error) {
    console.error('API Error:', error);

    if (error.code === 'VALIDATION_ERROR') {
      return res.status(400).json({ error: error.message });
    }

    res.status(500).json({ error: 'Internal server error' });
  }
}
```

### App Router

```jsx
export async function GET(request) {
  try {
    const data = await fetchData();

    if (!data) {
      return Response.json({ error: 'Not found' }, { status: 404 });
    }

    return Response.json(data);
  } catch (error) {
    console.error('API Error:', error);

    return Response.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

---

## Interview Questions

### Q1: What's the difference between API Routes and Route Handlers?

**Answer:**

**API Routes (Pages Router):**
- Located in `pages/api/`
- Use `req` and `res` objects (Node.js style)
- Single export default function

**Route Handlers (App Router):**
- Located in `app/api/*/route.js`
- Use Web APIs (Request, Response)
- Named exports per HTTP method

```jsx
// API Route
export default function handler(req, res) {
  res.status(200).json({ data });
}

// Route Handler
export async function GET(request) {
  return Response.json({ data });
}
```

---

### Q2: How do you handle different HTTP methods?

**Answer:**

**Pages Router:**
```jsx
export default function handler(req, res) {
  if (req.method === 'GET') {
    return res.status(200).json(getData());
  }

  if (req.method === 'POST') {
    return res.status(201).json(createData(req.body));
  }

  res.setHeader('Allow', ['GET', 'POST']);
  res.status(405).end(`Method ${req.method} Not Allowed`);
}
```

**App Router:**
```jsx
export async function GET(request) {
  return Response.json(getData());
}

export async function POST(request) {
  const body = await request.json();
  return Response.json(createData(body), { status: 201 });
}
```

---

### Q3: How do you access query parameters and request body?

**Answer:**

**Pages Router:**
```jsx
export default function handler(req, res) {
  // Query: /api/posts?id=123
  const { id } = req.query;

  // Body
  const { title } = req.body;

  res.status(200).json({ id, title });
}
```

**App Router:**
```jsx
export async function POST(request) {
  // Query parameters
  const { searchParams } = new URL(request.url);
  const id = searchParams.get('id');

  // Request body
  const body = await request.json();
  const { title } = body;

  return Response.json({ id, title });
}
```

---

### Q4: How do you implement authentication in API routes?

**Answer:**

```jsx
// lib/auth.js
export async function verifyAuth(req) {
  const token = req.cookies.token || req.headers.authorization?.split(' ')[1];

  if (!token) {
    throw new Error('No token provided');
  }

  const user = await verifyToken(token);
  return user;
}

// pages/api/protected.js
export default async function handler(req, res) {
  try {
    const user = await verifyAuth(req);

    // User is authenticated
    res.status(200).json({ user });
  } catch (error) {
    res.status(401).json({ error: 'Unauthorized' });
  }
}
```

---

### Q5: How do you enable CORS in API routes?

**Answer:**

**Pages Router:**
```jsx
export default function handler(req, res) {
  // Set CORS headers
  res.setHeader('Access-Control-Allow-Credentials', true);
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');

  // Handle preflight
  if (req.method === 'OPTIONS') {
    return res.status(200).end();
  }

  // Your API logic
  res.status(200).json({ message: 'Success' });
}
```

**App Router:**
```jsx
export async function GET(request) {
  return Response.json(
    { message: 'Success' },
    {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE',
        'Access-Control-Allow-Headers': 'Content-Type'
      }
    }
  );
}

export async function OPTIONS(request) {
  return new Response(null, {
    status: 200,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE',
      'Access-Control-Allow-Headers': 'Content-Type'
    }
  });
}
```

---

## Key Takeaways

1. **API Routes:** pages/api for Pages Router, app/api/*/route.js for App Router
2. **HTTP Methods:** req.method or named exports (GET, POST, etc.)
3. **Request:** Query params, headers, cookies, body
4. **Response:** JSON, status codes, headers, redirects
5. **Middleware:** Auth, CORS, logging
6. **Error Handling:** Try-catch, proper status codes
7. **Dynamic Routes:** [id].js for params, [...slug].js for catch-all

---

[← Back to Next.js](./README.md)
