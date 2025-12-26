# Scalability

## Overview
Scalability is the capability of a system to handle growing amounts of work by adding resources. A scalable system maintains performance and reliability as load increases, whether through more users, transactions, or data.

## Types of Scalability

### Vertical Scaling (Scale Up)

Adding more resources to a single machine.

**What it means:**
- Upgrade CPU: 4 cores → 16 cores
- Increase RAM: 16GB → 128GB
- Better storage: HDD → SSD
- Faster network: 1Gbps → 10Gbps

**Advantages:**
- **Simple**: No architectural changes needed
- **No Distribution Complexity**: Single machine, easier to manage
- **Data Consistency**: No distributed data issues
- **Inter-Process Communication**: Faster than network calls

**Disadvantages:**
- **Hard Limits**: Can't add infinite resources
- **Expensive**: High-end hardware costs exponentially more
- **Single Point of Failure**: One machine down = entire system down
- **Downtime Required**: Often need to take system offline to upgrade

**When to Use:**
- Early stages with limited traffic
- Databases requiring ACID transactions
- Systems with tight consistency requirements
- Legacy applications hard to distribute

**Example:**
```
Before: AWS t3.medium (2 vCPU, 4GB RAM) - $30/month
After:  AWS t3.2xlarge (8 vCPU, 32GB RAM) - $300/month

Handles: 1,000 req/s → 4,000 req/s
```

### Horizontal Scaling (Scale Out)

Adding more machines to distribute load.

**What it means:**
- Add more servers behind load balancer
- Distribute data across multiple databases
- Run parallel workers
- Geographic distribution

**Advantages:**
- **No Hard Limits**: Can add unlimited machines
- **Cost-Effective**: Commodity hardware is cheaper
- **High Availability**: Redundancy built-in
- **Fault Tolerance**: Failure of one machine doesn't affect system
- **Geographic Distribution**: Serve users from nearest location

**Disadvantages:**
- **Complexity**: Need load balancers, service discovery
- **Data Consistency**: Distributed data is harder to keep consistent
- **Network Overhead**: Communication between machines
- **More Moving Parts**: More things can go wrong

**When to Use:**
- High traffic applications
- Need high availability
- Unpredictable growth
- Global user base

**Example:**
```
Before: 1 server handling 5,000 req/s
After:  10 servers each handling 500 req/s

Cost: Same hardware × 10 = More capacity, better redundancy
```

## Horizontal Scaling Patterns

### Stateless Services

Services that don't store session data locally.

**Design:**
```
┌─────────┐     ┌────────────────┐
│ Client  │────▶│ Load Balancer  │
└─────────┘     └────────────────┘
                    │    │    │
         ┌──────────┼────┼────┼──────────┐
         ▼          ▼    ▼    ▼          ▼
    ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
    │Server 1│ │Server 2│ │Server 3│ │Server 4│
    └────────┘ └────────┘ └────────┘ └────────┘
         │          │         │           │
         └──────────┴─────────┴───────────┘
                    ▼
            ┌───────────────┐
            │ Session Store │
            │    (Redis)    │
            └───────────────┘
```

**Benefits:**
- Any server can handle any request
- Easy to add/remove servers
- Simple load balancing

**Implementation:**
```javascript
// ❌ Stateful - stores session in memory
const sessions = {};

app.post('/login', (req, res) => {
  const sessionId = generateId();
  sessions[sessionId] = { userId: req.body.userId };
  res.cookie('sessionId', sessionId);
});

// ✅ Stateless - stores session in Redis
app.post('/login', async (req, res) => {
  const sessionId = generateId();
  await redis.set(`session:${sessionId}`, JSON.stringify({
    userId: req.body.userId
  }), 'EX', 3600);
  res.cookie('sessionId', sessionId);
});
```

### Database Sharding

Partitioning data across multiple databases.

**Sharding Strategies:**

1. **Range-Based Sharding**
   ```
   User ID 1-1,000,000     → Shard 1
   User ID 1,000,001-2,000,000 → Shard 2
   User ID 2,000,001-3,000,000 → Shard 3
   ```

   **Pros:** Simple, range queries efficient
   **Cons:** Uneven distribution, hotspots

