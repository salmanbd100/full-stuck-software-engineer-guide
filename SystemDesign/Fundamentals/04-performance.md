# Performance Optimization

## Overview
Performance measures how fast a system responds to requests and handles load. Good performance directly impacts user satisfaction, conversion rates, and operational costs. Every 100ms delay can reduce conversions by 7%.

## Key Performance Metrics

### Latency

Time between request and response.

**Types:**
- **Network Latency**: Time for data to travel across network
- **Processing Latency**: Time for server to process request
- **Database Latency**: Time for database query
- **Total Latency**: Sum of all latencies

**Typical Latencies:**
```
L1 Cache:           0.5 ns
L2 Cache:           7 ns
RAM:                100 ns
SSD:                150 µs
Network (same DC):  0.5 ms
HDD:                10 ms
Network (cross-DC): 50-100 ms
Network (cross-continent): 100-300 ms
```

**P50, P95, P99:**
```
P50 (Median):    50% of requests faster than this
P95:             95% of requests faster than this
P99:             99% of requests faster than this

Example:
P50 = 100ms  → Half of users see < 100ms
P95 = 500ms  → 5% of users see > 500ms (slow outliers)
P99 = 2000ms → 1% of users see > 2s (very slow)
```

### Throughput

Requests processed per unit of time.

**Measurements:**
- **QPS (Queries Per Second)**: Database queries
- **RPS (Requests Per Second)**: API requests
- **TPS (Transactions Per Second)**: Business transactions

**Example:**
```
Server handles 10,000 requests/second
Average latency: 50ms

Concurrent requests = Throughput × Latency
                    = 10,000 × 0.05
                    = 500 concurrent requests
```

### Bandwidth

Amount of data transferred per unit of time.

```
Bandwidth = Data Size × Number of Requests

Example:
API response: 100 KB
1000 requests/second
Bandwidth = 100 KB × 1000 = 100 MB/s
```

## Database Performance

### Indexing

Speeds up data retrieval at cost of slower writes.

**Without Index:**
```sql
-- Full table scan - O(n)
SELECT * FROM users WHERE email = 'john@example.com';
-- Scans all 1 million rows
```

**With Index:**
```sql
-- Index lookup - O(log n)
CREATE INDEX idx_email ON users(email);

SELECT * FROM users WHERE email = 'john@example.com';
-- Uses B-tree index, scans ~20 rows
```

**Types of Indexes:**

1. **Single Column Index**
   ```sql
   CREATE INDEX idx_email ON users(email);
   ```

2. **Composite Index**
   ```sql
   CREATE INDEX idx_name_age ON users(last_name, first_name, age);

   -- Fast (uses index)
   SELECT * FROM users WHERE last_name = 'Smith';
   SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John';

   -- Slow (doesn't use index - wrong order)
   SELECT * FROM users WHERE first_name = 'John';
   ```

3. **Unique Index**
   ```sql
   CREATE UNIQUE INDEX idx_email ON users(email);
   ```

4. **Full-Text Index**
   ```sql
   CREATE FULLTEXT INDEX idx_content ON posts(content);

   SELECT * FROM posts WHERE MATCH(content) AGAINST('search term');
   ```

**Index Trade-offs:**
- ✅ Faster reads
- ❌ Slower writes
- ❌ More storage
- ❌ Index maintenance overhead

### Query Optimization

**Use EXPLAIN to analyze queries:**
```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 123;

-- Shows:
-- - Whether index is used
-- - Number of rows scanned
-- - Query execution plan
```

**Optimization Techniques:**

1. **Select Only Needed Columns**
   ```sql
   -- ❌ Bad - retrieves unnecessary data
   SELECT * FROM users;

   -- ✅ Good - retrieves only what's needed
   SELECT id, name, email FROM users;
   ```

2. **Avoid N+1 Queries**
   ```javascript
   // ❌ Bad - N+1 queries
   const users = await db.query('SELECT * FROM users');
   for (const user of users) {
     const orders = await db.query(
       'SELECT * FROM orders WHERE user_id = ?',
       [user.id]
     );
   }

   // ✅ Good - 2 queries with JOIN
   const usersWithOrders = await db.query(`
     SELECT users.*, orders.*
     FROM users
     LEFT JOIN orders ON users.id = orders.user_id
   `);
   ```

3. **Use Pagination**
   ```sql
   -- ❌ Bad - loads all records
   SELECT * FROM posts ORDER BY created_at DESC;

   -- ✅ Good - loads page at a time
   SELECT * FROM posts
   ORDER BY created_at DESC
   LIMIT 20 OFFSET 0;
   ```

4. **Avoid Wildcard Prefix**
   ```sql
   -- ❌ Bad - can't use index
   SELECT * FROM users WHERE name LIKE '%john%';

   -- ✅ Good - can use index
   SELECT * FROM users WHERE name LIKE 'john%';
   ```

### Connection Pooling

Reuse database connections instead of creating new ones.

