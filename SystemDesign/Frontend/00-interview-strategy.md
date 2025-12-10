# Frontend System Design Interview Strategy

## Overview

Frontend system design interviews are fundamentally different from coding interviews. Success requires demonstrating architectural thinking at scale, not just framework knowledge. This guide covers the critical mistakes that cause 95% of frontend developers to fail these interviews and how to avoid them.

## Why Most Developers Fail

The interviewer loses interest in 30 seconds. Why? Because system design isn't about your favorite framework. It's about demonstrating:

- **Architectural thinking** - How systems scale and interact
- **Trade-off analysis** - Understanding pros and cons of decisions
- **Requirements gathering** - Asking the right questions
- **End-to-end thinking** - Frontend as part of the larger system
- **Performance consciousness** - Building for scale from day one

## The 5 Critical Mistakes

### 1. Skipping Requirements Gathering

**What 95% do:**
Jump straight into drawing boxes and arrows without understanding the problem.

**What top performers do:**
Spend the first 10 minutes asking clarifying questions:

```
Critical Questions to Ask:

Scale & Users:
- What's the expected user scale? (1K vs 10M makes different architectures)
- What's the expected QPS?
- What are the growth projections?

User Flows:
- What are the core user flows?
- Which features are must-haves vs nice-to-haves?
- What's the user journey?

Performance Requirements:
- What are the performance requirements? (Time to Interactive, First Contentful Paint)
- What's the acceptable latency for API calls?
- What's the target for Core Web Vitals?

Platform & Constraints:
- Mobile, desktop, or both?
- Browser support requirements?
- Any specific constraints? (legacy systems, compliance, offline support)
- Geographic distribution of users?

Data & Content:
- How much data will be displayed?
- Is data real-time or eventually consistent?
- What's the media type? (text, images, video)
```

**Example: Design a Twitter Feed**

❌ **Bad Approach:**
"I'll use React with Redux for state management and display tweets in a list..."

✅ **Good Approach:**
"Before we start, let me clarify some requirements:
- How many daily active users are we expecting?
- What's the average number of tweets in a user's feed?
- Do we need real-time updates or is polling acceptable?
- What's our target for feed load time?
- Do we need to support infinite scroll?
- Are we building for mobile, desktop, or both?"

### 2. Ignoring Performance and Scalability

**The Red Flag:**
Saying "I'll optimize later" is an instant failure signal.

**What Strong Candidates Discuss:**

#### Code Splitting & Lazy Loading
```javascript
// Route-based code splitting
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./Dashboard'));
const Profile = lazy(() => import('./Profile'));
const Settings = lazy(() => import('./Settings'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<Profile />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}

// Component-level code splitting
const HeavyChart = lazy(() => import('./HeavyChart'));

function Analytics() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Analytics</button>
      {showChart && (
        <Suspense fallback={<Spinner />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

#### CDN Distribution
```
Architecture:
User → CloudFront (CDN) → Origin Server
       ├─ Static assets (JS, CSS, images) - Edge cached
       ├─ API calls - Passed through to origin
       └─ Dynamic content - Optional edge caching

Benefits:
- Reduced latency (content served from nearest edge location)
- Lower origin server load
- Better availability (content cached at multiple locations)
- Cost optimization (reduced bandwidth from origin)
```

#### Caching Strategies
```javascript
// Service Worker caching
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('v1').then((cache) => {
      return cache.addAll([
        '/static/css/main.css',
        '/static/js/bundle.js',
        '/static/images/logo.png'
      ]);
    })
  );
});

// API response caching with React Query
import { useQuery } from 'react-query';

function UserProfile({ userId }) {
  const { data, isLoading } = useQuery(
    ['user', userId],
    () => fetchUser(userId),
    {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      refetchOnWindowFocus: false
    }
  );

  if (isLoading) return <Skeleton />;
  return <Profile user={data} />;
}

// Browser cache with proper headers
// Server response:
Cache-Control: public, max-age=31536000, immutable  // Static assets
Cache-Control: private, max-age=300                 // User data
Cache-Control: no-cache                             // Always revalidate
```

#### Image Optimization
```jsx
// Responsive images
<picture>
  <source
    media="(min-width: 1024px)"
    srcSet="/images/hero-large.webp"
    type="image/webp"
  />
  <source
    media="(min-width: 768px)"
    srcSet="/images/hero-medium.webp"
    type="image/webp"
  />
  <img
    src="/images/hero-small.jpg"
    alt="Hero"
    loading="lazy"
    width="800"
    height="600"
  />
