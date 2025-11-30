# Next.js Best Practices

## Table of Contents
- [Project Structure](#project-structure)
- [Code Organization](#code-organization)
- [Performance Optimization](#performance-optimization)
- [Security Best Practices](#security-best-practices)
- [Error Handling](#error-handling)
- [Testing Strategies](#testing-strategies)
- [Interview Questions](#interview-questions)

---

## Project Structure

### Recommended Structure

```
my-app/
├── app/                    # App router (Next.js 13+)
│   ├── (marketing)/       # Route groups
│   │   ├── layout.js
│   │   └── page.js
│   ├── api/
│   │   └── users/
│   │       └── route.js
│   ├── layout.js
│   └── page.js
├── components/            # Shared components
│   ├── ui/               # UI components
│   │   ├── Button.js
│   │   └── Card.js
│   └── features/         # Feature-specific
│       └── UserProfile.js
├── lib/                  # Utility functions
│   ├── db.js
│   ├── auth.js
│   └── utils.js
├── public/               # Static files
│   ├── images/
│   └── fonts/
├── styles/               # Global styles
│   └── globals.css
├── types/                # TypeScript types
│   └── index.ts
├── .env.local            # Environment variables
├── next.config.js
└── package.json
```

---

## Code Organization

### Component Organization

```jsx
// ✅ Good: Single responsibility
function UserCard({ user }) {
  return (
    <div>
      <Avatar src={user.avatar} />
      <UserInfo name={user.name} email={user.email} />
    </div>
  );
}

// ❌ Bad: Too many responsibilities
function UserCard({ user }) {
  return (
    <div>
      <img src={user.avatar} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={() => deleteUser(user.id)}>Delete</button>
      <form onSubmit={handleEdit}>...</form>
    </div>
  );
}
```

### Separate Server and Client Code

```jsx
// ✅ Good: Clear separation
// app/page.js (Server Component)
async function Page() {
  const data = await fetchData(); // Server-side
  return <ClientComponent data={data} />;
}

// components/ClientComponent.js
'use client';
function ClientComponent({ data }) {
  const [state, setState] = useState(data);
  return <div onClick={() => setState(...)} />;
}

// ❌ Bad: Mixed concerns
'use client';
async function Page() {
  const data = await fetchData(); // Won't work in client component
  return <div>{data}</div>;
}
```

### Custom Hooks for Reusability

```jsx
// ✅ Good: Reusable logic
// hooks/useUser.js
export function useUser(id) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser(id).then(data => {
      setUser(data);
      setLoading(false);
    });
  }, [id]);

  return { user, loading };
}

// Usage
function Profile() {
  const { user, loading } = useUser(123);
  if (loading) return <Spinner />;
  return <div>{user.name}</div>;
}
```

---

## Performance Optimization

### Image Optimization

```jsx
// ✅ Good: Use next/image
import Image from 'next/image';

<Image
  src="/hero.jpg"
  width={1920}
  height={1080}
  priority  // For above-the-fold images
  alt="Hero"
/>

// ❌ Bad: Regular img tag
<img src="/hero.jpg" alt="Hero" />
```

### Dynamic Imports

```jsx
// ✅ Good: Dynamic imports for large components
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <Spinner />,
  ssr: false // Client-side only
});

// ❌ Bad: Import everything upfront
import HeavyComponent from './HeavyComponent';
```

### Memoization

```jsx
// ✅ Good: Memoize expensive calculations
import { useMemo, useCallback } from 'react';

function Component({ data }) {
  const expensiveValue = useMemo(() => {
    return data.reduce((acc, item) => acc + item.value, 0);
  }, [data]);

  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);

  return <div onClick={handleClick}>{expensiveValue}</div>;
}

// ❌ Bad: Recalculate on every render
function Component({ data }) {
  const expensiveValue = data.reduce((acc, item) => acc + item.value, 0);

  return <div onClick={() => console.log('Clicked')}>{expensiveValue}</div>;
}
```

### Font Optimization

```jsx
// app/layout.js
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

---

## Security Best Practices

### Environment Variables

```js
// ✅ Good: Server-side only
const apiKey = process.env.API_KEY;

// ✅ Good: Public (prefixed)
const publicUrl = process.env.NEXT_PUBLIC_API_URL;

// ❌ Bad: Exposing secrets to client
<script>const key = '{process.env.API_KEY}'</script>
```

### API Route Security

```js
// ✅ Good: Validate and sanitize
export async function POST(request) {
  const token = request.cookies.get('token');

  // Verify auth
  if (!token) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // Validate input
  const body = await request.json();
  const { email } = body;

  if (!isValidEmail(email)) {
    return Response.json({ error: 'Invalid email' }, { status: 400 });
  }

  // Sanitize
  const sanitizedEmail = sanitizeEmail(email);

  return Response.json({ email: sanitizedEmail });
}

// ❌ Bad: No validation
export async function POST(request) {
  const body = await request.json();
  await db.insert(body); // SQL injection risk!
}
```

### CSRF Protection

```js
// middleware.js
export function middleware(request) {
  if (request.method === 'POST') {
    const csrfToken = request.headers.get('x-csrf-token');
    const expectedToken = request.cookies.get('csrf-token')?.value;

    if (csrfToken !== expectedToken) {
      return new Response('Invalid CSRF token', { status: 403 });
    }
  }

  return NextResponse.next();
}
```

---

## Error Handling

### Error Boundaries

```jsx
// app/error.js
'use client';

export default function Error({ error, reset }) {
  useEffect(() => {
    // Log to error reporting service
    console.error(error);
  }, [error]);

  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

### API Error Handling

```js
// ✅ Good: Consistent error responses
export async function GET(request) {
  try {
    const data = await fetchData();

    if (!data) {
      return Response.json(
        { error: 'Not found', code: 'NOT_FOUND' },
        { status: 404 }
      );
    }

    return Response.json({ data });
  } catch (error) {
    console.error('API Error:', error);

    return Response.json(
      { error: 'Internal server error', code: 'SERVER_ERROR' },
      { status: 500 }
    );
  }
}

// ❌ Bad: Inconsistent errors
export async function GET() {
  const data = await fetchData(); // Unhandled error
  return Response.json(data);
}
```

---

## Testing Strategies

### Unit Tests

```jsx
// components/__tests__/Button.test.js
import { render, screen } from '@testing/library/react';
import Button from '../Button';

describe('Button', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('handles click', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click</Button>);

    screen.getByText('Click').click();
    expect(handleClick).toHaveBeenCalled();
  });
});
```

### Integration Tests

```jsx
// app/__tests__/page.test.js
import { render, screen } from '@testing-library/react';
import Page from '../page';

// Mock data fetching
jest.mock('../lib/api', () => ({
  fetchPosts: jest.fn(() => Promise.resolve([
    { id: 1, title: 'Post 1' }
  ]))
}));

describe('Home Page', () => {
  it('displays posts', async () => {
    render(await Page());

    expect(screen.getByText('Post 1')).toBeInTheDocument();
  });
});
```

---

## Interview Questions

### Q1: What's the recommended project structure?

**Answer:**

```
app/                    # Routes
components/            # Shared UI
  ├── ui/             # Generic UI
  └── features/       # Feature-specific
lib/                  # Utils, helpers
public/               # Static assets
styles/               # Global styles
types/                # TypeScript types
```

---

### Q2: How do you optimize performance?

**Answer:**

1. **Images:** Use next/image
2. **Code splitting:** Dynamic imports
3. **Memoization:** useMemo, useCallback
4. **Fonts:** next/font
5. **Lazy loading:** Load below-fold content lazily

```jsx
import Image from 'next/image';
import dynamic from 'next/dynamic';

const Heavy = dynamic(() => import('./Heavy'));

<Image src="/img.jpg" width={800} height={600} alt="Img" />
```

---

### Q3: What are security best practices?

**Answer:**

1. **Environment variables:** Keep secrets server-side
2. **Validation:** Validate all inputs
3. **Sanitization:** Sanitize user data
4. **CSRF protection:** Use tokens
5. **Authentication:** Secure API routes

```js
// Server-only secret
const apiKey = process.env.API_KEY;

// Validate input
if (!isValidEmail(email)) {
  return Response.json({ error: 'Invalid' }, { status: 400 });
}
```

---

### Q4: How do you handle errors in Next.js?

**Answer:**

**Route-level:**
```jsx
// app/dashboard/error.js
'use client';

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Error!</h2>
      <button onClick={reset}>Retry</button>
    </div>
  );
}
```

**API routes:**
```js
try {
  const data = await fetchData();
  return Response.json({ data });
} catch (error) {
  return Response.json({ error: 'Failed' }, { status: 500 });
}
```

---

### Q5: What testing strategies should you use?

**Answer:**

1. **Unit tests:** Individual components/functions
2. **Integration tests:** Component interactions
3. **E2E tests:** User flows

```jsx
// Unit test
import { render } from '@testing-library/react';

test('Button renders', () => {
  render(<Button>Click</Button>);
  expect(screen.getByText('Click')).toBeInTheDocument();
});

// E2E test (Playwright/Cypress)
test('user can login', async () => {
  await page.goto('/login');
  await page.fill('input[name=email]', 'user@example.com');
  await page.click('button[type=submit]');
  await expect(page).toHaveURL('/dashboard');
});
```

---

## Key Takeaways

1. **Structure:** Organize by feature, separate concerns
2. **Performance:** next/image, dynamic imports, memoization
3. **Security:** Validate inputs, protect secrets, secure APIs
4. **Errors:** Use error.js, consistent error responses
5. **Testing:** Unit + Integration + E2E tests
6. **Code quality:** Single responsibility, reusable hooks
7. **Best practices:** Follow Next.js conventions, use built-in optimizations

---

[← Back to Next.js](./README.md)
