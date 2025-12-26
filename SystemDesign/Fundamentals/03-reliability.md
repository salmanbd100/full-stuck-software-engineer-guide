# Reliability and Availability

## Overview
Reliability is the ability of a system to function correctly even when failures occur. Availability measures the percentage of time a system is operational. Together, they ensure systems users can depend on.

## Key Metrics

### Availability (Uptime)

Percentage of time system is operational.

**Formula:**
```
Availability = (Total Time - Downtime) / Total Time × 100%
```

**Service Level Objectives (SLOs):**
```
99%      → 3.65 days downtime/year   → "Two nines"
99.9%    → 8.76 hours downtime/year  → "Three nines"
99.99%   → 52.56 minutes/year        → "Four nines"
99.999%  → 5.26 minutes/year         → "Five nines"
99.9999% → 31.5 seconds/year         → "Six nines"
```

**Cost vs Availability:**
- 99% → Relatively inexpensive
- 99.9% → Moderate cost (standard for most apps)
- 99.99% → Expensive (critical services)
- 99.999% → Very expensive (financial systems)
- 99.9999% → Extremely expensive (rarely needed)

### Mean Time Between Failures (MTBF)

Average time between system failures.

```
MTBF = Total Operating Time / Number of Failures

Example:
1000 hours / 5 failures = 200 hours MTBF
```

### Mean Time To Repair (MTTR)

Average time to restore service after failure.

```
MTTR = Total Downtime / Number of Failures

Example:
10 hours / 5 failures = 2 hours MTTR
```

**Availability Formula:**
```
Availability = MTBF / (MTBF + MTTR)

Example:
MTBF = 200 hours
MTTR = 2 hours
Availability = 200 / (200 + 2) = 99%
```

## Failure Patterns

### Single Point of Failure (SPOF)

Component whose failure causes entire system to fail.

**Examples:**
- Single database server
- Single load balancer
- Single payment gateway
- Single datacenter

**Solutions:**
- Add redundancy
- Implement failover
- Use multiple availability zones
- Geographic distribution

### Cascading Failures

One failure triggers subsequent failures.

**Example:**
```
1. Database overloaded
2. API servers timeout waiting
3. Load balancer marks API servers as unhealthy
4. Traffic redistributed to remaining servers
5. Remaining servers also overload
6. Entire system fails
```

**Prevention:**
- Circuit breakers
- Rate limiting
- Timeouts
- Bulkheads (isolation)
- Graceful degradation

## Redundancy Strategies

### Active-Passive (Hot Standby)

Backup server ready to take over immediately.

```
Primary Server (Active)  ─┐
                          ├─▶ Serves Traffic
Standby Server (Passive) ─┘   (Waits)

On Failure:
Standby becomes Active
```

**Pros:** Fast failover, simple
**Cons:** Wasted resources (standby idle)

### Active-Active

Multiple servers actively serving traffic.

```
Server 1 (Active) ─┐
Server 2 (Active) ─┼─▶ Load Balancer ─▶ Clients
Server 3 (Active) ─┘
```

**Pros:** No wasted resources, higher capacity
**Cons:** Complex synchronization, potential conflicts

### Geographic Redundancy

Servers in multiple datacenters/regions.

```
US-East    │  US-West    │  EU-West
───────────┼─────────────┼──────────
Datacenter │ Datacenter  │ Datacenter
Active     │ Active      │ Active
```

**Benefits:**
- Disaster recovery
- Low latency (serve from nearest)
- Compliance (data residency)

## Fault Tolerance Patterns

### Circuit Breaker

Prevents repeated attempts to failing service.

```javascript
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    this.failureCount = 0;
    this.threshold = threshold;
    this.timeout = timeout;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.nextAttempt = Date.now();
  }

  async execute(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
    }

    try {
      const result = await fn();

      if (this.state === 'HALF_OPEN') {
        this.state = 'CLOSED';
        this.failureCount = 0;
      }

      return result;
    } catch (error) {
      this.failureCount++;

      if (this.failureCount >= this.threshold) {
        this.state = 'OPEN';
        this.nextAttempt = Date.now() + this.timeout;
      }

      throw error;
    }
  }
}

// Usage
const breaker = new CircuitBreaker();

try {
  const data = await breaker.execute(() => fetchFromAPI());
} catch (error) {
  // Fallback to cache or return error
  return getCachedData();
}
```

### Retry with Exponential Backoff

Automatically retry failed operations with increasing delays.

```javascript
async function retryWithBackoff(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;

      // Exponential backoff: 1s, 2s, 4s, 8s...
      const delay = Math.min(1000 * Math.pow(2, i), 30000);

      // Add jitter to prevent thundering herd
      const jitter = Math.random() * 1000;

      await new Promise(resolve =>
        setTimeout(resolve, delay + jitter)
      );
    }
  }
}

// Usage
const data = await retryWithBackoff(() => fetchData());
```

### Bulkhead Pattern

Isolate resources to prevent cascade failures.

```javascript
// Separate thread pools for different services
const userServicePool = new ThreadPool(10);
const paymentServicePool = new ThreadPool(5);
const emailServicePool = new ThreadPool(3);

// If email service fails, doesn't affect others
```

### Rate Limiting

Protect services from overload.

