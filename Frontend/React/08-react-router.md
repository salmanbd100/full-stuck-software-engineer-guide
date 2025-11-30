# React Router

React Router enables navigation and routing in single-page React applications.

## 📚 Core Concepts

### 1. Basic Setup

```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
    return (
        <BrowserRouter>
            <nav>
                <Link to="/">Home</Link>
                <Link to="/about">About</Link>
                <Link to="/users">Users</Link>
            </nav>

            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/about" element={<About />} />
                <Route path="/users" element={<Users />} />
                <Route path="*" element={<NotFound />} />
            </Routes>
        </BrowserRouter>
    );
}
```

### 2. Dynamic Routes

```jsx
import { useParams, useNavigate } from 'react-router-dom';

function App() {
    return (
        <Routes>
            <Route path="/users/:id" element={<UserDetail />} />
            <Route path="/posts/:postId/comments/:commentId" element={<Comment />} />
        </Routes>
    );
}

function UserDetail() {
    const { id } = useParams();
    const navigate = useNavigate();

    return (
        <div>
            <h2>User {id}</h2>
            <button onClick={() => navigate('/users')}>
                Back to Users
            </button>
        </div>
    );
}
```

### 3. Nested Routes

```jsx
function App() {
    return (
        <Routes>
            <Route path="/dashboard" element={<Dashboard />}>
                <Route path="profile" element={<Profile />} />
                <Route path="settings" element={<Settings />} />
            </Route>
        </Routes>
    );
}

function Dashboard() {
    return (
        <div>
            <h1>Dashboard</h1>
            <nav>
                <Link to="profile">Profile</Link>
                <Link to="settings">Settings</Link>
            </nav>
            <Outlet /> {/* Renders child routes */}
        </div>
    );
}
```

### 4. Protected Routes

```jsx
function ProtectedRoute({ children }) {
    const { isAuthenticated } = useAuth();

    if (!isAuthenticated) {
        return <Navigate to="/login" replace />;
    }

    return children;
}

function App() {
    return (
        <Routes>
            <Route path="/login" element={<Login />} />
            <Route
                path="/dashboard"
                element={
                    <ProtectedRoute>
                        <Dashboard />
                    </ProtectedRoute>
                }
            />
        </Routes>
    );
}
```

## 🎯 Common Interview Questions

### Q1: How to navigate programmatically?

**Answer:** Use the `useNavigate` hook:

```jsx
const navigate = useNavigate();

// Navigate to path
navigate('/home');

// Navigate with state
navigate('/user', { state: { from: 'login' } });

// Go back
navigate(-1);
```

### Q2: How to access URL parameters?

**Answer:** Use `useParams`, `useSearchParams`, or `useLocation`:

```jsx
// Path params: /users/:id
const { id } = useParams();

// Query params: /search?q=test
const [searchParams] = useSearchParams();
const query = searchParams.get('q');

// Full location
const location = useLocation();
```

## 🎓 Best Practices

1. **Use Links for navigation** (not <a> tags)
2. **Protect sensitive routes**
3. **Use nested routes** for layouts
4. **Handle 404s** with catch-all route
5. **Use lazy loading** for code splitting

---

[← Back to React](./README.md)