</picture>

// Next.js Image component (automatic optimization)
import Image from 'next/image';

<Image
  src="/images/product.jpg"
  alt="Product"
  width={500}
  height={300}
  placeholder="blur"
  blurDataURL="/images/product-blur.jpg"
  loading="lazy"
/>

// Progressive image loading
function ProgressiveImage({ src, placeholder }) {
  const [imgSrc, setImgSrc] = useState(placeholder);

  useEffect(() => {
    const img = new Image();
    img.src = src;
    img.onload = () => setImgSrc(src);
  }, [src]);

  return (
    <img
      src={imgSrc}
      className={imgSrc === placeholder ? 'blur' : 'sharp'}
      alt="Content"
    />
  );
}
```

#### Bundle Size Management
```javascript
// Tree-shaking with ES modules
import { debounce } from 'lodash-es'; // Only imports debounce
// vs
import debounce from 'lodash/debounce'; // Even better

// Dynamic imports for large libraries
async function loadChart() {
  const { Chart } = await import('chart.js');
  return new Chart(ctx, config);
}

// Bundle analysis
// package.json
{
  "scripts": {
    "analyze": "webpack-bundle-analyzer dist/stats.json"
  }
}

// Webpack configuration
module.exports = {
  optimization: {
    usedExports: true,        // Tree-shaking
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10
        },
        common: {
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

#### Rendering Patterns
```javascript
// CSR (Client-Side Rendering) - SPA
// ❌ Slow initial load, poor SEO
// ✅ Fast subsequent navigation, rich interactions

// SSR (Server-Side Rendering) - Traditional
// ❌ Higher server costs, slower TTFB
// ✅ Better SEO, faster initial load

// SSG (Static Site Generation) - JAMstack
// ❌ Build time increases with pages
// ✅ Best performance, lowest cost

// ISR (Incremental Static Regeneration) - Next.js
// ❌ Complexity in cache invalidation
// ✅ Best of both worlds - static + dynamic

// Choosing the right pattern:
const renderingStrategy = {
  marketing: 'SSG',           // Static content, best SEO
  blog: 'ISR',                // Mix of static + new posts
  dashboard: 'CSR',           // Authenticated, dynamic data
  ecommerce: 'SSR + CSR',     // Product pages SSR, cart CSR
  documentation: 'SSG'        // Static, searchable
};

// Next.js implementation
export async function getStaticProps() {
  // SSG - Built at build time
  return { props: { data } };
}

export async function getServerSideProps() {
  // SSR - Built on each request
  return { props: { data } };
}

export async function getStaticProps() {
  // ISR - Rebuilt after specified time
  return {
    props: { data },
    revalidate: 60 // Rebuild every 60 seconds
  };
}
```

### 3. Treating Frontend as an Island

**The Problem:**
Failing to discuss how frontend integrates with the broader system.

**What to Discuss:**

#### API Design & Integration
```javascript
// RESTful API considerations
const apiDesign = {
  versioning: '/api/v1/users',           // URL versioning
  pagination: '?page=1&limit=20',        // Cursor-based better
  filtering: '?status=active&role=admin',
  sorting: '?sort=-createdAt',           // - for descending
  fieldSelection: '?fields=id,name,email' // Reduce payload
};

// GraphQL considerations
const graphqlQuery = `
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
      posts(limit: 10) {
        id
        title
      }
    }
  }
`;
// Pros: Single request, no over-fetching
// Cons: Complex caching, backend complexity

// REST vs GraphQL decision matrix:
// - Use REST when: Simple CRUD, standard caching, public API
// - Use GraphQL when: Complex nested data, multiple clients, flexibility needed
```

