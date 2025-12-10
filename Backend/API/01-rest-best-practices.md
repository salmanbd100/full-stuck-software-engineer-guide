# REST API Best Practices

## Overview

REST (Representational State Transfer) is an architectural style for designing networked applications. RESTful APIs use HTTP requests to perform CRUD operations.

**Key Principle:** Resources are represented by URLs, and HTTP methods define actions on those resources.

## 🎯 HTTP Methods (CRUD)

### Standard HTTP Methods

| Method | Action | Idempotent | Safe | Body |
|--------|--------|------------|------|------|
| GET | Read/Retrieve | ✅ | ✅ | No |
| POST | Create | ❌ | ❌ | Yes |
| PUT | Update/Replace | ✅ | ❌ | Yes |
| PATCH | Partial Update | ❌ | ❌ | Yes |
| DELETE | Delete | ✅ | ❌ | Optional |

**Idempotent:** Multiple identical requests have the same effect as a single request
**Safe:** Does not modify resources

### Examples

```javascript
// GET - Retrieve resources
GET /api/users              // Get all users
GET /api/users/123          // Get specific user
GET /api/users/123/posts    // Get user's posts

// POST - Create new resource
POST /api/users
Body: { "name": "John", "email": "john@example.com" }

// PUT - Replace entire resource
PUT /api/users/123
Body: { "name": "John Doe", "email": "john@example.com", "age": 30 }

// PATCH - Update specific fields
PATCH /api/users/123
Body: { "email": "newemail@example.com" }

// DELETE - Remove resource
DELETE /api/users/123
```

---

## 📐 URL Design Best Practices

### 1. Use Nouns, Not Verbs

```
❌ BAD:
GET /getUsers
POST /createUser
PUT /updateUser/123

✅ GOOD:
GET /users
POST /users
PUT /users/123
```

### 2. Use Plural Nouns

```
❌ BAD:
GET /user
GET /user/123

✅ GOOD:
GET /users
GET /users/123
```

### 3. Nested Resources

```
GET /users/123/posts          // User's posts
GET /users/123/posts/456      // Specific post
POST /users/123/posts         // Create post for user
GET /posts/456/comments       // Post's comments
```

### 4. Keep URLs Lowercase

```
❌ BAD:
/getUserProfile
/UserProfile

✅ GOOD:
/user-profile
/api/user-preferences
```

### 5. Version Your API

```
✅ GOOD:
/api/v1/users
/api/v2/users
```

### 6. Query Parameters for Filtering

```
// Filtering
GET /users?role=admin&status=active

// Sorting
GET /users?sort=createdAt:desc

// Pagination
GET /users?page=2&limit=20

// Combined
GET /users?role=admin&sort=-createdAt&page=1&limit=10
```

---

## 📊 HTTP Status Codes

### Success (2xx)

| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | GET, PUT, PATCH successful |
| 201 | Created | POST successful |
| 204 | No Content | DELETE successful |

### Client Errors (4xx)

| Code | Meaning | Use Case |
|------|---------|----------|
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Not authorized |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate email |
| 422 | Unprocessable Entity | Validation errors |
| 429 | Too Many Requests | Rate limit exceeded |

### Server Errors (5xx)

| Code | Meaning | Use Case |
|------|---------|----------|
| 500 | Internal Server Error | Unhandled error |
| 503 | Service Unavailable | Temporary downtime |

---

## 📄 Pagination

```javascript
app.get('/users', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 20;
  const offset = (page - 1) * limit;

  const users = await User.find()
    .skip(offset)
    .limit(limit);

  const total = await User.countDocuments();

  res.json({
    data: users,
    meta: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
    },
    links: {
      next: page < Math.ceil(total / limit)
        ? `/users?page=${page + 1}`
        : null,
    },
  });
});
```

---

## 🔒 Authentication

```javascript
const jwt = require('jsonwebtoken');

// Authentication middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization?.replace('Bearer ', '');

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
};

// Usage
app.get('/profile', authenticate, (req, res) => {
  res.json(req.user);
});
```

---

## 🚦 Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests',
});

app.use('/api/', limiter);
```

---

## ✅ Key Takeaways

1. **Use HTTP methods correctly**: GET (read), POST (create), PUT/PATCH (update), DELETE (delete)
2. **Design URLs with nouns**: `/users`, not `/getUsers`
3. **Return appropriate status codes**: 200, 201, 204, 400, 401, 403, 404, 500
4. **Implement pagination** for list endpoints
5. **Rate limit** to prevent abuse
6. **Version your API**: `/api/v1/`
7. **Validate all input**
8. **Use JWT for authentication**

---

[← Back to Backend](../README.md) | [Next: GraphQL →](./02-graphql.md)
