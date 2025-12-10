# PostgreSQL

## Overview

**PostgreSQL** is an advanced, enterprise-class open-source relational database management system (RDBMS) known for its reliability, robustness, and performance. It supports both SQL (relational) and JSON (non-relational) querying.

**Key Strengths:**
- **ACID Compliant**: Ensures data integrity and consistency
- **Advanced Data Types**: JSONB, Arrays, UUID, Geometric types
- **Extensibility**: Custom functions, data types, and extensions
- **Full SQL Standard**: Superior SQL compliance compared to MySQL
- **Performance**: Handles complex queries and large datasets efficiently

---

## 🚀 Getting Started with PostgreSQL in Node.js

### Installation

```bash
npm install pg  # PostgreSQL client for Node.js
npm install dotenv  # For environment variables
```

### Basic Connection

```javascript
const { Pool } = require('pg');

// Create connection pool
const pool = new Pool({
  user: process.env.DB_USER || 'postgres',
  host: process.env.DB_HOST || 'localhost',
  database: process.env.DB_NAME || 'mydb',
  password: process.env.DB_PASSWORD,
  port: process.env.DB_PORT || 5432,
  max: 20,  // Maximum number of clients in pool
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Test connection
pool.query('SELECT NOW()', (err, res) => {
  if (err) {
    console.error('Database connection error:', err);
  } else {
    console.log('Connected to PostgreSQL:', res.rows[0].now);
  }
});

module.exports = pool;
```

### Connection with async/await

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: process.env.NODE_ENV === 'production' ? { rejectUnauthorized: false } : false
});

// Query helper function
async function query(text, params) {
  const start = Date.now();
  try {
    const res = await pool.query(text, params);
    const duration = Date.now() - start;
    console.log('Query executed:', { text, duration, rows: res.rowCount });
    return res;
  } catch (error) {
    console.error('Query error:', error);
    throw error;
  }
}

module.exports = { query, pool };
```

---

## 📊 Database Schema Design

### Creating Tables

```sql
-- Users table
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(100) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL,
  name VARCHAR(100) NOT NULL,
  role VARCHAR(20) DEFAULT 'user',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Posts table with foreign key
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(200) NOT NULL,
  content TEXT NOT NULL,
  author_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  published BOOLEAN DEFAULT FALSE,
  views INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Comments table (nested relationship)