#### WebSocket for Real-Time Features
```javascript
// WebSocket connection management
class WebSocketService {
  constructor(url) {
    this.url = url;
    this.ws = null;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
  }

  connect() {
    this.ws = new WebSocket(this.url);

    this.ws.onopen = () => {
      console.log('Connected');
      this.reconnectAttempts = 0;
    };

    this.ws.onclose = () => {
      this.reconnect();
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };

    this.ws.onmessage = (event) => {
      this.handleMessage(JSON.parse(event.data));
    };
  }

  reconnect() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      const delay = Math.min(1000 * 2 ** this.reconnectAttempts, 30000);
      setTimeout(() => this.connect(), delay);
    }
  }

  send(data) {
    if (this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(data));
    }
  }
}

// When to use WebSockets:
// ✅ Real-time chat, live updates, collaborative editing
// ❌ Simple notifications (use SSE), infrequent updates (use polling)
```

#### Authentication & Authorization Flows
```javascript
// JWT-based auth flow
const authFlow = `
1. User Login:
   POST /api/auth/login { email, password }
   ← { accessToken, refreshToken }

2. Store tokens:
   - accessToken: Memory (httpOnly cookie or memory)
   - refreshToken: httpOnly cookie (more secure)

3. Authenticated requests:
   GET /api/users
   Headers: { Authorization: 'Bearer <accessToken>' }

4. Token refresh:
   POST /api/auth/refresh
   Cookie: refreshToken
   ← { accessToken }

5. Logout:
   POST /api/auth/logout
   - Clear client tokens
   - Invalidate refresh token server-side
`;

// Implementation
class AuthService {
  async login(email, password) {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    });

    const { accessToken, refreshToken } = await response.json();

    // Store access token in memory
    this.accessToken = accessToken;

    // Refresh token stored in httpOnly cookie by server
    return { success: true };
  }

  async refreshToken() {
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      credentials: 'include' // Send cookies
    });

    if (response.ok) {
      const { accessToken } = await response.json();
      this.accessToken = accessToken;
      return true;
    }
    return false;
  }

  // Axios interceptor for token refresh
  setupInterceptors(axios) {
    axios.interceptors.response.use(
      (response) => response,
      async (error) => {
        const originalRequest = error.config;

        if (error.response?.status === 401 && !originalRequest._retry) {
          originalRequest._retry = true;

          const refreshed = await this.refreshToken();
          if (refreshed) {
            originalRequest.headers.Authorization = `Bearer ${this.accessToken}`;
            return axios(originalRequest);
          }
        }

        return Promise.reject(error);
      }
    );
  }
}
```

#### Error Handling & Retry Strategies
```javascript
// Exponential backoff retry
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }

      return await response.json();
    } catch (error) {
      const isLastAttempt = i === maxRetries - 1;

      if (isLastAttempt) {
        throw error;
      }

      // Exponential backoff: 1s, 2s, 4s, 8s...
      const delay = Math.min(1000 * 2 ** i, 10000);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// React Query with automatic retries
import { useQuery } from 'react-query';

function UserData({ userId }) {
  const { data, error, isLoading } = useQuery(
    ['user', userId],
    () => fetchUser(userId),
    {
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      onError: (error) => {
        // Log to monitoring service
        errorTracker.log(error);
      }
    }
  );

  if (error) return <ErrorBoundary error={error} />;
  if (isLoading) return <Skeleton />;
  return <UserProfile user={data} />;
}
```

#### Monitoring & Observability
```javascript
// Error tracking with Sentry
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: 'YOUR_SENTRY_DSN',
  integrations: [
    new Sentry.BrowserTracing(),
    new Sentry.Replay()
  ],
  tracesSampleRate: 0.1,
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0
});

// Performance monitoring
const analytics = {
  trackPageLoad: () => {
    window.addEventListener('load', () => {
      const perfData = performance.getEntriesByType('navigation')[0];

      analytics.send({
        metric: 'page_load',
        ttfb: perfData.responseStart - perfData.requestStart,
        fcp: performance.getEntriesByName('first-contentful-paint')[0]?.startTime,
        lcp: performance.getEntriesByType('largest-contentful-paint')[0]?.startTime,
        cls: getCLS(),
        fid: getFID()
      });
    });
  },

  trackAPICall: (endpoint, duration, status) => {
    analytics.send({
      metric: 'api_call',
      endpoint,
      duration,
      status
    });
  }
};

// Logging strategy
const logger = {
  info: (message, meta) => {
    console.log(message, meta);
    // Send to logging service
  },
  error: (error, context) => {
    console.error(error, context);
    Sentry.captureException(error, { contexts: { custom: context } });
  },
  performance: (metric) => {
    // Send to analytics
    analytics.send(metric);
  }
};
```