2. **Hash-Based Sharding**
   ```
   Shard = hash(user_id) % number_of_shards

   hash(user_123) % 4 = 3 → Shard 3
   hash(user_456) % 4 = 1 → Shard 1
   ```

   **Pros:** Even distribution
   **Cons:** Range queries require hitting all shards, rebalancing is hard

3. **Geographic Sharding**
   ```
   US users      → US Shard
   Europe users  → EU Shard
   Asia users    → Asia Shard
   ```

   **Pros:** Low latency, data locality
   **Cons:** Uneven load, cross-region queries expensive

4. **Directory-Based Sharding**
   ```
   Lookup Table:
   user_123 → Shard 2
   user_456 → Shard 1
   user_789 → Shard 3
   ```

   **Pros:** Flexible, easy rebalancing
   **Cons:** Lookup table is bottleneck, extra hop

**Implementation:**
```javascript
// Shard selector
class ShardManager {
  constructor(shards) {
    this.shards = shards; // ['shard1.db.com', 'shard2.db.com', ...]
  }

  // Hash-based sharding
  getShard(userId) {
    const hash = this.hashCode(userId);
    const index = hash % this.shards.length;
    return this.shards[index];
  }

  hashCode(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      hash = ((hash << 5) - hash) + str.charCodeAt(i);
      hash |= 0; // Convert to 32-bit integer
    }
    return Math.abs(hash);
  }

  async getUser(userId) {
    const shard = this.getShard(userId);
    return await db.connect(shard).query(
      'SELECT * FROM users WHERE id = ?',
      [userId]
    );
  }
}
```

### Database Replication

Copying data to multiple databases.

**Master-Slave (Primary-Replica):**
```
                ┌────────────┐
                │   Master   │
                │  (Writes)  │
                └────────────┘
                   │    │    │
            ┌──────┴────┴────┴──────┐
            ▼          ▼            ▼
       ┌────────┐ ┌────────┐  ┌────────┐
       │ Slave 1│ │ Slave 2│  │ Slave 3│
       │(Reads) │ │(Reads) │  │(Reads) │
       └────────┘ └────────┘  └────────┘
```

**Characteristics:**
- **Writes**: Go to master only
- **Reads**: Distributed across slaves
- **Replication**: Async (eventual consistency)

**Pros:**
- **Read Scalability**: Add more read replicas
- **Backup**: Slaves serve as hot backups

**Cons:**
- **Write Bottleneck**: Single master for writes
- **Replication Lag**: Slaves may be slightly behind
- **Failover Complexity**: Need to promote slave to master

**Master-Master (Multi-Master):**
```
    ┌────────────┐  ◀────▶  ┌────────────┐
    │  Master 1  │          │  Master 2  │
    │ (R/W - US) │          │ (R/W - EU) │
    └────────────┘  ────▶   └────────────┘
```

**Pros:**
- **Write Scalability**: Multiple masters
- **High Availability**: No single point of failure

**Cons:**
- **Conflict Resolution**: Need to handle write conflicts
- **Complexity**: More complex to manage

### Caching Layers

Fast access to frequently used data.

**Multi-Level Caching:**
```
Client
  ↓
Browser Cache (HTTP cache)
  ↓
CDN Cache (CloudFront, Cloudflare)
  ↓
Application Cache (Redis, Memcached)
  ↓
Database Query Cache
  ↓
Database
```

**Cache Patterns:**

1. **Cache-Aside**
   ```python
   def get_user(user_id):
       # Try cache first
       user = cache.get(f"user:{user_id}")
       if user:
           return user

       # Cache miss - fetch from DB
       user = db.query("SELECT * FROM users WHERE id = ?", user_id)

       # Store in cache
       cache.set(f"user:{user_id}", user, ttl=3600)
       return user
   ```

2. **Read-Through**
   ```python
   # Cache handles DB fetch automatically
   user = cache.get(f"user:{user_id}")  # Cache fetches from DB if miss
   ```

