# System Design Interview Framework

## Overview
A structured framework helps you navigate system design interviews systematically, ensuring you cover all critical aspects while demonstrating strong communication and technical reasoning. This guide provides a step-by-step approach to tackle any system design question.

## Interview Structure

### Typical Timeline (45-60 minutes)

```
Requirements & Scope:           5-10 minutes
High-Level Design:              10-15 minutes
Deep Dive:                      15-20 minutes
Bottlenecks & Trade-offs:       5-10 minutes
Questions & Wrap-up:            5 minutes
```

**Key Principle:** Spend more time on breadth first, then depth based on interviewer's interest.

## RADIO Framework

The RADIO framework provides a structured approach to system design interviews.

### R - Requirements

Clarify functional and non-functional requirements before designing.

**Functional Requirements (What system does):**
- What features are we building?
- What are the core use cases?
- Who are the users?
- What actions can users perform?

**Non-Functional Requirements (How system performs):**
- **Scale:** How many users? Requests per second?
- **Performance:** Latency requirements?
- **Availability:** Uptime requirements (99.9%, 99.99%)?
- **Consistency:** Strong or eventual?
- **Durability:** Data loss acceptable?

**Questions to Ask:**
```
"How many daily active users are we expecting?"
"What's the read-to-write ratio?"
"Are there any latency requirements?"
"Do we need to support mobile clients?"
"What's the expected growth rate?"
"Any specific compliance requirements (GDPR, HIPAA)?"
```

**Example: Design Twitter**
```
Functional:
✓ Users can post tweets (280 chars)
✓ Users can follow other users
✓ Users view home timeline
✓ Users can like/retweet
✗ Direct messaging (out of scope)
✗ Trending topics (out of scope)

Non-Functional:
- 500M daily active users
- 100:1 read-to-write ratio
- Timeline latency < 200ms
- 99.9% availability
- Eventual consistency OK
```

### A - Architecture

Design high-level architecture with major components.

**Components to Consider:**
- Client applications (web, mobile)
- Load balancers
- Application servers
- Databases (SQL/NoSQL)
- Caches (Redis, Memcached)
- Message queues (Kafka, RabbitMQ)
- Object storage (S3)
- CDN

**Drawing the Architecture:**
```
┌──────────┐
│  Client  │
└────┬─────┘
     │
┌────▼──────────┐
│ Load Balancer │
└────┬──────────┘
     │
     ├────────────────────┐
     │                    │
┌────▼─────┐       ┌──────▼────┐
│ App      │       │ App       │
│ Server 1 │       │ Server 2  │
└────┬─────┘       └──────┬────┘
     │                    │
     └──────┬─────────────┘
            │
     ┌──────▼──────┐
     │   Cache     │
     │  (Redis)    │
     └──────┬──────┘
            │
     ┌──────▼──────┐
     │  Database   │
     │ (Primary)   │
     └──────┬──────┘
            │
     ┌──────▼──────┐
     │   Replicas  │
     └─────────────┘
```

**Talk Through the Flow:**
```
1. Client sends request to load balancer
2. Load balancer routes to available app server
3. App server checks cache for data
4. If cache miss, query database
5. Update cache with result
6. Return response to client
```

### D - Data Model

Design database schema and choose appropriate storage.

**Database Selection:**

**Use SQL when:**
- Structured data with relationships
- ACID transactions required
- Complex queries and joins
- Data integrity critical

**Use NoSQL when:**
- Flexible schema needed
- High write throughput
- Horizontal scaling required
- Simple query patterns

**Schema Design Example: Twitter**

**SQL Schema:**
```sql
-- Users table
CREATE TABLE users (
  user_id BIGINT PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP,
  INDEX idx_username (username)
);

-- Tweets table
CREATE TABLE tweets (
  tweet_id BIGINT PRIMARY KEY,
  user_id BIGINT NOT NULL,
  content VARCHAR(280),
  created_at TIMESTAMP,
  likes_count INT DEFAULT 0,
  retweets_count INT DEFAULT 0,
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  INDEX idx_user_created (user_id, created_at DESC)
);

-- Follows table (adjacency list)
CREATE TABLE follows (
  follower_id BIGINT,
  followee_id BIGINT,
  created_at TIMESTAMP,
  PRIMARY KEY (follower_id, followee_id),
  INDEX idx_followee (followee_id)
);
```

**NoSQL Schema (Cassandra):**
```
-- Timeline table (denormalized)
CREATE TABLE timeline (
  user_id BIGINT,
  tweet_id BIGINT,
  author_id BIGINT,
  content TEXT,
  created_at TIMESTAMP,
  PRIMARY KEY (user_id, created_at, tweet_id)
) WITH CLUSTERING ORDER BY (created_at DESC);
```

