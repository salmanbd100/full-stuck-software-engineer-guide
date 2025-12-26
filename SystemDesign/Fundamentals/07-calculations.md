# Back-of-Envelope Calculations

## Overview
Back-of-envelope calculations help estimate system capacity, storage, bandwidth, and resource requirements during system design interviews. These quick approximations demonstrate your ability to reason about scale and make informed architectural decisions.

## Why Calculations Matter

### Interview Perspective

Calculations show you can:
- Think quantitatively about scale
- Make realistic assumptions
- Identify bottlenecks early
- Choose appropriate technologies
- Justify architectural decisions

### Real-World Perspective

Accurate estimates help:
- Plan infrastructure capacity
- Budget for cloud costs
- Set performance targets
- Prevent over/under-provisioning
- Make data-driven decisions

## Essential Numbers to Remember

### Latency Numbers

Every programmer should know these fundamental latencies.

```
L1 cache reference:                     0.5 ns
Branch mispredict:                      5 ns
L2 cache reference:                     7 ns
Mutex lock/unlock:                      100 ns
Main memory reference:                  100 ns
Compress 1KB with Snappy:               10,000 ns = 10 µs
Send 2KB over 1 Gbps network:           20,000 ns = 20 µs
Read 1 MB sequentially from memory:     250,000 ns = 250 µs
Round trip within same datacenter:      500,000 ns = 500 µs
Disk seek:                              10,000,000 ns = 10 ms
Read 1 MB sequentially from network:    10,000,000 ns = 10 ms
Read 1 MB sequentially from disk:       30,000,000 ns = 30 ms
Send packet CA → Netherlands → CA:      150,000,000 ns = 150 ms
```

**Key Takeaways:**
- Memory is ~100x faster than disk
- Sequential reads >> random reads
- Same datacenter ~0.5ms, cross-continent ~150ms
- Network within datacenter is very fast

### Data Size Units

```
1 Byte = 8 bits
1 KB = 1,000 bytes (or 1,024 bytes = 2^10)
1 MB = 1,000 KB = 1 million bytes
1 GB = 1,000 MB = 1 billion bytes
1 TB = 1,000 GB = 1 trillion bytes
1 PB = 1,000 TB = 1 quadrillion bytes
```

**Common Data Sizes:**
```
Character (ASCII):        1 byte
Character (Unicode):      2-4 bytes
Integer (32-bit):         4 bytes
Long (64-bit):            8 bytes
Timestamp:                8 bytes
UUID:                     16 bytes
IPv4 Address:             4 bytes
IPv6 Address:             16 bytes

Tweet:                    ~280 bytes (280 chars)
Small image (thumbnail):  ~10 KB
Medium image:             ~200 KB
Large image (HD):         ~2 MB
Video (1 min, 720p):      ~50 MB
Video (1 min, 1080p):     ~100 MB
```

### Time Units

```
1 second = 1,000 milliseconds (ms)
1 second = 1,000,000 microseconds (µs)
1 second = 1,000,000,000 nanoseconds (ns)

1 day = 86,400 seconds ≈ 100,000 seconds (for calculations)
1 month ≈ 2,592,000 seconds ≈ 2.5 million seconds
1 year ≈ 31,536,000 seconds ≈ 30 million seconds
```

**Requests Per Second:**
```
1 request/second = 86,400 requests/day ≈ 100K requests/day
100 requests/second = 8.64M requests/day ≈ 10M requests/day
1,000 requests/second = 86.4M requests/day ≈ 100M requests/day
```

## Storage Calculations

### Basic Storage Estimation

**Formula:**
```
Total Storage = Data per item × Number of items × Retention period
```

**Example: Twitter-like System**

**Assumptions:**
- 500 million daily active users
- Each user tweets 2 times/day
- Each tweet: 280 chars + metadata
- Store for 5 years

**Calculation:**
```
Tweets per day = 500M users × 2 tweets = 1 billion tweets/day

Tweet size:
- Text: 280 bytes
- Metadata (user_id, timestamp, etc.): 100 bytes
- Total: ~400 bytes ≈ 0.4 KB

Daily storage = 1B tweets × 0.4 KB = 400 GB/day

Yearly storage = 400 GB × 365 = 146 TB/year

5-year storage = 146 TB × 5 = 730 TB ≈ 0.7 PB
```

