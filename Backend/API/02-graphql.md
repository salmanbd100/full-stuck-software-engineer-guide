# GraphQL

## Overview

GraphQL is a query language and runtime for APIs that provides a more efficient, powerful alternative to REST. Developed by Facebook in 2012 and open-sourced in 2015.

**Key Advantage:** Clients can request exactly the data they need, nothing more, nothing less.

## 🎯 Core Concepts

### 1. Schema Definition Language (SDL)

The GraphQL schema defines the API contract using a type system.

```graphql
# Define types
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  published: Boolean!
  author: User!
  comments: [Comment!]!
}

type Comment {
  id: ID!
  content: String!
  author: User!
  post: Post!
}

# Root types
type Query {
  user(id: ID!): User
  users(limit: Int, offset: Int): [User!]!
  post(id: ID!): Post
  posts(published: Boolean): [Post!]!
}

type Mutation {
  createUser(name: String!, email: String!): User!
  updateUser(id: ID!, name: String, email: String): User!
  deleteUser(id: ID!): Boolean!
  createPost(title: String!, content: String!, authorId: ID!): Post!
  publishPost(id: ID!): Post!
}

type Subscription {
  postCreated: Post!
  commentAdded(postId: ID!): Comment!
}
```

**Type Modifiers:**
- `String!` - Non-null (required)
- `[String]` - Array of strings (can be null)
- `[String!]!` - Non-null array of non-null strings

---

### 2. Queries

Request specific data from the server.

```graphql
# Simple query
query {
  user(id: "1") {
    id
    name
    email
  }
}

# Response
{
  "data": {
    "user": {
      "id": "1",
      "name": "John Doe",
      "email": "john@example.com"
    }
  }
}
```

**Nested Queries:**
```graphql
query {
  user(id: "1") {
    id
    name
    posts {
      id
      title
      comments {
        id
        content
        author {
          name
        }
      }
    }
  }
}
```

**With Variables:**
```graphql
query GetUser($userId: ID!) {
  user(id: $userId) {
    id
    name
    email
  }
}

# Variables
{
  "userId": "1"
}
```

---

### 3. Mutations

Modify server-side data.

```graphql
mutation {
  createUser(name: "Jane Doe", email: "jane@example.com") {
    id
    name
    email
    createdAt
  }
}

# Response
{
  "data": {
    "createUser": {
      "id": "2",
      "name": "Jane Doe",
      "email": "jane@example.com",
      "createdAt": "2024-12-08T10:00:00Z"
    }
  }
}
```

**Multiple Mutations:**
```graphql
mutation {
  user1: createUser(name: "Alice", email: "alice@example.com") {
    id
    name
  }
  user2: createUser(name: "Bob", email: "bob@example.com") {
    id
    name
  }
}
```

---

### 4. Subscriptions

Real-time updates via WebSocket.

```graphql
subscription {
  postCreated {
    id
    title
    author {
      name
    }
  }
}

# Client receives real-time updates when posts are created
```

---

## 💻 Implementation (Node.js + Apollo Server)

### Setup

```bash
npm install apollo-server graphql
```

### Basic Server

```javascript
const { ApolloServer, gql } = require('apollo-server');

// 1. Define schema
const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
  }

  type Query {
    users: [User!]!
    user(id: ID!): User
  }

  type Mutation {
    createUser(name: String!, email: String!): User!
  }
`;

// 2. In-memory data (replace with database)
let users = [
  { id: '1', name: 'John Doe', email: 'john@example.com' },
  { id: '2', name: 'Jane Smith', email: 'jane@example.com' },
];

// 3. Define resolvers
const resolvers = {
  Query: {
    users: () => users,
    user: (parent, args) => {
      return users.find(user => user.id === args.id);
    },
  },
  Mutation: {
    createUser: (parent, args) => {
      const newUser = {
        id: String(users.length + 1),
        name: args.name,
        email: args.email,
      };
      users.push(newUser);
      return newUser;
    },
  },
};

// 4. Create server
const server = new ApolloServer({
  typeDefs,
  resolvers,
});

// 5. Start server
server.listen().then(({ url }) => {
  console.log(`🚀 Server ready at ${url}`);
});
```

---

### With Database (MongoDB + Mongoose)

```javascript
const { ApolloServer, gql } = require('apollo-server');
const mongoose = require('mongoose');

// Mongoose models
const User = mongoose.model('User', {
  name: String,
  email: String,
});