```javascript
class RateLimiter {
  constructor(maxRequests, windowMs) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
    this.requests = new Map();
  }

  isAllowed(key) {
    const now = Date.now();
    const windowStart = now - this.windowMs;

    // Clean old requests
    const userRequests = this.requests.get(key) || [];
    const validRequests = userRequests.filter(
      time => time > windowStart
    );

    if (validRequests.length >= this.maxRequests) {
      return false;
    }

    validRequests.push(now);
    this.requests.set(key, validRequests);
    return true;
  }
}

// Usage: 100 requests per minute
const limiter = new RateLimiter(100, 60000);

if (!limiter.isAllowed(userId)) {
  return res.status(429).json({ error: 'Too many requests' });
}
```

## Health Checks and Monitoring

### Health Check Endpoints

Monitor service health.

```javascript
app.get('/health', async (req, res) => {
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
    externalAPI: await checkExternalAPI()
  };

  const allHealthy = Object.values(checks).every(c => c.healthy);

  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'healthy' : 'unhealthy',
    checks,
    timestamp: new Date().toISOString()
  });
});

async function checkDatabase() {
  try {
    await db.query('SELECT 1');
    return { healthy: true };
  } catch (error) {
    return { healthy: false, error: error.message };
  }
}
```

### Heartbeat Monitoring

Regular ping to detect failures.

```javascript
setInterval(async () => {
  const services = ['api-1', 'api-2', 'api-3'];

  for (const service of services) {
    try {
      await fetch(`http://${service}/health`, { timeout: 5000 });
      markServiceHealthy(service);
    } catch (error) {
      markServiceUnhealthy(service);
      alertOps(`Service ${service} is down!`);
    }
  }
}, 30000); // Every 30 seconds
```

## Backup and Recovery

### Backup Strategies

1. **Full Backup**
   - Complete copy of all data
   - Slowest, most storage
   - Fastest recovery

2. **Incremental Backup**
   - Only changes since last backup
   - Faster, less storage
   - Slower recovery (need all incrementals)

3. **Differential Backup**
   - Changes since last full backup
   - Middle ground

**3-2-1 Rule:**
- 3 copies of data
- 2 different media types
- 1 offsite backup

### Recovery Point Objective (RPO)

Maximum acceptable data loss.

```
RPO = 1 hour

If failure at 3:00 PM, can lose data since 2:00 PM
Need: Backups every hour or less
```

### Recovery Time Objective (RTO)

Maximum acceptable downtime.

```
RTO = 15 minutes

System must be restored within 15 minutes of failure
Need: Hot standby or fast failover
```

## Disaster Recovery

### Backup Site Strategies

1. **Cold Site**
   - Empty datacenter
   - RTO: Days to weeks
   - Cost: Low

2. **Warm Site**
   - Partially equipped
   - RTO: Hours to days
   - Cost: Medium

3. **Hot Site**
   - Fully operational replica
   - RTO: Minutes
   - Cost: High

### Multi-Region Architecture

```
Region 1 (Primary)     Region 2 (DR)
──────────────────     ─────────────
Load Balancer          Load Balancer
App Servers (Active)   App Servers (Standby)
Database (Master)      Database (Replica)
         │                     ↑
         └─────Replication─────┘
```

**Failover Process:**
1. Detect primary failure
2. Promote DR database to master
3. Redirect traffic to DR region
4. Monitor and alert

## Interview Questions

**Q: How do you achieve 99.99% availability?**

A: Multiple strategies combined:
1. **Eliminate SPOFs**: Redundant load balancers, databases, servers
2. **Multi-AZ Deployment**: Spread across availability zones
3. **Auto-scaling**: Handle traffic spikes
4. **Health checks**: Detect and remove unhealthy instances
5. **Automated failover**: Quick recovery from failures
6. **Monitoring & Alerts**: Detect issues before users
7. **Regular testing**: Chaos engineering, disaster recovery drills

**Q: Explain the difference between high availability and disaster recovery.**

A:
- **High Availability (HA)**: Minimize downtime during normal operations
  - Redundant components
  - Auto-failover
  - Load balancing
  - Goal: Prevent downtime

- **Disaster Recovery (DR)**: Restore operations after catastrophic failure
  - Backups
  - Recovery procedures
  - Secondary datacenter
  - Goal: Recover from disasters

**Q: How do you prevent cascading failures?**

A: Defense in depth:
1. **Circuit breakers**: Stop calling failing services
2. **Timeouts**: Don't wait indefinitely
3. **Bulkheads**: Isolate failures
4. **Rate limiting**: Prevent overload
5. **Graceful degradation**: Provide limited functionality
6. **Load shedding**: Drop low-priority requests

## Best Practices

✅ **Design:**
- Eliminate single points of failure
- Design for failure (assume everything can fail)
- Implement retry with backoff
- Use circuit breakers
- Set appropriate timeouts

✅ **Operations:**
- Monitor key metrics (availability, latency, errors)
- Set up alerts before thresholds
- Regular backup testing
- Disaster recovery drills
- Incident postmortems

✅ **Testing:**
- Chaos engineering (Netflix's Chaos Monkey)
- Failure injection testing
- Load testing
- Disaster recovery simulation

❌ **Avoid:**
- Single point of failure
- No health checks
- Manual failover processes
- Untested backups
- No monitoring/alerting

## Summary

- **Availability** measures uptime percentage (99.9% = 8.76 hours/year downtime)
- **Reliability** ensures correct operation despite failures
- **Redundancy** (active-passive, active-active) eliminates single points of failure
- **Circuit breakers** prevent cascading failures
- **Backups** with defined RPO/RTO enable disaster recovery
- **Monitoring** and health checks detect failures quickly
- Design assuming failures will happen, not if they happen

---
[← Back to SystemDesign](../README.md)