**Data Partitioning:**
```
Partition by user_id:
- Shard 0: users 0-999,999
- Shard 1: users 1M-1,999,999
- Shard 2: users 2M-2,999,999

Hash-based: shard = hash(user_id) % num_shards
```

### I - Interface (API Design)

Define API endpoints and contracts.

**REST API Design:**

```javascript
// User APIs
POST   /api/v1/users              // Create user
GET    /api/v1/users/{user_id}    // Get user profile
PUT    /api/v1/users/{user_id}    // Update user
DELETE /api/v1/users/{user_id}    // Delete user

// Tweet APIs
POST   /api/v1/tweets              // Create tweet
GET    /api/v1/tweets/{tweet_id}  // Get tweet
DELETE /api/v1/tweets/{tweet_id}  // Delete tweet
POST   /api/v1/tweets/{tweet_id}/like    // Like tweet
POST   /api/v1/tweets/{tweet_id}/retweet // Retweet

// Timeline APIs
GET    /api/v1/timelines/home?limit=20&offset=0  // Home timeline
GET    /api/v1/timelines/user/{user_id}?limit=20 // User timeline

// Follow APIs
POST   /api/v1/follows             // Follow user
DELETE /api/v1/follows/{user_id}   // Unfollow user
GET    /api/v1/follows/followers/{user_id}  // Get followers
GET    /api/v1/follows/following/{user_id}  // Get following
```

**Request/Response Examples:**

```javascript
// POST /api/v1/tweets
Request:
{
  "content": "Hello World!",
  "user_id": 12345
}

Response:
{
  "tweet_id": 98765,
  "user_id": 12345,
  "username": "john_doe",
  "content": "Hello World!",
  "created_at": "2024-01-15T10:30:00Z",
  "likes_count": 0,
  "retweets_count": 0
}

// GET /api/v1/timelines/home?limit=20
Response:
{
  "tweets": [
    {
      "tweet_id": 98765,
      "user_id": 12345,
      "username": "john_doe",
      "content": "Hello World!",
      "created_at": "2024-01-15T10:30:00Z",
      "likes_count": 100,
      "retweets_count": 20
    },
    // ... more tweets
  ],
  "next_offset": 20,
  "has_more": true
}
```

### O - Optimizations

Identify bottlenecks and propose optimizations.

**Common Optimizations:**

1. **Caching**
   ```
   Problem: Database queries slow
   Solution: Cache frequently accessed data in Redis

   Cache layers:
   - Browser cache (static assets)
   - CDN (images, videos)
   - Application cache (API responses)
   - Database query cache
   ```

2. **Database Optimization**
   ```
   Problem: Slow queries
   Solutions:
   - Add indexes on frequently queried columns
   - Use read replicas for read-heavy workloads
   - Implement database sharding
   - Denormalize data for faster reads
   ```

3. **Asynchronous Processing**
   ```
   Problem: Slow operations blocking requests
   Solution: Move to background queue

   Example:
   - Email notifications → Queue
   - Image processing → Queue
   - Analytics aggregation → Queue
   ```

4. **CDN for Static Assets**
   ```
   Problem: High bandwidth costs, slow asset delivery
   Solution: CloudFront/Cloudflare CDN

   Benefits:
   - Reduced latency (edge locations)
   - Lower bandwidth costs
   - DDoS protection
   ```

5. **Load Balancing**
   ```
   Problem: Uneven traffic distribution
   Solutions:
   - Application load balancer (Layer 7)
   - Geographic load balancing
   - Weighted round-robin
   ```

## Alternative Frameworks

### PEDALS Framework

**P - Problem Scoping**
- Clarify requirements
- Define success metrics

**E - Estimation**
- Calculate storage, bandwidth, QPS
- Determine scale

**D - Design**
- High-level architecture
- Component interactions

**A - API Design**
- Define endpoints
- Request/response formats

**L - Logic/Algorithms**
- Core algorithms
- Data structures

**S - Scalability**
- Bottlenecks
- Optimizations

### SNAKE Framework

**S - Scenario**
- Use cases
- Constraints

**N - Necessary**
- Functional requirements
- Non-functional requirements

**A - Application**
- High-level design
- Services

**K - Kilobyte**
- Data model
- Storage estimation

**E - Evolve**
- Bottlenecks
- Scaling strategies

## Communication Best Practices

### Do's