const Post = mongoose.model('Post', {
  title: String,
  content: String,
  published: Boolean,
  authorId: mongoose.Schema.Types.ObjectId,
});

const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
  }

  type Post {
    id: ID!
    title: String!
    content: String!
    published: Boolean!
    author: User!
  }

  type Query {
    users: [User!]!
    user(id: ID!): User
    posts: [Post!]!
    post(id: ID!): Post
  }

  type Mutation {
    createUser(name: String!, email: String!): User!
    createPost(
      title: String!
      content: String!
      authorId: ID!
    ): Post!
  }
`;

const resolvers = {
  Query: {
    users: async () => await User.find(),
    user: async (parent, { id }) => await User.findById(id),
    posts: async () => await Post.find(),
    post: async (parent, { id }) => await Post.findById(id),
  },

  Mutation: {
    createUser: async (parent, { name, email }) => {
      const user = new User({ name, email });
      await user.save();
      return user;
    },
    createPost: async (parent, { title, content, authorId }) => {
      const post = new Post({
        title,
        content,
        published: false,
        authorId,
      });
      await post.save();
      return post;
    },
  },

  // Field resolvers
  User: {
    posts: async (parent) => {
      return await Post.find({ authorId: parent.id });
    },
  },

  Post: {
    author: async (parent) => {
      return await User.findById(parent.authorId);
    },
  },
};

// Connect to MongoDB and start server
mongoose.connect('mongodb://localhost:27017/graphql-demo')
  .then(() => {
    const server = new ApolloServer({ typeDefs, resolvers });
    return server.listen();
  })
  .then(({ url }) => {
    console.log(`🚀 Server ready at ${url}`);
  });
```

---

## 🔍 Advanced Patterns

### 1. DataLoader (N+1 Problem Solution)

**Problem:** Without DataLoader, nested queries cause multiple database calls.

```javascript
// ❌ BAD: N+1 queries
Post: {
  author: async (parent) => {
    // Called once for EACH post - N+1 problem!
    return await User.findById(parent.authorId);
  }
}

// For 100 posts, makes 100+ database queries!
```

**Solution:** Use DataLoader to batch and cache requests.

```javascript
const DataLoader = require('dataloader');

// Batch function
async function batchUsers(ids) {
  const users = await User.find({ _id: { $in: ids } });
  // Return users in same order as ids
  return ids.map(id => users.find(user => user.id.equals(id)));
}

// Create loader
const userLoader = new DataLoader(batchUsers);

// Use in resolver
Post: {
  author: async (parent) => {
    return await userLoader.load(parent.authorId);
  }
}

// For 100 posts, makes only 1 database query!
```

---

### 2. Context & Authentication

```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    // Get token from header
    const token = req.headers.authorization || '';

    // Verify token and get user
    const user = verifyToken(token);

    // Add to context
    return { user, userLoader };
  },
});

// Use in resolvers
const resolvers = {
  Query: {
    me: (parent, args, context) => {
      if (!context.user) {
        throw new Error('Not authenticated');
      }
      return context.user;
    },
  },

  Mutation: {
    createPost: (parent, args, context) => {
      if (!context.user) {
        throw new Error('Not authenticated');
      }

      const post = new Post({
        ...args,
        authorId: context.user.id,
      });

      return post.save();
    },
  },
};
```

---

### 3. Error Handling

```javascript
const { ApolloError, UserInputError, AuthenticationError } = require('apollo-server');

const resolvers = {
  Query: {
    user: async (parent, { id }) => {
      const user = await User.findById(id);

      if (!user) {
        throw new UserInputError('User not found', {
          invalidArgs: { id },
        });
      }

      return user;
    },
  },

  Mutation: {
    createUser: async (parent, { email, name }, context) => {
      // Check auth
      if (!context.user) {
        throw new AuthenticationError('Must be logged in');
      }

      // Validate input
      if (!email.includes('@')) {
        throw new UserInputError('Invalid email format', {
          invalidArgs: { email },
        });
      }

      // Check duplicates
      const existing = await User.findOne({ email });
      if (existing) {
        throw new ApolloError('Email already exists', 'DUPLICATE_EMAIL');
      }

      const user = new User({ email, name });
      await user.save();
      return user;
    },
  },
};
```

---

### 4. Pagination

**Offset-based:**
```graphql
type Query {
  posts(limit: Int, offset: Int): PostConnection!
}

type PostConnection {
  posts: [Post!]!
  totalCount: Int!
  hasMore: Boolean!
}
```