```javascript
// ❌ Bad - creates new connection each time
async function getUser(id) {
  const connection = await mysql.createConnection(config);
  const user = await connection.query('SELECT * FROM users WHERE id = ?', [id]);
  await connection.end();
  return user;
}

// ✅ Good - reuses connections from pool
const pool = mysql.createPool({
  host: 'localhost',
  user: 'root',
  database: 'mydb',
  connectionLimit: 10  // Max 10 concurrent connections
});

async function getUser(id) {
  const connection = await pool.getConnection();
  const user = await connection.query('SELECT * FROM users WHERE id = ?', [id]);
  connection.release();  // Return to pool
  return user;
}
```

**Benefits:**
- Avoid connection overhead (100-200ms per connection)
- Limit concurrent connections to database
- Better resource utilization

## Caching Strategies

### Cache Layers

**Multi-level caching:**
```
Browser Cache (HTTP Cache-Control)
    ↓
CDN Cache (CloudFront, Cloudflare)
    ↓
Application Cache (Redis, Memcached)
    ↓
Database Query Cache
    ↓
Database
```

### Cache Hit Ratio

Percentage of requests served from cache.

```
Hit Ratio = Cache Hits / Total Requests × 100%

Example:
1000 requests
900 served from cache
100 from database

Hit Ratio = 900/1000 × 100% = 90%
```

**Target:** 80%+ hit ratio for most applications

### Cache Eviction Policies

**LRU (Least Recently Used):**
```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }

  get(key) {
    if (!this.cache.has(key)) return null;

    // Move to end (most recently used)
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);

    return value;
  }

  put(key, value) {
    // Remove if exists
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }

    // Add to end
    this.cache.set(key, value);

    // Evict oldest if over capacity
    if (this.cache.size > this.capacity) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
  }
}
```

**Other Policies:**
- **LFU (Least Frequently Used)**: Remove least accessed
- **FIFO**: Remove oldest added
- **Random**: Remove random entry

### Cache Invalidation

**Strategies:**

1. **TTL (Time To Live)**
   ```javascript
   // Cache for 5 minutes
   redis.set('user:123', userData, 'EX', 300);
   ```

2. **Write-Through (Update on Write)**
   ```javascript
   async function updateUser(id, data) {
     // Update DB
     await db.query('UPDATE users SET ... WHERE id = ?', [id]);

     // Update cache
     await redis.set(`user:${id}`, JSON.stringify(data));
   }
   ```

3. **Cache-Aside with Invalidation**
   ```javascript
   async function updateUser(id, data) {
     // Update DB
     await db.query('UPDATE users SET ... WHERE id = ?', [id]);

     // Invalidate cache
     await redis.del(`user:${id}`);
   }
   ```

## Content Delivery Network (CDN)

### How CDN Works

```
User in Tokyo → CDN Tokyo → Origin (US)
                  ↓
              Cached Content
                (fast)

User in London → CDN London → Origin (US)
                   ↓
               Cached Content
                 (fast)
```

**Benefits:**
- **Reduced Latency**: Serve from nearest location
- **Reduced Load**: Less traffic to origin
- **DDoS Protection**: Distributed infrastructure
- **Cost Savings**: Cheaper bandwidth

### Cache-Control Headers

```javascript
// Express.js
app.use('/static', express.static('public', {
  maxAge: '1y',  // Cache for 1 year
  immutable: true
}));

// Response headers
res.setHeader('Cache-Control', 'public, max-age=31536000, immutable');
```

**Cache-Control Directives:**
```
no-cache:   Validate with server before using cache
no-store:   Never cache
public:     Can be cached by CDN
private:    Only cache in browser
max-age:    Cache for N seconds
immutable:  Content never changes (versioned files)
```

## Application Performance

### Lazy Loading

Load resources only when needed.

```javascript
// Code splitting
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}

// Image lazy loading
<img src="large-image.jpg" loading="lazy" alt="Lazy loaded" />
```

### Compression

Reduce data size before transmission.

**Gzip/Brotli:**
```javascript
// Express.js
const compression = require('compression');
app.use(compression());

// Reduces text files by 70-90%
```

**Response Sizes:**
```
Original:    500 KB
Gzip:        100 KB (80% reduction)
Brotli:      85 KB (83% reduction)
```

### Bundle Optimization

**Code Splitting:**
```javascript
// Webpack configuration
module.exports = {
  optimization: {
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
          priority: 5
        }
      }
    }
  }
};
```

**Tree Shaking:**
```javascript
// ❌ Bad - imports entire library
import _ from 'lodash';

// ✅ Good - imports only what's needed
import { debounce } from 'lodash-es';
```

### Asynchronous Processing

Move slow tasks to background.