**With media (20% have images):**
```
Images per day = 1B tweets × 20% = 200M images
Average image size = 200 KB

Daily media storage = 200M × 200 KB = 40 TB/day
Yearly media storage = 40 TB × 365 = 14.6 PB/year
5-year media storage = 14.6 PB × 5 = 73 PB
```

### Database Storage with Indexes

**Rule of Thumb:**
- Indexes typically add 20-30% overhead
- Include space for growth (2-3x initial estimate)

**Example:**
```
Base data: 100 GB
Indexes: 30 GB (30% overhead)
Total: 130 GB
With growth buffer (3x): 390 GB ≈ 400 GB
```

## Bandwidth Calculations

### Incoming Traffic (Write)

**Formula:**
```
Bandwidth = Requests per second × Request size
```

**Example: Video Upload Service (YouTube-like)**

**Assumptions:**
- 500 video uploads per second
- Average video size: 50 MB

**Calculation:**
```
Upload bandwidth = 500 uploads/sec × 50 MB = 25,000 MB/sec = 25 GB/sec

In Gbps:
25 GB/sec × 8 bits/byte = 200 Gbps
```

### Outgoing Traffic (Read)

**Example: Video Streaming**

**Assumptions:**
- 10,000 concurrent viewers
- 1080p video: 5 Mbps per stream

**Calculation:**
```
Streaming bandwidth = 10,000 viewers × 5 Mbps = 50,000 Mbps = 50 Gbps
```

### Read-to-Write Ratio

Most systems are read-heavy.

**Common Ratios:**
- **Social media**: 100:1 (read:write)
- **E-commerce**: 10:1
- **Messaging**: 1:1

**Example: Social Media (100:1 ratio)**
```
Writes: 1,000 requests/sec
Reads: 100,000 requests/sec
```

## QPS/TPS Calculations

### Queries Per Second (QPS)

**Formula:**
```
QPS = Daily Active Users × Actions per user / 86,400 seconds
```

**Example: URL Shortener**

**Assumptions:**
- 100 million URLs created per month
- 100:1 read-to-write ratio

**Calculation:**
```
Write QPS:
- URLs per day = 100M / 30 = 3.3M URLs/day
- Write QPS = 3.3M / 86,400 ≈ 40 writes/sec

Read QPS:
- Read QPS = 40 × 100 = 4,000 reads/sec

Peak QPS (2x average):
- Peak write: 80 writes/sec
- Peak read: 8,000 reads/sec
```

### Peak Traffic Calculations

**Rule of Thumb:**
- Peak traffic is typically 2-3x average
- Use peak for capacity planning

**Example:**
```
Average: 10,000 QPS
Peak (2x): 20,000 QPS

Capacity planning: Design for 20,000 QPS
Safety margin (1.5x peak): 30,000 QPS
```

## Memory/Cache Calculations

### Cache Size Estimation

**Rule of Thumb:**
- Cache 20% of data that serves 80% of requests (Pareto principle)

**Example: Content Caching**

**Assumptions:**
- Total data: 10 TB
- Want to cache hot data

**Calculation:**
```
Cache size = 10 TB × 20% = 2 TB

If cache hit ratio target is 80%:
- 80% of requests served from 2 TB cache
- 20% of requests go to database
```

### Memory for Connection Pools

**Formula:**
```
Memory = Connections × Memory per connection
```

**Example: Database Connections**

**Assumptions:**
- 1,000 concurrent connections
- Each connection: 10 MB

**Calculation:**
```
Total memory = 1,000 × 10 MB = 10 GB
```

## Server Count Estimation

### Application Servers

**Formula:**
```
Servers = Total QPS / (QPS per server)
```

**Example: API Servers**

**Assumptions:**
- Peak QPS: 100,000
- Each server handles: 1,000 QPS

**Calculation:**
```
Servers needed = 100,000 / 1,000 = 100 servers

With redundancy (N+1):
- Active: 100 servers
- Standby: 10 servers (10%)
- Total: 110 servers
```

### Database Servers

**Example: Read Replicas**

**Assumptions:**
- Read QPS: 50,000
- Each replica handles: 5,000 QPS

**Calculation:**
```
Read replicas = 50,000 / 5,000 = 10 replicas

With master:
- 1 master (for writes)
- 10 read replicas
- Total: 11 database servers
```

## CDN Calculations

### CDN Storage