```javascript
Query: {
  posts: async (parent, { limit = 10, offset = 0 }) => {
    const posts = await Post.find()
      .skip(offset)
      .limit(limit);

    const totalCount = await Post.countDocuments();

    return {
      posts,
      totalCount,
      hasMore: offset + limit < totalCount,
    };
  },
}
```

**Cursor-based (Relay-style):**
```graphql
type Query {
  posts(first: Int, after: String): PostConnection!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  endCursor: String
}
```

---

### 5. Subscriptions with WebSocket

```javascript
const { ApolloServer, gql, PubSub } = require('apollo-server');

const pubsub = new PubSub();
const POST_CREATED = 'POST_CREATED';

const resolvers = {
  Mutation: {
    createPost: async (parent, args) => {
      const post = new Post(args);
      await post.save();

      // Publish event
      pubsub.publish(POST_CREATED, { postCreated: post });

      return post;
    },
  },

  Subscription: {
    postCreated: {
      subscribe: () => pubsub.asyncIterator([POST_CREATED]),
    },
  },
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  subscriptions: {
    path: '/subscriptions',
    onConnect: (connectionParams) => {
      // Authenticate WebSocket connection
      const token = connectionParams.authorization;
      const user = verifyToken(token);
      return { user };
    },
  },
});
```

---

## 🆚 GraphQL vs REST

| Feature | GraphQL | REST |
|---------|---------|------|
| **Data Fetching** | Single request for all data | Multiple endpoints |
| **Over-fetching** | Request only needed fields | Returns all fields |
| **Under-fetching** | Get related data in one query | Multiple requests needed |
| **Versioning** | No versioning needed | /v1/, /v2/ |
| **Type System** | Strong typing with schema | No built-in typing |
| **Tooling** | GraphQL Playground, introspection | Swagger, Postman |
| **Caching** | More complex (need libs) | HTTP caching built-in |
| **Learning Curve** | Steeper | Gentler |
| **Best For** | Complex, related data | Simple CRUD, public APIs |

---

## 🎯 GraphQL Best Practices

### 1. Schema Design

```graphql
# ✅ GOOD: Descriptive, specific types
type Product {
  id: ID!
  name: String!
  price: Money!  # Custom scalar
  category: Category!
  reviews(limit: Int): [Review!]!
}

type Money {
  amount: Float!
  currency: String!
}

# ❌ BAD: Generic, unclear types
type Product {
  id: ID!
  name: String!
  price: Float!  # What currency?
  cat: String!   # Unclear abbreviation
}
```

### 2. Naming Conventions

```graphql
# ✅ GOOD: Clear, consistent naming
type Query {
  user(id: ID!): User          # Singular for single item
  users(limit: Int): [User!]!  # Plural for lists
  searchUsers(query: String!): [User!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}

input CreateUserInput {
  name: String!
  email: String!
}
```

### 3. Input Types for Mutations

```graphql
# ✅ GOOD: Use input types
input CreatePostInput {
  title: String!
  content: String!
  tags: [String!]
  published: Boolean
}

type Mutation {
  createPost(input: CreatePostInput!): Post!
}

# ❌ BAD: Many individual arguments
type Mutation {
  createPost(
    title: String!
    content: String!
    tags: [String!]
    published: Boolean
  ): Post!
}
```

### 4. Error Handling

```javascript
// ✅ GOOD: Detailed error responses
{
  "errors": [
    {
      "message": "User not found",
      "extensions": {
        "code": "USER_NOT_FOUND",
        "userId": "123"
      }
    }
  ]
}

// Use custom error classes
class UserNotFoundError extends ApolloError {
  constructor(userId) {
    super('User not found', 'USER_NOT_FOUND', { userId });
  }
}
```

---

## 📊 Performance Optimization

### 1. Query Complexity Analysis

```javascript
const { createComplexityLimitRule } = require('graphql-validation-complexity');

const server = new ApolloServer({
  validationRules: [
    createComplexityLimitRule(1000, {
      onCost: (cost) => console.log('Query cost:', cost),
    }),
  ],
});
```

### 2. Query Depth Limiting

```javascript
const depthLimit = require('graphql-depth-limit');

const server = new ApolloServer({
  validationRules: [depthLimit(5)],  // Max depth of 5
});
```

### 3. Batch Resolvers

```javascript
// Use DataLoader for all one-to-many relationships
const loaders = {
  user: new DataLoader(ids => batchGetUsers(ids)),
  post: new DataLoader(ids => batchGetPosts(ids)),
  comment: new DataLoader(ids => batchGetComments(ids)),
};

// Add to context
context: () => ({ loaders })
```