#### Data Consistency Across Distributed Systems
```javascript
// Optimistic updates with rollback
function useTodoMutation() {
  const queryClient = useQueryClient();

  return useMutation(
    (newTodo) => api.createTodo(newTodo),
    {
      // Optimistic update
      onMutate: async (newTodo) => {
        await queryClient.cancelQueries(['todos']);

        const previousTodos = queryClient.getQueryData(['todos']);

        queryClient.setQueryData(['todos'], (old) => [
          ...old,
          { ...newTodo, id: 'temp-' + Date.now() }
        ]);

        return { previousTodos };
      },

      // Rollback on error
      onError: (err, newTodo, context) => {
        queryClient.setQueryData(['todos'], context.previousTodos);
        toast.error('Failed to create todo');
      },

      // Refetch on success
      onSettled: () => {
        queryClient.invalidateQueries(['todos']);
      }
    }
  );
}

// Eventual consistency handling
function SyncStatus() {
  const [syncState, setSyncState] = useState('synced');

  useEffect(() => {
    const handleOnline = () => {
      setSyncState('syncing');
      syncLocalChanges().then(() => {
        setSyncState('synced');
      });
    };

    window.addEventListener('online', handleOnline);
    return () => window.removeEventListener('online', handleOnline);
  }, []);

  return (
    <div>
      {syncState === 'offline' && '⚠️ Offline - changes saved locally'}
      {syncState === 'syncing' && '🔄 Syncing...'}
      {syncState === 'synced' && '✓ All changes synced'}
    </div>
  );
}
```

### 4. Weak Component Architecture

**The Problem:**
Drawing a component tree isn't architecture. You need to explain design decisions.

**What to Discuss:**

#### State Management Patterns
```javascript
// Avoid prop drilling with Context
const UserContext = createContext();

function App() {
  const user = useAuth();

  return (
    <UserContext.Provider value={user}>
      <Dashboard />
    </UserContext.Provider>
  );
}

// Deep nested component
function UserAvatar() {
  const user = useContext(UserContext);  // No prop drilling!
  return <img src={user.avatar} alt={user.name} />;
}

// When to use different state solutions:
const stateManagement = {
  local: 'useState/useReducer - Component-specific state',
  context: 'Context API - Small to medium global state',
  redux: 'Redux - Large apps, complex state, time-travel debugging',
  zustand: 'Zustand - Simpler alternative to Redux',
  reactQuery: 'React Query - Server state management',
  recoil: 'Recoil - Atom-based state, granular updates'
};
```

#### Component Composition vs Inheritance
```jsx
// ❌ Inheritance (anti-pattern in React)
class BaseButton extends React.Component {
  render() {
    return <button>{this.props.children}</button>;
  }
}

class PrimaryButton extends BaseButton {
  render() {
    return <button className="primary">{this.props.children}</button>;
  }
}

// ✅ Composition (recommended)
function Button({ variant = 'default', children, ...props }) {
  const className = `btn btn-${variant}`;
  return <button className={className} {...props}>{children}</button>;
}

// Usage
<Button variant="primary">Click me</Button>

// Composition with children
function Card({ children, header, footer }) {
  return (
    <div className="card">
      {header && <div className="card-header">{header}</div>}
      <div className="card-body">{children}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}

// Flexible usage
<Card
  header={<h3>Title</h3>}
  footer={<button>Action</button>}
>
  <p>Content</p>
</Card>

// Render props pattern
function DataFetcher({ url, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(url)
      .then(r => r.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [url]);

  return render({ data, loading });
}

// Usage
<DataFetcher
  url="/api/users"
  render={({ data, loading }) => (
    loading ? <Loading /> : <UserList users={data} />
  )}
/>
```