✅ **Think Out Loud**
```
"I'm thinking we need a cache here because..."
"Let me consider the trade-offs between SQL and NoSQL..."
"This could be a bottleneck, so we should..."
```

✅ **Ask Clarifying Questions**
```
"Should we prioritize consistency or availability?"
"What's more important: read latency or write latency?"
"Are there any constraints on technology stack?"
```

✅ **State Assumptions**
```
"I'm assuming 100M daily active users"
"Let's say the average tweet is 200 bytes"
"I'll assume a 100:1 read-to-write ratio"
```

✅ **Discuss Trade-offs**
```
"SQL gives us ACID, but NoSQL scales better horizontally"
"Strong consistency means higher latency"
"Caching improves performance but adds complexity"
```

✅ **Draw Diagrams**
- Use boxes for components
- Arrows for data flow
- Label clearly
- Keep it simple

✅ **Check for Understanding**
```
"Does this approach make sense?"
"Should I dive deeper into this component?"
"Would you like me to discuss alternatives?"
```

### Don'ts

❌ **Jumping to Solution**
- Don't start designing without requirements

❌ **Over-Engineering**
- Don't add unnecessary complexity
- Start simple, then scale

❌ **Ignoring Scale**
- Don't forget to do calculations
- Always consider growth

❌ **Silent Thinking**
- Don't go quiet for long periods
- Explain your thought process

❌ **Single Solution**
- Don't present only one option
- Discuss alternatives and trade-offs

❌ **Forgetting About Failures**
- Don't ignore failure scenarios
- Consider fault tolerance

## Handling Different Question Types

### Design a System (Twitter, Uber, YouTube)

**Approach:**
1. Clarify scope (which features?)
2. Estimate scale (users, QPS, storage)
3. High-level design
4. Database schema
5. API design
6. Deep dive into 2-3 components
7. Bottlenecks and optimizations

**Time Allocation:**
- Requirements: 7 minutes
- Design: 15 minutes
- Deep dive: 20 minutes
- Optimizations: 8 minutes

### Design a Component (Rate Limiter, Cache, Queue)

**Approach:**
1. Requirements and use cases
2. Algorithm/data structure
3. Implementation details
4. Distributed considerations
5. Edge cases

**Example: Rate Limiter**
```
1. Requirements:
   - Limit requests per user
   - Distributed system
   - Low latency

2. Algorithm:
   - Token bucket
   - Fixed window counter
   - Sliding window log

3. Implementation:
   - Redis with INCR and EXPIRE
   - Distributed rate limiting

4. Distributed:
   - Sticky sessions
   - Central Redis
   - Eventual consistency OK

5. Edge cases:
   - Clock skew
   - Redis failure
   - Burst traffic
```

### Capacity Estimation

**Approach:**
1. Understand requirements
2. Break down into components
3. Calculate each metric
4. Show your work

**Example:**
```
Storage for 1B photos:
- Photo size: 2 MB average
- Total: 1B × 2 MB = 2 PB
- With compression (50%): 1 PB
- With replicas (3x): 3 PB
```

## Common Interview Questions

### Q: How do you handle millions of requests per second?

**Answer Framework:**
```
1. Horizontal Scaling:
   - Add more servers
   - Load balancing
   - Auto-scaling

2. Caching:
   - Redis/Memcached
   - CDN for static assets
   - Browser caching

3. Database Optimization:
   - Read replicas
   - Sharding
   - Indexes

4. Async Processing:
   - Message queues
   - Background workers

5. Rate Limiting:
   - Prevent abuse
   - Protect system
```

### Q: How do you ensure high availability?

**Answer Framework:**
```
1. Redundancy:
   - Multiple availability zones
   - Database replicas
   - No single point of failure

2. Load Balancing:
   - Health checks
   - Auto-failover

3. Monitoring:
   - Metrics and alerts
   - Automated recovery

4. Graceful Degradation:
   - Fallback mechanisms
   - Circuit breakers

5. Regular Testing:
   - Disaster recovery drills
   - Chaos engineering
```

### Q: SQL vs NoSQL?

**Answer Framework:**
```
Choose SQL when:
- Structured data
- ACID transactions
- Complex queries/joins
- Data integrity critical
- Examples: PostgreSQL, MySQL

Choose NoSQL when:
- Flexible schema
- High write throughput
- Horizontal scaling
- Simple queries
- Examples: MongoDB, Cassandra, DynamoDB

Trade-offs:
SQL: Strong consistency, limited scale
NoSQL: High scale, eventual consistency
```

## Example Walkthrough: Design URL Shortener

### Step 1: Requirements (5 min)