---

## 🔒 Security

### 1. Disable Introspection in Production

```javascript
const server = new ApolloServer({
  introspection: process.env.NODE_ENV !== 'production',
  playground: process.env.NODE_ENV !== 'production',
});
```

### 2. Rate Limiting

```javascript
const { RateLimitDirective } = require('graphql-rate-limit-directive');

const typeDefs = gql`
  directive @rateLimit(
    limit: Int!
    duration: Int!
  ) on FIELD_DEFINITION

  type Query {
    users: [User!]! @rateLimit(limit: 100, duration: 60)
    user(id: ID!): User @rateLimit(limit: 1000, duration: 60)
  }
`;
```

### 3. Validate Input

```javascript
const Joi = require('joi');

const createUserSchema = Joi.object({
  name: Joi.string().min(2).max(50).required(),
  email: Joi.string().email().required(),
});

Mutation: {
  createUser: async (parent, { input }) => {
    const { error } = createUserSchema.validate(input);
    if (error) {
      throw new UserInputError(error.message);
    }
    // Proceed with creation
  }
}
```

---

## 📚 Interview Questions

### Q1: Explain the N+1 problem in GraphQL and how to solve it.

**Answer:**

The N+1 problem occurs when fetching related data results in 1 query to fetch parent records + N queries to fetch related child records.

**Example:**
```graphql
query {
  posts {          # 1 query
    title
    author {       # N queries (one per post)
      name
    }
  }
}
```

**Solutions:**

1. **DataLoader**: Batches and caches database requests
```javascript
const userLoader = new DataLoader(async (userIds) => {
  const users = await User.find({ _id: { $in: userIds } });
  return userIds.map(id => users.find(u => u.id.equals(id)));
});

// In resolver
author: (post) => userLoader.load(post.authorId)
```

2. **Select fields intelligently**: Only fetch needed data
3. **Use projections**: Reduce database query overhead

---

### Q2: When would you choose GraphQL over REST?

**Answer:**

**Choose GraphQL when:**
- Frontend needs flexible data fetching
- Mobile apps need to minimize data transfer
- Multiple related resources accessed together
- Schema changes frequently
- Strong typing beneficial

**Choose REST when:**
- Simple CRUD operations
- HTTP caching important
- File uploads/downloads primary use case
- Team unfamiliar with GraphQL
- Public API with wide adoption needed

**Example:** E-commerce dashboard showing user + orders + products → GraphQL perfect for fetching all in one query.

---

### Q3: How do you handle authentication in GraphQL?

**Answer:**

```javascript
const server = new ApolloServer({
  context: ({ req }) => {
    // 1. Extract token from header
    const token = req.headers.authorization?.replace('Bearer ', '');

    // 2. Verify token
    let user = null;
    if (token) {
      try {
        user = jwt.verify(token, process.env.JWT_SECRET);
      } catch (err) {
        // Invalid token - ignore or throw error
      }
    }

    // 3. Add user to context
    return { user };
  },
});

// 4. Check auth in resolvers
const resolvers = {
  Query: {
    me: (parent, args, { user }) => {
      if (!user) throw new AuthenticationError('Not authenticated');
      return user;
    },
  },
  Mutation: {
    createPost: (parent, args, { user }) => {
      if (!user) throw new AuthenticationError('Not authenticated');
      // Create post
    },
  },
};
```

---

### Q4: How do you implement pagination in GraphQL?

**Answer:**

**Two approaches:**

**1. Offset-based (simpler):**
```graphql
query {
  posts(limit: 10, offset: 20) {
    id
    title
  }
}
```

Pros: Simple, familiar
Cons: Performance issues with large offsets, inconsistent if data changes

**2. Cursor-based (recommended):**
```graphql
query {
  posts(first: 10, after: "cursor123") {
    edges {
      cursor
      node {
        id
        title
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

Pros: Consistent, performant, handles real-time changes
Cons: More complex implementation

---

## ✅ Key Takeaways

1. **GraphQL solves over/under-fetching** by allowing clients to request exactly what they need
2. **N+1 problem** is common - use DataLoader to batch requests
3. **Schema-first development** with strong typing catches errors early
4. **Authentication via context** - verify JWT in context function
5. **Pagination: cursor-based > offset-based** for large datasets
6. **Disable introspection** in production for security
7. **GraphQL complements REST** - doesn't always replace it

---

[← Back to Backend](../README.md) | [Next: API Versioning →](./03-versioning.md)