#### Shared Component Libraries (Design Systems)
```javascript
// Design system structure
/*
design-system/
├── tokens/
│   ├── colors.ts        # Color palette
│   ├── spacing.ts       # Spacing scale
│   ├── typography.ts    # Font sizes, weights
│   └── breakpoints.ts   # Responsive breakpoints
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   ├── Button.stories.tsx
│   │   └── Button.styles.ts
│   └── ...
└── hooks/
    └── useTheme.ts
*/

// Design tokens
export const colors = {
  primary: {
    50: '#e3f2fd',
    100: '#bbdefb',
    500: '#2196f3',
    900: '#0d47a1'
  },
  semantic: {
    success: '#4caf50',
    warning: '#ff9800',
    error: '#f44336',
    info: '#2196f3'
  }
};

export const spacing = {
  xs: '4px',
  sm: '8px',
  md: '16px',
  lg: '24px',
  xl: '32px'
};

// Button component with variants
import styled from 'styled-components';

const StyledButton = styled.button`
  padding: ${({ size }) => spacing[size]};
  background: ${({ variant }) =>
    variant === 'primary' ? colors.primary[500] : 'transparent'
  };
  border: 1px solid ${colors.primary[500]};
  border-radius: 4px;

  &:hover {
    background: ${colors.primary[600]};
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
`;

export function Button({ variant = 'primary', size = 'md', ...props }) {
  return <StyledButton variant={variant} size={size} {...props} />;
}

// Versioning strategy
// package.json
{
  "name": "@company/design-system",
  "version": "2.1.0",  // Major.Minor.Patch
  "peerDependencies": {
    "react": "^18.0.0"
  }
}

// Breaking changes = Major version bump
// New features = Minor version bump
// Bug fixes = Patch version bump
```

#### Micro-Frontend Considerations
```javascript
// When to use micro-frontends:
// ✅ Multiple teams working independently
// ✅ Different tech stacks needed
// ✅ Independent deployment requirements
// ✅ Very large applications (100+ pages)
// ❌ Small teams
// ❌ Simple applications
// ❌ Tight coupling needed

// Module Federation (Webpack 5)
// Host app webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        productApp: 'productApp@http://localhost:3001/remoteEntry.js',
        checkoutApp: 'checkoutApp@http://localhost:3002/remoteEntry.js'
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true }
      }
    })
  ]
};

// Remote app webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'productApp',
      filename: 'remoteEntry.js',
      exposes: {
        './ProductList': './src/ProductList'
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true }
      }
    })
  ]
};

// Usage in host
import ProductList from 'productApp/ProductList';

function App() {
  return (
    <div>
      <h1>E-commerce App</h1>
      <React.Suspense fallback="Loading...">
        <ProductList />
      </React.Suspense>
    </div>
  );
}

// Communication between micro-frontends
// Event bus pattern
class EventBus {
  constructor() {
    this.events = {};
  }

  subscribe(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }

  publish(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(data));
    }
  }
}

export const eventBus = new EventBus();

// In Product micro-frontend
eventBus.publish('addToCart', { productId: 123, quantity: 1 });

// In Cart micro-frontend
eventBus.subscribe('addToCart', (item) => {
  updateCart(item);
});
```

### 5. No Discussion of Trade-offs

**The Problem:**
Every architectural decision has trade-offs. Failing to acknowledge them signals inexperience.

**How to Discuss Trade-offs:**

#### Rendering Strategy Trade-offs
```
Client-Side Rendering (CSR):
✅ Pros:
   - Simpler to deploy (static files)
   - Rich interactivity
   - Fast subsequent navigation
   - Lower server costs
❌ Cons:
   - Slow initial load (large JS bundle)
   - Poor SEO (content not in initial HTML)
   - Slower First Contentful Paint
   - Requires more client resources

Server-Side Rendering (SSR):
✅ Pros:
   - Better SEO (content in initial HTML)
   - Faster First Contentful Paint
   - Works without JavaScript
   - Better for low-powered devices
❌ Cons:
   - Higher server costs
   - Slower Time to First Byte
   - More complex deployment
   - Server load increases with traffic

Static Site Generation (SSG):
✅ Pros:
   - Best performance (pre-built)
   - Lowest server costs
   - Best SEO
   - High availability (CDN)
❌ Cons:
   - Build time increases with pages
   - Stale data (need rebuilds)
   - Not suitable for dynamic content
   - Longer deployment times

Decision Matrix:
- Marketing site → SSG (best performance, SEO)
- E-commerce → SSR + CSR (product pages SSR, cart CSR)
- Dashboard → CSR (authenticated, dynamic)
- Blog → ISR (mix of static + fresh content)
```

#### State Management Trade-offs
```
Redux:
✅ Pros:
   - Predictable state updates
   - Time-travel debugging
   - Excellent DevTools
   - Large ecosystem
   - Works with any UI library
❌ Cons:
   - Boilerplate code
   - Learning curve
   - Bundle size
   - Can be overkill for simple apps

Context API:
✅ Pros:
   - Built into React
   - No extra dependencies
   - Simple API
   - Good for small to medium apps
❌ Cons:
   - Re-render issues
   - No DevTools
   - Can cause prop drilling
   - Performance issues at scale

Zustand:
✅ Pros:
   - Minimal boilerplate
   - Small bundle size
   - Simple API
   - Good performance
❌ Cons:
   - Smaller ecosystem
   - Less mature
   - No time-travel debugging
   - Fewer learning resources

Decision:
- Small app (<10 components) → useState/useReducer
- Medium app (10-50 components) → Context API or Zustand
- Large app (50+ components, complex state) → Redux or Zustand
```

#### Monorepo vs Polyrepo Trade-offs
```
Monorepo:
✅ Pros:
   - Simplified dependency management
   - Easier code sharing
   - Atomic commits across projects
   - Easier refactoring
   - Single source of truth
❌ Cons:
   - Slower CI/CD at scale
   - Larger repository size
   - More complex tooling
   - Potential merge conflicts
   - Requires good tooling (Nx, Turborepo)

Polyrepo:
✅ Pros:
   - Independent deployments
   - Smaller, faster repos
   - Team autonomy
   - Simpler CI/CD per repo
   - Clear ownership
❌ Cons:
   - Dependency hell
   - Code duplication
   - Difficult to share code
   - Versioning complexity
   - Harder to refactor across repos

Decision:
- Multiple related projects, shared code → Monorepo
- Independent services, different teams → Polyrepo
- Micro-frontends → Can work with either
```

## Interview Framework for Frontend System Design

### The Structured Approach

```
1. Requirements Clarification (10 min)
   ├─ Functional requirements
   ├─ Non-functional requirements (scale, performance)
   ├─ User flows
   └─ Constraints

2. High-Level Architecture (15 min)
   ├─ Client architecture (SPA, MPA, PWA)
   ├─ Rendering strategy (CSR, SSR, SSG)
   ├─ State management approach
   ├─ API integration
   └─ CDN and caching

3. Component Design (10 min)
   ├─ Component hierarchy
   ├─ State management
   ├─ Data flow
   └─ Reusable patterns

4. Performance Optimization (10 min)
   ├─ Code splitting
   ├─ Lazy loading
   ├─ Caching strategies
   ├─ Bundle optimization
   └─ Image optimization

5. Deep Dive (10 min)
   ├─ Real-time features
   ├─ Offline support
   ├─ Security considerations
   ├─ Error handling
   └─ Monitoring

6. Trade-offs & Alternatives (5 min)
   ├─ Technology choices
   ├─ Architectural decisions
   ├─ Performance vs complexity
   └─ Cost considerations
```

## Common Frontend Interview Questions

### Easy
- Design a todo list application
- Design a image carousel
- Design a infinite scroll feed
- Design a autocomplete search

### Medium
- Design Twitter feed
- Design Instagram stories
- Design Facebook news feed
- Design a collaborative text editor (Google Docs lite)

### Hard
- Design Netflix video player
- Design Figma (collaborative design tool)
- Design Google Maps frontend
- Design Notion (collaborative workspace)

## Best Practices Summary

✅ **Always Do:**
- Ask clarifying questions (10 minutes minimum)
- Discuss performance proactively
- Consider the entire system (frontend + backend + infrastructure)
- Explain trade-offs for every decision
- Think about scale from the start
- Draw clear diagrams
- Communicate your thought process
- Consider real-world constraints

❌ **Never Do:**
- Jump straight to solutions
- Ignore performance and scalability
- Focus only on your favorite framework
- Forget about error handling
- Ignore monitoring and observability
- Overcomplicate simple solutions
- Assume requirements
- Ignore non-functional requirements

## Summary

Frontend system design interviews test your ability to:
1. **Gather requirements** - Ask the right questions
2. **Think architecturally** - Design scalable systems
3. **Consider performance** - Optimize from day one
4. **Integrate with systems** - Frontend as part of the whole
5. **Make trade-offs** - Understand pros and cons

Success comes from demonstrating systematic thinking, not memorizing solutions. Practice these patterns, understand the trade-offs, and always think about scale.

---
[← Back to SystemDesign](../README.md)