```javascript
// ❌ Bad - blocks request
app.post('/register', async (req, res) => {
  await createUser(req.body);
  await sendWelcomeEmail(req.body.email);  // Slow!
  await generateReport(req.body);           // Slow!
  res.json({ success: true });
});

// ✅ Good - responds immediately
app.post('/register', async (req, res) => {
  await createUser(req.body);

  // Queue background tasks
  queue.enqueue('send_email', { email: req.body.email });
  queue.enqueue('generate_report', { userId: user.id });

  res.json({ success: true });  // Fast response
});
```

## Load Balancing for Performance

### Algorithm Selection

**Round Robin:**
- Simple, even distribution
- Doesn't account for server load

**Least Connections:**
- Routes to server with fewest active connections
- Better for long-running requests

**Weighted Round Robin:**
- Distribute based on server capacity
- Use for heterogeneous servers

**Least Response Time:**
- Routes to fastest responding server
- Best overall performance

## Monitoring Performance

### Key Metrics to Track

1. **Response Time**
   - Average, P50, P95, P99
   - By endpoint
   - By region

2. **Throughput**
   - Requests per second
   - By endpoint
   - Peak vs average

3. **Error Rate**
   - 4xx errors (client errors)
   - 5xx errors (server errors)
   - By endpoint

4. **Resource Utilization**
   - CPU usage
   - Memory usage
   - Disk I/O
   - Network bandwidth

### Application Performance Monitoring (APM)

```javascript
// Track slow queries
const startTime = Date.now();
const result = await db.query('SELECT ...');
const duration = Date.now() - startTime;

if (duration > 1000) {
  logger.warn('Slow query detected', {
    query: 'SELECT ...',
    duration,
    threshold: 1000
  });
}

// Track API latency
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;

    metrics.record({
      endpoint: req.path,
      method: req.method,
      status: res.statusCode,
      duration
    });
  });

  next();
});
```

## Interview Questions

**Q: How do you optimize a slow API endpoint?**

A: Systematic approach:

1. **Measure**: Use APM to identify bottleneck
   - Database query slow?
   - External API slow?
   - CPU-intensive computation?

2. **Database Optimization**:
   - Add indexes
   - Optimize queries
   - Use connection pooling

3. **Caching**:
   - Cache database queries
   - Cache API responses
   - Use CDN for static content

4. **Code Optimization**:
   - Remove unnecessary operations
   - Use efficient algorithms
   - Parallel processing where possible

5. **Async Processing**:
   - Move slow tasks to background queue

**Q: What's the difference between latency and throughput?**

A:
- **Latency**: Time for single request (ms)
  - Example: API responds in 50ms

- **Throughput**: Requests processed per time (req/s)
  - Example: Server handles 1000 req/s

**Relationship:**
```
Throughput = Concurrency / Latency

Example:
500 concurrent connections
50ms average latency
Throughput = 500 / 0.05 = 10,000 req/s
```

**Q: How do you handle cache invalidation?**

A: "There are only two hard things in Computer Science: cache invalidation and naming things."

**Strategies:**

1. **TTL**: Expire after time
   - Simple, eventually consistent
   - Good for data that changes regularly

2. **Write-Through**: Update on write
   - Always consistent
   - Write latency increased

3. **Invalidate on Write**: Delete on write
   - Simple, next read updates cache
   - Brief inconsistency window

4. **Event-Based**: Invalidate via pub/sub
   - Good for distributed systems
   - More complex

**Q: What causes high database CPU usage?**

A: Common causes:

1. **Missing Indexes**: Full table scans
2. **Inefficient Queries**: Complex joins, subqueries
3. **High QPS**: Too many queries
4. **Lock Contention**: Waiting for locks
5. **Large Result Sets**: Returning too much data

**Solutions:**
- Add appropriate indexes
- Optimize queries (use EXPLAIN)
- Implement caching
- Use read replicas
- Consider sharding

## Best Practices

**Database:**
✅ Index frequently queried columns
✅ Use connection pooling
✅ Implement read replicas for read-heavy workloads
✅ Cache frequently accessed data
✅ Monitor slow queries

**Caching:**
✅ Cache at multiple levels
✅ Set appropriate TTLs
✅ Monitor hit ratios
✅ Implement cache warming
✅ Handle cache failures gracefully

**Application:**
✅ Use CDN for static assets
✅ Compress responses
✅ Lazy load resources
✅ Code splitting
✅ Async processing for slow tasks

**Monitoring:**
✅ Track P95, P99 latencies (not just averages)
✅ Set up alerts for anomalies
✅ Monitor resource utilization
✅ Regular performance testing
✅ APM tools (DataDog, New Relic)

## Summary

- **Performance** measured by latency, throughput, and bandwidth
- **P95/P99** metrics more important than averages (catch outliers)
- **Database indexing** dramatically improves query performance
- **Caching** at multiple levels reduces load and latency
- **CDN** reduces latency for geographically distributed users
- **Connection pooling** eliminates connection overhead
- **Async processing** keeps API responses fast
- **Monitor** key metrics and set alerts for degradation

---
[← Back to SystemDesign](../README.md)