3. **Write-Through**
   ```python
   def update_user(user_id, data):
       # Write to cache AND database
       cache.set(f"user:{user_id}", data)
       db.query("UPDATE users SET ... WHERE id = ?", user_id)
   ```

4. **Write-Behind (Write-Back)**
   ```python
   def update_user(user_id, data):
       # Write to cache immediately
       cache.set(f"user:{user_id}", data)

       # Queue DB write for later (async)
       queue.enqueue('db_writes', {
           'table': 'users',
           'id': user_id,
           'data': data
       })
   ```

**Cache Eviction Policies:**
- **LRU (Least Recently Used)**: Remove oldest unused
- **LFU (Least Frequently Used)**: Remove least accessed
- **FIFO**: Remove oldest added
- **TTL**: Expire after time limit

## Load Balancing

### Load Balancing Algorithms

1. **Round Robin**
   ```
   Request 1 → Server 1
   Request 2 → Server 2
   Request 3 → Server 3
   Request 4 → Server 1 (cycle repeats)
   ```

   **Pros:** Simple, fair distribution
   **Cons:** Doesn't consider server load

2. **Weighted Round Robin**
   ```
   Server 1 (weight: 3) gets 3 requests
   Server 2 (weight: 1) gets 1 request
   ```

   **Use:** Servers with different capacities

3. **Least Connections**
   ```
   Choose server with fewest active connections
   ```

   **Pros:** Accounts for varying request durations
   **Cons:** Need to track connections

4. **Least Response Time**
   ```
   Choose server with fastest recent response time
   ```

   **Pros:** Accounts for actual performance
   **Cons:** More complex to implement

5. **IP Hash (Session Affinity)**
   ```
   server = hash(client_ip) % num_servers
   ```

   **Pros:** Same client → same server (sticky sessions)
   **Cons:** Uneven distribution

### Load Balancer Types

**Layer 4 (Transport Layer)**
- Routes based on IP and port
- Fast, simple
- No content awareness
- Example: AWS NLB, HAProxy

**Layer 7 (Application Layer)**
- Routes based on HTTP headers, cookies, URL path
- Content-aware routing
- SSL termination
- Example: AWS ALB, Nginx

**Global Load Balancing (DNS)**
- Routes to nearest geographic location
- Disaster recovery
- Example: Route 53, Cloudflare

## Asynchronous Processing

Decouple time-consuming tasks from user requests.

**Message Queue Pattern:**
```
┌────────┐      ┌───────┐      ┌─────────┐      ┌──────────┐
│ Client │─────▶│  API  │─────▶│  Queue  │─────▶│  Worker  │
└────────┘      └───────┘      └─────────┘      └──────────┘
                    ↓                                  ↓
               200 OK                            Process
            (immediate)                        (background)
```

**Use Cases:**

1. **Email Sending**
   ```javascript
   // Synchronous (bad - blocks user)
   app.post('/register', async (req, res) => {
     await createUser(req.body);
     await sendWelcomeEmail(req.body.email);  // Slow!
     res.json({ success: true });
   });

   // Asynchronous (good - responsive)
   app.post('/register', async (req, res) => {
     await createUser(req.body);
     await queue.enqueue('send_email', {
       type: 'welcome',
       email: req.body.email
     });
     res.json({ success: true });  // Returns immediately
   });
   ```

2. **Image Processing**
   ```javascript
   app.post('/upload', async (req, res) => {
     const file = await saveFile(req.file);

     // Queue thumbnail generation
     await queue.enqueue('process_image', {
       fileId: file.id,
       tasks: ['thumbnail', 'watermark', 'compress']
     });

     res.json({ fileId: file.id });
   });
   ```

3. **Report Generation**
   ```javascript
   app.post('/reports/generate', async (req, res) => {
     const reportId = generateId();

     await queue.enqueue('generate_report', {
       reportId,
       type: req.body.type,
       dateRange: req.body.dateRange
     });

     res.json({
       reportId,
       status: 'processing',
       checkUrl: `/reports/${reportId}/status`
     });
   });
   ```

## Microservices Architecture

Breaking monolith into independent services.