**Example: Static Assets**

**Assumptions:**
- 1 million users
- Each user profile has: 5 images
- Average image: 100 KB

**Calculation:**
```
Total images = 1M users × 5 = 5M images
Storage = 5M × 100 KB = 500 GB
```

### CDN Bandwidth

**Assumptions:**
- 10 million page views/day
- Each page: 2 MB (images, CSS, JS)
- 80% served from CDN

**Calculation:**
```
Daily data transfer = 10M × 2 MB = 20 TB/day
CDN bandwidth = 20 TB × 80% = 16 TB/day

Per second:
16 TB / 86,400 sec ≈ 185 MB/sec ≈ 1.5 Gbps
```

## Practical Examples

### Example 1: Instagram-like Service

**Requirements:**
- 500M daily active users
- Each user uploads 2 photos/day
- Each user views 50 photos/day
- Photo size: 2 MB average
- Store for 3 years

**Calculations:**

**Storage:**
```
Photos per day = 500M × 2 = 1B photos/day
Daily storage = 1B × 2 MB = 2 PB/day
Yearly storage = 2 PB × 365 = 730 PB/year
3-year storage = 730 PB × 3 = 2,190 PB ≈ 2 EB
```

**Bandwidth:**
```
Uploads (write):
- 1B photos/day ÷ 86,400 = 11,574 photos/sec
- 11,574 × 2 MB = 23 GB/sec upload bandwidth

Downloads (read):
- 500M users × 50 photos = 25B photos/day
- 25B ÷ 86,400 = 289,351 photos/sec
- 289,351 × 2 MB = 578 GB/sec download bandwidth
```

**QPS:**
```
Upload API: 11,574 writes/sec
View API: 289,351 reads/sec
Peak (2x):
- Write: 23,148/sec
- Read: 578,702/sec
```

### Example 2: URL Shortener

**Requirements:**
- 100M new URLs per month
- 100:1 read-to-write ratio
- Store for 10 years

**Calculations:**

**Storage:**
```
URLs per day = 100M / 30 ≈ 3.3M URLs/day

URL data:
- Short URL: 7 chars = 7 bytes
- Long URL: 200 bytes (average)
- Metadata: 50 bytes
- Total: ~260 bytes ≈ 0.26 KB

Daily storage = 3.3M × 0.26 KB ≈ 850 MB/day
Yearly storage = 850 MB × 365 ≈ 310 GB/year
10-year storage = 310 GB × 10 = 3.1 TB
```

**QPS:**
```
Write QPS = 3.3M / 86,400 ≈ 40 writes/sec
Read QPS = 40 × 100 = 4,000 reads/sec
Peak: 8,000 reads/sec
```

**Cache:**
```
Total URLs after 10 years = 100M × 12 × 10 = 12B URLs
Total size = 12B × 0.26 KB = 3.1 TB

Cache 20% hot URLs:
Cache size = 3.1 TB × 20% = 620 GB
```

### Example 3: Messaging App (WhatsApp-like)

**Requirements:**
- 1B daily active users
- Each user sends 50 messages/day
- Each message: 100 bytes
- Store for 1 year

**Calculations:**

**Storage:**
```
Messages per day = 1B × 50 = 50B messages/day
Daily storage = 50B × 100 bytes = 5 TB/day
Yearly storage = 5 TB × 365 = 1.8 PB/year
```

**QPS:**
```
Message QPS = 50B / 86,400 ≈ 578,703 messages/sec
Peak (2x) = 1,157,406 messages/sec
```

**Servers:**
```
Assuming each server handles 5,000 QPS:
Servers = 578,703 / 5,000 ≈ 116 servers

Peak servers = 1,157,406 / 5,000 ≈ 232 servers
With redundancy (+20%) = 280 servers
```

## Calculation Shortcuts

### Quick Multipliers

**Traffic Patterns:**
```
Average → Peak: 2x-3x
Weekday → Weekend: 1.5x-2x
Business hours → Off hours: 3x-5x
```

**Storage Growth:**
```
Year 1: Base
Year 2: 2x base
Year 3: 3.5x base
Year 5: 8x base
```

**Rounding for Estimates:**
```
86,400 seconds/day → 100,000 (easier math)
365 days/year → 400 (easier math)
1,024 bytes → 1,000 (KB approximation)
```

### Rule of 72 (Doubling Time)