CREATE TABLE comments (
  id SERIAL PRIMARY KEY,
  post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for performance
CREATE INDEX idx_posts_author ON posts(author_id);
CREATE INDEX idx_comments_post ON comments(post_id);
CREATE INDEX idx_comments_user ON comments(user_id);
CREATE INDEX idx_users_email ON users(email);
```

---

## 🔨 CRUD Operations

### Create (INSERT)

```javascript
const db = require('./db');

// Insert single user
async function createUser(email, password, name) {
  const query = `
    INSERT INTO users (email, password, name)
    VALUES ($1, $2, $3)
    RETURNING id, email, name, created_at
  `;

  try {
    const result = await db.query(query, [email, password, name]);
    return result.rows[0];
  } catch (error) {
    if (error.code === '23505') {  // Unique violation
      throw new Error('Email already exists');
    }
    throw error;
  }
}

// Insert multiple users
async function createUsers(users) {
  const query = `
    INSERT INTO users (email, password, name)
    VALUES ($1, $2, $3), ($4, $5, $6), ($7, $8, $9)
    RETURNING *
  `;

  const values = users.flatMap(u => [u.email, u.password, u.name]);
  const result = await db.query(query, values);
  return result.rows;
}

// Using unnest for bulk insert
async function bulkCreateUsers(users) {
  const emails = users.map(u => u.email);
  const passwords = users.map(u => u.password);
  const names = users.map(u => u.name);

  const query = `
    INSERT INTO users (email, password, name)
    SELECT * FROM UNNEST($1::varchar[], $2::varchar[], $3::varchar[])
    RETURNING *
  `;

  const result = await db.query(query, [emails, passwords, names]);
  return result.rows;
}
```

### Read (SELECT)

```javascript
// Find all users
async function getAllUsers() {
  const query = 'SELECT id, email, name, role, created_at FROM users';
  const result = await db.query(query);
  return result.rows;
}

// Find user by ID
async function getUserById(id) {
  const query = `
    SELECT id, email, name, role, created_at
    FROM users
    WHERE id = $1
  `;

  const result = await db.query(query, [id]);
  return result.rows[0];
}

// Find user by email
async function getUserByEmail(email) {
  const query = `
    SELECT id, email, password, name, role
    FROM users
    WHERE email = $1
  `;

  const result = await db.query(query, [email]);
  return result.rows[0];
}

// Pagination
async function getUsersPaginated(page = 1, limit = 20) {
  const offset = (page - 1) * limit;

  const query = `
    SELECT id, email, name, role, created_at
    FROM users
    ORDER BY created_at DESC
    LIMIT $1 OFFSET $2
  `;

  const countQuery = 'SELECT COUNT(*) FROM users';

  const [usersResult, countResult] = await Promise.all([
    db.query(query, [limit, offset]),
    db.query(countQuery)
  ]);

  return {
    users: usersResult.rows,
    total: parseInt(countResult.rows[0].count),
    page,
    totalPages: Math.ceil(countResult.rows[0].count / limit)
  };
}

// Search with LIKE
async function searchUsers(searchTerm) {
  const query = `
    SELECT id, email, name, role
    FROM users
    WHERE name ILIKE $1 OR email ILIKE $1
    ORDER BY name
  `;

  const result = await db.query(query, [`%${searchTerm}%`]);
  return result.rows;
}
```

### Update

```javascript
// Update user
async function updateUser(id, updates) {
  const { name, email, role } = updates;

  const query = `
    UPDATE users
    SET
      name = COALESCE($1, name),
      email = COALESCE($2, email),
      role = COALESCE($3, role),
      updated_at = CURRENT_TIMESTAMP
    WHERE id = $4
    RETURNING id, email, name, role, updated_at
  `;

  const result = await db.query(query, [name, email, role, id]);

  if (result.rowCount === 0) {
    throw new Error('User not found');
  }

  return result.rows[0];
}

// Increment views count
async function incrementViews(postId) {
  const query = `
    UPDATE posts
    SET views = views + 1
    WHERE id = $1
    RETURNING views
  `;

  const result = await db.query(query, [postId]);
  return result.rows[0];
}
```

### Delete

```javascript
// Delete user
async function deleteUser(id) {
  const query = 'DELETE FROM users WHERE id = $1 RETURNING id';
  const result = await db.query(query, [id]);

  if (result.rowCount === 0) {
    throw new Error('User not found');
  }

  return result.rows[0];
}

// Soft delete (mark as deleted)
async function softDeleteUser(id) {
  const query = `
    UPDATE users
    SET deleted_at = CURRENT_TIMESTAMP
    WHERE id = $1 AND deleted_at IS NULL
    RETURNING id
  `;

  const result = await db.query(query, [id]);

  if (result.rowCount === 0) {
    throw new Error('User not found or already deleted');
  }

  return result.rows[0];
}
```

---

## 🔗 Joins and Relationships

### INNER JOIN

```javascript
// Get posts with author information
async function getPostsWithAuthors() {
  const query = `
    SELECT
      p.id,
      p.title,
      p.content,
      p.created_at,
      u.id as author_id,
      u.name as author_name,
      u.email as author_email
    FROM posts p
    INNER JOIN users u ON p.author_id = u.id
    WHERE p.published = true
    ORDER BY p.created_at DESC
  `;

  const result = await db.query(query);
  return result.rows;
}
```

### LEFT JOIN

```javascript
// Get all users and their post count (including users with 0 posts)
async function getUsersWithPostCount() {
  const query = `
    SELECT
      u.id,
      u.name,
      u.email,
      COUNT(p.id) as post_count
    FROM users u
    LEFT JOIN posts p ON u.id = p.author_id
    GROUP BY u.id, u.name, u.email
    ORDER BY post_count DESC
  `;

  const result = await db.query(query);
  return result.rows;
}
```

### Multiple JOINs

```javascript
// Get posts with author and comment count
async function getPostsWithDetails() {
  const query = `
    SELECT
      p.id,
      p.title,
      p.views,
      p.created_at,
      u.name as author_name,
      COUNT(c.id) as comment_count
    FROM posts p
    INNER JOIN users u ON p.author_id = u.id
    LEFT JOIN comments c ON p.id = c.post_id
    WHERE p.published = true
    GROUP BY p.id, p.title, p.views, p.created_at, u.name
    ORDER BY p.created_at DESC
  `;

  const result = await db.query(query);
  return result.rows;
}
```

### Nested Relationships

```javascript
// Get post with author and all comments (with comment authors)
async function getPostWithComments(postId) {
  const postQuery = `
    SELECT
      p.id,
      p.title,
      p.content,
      p.created_at,
      json_build_object(
        'id', u.id,
        'name', u.name,
        'email', u.email
      ) as author
    FROM posts p
    INNER JOIN users u ON p.author_id = u.id
    WHERE p.id = $1
  `;

  const commentsQuery = `
    SELECT
      c.id,
      c.content,
      c.created_at,
      json_build_object(
        'id', u.id,
        'name', u.name
      ) as user
    FROM comments c
    INNER JOIN users u ON c.user_id = u.id
    WHERE c.post_id = $1
    ORDER BY c.created_at ASC
  `;

  const [postResult, commentsResult] = await Promise.all([
    db.query(postQuery, [postId]),
    db.query(commentsQuery, [postId])
  ]);

  if (postResult.rowCount === 0) {
    throw new Error('Post not found');
  }

  return {
    ...postResult.rows[0],
    comments: commentsResult.rows
  };
}
```

---

## 📦 PostgreSQL-Specific Features

### 1. JSONB Data Type

```javascript
// Create table with JSONB
const createProductsTable = `
  CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    attributes JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );

  -- Index for JSONB queries
  CREATE INDEX idx_attributes ON products USING gin(attributes);
`;

// Insert with JSONB
async function createProduct(name, price, attributes) {
  const query = `
    INSERT INTO products (name, price, attributes)
    VALUES ($1, $2, $3)
    RETURNING *
  `;

  const result = await db.query(query, [name, price, JSON.stringify(attributes)]);
  return result.rows[0];
}

// Query JSONB
async function searchProducts(brand) {
  const query = `
    SELECT *
    FROM products
    WHERE attributes->>'brand' = $1
  `;

  const result = await db.query(query, [brand]);
  return result.rows;
}

// Query nested JSONB
async function getProductsBySpec(key, value) {
  const query = `
    SELECT *
    FROM products
    WHERE attributes->>'specs'->>$1 = $2
  `;

  const result = await db.query(query, [key, value]);
  return result.rows;
}

// Update JSONB (merge)
async function updateProductAttributes(id, newAttributes) {
  const query = `
    UPDATE products
    SET attributes = attributes || $1::jsonb
    WHERE id = $2
    RETURNING *
  `;

  const result = await db.query(query, [JSON.stringify(newAttributes), id]);
  return result.rows[0];
}

// Query JSONB array
async function getProductsWithTag(tag) {
  const query = `
    SELECT *
    FROM products
    WHERE attributes->'tags' ? $1
  `;

  const result = await db.query(query, [tag]);
  return result.rows;
}
```

### 2. Array Data Type

```javascript
// Create table with array
const createPostsTable = `
  CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    tags TEXT[],
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );

  -- Index for array queries
  CREATE INDEX idx_tags ON posts USING gin(tags);
`;

// Insert with array
async function createPost(title, content, tags) {
  const query = `
    INSERT INTO posts (title, content, tags)
    VALUES ($1, $2, $3)
    RETURNING *
  `;

  const result = await db.query(query, [title, content, tags]);
  return result.rows[0];
}

// Query array (contains element)
async function getPostsByTag(tag) {
  const query = `
    SELECT *
    FROM posts
    WHERE $1 = ANY(tags)
  `;

  const result = await db.query(query, [tag]);
  return result.rows;
}

// Query array (contains all elements)
async function getPostsByTags(tags) {
  const query = `
    SELECT *
    FROM posts
    WHERE tags @> $1::text[]
  `;

  const result = await db.query(query, [tags]);
  return result.rows;
}
```

### 3. Full-Text Search

```javascript
// Create table with tsvector
const createArticlesTable = `
  CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    search_vector tsvector
  );

  -- Create GIN index for full-text search
  CREATE INDEX idx_search ON articles USING gin(search_vector);

  -- Trigger to automatically update search_vector
  CREATE OR REPLACE FUNCTION update_search_vector() RETURNS trigger AS $$
  BEGIN
    NEW.search_vector := to_tsvector('english', COALESCE(NEW.title, '') || ' ' || COALESCE(NEW.content, ''));
    RETURN NEW;
  END;
  $$ LANGUAGE plpgsql;

  CREATE TRIGGER trig_update_search
  BEFORE INSERT OR UPDATE ON articles
  FOR EACH ROW
  EXECUTE FUNCTION update_search_vector();
`;

// Full-text search
async function searchArticles(searchTerm) {
  const query = `
    SELECT
      id,
      title,
      ts_rank(search_vector, query) AS rank
    FROM articles,
         to_tsquery('english', $1) query
    WHERE search_vector @@ query
    ORDER BY rank DESC
    LIMIT 10
  `;

  // Convert search term to tsquery format
  const tsquery = searchTerm.split(' ').join(' & ');
  const result = await db.query(query, [tsquery]);
  return result.rows;
}
```

### 4. Window Functions

```javascript
// Rank users by post count
async function getUsersRankedByPosts() {
  const query = `
    SELECT
      u.id,
      u.name,
      COUNT(p.id) as post_count,
      RANK() OVER (ORDER BY COUNT(p.id) DESC) as rank
    FROM users u
    LEFT JOIN posts p ON u.id = p.author_id
    GROUP BY u.id, u.name
    ORDER BY rank
  `;

  const result = await db.query(query);
  return result.rows;
}

// Running total of orders
async function getOrdersWithRunningTotal() {
  const query = `
    SELECT
      id,
      order_date,
      amount,
      SUM(amount) OVER (ORDER BY order_date) as running_total
    FROM orders
    ORDER BY order_date
  `;

  const result = await db.query(query);
  return result.rows;
}

// Moving average
async function getSalesWithMovingAverage() {
  const query = `
    SELECT
      sale_date,
      amount,
      AVG(amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
      ) as moving_avg_7days
    FROM sales
    ORDER BY sale_date
  `;

  const result = await db.query(query);
  return result.rows;
}
```

### 5. Common Table Expressions (CTEs)

```javascript
// Simple CTE
async function getTopAuthorsWithPosts() {
  const query = `
    WITH author_stats AS (
      SELECT
        u.id,
        u.name,
        COUNT(p.id) as post_count
      FROM users u
      LEFT JOIN posts p ON u.id = p.author_id
      GROUP BY u.id, u.name
    )
    SELECT * FROM author_stats
    WHERE post_count > 5
    ORDER BY post_count DESC
  `;

  const result = await db.query(query);
  return result.rows;
}

// Recursive CTE (organizational hierarchy)
async function getEmployeeHierarchy(managerId) {
  const query = `
    WITH RECURSIVE employee_tree AS (
      -- Base case: top-level manager
      SELECT id, name, manager_id, 0 as level, name as path
      FROM employees
      WHERE manager_id IS NULL

      UNION ALL

      -- Recursive case: employees under each manager
      SELECT e.id, e.name, e.manager_id, et.level + 1, et.path || ' > ' || e.name
      FROM employees e
      INNER JOIN employee_tree et ON e.manager_id = et.id
    )
    SELECT * FROM employee_tree
    ORDER BY level, name
  `;

  const result = await db.query(query);
  return result.rows;
}
```

### 6. UUID Primary Keys

```javascript
// Enable UUID extension
const setupUUID = `
  CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
`;

// Create table with UUID
const createUsersTable = `
  CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );
`;

// Insert with UUID
async function createUser(email, name) {
  const query = `
    INSERT INTO users (email, name)
    VALUES ($1, $2)
    RETURNING *
  `;

  const result = await db.query(query, [email, name]);
  return result.rows[0];
}
```

---

## 🔄 Transactions

### Basic Transaction

```javascript
async function transferFunds(fromUserId, toUserId, amount) {
  const client = await db.pool.connect();

  try {
    await client.query('BEGIN');

    // Deduct from sender
    const deductQuery = `
      UPDATE accounts
      SET balance = balance - $1
      WHERE user_id = $2 AND balance >= $1
      RETURNING balance
    `;
    const deductResult = await client.query(deductQuery, [amount, fromUserId]);

    if (deductResult.rowCount === 0) {
      throw new Error('Insufficient funds');
    }

    // Add to recipient
    const addQuery = `
      UPDATE accounts
      SET balance = balance + $1
      WHERE user_id = $2
      RETURNING balance
    `;
    await client.query(addQuery, [amount, toUserId]);

    // Record transaction
    const recordQuery = `
      INSERT INTO transactions (from_user_id, to_user_id, amount)
      VALUES ($1, $2, $3)
    `;
    await client.query(recordQuery, [fromUserId, toUserId, amount]);

    await client.query('COMMIT');

    return { success: true, message: 'Transfer completed' };
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

### Transaction Isolation Levels

```javascript
async function updateWithIsolation() {
  const client = await db.pool.connect();

  try {
    // Set isolation level
    await client.query('BEGIN ISOLATION LEVEL SERIALIZABLE');

    // Perform operations
    const result = await client.query('SELECT * FROM accounts WHERE id = $1 FOR UPDATE', [1]);

    // Update logic
    await client.query('UPDATE accounts SET balance = balance + 100 WHERE id = $1', [1]);

    await client.query('COMMIT');
    return result.rows[0];
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

---

## 🚀 Performance Optimization

### Indexing Strategies

```sql
-- B-tree index (default) - good for equality and range queries
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_created ON posts(created_at DESC);

-- Partial index - index only subset of data
CREATE INDEX idx_published_posts ON posts(created_at)
WHERE published = true;

-- Composite index - multiple columns
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);

-- GIN index - for JSONB, arrays, full-text search
CREATE INDEX idx_attributes ON products USING gin(attributes);
CREATE INDEX idx_tags ON posts USING gin(tags);

-- GiST index - for geometric data and full-text search
CREATE INDEX idx_location ON places USING gist(location);
```

### EXPLAIN ANALYZE

```javascript
async function analyzeQuery() {
  const query = `
    EXPLAIN ANALYZE
    SELECT p.*, u.name as author_name
    FROM posts p
    INNER JOIN users u ON p.author_id = u.id
    WHERE p.published = true
    ORDER BY p.created_at DESC
    LIMIT 20
  `;

  const result = await db.query(query);
  console.log(result.rows);
  return result.rows;
}
```

### Materialized Views

```javascript
// Create materialized view
const createMaterializedView = `
  CREATE MATERIALIZED VIEW user_stats AS
  SELECT
    u.id,
    u.name,
    COUNT(DISTINCT p.id) as post_count,
    COUNT(DISTINCT c.id) as comment_count,
    MAX(p.created_at) as last_post_date
  FROM users u
  LEFT JOIN posts p ON u.id = p.author_id
  LEFT JOIN comments c ON u.id = c.user_id
  GROUP BY u.id, u.name;

  -- Create index on materialized view
  CREATE INDEX idx_user_stats_id ON user_stats(id);
`;

// Refresh materialized view
async function refreshUserStats() {
  await db.query('REFRESH MATERIALIZED VIEW user_stats');
}

// Query materialized view
async function getUserStats(userId) {
  const query = 'SELECT * FROM user_stats WHERE id = $1';
  const result = await db.query(query, [userId]);
  return result.rows[0];
}
```

### Query Optimization Tips

```javascript
// ❌ BAD: N+1 query problem
async function getPostsWithAuthorsBad() {
  const posts = await db.query('SELECT * FROM posts');

  for (const post of posts.rows) {
    const author = await db.query('SELECT * FROM users WHERE id = $1', [post.author_id]);
    post.author = author.rows[0];
  }

  return posts.rows;
}

// ✅ GOOD: Single JOIN query
async function getPostsWithAuthorsGood() {
  const query = `
    SELECT
      p.*,
      json_build_object('id', u.id, 'name', u.name, 'email', u.email) as author
    FROM posts p
    INNER JOIN users u ON p.author_id = u.id
  `;

  const result = await db.query(query);
  return result.rows;
}

// ✅ GOOD: Use SELECT * only when needed
async function getUsersOptimized() {
  // Select only needed columns
  const query = 'SELECT id, name, email FROM users';
  const result = await db.query(query);
  return result.rows;
}
```

---

## 📚 Interview Questions

### Q1: What is the difference between JSON and JSONB in PostgreSQL?

**Answer:**

**JSON:**
- Stores exact text representation of JSON
- Faster to insert (no processing)
- Slower to query (needs parsing each time)
- Preserves whitespace and key order
- No indexing support

**JSONB (Binary JSON):**
- Stores decomposed binary format
- Slower to insert (needs processing)
- Much faster to query (already parsed)
- Does not preserve whitespace or key order
- Supports indexing with GIN indexes
- Removes duplicate keys

**Recommendation:** Use JSONB for most cases due to better query performance and indexing capabilities.

**Example:**
```javascript
// Create table with JSONB
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(200),
  attributes JSONB
);

// Index JSONB for fast queries
CREATE INDEX idx_attributes ON products USING gin(attributes);

// Query JSONB
SELECT * FROM products WHERE attributes->>'brand' = 'Apple';
SELECT * FROM products WHERE attributes->'specs'->>'ram' = '16GB';
```

---

### Q2: Explain PostgreSQL indexes and when to use them.

**Answer:**

**Index Types:**

**1. B-tree (default)** - Best for equality and range queries
```sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_date ON posts(created_at);

-- Use for: WHERE, ORDER BY, <, >, <=, >=, =
SELECT * FROM users WHERE email = 'test@example.com';
SELECT * FROM posts WHERE created_at > '2024-01-01' ORDER BY created_at;
```

**2. GIN (Generalized Inverted Index)** - For full-text search, JSONB, arrays
```sql
CREATE INDEX idx_jsonb ON products USING gin(attributes);
CREATE INDEX idx_array ON posts USING gin(tags);

-- Use for JSONB and array queries
SELECT * FROM products WHERE attributes @> '{"brand": "Apple"}';
SELECT * FROM posts WHERE tags @> ARRAY['postgresql', 'database'];
```

**3. Partial Index** - Index only subset of rows
```sql
CREATE INDEX idx_active_users ON users(created_at) WHERE status = 'active';

-- More efficient than full index if querying specific subset frequently
SELECT * FROM users WHERE status = 'active' ORDER BY created_at;
```

**4. Composite Index** - Multiple columns
```sql
CREATE INDEX idx_posts_author_date ON posts(author_id, created_at DESC);

-- Use for queries with multiple WHERE conditions
SELECT * FROM posts WHERE author_id = 123 ORDER BY created_at DESC;
```

**When to Use Indexes:**
- ✅ Columns frequently used in WHERE clauses
- ✅ Columns used in JOIN conditions
- ✅ Columns used in ORDER BY
- ✅ Foreign keys
- ❌ Small tables (< 1000 rows)
- ❌ Columns that are frequently updated
- ❌ Columns with low cardinality (few unique values)

---

### Q3: What are CTEs and when would you use them?

**Answer:**

**CTE (Common Table Expression)** is a temporary named result set that exists only during query execution. Defined using `WITH` clause.

**Benefits:**
- Improves query readability
- Allows recursive queries
- Can be referenced multiple times
- Simplifies complex queries

**Example 1: Basic CTE**
```sql
WITH high_value_customers AS (
  SELECT user_id, SUM(total) as total_spent
  FROM orders
  GROUP BY user_id
  HAVING SUM(total) > 10000
)
SELECT u.name, h.total_spent
FROM users u
INNER JOIN high_value_customers h ON u.id = h.user_id
ORDER BY h.total_spent DESC;
```

**Example 2: Multiple CTEs**
```sql
WITH
  monthly_sales AS (
    SELECT DATE_TRUNC('month', order_date) as month, SUM(total) as total
    FROM orders
    GROUP BY month
  ),
  avg_sales AS (
    SELECT AVG(total) as avg_monthly
    FROM monthly_sales
  )
SELECT m.month, m.total, a.avg_monthly
FROM monthly_sales m
CROSS JOIN avg_sales a
WHERE m.total > a.avg_monthly;
```

**Example 3: Recursive CTE (Hierarchy)**
```sql
-- Employee organizational chart
WITH RECURSIVE org_chart AS (
  -- Base case: CEO (no manager)
  SELECT id, name, manager_id, 0 as level
  FROM employees
  WHERE manager_id IS NULL

  UNION ALL

  -- Recursive case: employees under each manager
  SELECT e.id, e.name, e.manager_id, oc.level + 1
  FROM employees e
  INNER JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level, name;
```

**When to Use:**
- Complex queries with repeated subqueries
- Hierarchical data (organizational charts, comment trees)
- Graph traversal
- Breaking down complex logic into readable steps

---

### Q4: How do you handle transactions and what are isolation levels?

**Answer:**

**Transactions** ensure atomicity, consistency, isolation, and durability (ACID). All operations succeed together or fail together.

**Basic Transaction:**
```javascript
const client = await pool.connect();
try {
  await client.query('BEGIN');

  await client.query('UPDATE accounts SET balance = balance - 100 WHERE id = 1');
  await client.query('UPDATE accounts SET balance = balance + 100 WHERE id = 2');

  await client.query('COMMIT');
} catch (error) {
  await client.query('ROLLBACK');
  throw error;
} finally {
  client.release();
}
```

**Isolation Levels** (from weakest to strongest):

**1. READ UNCOMMITTED** (not supported in PostgreSQL)
- Can read uncommitted changes from other transactions
- Dirty reads possible

**2. READ COMMITTED** (PostgreSQL default)
- Only sees committed data
- Different reads might return different results
- Prevents dirty reads
```sql
BEGIN ISOLATION LEVEL READ COMMITTED;
```

**3. REPEATABLE READ**
- Ensures same query returns same results within transaction
- Prevents non-repeatable reads
- Phantom reads still possible
```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
```

**4. SERIALIZABLE** (strongest)
- Complete isolation, as if transactions run sequentially
- Prevents all anomalies
- May cause more transaction conflicts
```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
```

**Example: Bank Transfer with Lock**
```javascript
async function transferMoney(fromId, toId, amount) {
  const client = await pool.connect();

  try {
    await client.query('BEGIN ISOLATION LEVEL SERIALIZABLE');

    // Lock rows to prevent concurrent modifications
    const fromAccount = await client.query(
      'SELECT balance FROM accounts WHERE id = $1 FOR UPDATE',
      [fromId]
    );

    if (fromAccount.rows[0].balance < amount) {
      throw new Error('Insufficient funds');
    }

    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
      [amount, fromId]
    );

    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
      [amount, toId]
    );

    await client.query('COMMIT');
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

---

### Q5: What are the advantages of PostgreSQL over MySQL?

**Answer:**

**1. Data Types**
- **PostgreSQL**: JSONB, Arrays, UUID, Geometric types, Network types, Custom types
- **MySQL**: Limited (basic JSON support, no arrays)

**2. SQL Standard Compliance**
- **PostgreSQL**: Strict SQL standard compliance
- **MySQL**: Less strict, many proprietary extensions

**3. JSONB Support**
- **PostgreSQL**: Native JSONB with indexing (GIN)
- **MySQL**: JSON support but slower, no binary format

**4. Full-Text Search**
- **PostgreSQL**: Built-in, powerful full-text search with tsvector
- **MySQL**: Basic FULLTEXT index, less flexible

**5. Advanced Features**
- **PostgreSQL**: Window functions, CTEs, recursive queries, materialized views, table inheritance
- **MySQL**: Limited support for some features

**6. Concurrency**
- **PostgreSQL**: MVCC (Multi-Version Concurrency Control) - better for high-concurrency workloads
- **MySQL InnoDB**: Row-level locking, but less sophisticated

**7. Extensibility**
- **PostgreSQL**: Can create custom data types, functions, operators, extensions (PostGIS, pg_trgm)
- **MySQL**: Limited extensibility

**8. Complex Queries**
- **PostgreSQL**: Better optimizer for complex queries with multiple joins
- **MySQL**: Simpler optimizer, may struggle with complex queries

**When to Choose PostgreSQL:**
- Complex queries and analytics
- JSONB/NoSQL-like features needed
- Full-text search
- Advanced data types
- Strict data integrity
- Open-source with permissive license

**When to Choose MySQL:**
- Simple read-heavy applications
- Replication is priority
- Existing ecosystem/tools
- Simpler deployment

**Comparison:**
```javascript
// PostgreSQL: JSONB query
SELECT * FROM products
WHERE attributes @> '{"brand": "Apple", "color": "silver"}';

// PostgreSQL: Array query
SELECT * FROM posts WHERE tags @> ARRAY['postgresql', 'database'];

// PostgreSQL: Full-text search
SELECT * FROM articles
WHERE search_vector @@ to_tsquery('postgresql & performance');

// These features don't exist or are much more limited in MySQL
```

---

### Q6: How do you optimize slow queries in PostgreSQL?

**Answer:**

**Step-by-step optimization process:**

**1. Identify Slow Queries**
```sql
-- Enable query logging
ALTER DATABASE mydb SET log_min_duration_statement = 1000;  -- Log queries > 1s

-- Or use pg_stat_statements extension
CREATE EXTENSION pg_stat_statements;

SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;
```

**2. Use EXPLAIN ANALYZE**
```sql
EXPLAIN ANALYZE
SELECT p.*, u.name
FROM posts p
INNER JOIN users u ON p.author_id = u.id
WHERE p.published = true
ORDER BY p.created_at DESC
LIMIT 20;
```

**Look for:**
- **Seq Scan** on large tables → Add index
- **High cost numbers** → Optimize query
- **Actual time vs planned** → Statistics might be outdated

**3. Add Appropriate Indexes**
```sql
-- Add index for WHERE clause
CREATE INDEX idx_posts_published ON posts(published);

-- Composite index for JOIN + WHERE
CREATE INDEX idx_posts_published_date ON posts(published, created_at DESC);

-- Partial index for frequently queried subset
CREATE INDEX idx_active_posts ON posts(created_at)
WHERE published = true;
```

**4. Optimize Query Structure**
```javascript
// ❌ BAD: N+1 queries
async function getPostsBad() {
  const posts = await db.query('SELECT * FROM posts LIMIT 20');

  for (const post of posts.rows) {
    const author = await db.query('SELECT * FROM users WHERE id = $1', [post.author_id]);
    post.author = author.rows[0];
  }
}

// ✅ GOOD: Single JOIN query
async function getPostsGood() {
  const query = `
    SELECT p.*, u.name as author_name
    FROM posts p
    INNER JOIN users u ON p.author_id = u.id
    LIMIT 20
  `;
  return await db.query(query);
}
```

**5. Use Connection Pooling**
```javascript
// ✅ Connection pool reduces connection overhead
const pool = new Pool({
  max: 20,  // Maximum connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
});
```

**6. Pagination**
```javascript
// ❌ BAD: OFFSET is slow for large offsets
SELECT * FROM posts ORDER BY created_at LIMIT 20 OFFSET 10000;

// ✅ GOOD: Cursor-based pagination
SELECT * FROM posts
WHERE created_at < $1  -- Last seen timestamp
ORDER BY created_at DESC
LIMIT 20;
```

**7. Materialized Views**
```sql
-- For expensive aggregations
CREATE MATERIALIZED VIEW user_stats AS
SELECT user_id, COUNT(*) as post_count, AVG(views) as avg_views
FROM posts
GROUP BY user_id;

-- Refresh periodically
REFRESH MATERIALIZED VIEW user_stats;
```

**8. Update Statistics**
```sql
ANALYZE posts;  -- Update statistics for query planner
VACUUM ANALYZE; -- Reclaim space and update statistics
```

**9. Denormalization (if needed)**
```javascript
// Store computed values for fast access
ALTER TABLE posts ADD COLUMN comment_count INTEGER DEFAULT 0;

// Update with trigger or application logic
CREATE OR REPLACE FUNCTION update_comment_count()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE posts SET comment_count = comment_count + 1
  WHERE id = NEW.post_id;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

---

## ✅ Best Practices

### Do's ✅

1. **Use connection pooling**
```javascript
const pool = new Pool({ max: 20 });
```

2. **Use parameterized queries** (prevent SQL injection)
```javascript
// ✅ GOOD
await db.query('SELECT * FROM users WHERE email = $1', [email]);

// ❌ BAD - SQL injection risk
await db.query(`SELECT * FROM users WHERE email = '${email}'`);
```

3. **Use transactions for related operations**
```javascript
await client.query('BEGIN');
// Multiple related queries
await client.query('COMMIT');
```

4. **Index foreign keys and frequently queried columns**
```sql
CREATE INDEX idx_posts_author ON posts(author_id);
CREATE INDEX idx_users_email ON users(email);
```

5. **Use JSONB instead of JSON**
```sql
CREATE TABLE products (attributes JSONB);
CREATE INDEX idx_attrs ON products USING gin(attributes);
```

6. **Analyze query performance**
```sql
EXPLAIN ANALYZE SELECT * FROM posts WHERE author_id = 123;
```

7. **Use appropriate data types**
```sql
-- UUID for distributed systems
id UUID PRIMARY KEY DEFAULT uuid_generate_v4()

-- TIMESTAMP for dates
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

-- DECIMAL for money
price DECIMAL(10,2)
```

### Don'ts ❌

1. **Don't use SELECT * in production**
```javascript
// ❌ BAD
SELECT * FROM users;

// ✅ GOOD
SELECT id, email, name FROM users;
```

2. **Don't forget to close/release connections**
```javascript
// ❌ BAD - connection leak
const client = await pool.connect();
await client.query('SELECT * FROM users');

// ✅ GOOD
const client = await pool.connect();
try {
  await client.query('SELECT * FROM users');
} finally {
  client.release();
}
```

3. **Don't store sensitive data in plain text**
```javascript
// ✅ Hash passwords
const hashedPassword = await bcrypt.hash(password, 10);
```

4. **Don't ignore database constraints**
```sql
-- Use constraints for data integrity
email VARCHAR(100) UNIQUE NOT NULL,
age INTEGER CHECK (age >= 0 AND age <= 120),
status VARCHAR(20) CHECK (status IN ('active', 'inactive', 'banned'))
```

5. **Don't use OFFSET for large pagination**
```javascript
// ❌ BAD - Slow for large offsets
LIMIT 20 OFFSET 100000

// ✅ GOOD - Cursor-based
WHERE id > $1 LIMIT 20
```

---

## 🎯 Summary

- **PostgreSQL** is a powerful, feature-rich RDBMS with advanced capabilities
- **JSONB** provides NoSQL flexibility with SQL reliability
- **Indexes** are crucial for query performance (B-tree, GIN, partial, composite)
- **Transactions** ensure data consistency with ACID properties
- **CTEs** improve query readability and enable recursive queries
- **Window functions** enable advanced analytics without subqueries
- **Connection pooling** reduces overhead and improves performance
- **Parameterized queries** prevent SQL injection
- **EXPLAIN ANALYZE** is essential for query optimization

---

[← Previous: Transactions](./04-transactions.md) | [Next: ORMs →](./06-orms.md)