**Monolith:**
```
┌─────────────────────────────┐
│      Single Application     │
│  ┌────────────────────────┐ │
│  │  User Management       │ │
│  │  Product Catalog       │ │
│  │  Shopping Cart         │ │
│  │  Payment Processing    │ │
│  │  Order Management      │ │
│  └────────────────────────┘ │
│      Single Database        │
└─────────────────────────────┘
```

**Microservices:**
```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│   User   │  │ Product  │  │   Cart   │  │ Payment  │
│ Service  │  │ Service  │  │ Service  │  │ Service  │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
     ↓             ↓             ↓             ↓
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│ User DB  │  │Product DB│  │ Cart DB  │  │Payment DB│
└──────────┘  └──────────┘  └──────────┘  └──────────┘
```

**Benefits:**
- **Independent Deployment**: Deploy services separately
- **Technology Flexibility**: Different tech stack per service
- **Isolated Failures**: One service failure doesn't crash all
- **Team Autonomy**: Teams own entire service

**Challenges:**
- **Complexity**: More services to manage
- **Distributed Transactions**: Hard to maintain consistency
- **Network Latency**: Service-to-service calls
- **Debugging**: Harder to trace issues across services

## Common Interview Questions

**Q: When should you use vertical vs horizontal scaling?**

A:
**Vertical (Scale Up):**
- Early stages with predictable load
- Databases requiring strong consistency
- Systems hard to distribute (legacy apps)
- Example: MySQL database for startup

**Horizontal (Scale Out):**
- High traffic, unpredictable growth
- Need high availability
- Stateless applications
- Example: Web application layer

**Start vertical for simplicity, move to horizontal for scale.**

**Q: How do you handle database hotspots in sharding?**

A: Hotspots occur when one shard gets disproportionate traffic.

**Solutions:**
1. **Better Shard Key**: Choose key with even distribution
2. **Consistent Hashing**: Minimize redistribution when adding shards
3. **Shard Splitting**: Split hot shard into multiple
4. **Caching**: Cache hot data to reduce DB load
5. **Read Replicas**: Add replicas for hot shard

**Example:**
```
Problem: Celebrity user on Shard 1 gets 1M followers
Solution: Cache celebrity's profile, add read replicas
```

**Q: What are the trade-offs of master-slave replication?**

A:
**Pros:**
- Read scalability (add more slaves)
- Backup/failover (promote slave)
- Separate analytics workload

**Cons:**
- Write bottleneck (single master)
- Replication lag (eventual consistency)
- Failover complexity

**When it works:** Read-heavy applications (social media feeds)
**When it doesn't:** Write-heavy applications (financial transactions)

**Q: How do you scale a write-heavy application?**

A: Strategies:
1. **Database Sharding**: Distribute writes across shards
2. **Write-Behind Caching**: Write to cache, async to DB
3. **Queue Writes**: Buffer writes in queue
4. **Multi-Master Replication**: Multiple write nodes
5. **Batch Writes**: Combine multiple writes

**Example:**
```
Analytics system with 100K writes/sec:
- Shard by time (current month shard gets writes)
- Use Cassandra (optimized for writes)
- Write to Kafka, batch insert to database
```

## Best Practices

**Design for Scalability:**
✅ Make services stateless
✅ Use caching aggressively
✅ Implement async processing for slow tasks
✅ Design database schema for sharding
✅ Monitor and measure performance
✅ Plan for failure

**Common Mistakes:**
❌ Premature optimization
❌ Ignoring database bottlenecks
❌ Not monitoring key metrics
❌ Over-engineering for scale you don't have
❌ Underestimating network latency
❌ Not planning for data growth

## Summary

- **Scalability** is handling growth by adding resources
- **Vertical scaling** (scale up) is simpler but has limits
- **Horizontal scaling** (scale out) offers unlimited growth but adds complexity
- **Stateless services** enable easy horizontal scaling
- **Database sharding** distributes data, but adds complexity
- **Caching** reduces load on databases and external services
- **Load balancing** distributes traffic across servers
- **Async processing** decouples slow tasks from user requests
- Choose scaling strategy based on requirements, not trends

---
[← Back to SystemDesign](../README.md)