For exponential growth: `Years to double ≈ 72 / Growth rate`

**Example:**
```
20% annual growth → 72/20 = 3.6 years to double
50% annual growth → 72/50 = 1.4 years to double
```

## Interview Tips

### Approach

1. **State Assumptions Clearly**
   ```
   "Assuming 100M daily active users..."
   "Let's say average photo is 2 MB..."
   ```

2. **Round Numbers**
   ```
   3.3M → 3M (or 3.5M if precision matters)
   86,400 → 100,000
   ```

3. **Show Your Work**
   ```
   "500M users × 2 tweets = 1B tweets/day"
   "1B tweets ÷ 100K seconds ≈ 10K tweets/sec"
   ```

4. **Sanity Check**
   ```
   "10K writes/sec seems reasonable for this scale"
   "2 PB seems too high, let me recalculate..."
   ```

5. **Consider Peak vs Average**
   ```
   "Average is 10K QPS, so peak might be 20-30K QPS"
   ```

### Common Mistakes

❌ **Being too precise**
- "Exactly 86,400 seconds per day" → Just use 100K

❌ **Forgetting about growth**
- Always plan for 2-3x current requirements

❌ **Ignoring redundancy**
- Don't forget replicas, backups, failover capacity

❌ **Not considering peak traffic**
- Design for peak, not average

❌ **Mixing units**
- Be consistent: MB/sec or Gbps, not both

## Practice Problems

### Problem 1: Design Twitter

**Given:**
- 200M daily active users
- Each user reads 100 tweets/day
- Each user writes 2 tweets/day
- 10% of tweets have images (200 KB avg)

**Calculate:**
1. Storage needed for 1 year
2. Read QPS
3. Write QPS
4. Bandwidth requirements

**Solution:**
```
1. Storage:
   Tweets/day = 200M × 2 = 400M
   Text: 400M × 0.3 KB = 120 GB/day
   Images: 400M × 10% × 200 KB = 8 TB/day
   Total: 8.12 TB/day
   Yearly: 8.12 TB × 365 ≈ 3 PB

2. Read QPS:
   Reads = 200M × 100 = 20B reads/day
   QPS = 20B / 100K ≈ 200K reads/sec

3. Write QPS:
   Writes = 400M/day
   QPS = 400M / 100K ≈ 4K writes/sec

4. Bandwidth:
   Read: 200K reads/sec × 0.3 KB ≈ 60 MB/sec
   Write: 4K writes/sec × 0.3 KB ≈ 1.2 MB/sec
   Images: 4K × 10% × 200 KB ≈ 80 MB/sec
```

### Problem 2: Design YouTube

**Given:**
- 100M daily active users
- Each user watches 5 videos/day (5 min each)
- 1M videos uploaded/day
- Video bitrate: 5 Mbps

**Calculate:**
1. Storage needed for 1 year
2. Streaming bandwidth
3. Upload bandwidth

**Solution:**
```
1. Storage:
   Upload size = 1M videos × 5 min × 5 Mbps
               = 1M × 300 sec × 5 Mb/sec
               = 1.5M Mb = 187.5 TB/day
   Yearly: 187.5 TB × 365 ≈ 68 PB

2. Streaming bandwidth:
   Concurrent viewers = 100M users × 5 videos × 5 min / (24 × 60)
                     ≈ 1.7M concurrent streams
   Bandwidth = 1.7M × 5 Mbps = 8.5 Tbps

3. Upload bandwidth:
   Uploads = 1M videos × 5 min × 5 Mbps / 86,400 sec
          ≈ 289 Gbps
```

## Summary

- **Know the numbers**: Memorize key latencies and data sizes
- **Make assumptions**: State them clearly and make them realistic
- **Round liberally**: 86,400 → 100,000 for easier mental math
- **Think in orders of magnitude**: KB vs MB vs GB vs TB vs PB
- **Consider peak traffic**: 2-3x average for capacity planning
- **Include redundancy**: Add 20-50% for replicas and failover
- **Sanity check**: Does 2 PB/day make sense? Question outliers
- **Show your work**: Interviewers want to see your thought process
- **Storage formula**: Items × Size × Retention
- **Bandwidth formula**: QPS × Payload size
- **QPS formula**: Daily requests / 86,400
- **Always plan for growth**: 2-3x initial estimates

---
[← Back to SystemDesign](../README.md)