**Functional:**
- Shorten long URL to short URL
- Redirect short URL to original
- Custom short URLs (optional)
- Analytics (optional)

**Non-Functional:**
- 100M URLs created per month
- 100:1 read-to-write ratio
- Low latency (<50ms)
- High availability (99.9%)
- 10-year retention

**Scope:**
✓ URL shortening and redirection
✗ User accounts (out of scope)
✗ URL editing (out of scope)

### Step 2: Estimation (3 min)

```
Write QPS:
- 100M URLs/month ÷ 30 days ÷ 86,400 sec ≈ 40 writes/sec

Read QPS:
- 40 × 100 = 4,000 reads/sec

Storage (10 years):
- Total URLs: 100M × 12 × 10 = 12B URLs
- Per URL: 500 bytes (short URL + long URL + metadata)
- Total: 12B × 500 bytes = 6 TB

Short URL length:
- Base62 (a-z, A-Z, 0-9): 62 characters
- 62^7 = 3.5 trillion combinations (sufficient)
```

### Step 3: High-Level Design (10 min)

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
┌────▼──────────┐
│ Load Balancer │
└────┬──────────┘
     │
     ├─────────────────────┐
     │                     │
┌────▼─────┐        ┌──────▼────┐
│   API    │        │    API    │
│ Server 1 │        │  Server 2 │
└────┬─────┘        └──────┬────┘
     │                     │
     └──────┬──────────────┘
            │
     ┌──────▼──────┐
     │    Cache    │
     │   (Redis)   │
     └──────┬──────┘
            │
     ┌──────▼──────┐
     │  Database   │
     │ (Cassandra) │
     └─────────────┘
```

### Step 4: Data Model (5 min)

```sql
-- Cassandra schema
CREATE TABLE urls (
  short_url VARCHAR(7) PRIMARY KEY,
  long_url TEXT,
  created_at TIMESTAMP,
  expiry_at TIMESTAMP,
  click_count BIGINT
);

CREATE INDEX idx_created ON urls(created_at);
```

### Step 5: API Design (5 min)

```javascript
// Create short URL
POST /api/v1/shorten
Request:
{
  "long_url": "https://example.com/very/long/url",
  "custom_short": "abc123" // optional
}

Response:
{
  "short_url": "https://short.ly/Xy3aB9",
  "long_url": "https://example.com/very/long/url",
  "created_at": "2024-01-15T10:30:00Z"
}

// Redirect
GET /{short_url}
Response: 301 Redirect to long_url
```

### Step 6: Core Algorithm (5 min)

```javascript
function generateShortURL(longURL) {
  // Option 1: Hash-based
  const hash = md5(longURL);
  const shortURL = base62Encode(hash.substring(0, 7));

  // Handle collisions
  while (exists(shortURL)) {
    // Append random string and retry
    shortURL = base62Encode(hash + random());
  }

  return shortURL;
}

// Option 2: Counter-based (better)
function generateShortURL() {
  const counter = getNextCounter(); // Distributed counter
  return base62Encode(counter); // Guaranteed unique
}

function base62Encode(num) {
  const chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
  let result = '';

  while (num > 0) {
    result = chars[num % 62] + result;
    num = Math.floor(num / 62);
  }

  return result.padStart(7, 'a');
}
```

### Step 7: Optimizations (7 min)

**1. Caching:**
```
Cache hot URLs in Redis:
- 20% of URLs get 80% of traffic
- Cache size: 6 TB × 20% = 1.2 TB
- TTL: 1 hour
```

**2. Database:**
```
- Use Cassandra for horizontal scaling
- Partition by short_url hash
- Replicate across 3 nodes
```

**3. CDN:**
```
- Cache 301 redirects at edge
- Reduced latency
- Lower origin load
```

**4. Rate Limiting:**
```
- Prevent abuse
- Limit: 10 URLs per IP per minute
```

**5. Analytics (Async):**
```
- Track clicks asynchronously
- Use Kafka for event stream
- Aggregate in background
```

## Summary

- **Use a framework** (RADIO, PEDALS, SNAKE) for structured approach
- **Clarify requirements** before designing (5-10 minutes)
- **Make assumptions** and state them clearly
- **Do calculations** to show you understand scale
- **Draw diagrams** to visualize architecture
- **Think out loud** to show your reasoning
- **Discuss trade-offs** for every decision
- **Start simple** then optimize
- **Consider failures** and edge cases
- **Manage time** - don't spend too long on one area
- **Ask questions** throughout the interview
- **Be flexible** - adapt based on interviewer feedback
- Practice makes perfect - do mock interviews

---
[← Back to SystemDesign](../README.md)
