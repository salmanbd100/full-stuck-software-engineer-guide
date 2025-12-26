# CAP Theorem

## Overview
CAP Theorem states that a distributed system can only guarantee two out of three properties: Consistency, Availability, and Partition Tolerance. Understanding CAP helps you make informed trade-offs when designing distributed systems.

## The Three Properties

### Consistency (C)

All nodes see the same data at the same time.

**What it means:**
- Read always returns most recent write
- All replicas have identical data
- No stale data served

**Example:**
```
User writes: balance = $100
User immediately reads: balance = $100  ✓ Consistent

NOT:
User writes: balance = $100
User reads: balance = $50  ✗ Inconsistent (stale data)
```

**Real-World:**
- Bank account balance must be consistent
- Inventory count must be accurate
- User authentication state

### Availability (A)

Every request receives a response (success or failure).

**What it means:**
- System always operational
- No downtime
- Every request gets response (even if not latest data)

**Example:**
```
Request to read balance → Always get a response
(even if one server is down)
```

**Real-World:**
- Social media feeds can show slightly old data
- E-commerce product listings
- Search results

### Partition Tolerance (P)

System continues operating despite network failures between nodes.

**What it means:**
- Network can split into partitions
- Nodes can't communicate
- System still functions

**Example:**
```
Network Partition:
[Server A] ─X─ [Server B]
     │              │
  Can't communicate
     │              │
  Still serves requests
```

**Real-World:**
- **Datacenter link failure**: US-East can't talk to US-West
- **Network congestion**: Messages delayed or dropped
- **Hardware failure**: Switch/router down

**Important:** In distributed systems, partitions **will** happen, so P is not optional. The real choice is between C and A.

## Trade-offs

### CP (Consistency + Partition Tolerance)

Sacrifice availability to maintain consistency.

**How it works:**
```
Normal Operation:
User → Server A (Master) → Server B (Replica)
        Write               Replicate
                            ↓
                         Success

Network Partition:
User → Server A ─X─ Server B
        ↓
      Error! (Can't guarantee consistency)
      Return 503 Service Unavailable
```

**Behavior:**
- When partition occurs, system becomes unavailable
- Prevents serving stale data
- Waits for partition to heal

**Examples:**
- **Banking systems**: Better unavailable than wrong balance
- **Distributed databases**: MongoDB (with majority writes), HBase
- **Coordination services**: ZooKeeper, etcd

**Use When:**
- Data accuracy is critical
- Can tolerate downtime
- Financial transactions
- Inventory management

### AP (Availability + Partition Tolerance)

Sacrifice consistency to maintain availability.

**How it works:**
```
Network Partition:
User → Server A ─X─ Server B
        ↓              ↓
     Write $100     Read $50
     (succeeds)     (stale but available)
```

**Behavior:**
- System always responds
- May serve stale data
- Eventually becomes consistent

**Examples:**
- **Cassandra**: Always available, eventual consistency
- **DynamoDB**: Highly available
- **DNS**: Stale DNS records acceptable
- **Caching systems**: Redis, Memcached

**Use When:**
- Availability more important than consistency
- Can handle eventual consistency
- Social media feeds
- Product catalogs
- Recommendation systems

### CA (Consistency + Availability)

**Important:** Not possible in distributed systems with network partitions!

**Why?**
- Networks are unreliable
- Partitions will happen
- Must choose C or A when partition occurs

**Where CA exists:**
- Single server (no distribution)
- Traditional RDBMS on single machine
- Not fault-tolerant

## Real-World Examples

### Banking System (CP)

**Scenario:** Transfer $100 from Account A to Account B

```
Normal Operation:
1. Check balance A ≥ $100
2. Deduct $100 from A
3. Add $100 to B
4. Replicate to all nodes
5. Confirm success

Network Partition:
1. Check balance A ≥ $100
2. Try to replicate
3. Partition detected!
4. Abort transaction
5. Return error to user

Result: User gets error, but data stays consistent
```

**Trade-off:** Better to show error than wrong balance

### Social Media Feed (AP)

**Scenario:** User posts status update

```
Normal Operation:
1. Write to Server A
2. Replicate to Servers B, C, D
3. All servers updated

Network Partition:
1. Write to Server A (succeeds)
2. Can't reach Server B (partition)
3. Server A still accepts write
4. Users on Server B see old feed
5. When partition heals, sync

Result: Temporary inconsistency, but always available
```

**Trade-off:** Better to show slightly stale feed than error

## PACELC Extension

CAP doesn't tell the whole story. PACELC extends it:

**IF Partition:**
- Choose between Availability (A) or Consistency (C)

**ELSE (normal operation):**
- Choose between Latency (L) or Consistency (C)

### Systems Classified

**PA/EL (Prioritize Availability & Latency):**
- Cassandra
- DynamoDB
- Couchbase

**PC/EC (Prioritize Consistency):**
- MongoDB (with majority writes)
- HBase
- Redis (with replication)

**PA/EC:**
- Kafka
- Cosmos DB (tunable consistency)

## Consistency Models

Beyond strong consistency, distributed systems offer various levels:

### Strong Consistency

Read always returns latest write.

```
Time →
Write X → Read X  (always latest)
```

**Systems:** Traditional databases, ZooKeeper

### Eventual Consistency

System becomes consistent given enough time.

```
Time →
Write X → Read Y (stale) → Read Y (stale) → Read X (eventually consistent)
```

**Systems:** Cassandra, DynamoDB, DNS

### Read-Your-Writes Consistency

User sees their own writes immediately.

```
User A writes X → User A reads X  ✓ (sees own write)
User B reads Y  (might be stale)  ✓ (ok)
```

**Use:** User profiles, settings

### Causal Consistency

Causally related operations seen in order.

```
Comment reply must come after original comment
Original → Reply  (always in order)
```

**Use:** Social media threads, messaging

## Interview Questions

**Q: Explain CAP Theorem in simple terms.**

A: CAP Theorem says distributed systems can only provide 2 of 3 guarantees:
- **Consistency**: All nodes see same data
- **Availability**: System always responds
- **Partition Tolerance**: Works despite network failures

Since network partitions **will** happen, you must choose between **Consistency (CP)** or **Availability (AP)**.

**CP Example:** Banking - better unavailable than wrong balance
**AP Example:** Social media - better show stale feed than error

**Q: Why can't we have all three (C, A, P)?**

A: During a network partition:
- If you want **Consistency**: Must reject requests (lose Availability)
- If you want **Availability**: Must serve potentially stale data (lose Consistency)

You can't have both during partition because nodes can't communicate to synchronize data.

**Q: Is CAP Theorem relevant for single-server systems?**

A: No. CAP only applies to distributed systems with multiple nodes. Single servers don't have partitions, so they can be both consistent and available (CA).

**Q: How does Cassandra achieve high availability?**

A: Cassandra is AP (Available + Partition Tolerant):
- **Replication**: Data copied to multiple nodes
- **Tunable Consistency**: Configure read/write quorum
- **Hinted Handoff**: Stores writes when nodes down
- **Anti-entropy**: Background sync to fix inconsistencies
- **Always accepts writes**: Even during partitions

Trade-off: May serve stale data temporarily (eventual consistency)

**Q: When would you choose CP over AP?**

A: Choose **CP** (Consistency over Availability) when:
- Data accuracy is critical
- Incorrect data causes problems
- Can tolerate downtime

**Examples:**
- Financial transactions
- Inventory systems
- Booking systems (airplane seats, hotel rooms)
- Authentication systems

Choose **AP** (Availability over Consistency) when:
- Always being online is critical
- Temporary inconsistency is acceptable
- User experience matters more than perfect data

**Examples:**
- Social media feeds
- Product catalogs
- Shopping carts
- Recommendation systems

## Practical Implications

### Database Selection

**Need Strong Consistency (CP):**
- PostgreSQL (with synchronous replication)
- MongoDB (with majority writes)
- Google Spanner
- CockroachDB

**Need High Availability (AP):**
- Cassandra
- DynamoDB
- Riak
- Couchbase

**Tunable (Choose based on operation):**
- Cosmos DB (5 consistency levels)
- Cassandra (quorum levels)

### Designing for Partitions

**CP System:**
```javascript
async function transferMoney(from, to, amount) {
  const transaction = await db.beginTransaction();

  try {
    // Acquire locks
    await db.lock([from, to]);

    const fromBalance = await db.get(from);
    if (fromBalance < amount) {
      throw new Error('Insufficient funds');
    }

    await db.update(from, fromBalance - amount);
    await db.update(to, await db.get(to) + amount);

    // Wait for replication to majority
    await transaction.commit({ w: 'majority' });

    return { success: true };
  } catch (error) {
    await transaction.rollback();

    // Return error during partition
    return { success: false, error: 'Service unavailable' };
  }
}
```

**AP System:**
```javascript
async function updateFeed(userId, post) {
  try {
    // Always accept write
    await db.write(userId, post, { consistency: 'ONE' });

    // Replicate asynchronously
    replicateAsync(post);

    return { success: true };
  } catch (error) {
    // Retry later
    queue.enqueue('retry_post', { userId, post });
    return { success: true };  // Still return success
  }
}
```

## Best Practices

**System Design:**
✅ Understand your consistency requirements
✅ Know your SLAs (99.9% vs 99.99%)
✅ Design for partition handling
✅ Use appropriate consistency level
✅ Monitor replication lag

**Trade-offs:**
✅ Document consistency choices
✅ Communicate trade-offs to stakeholders
✅ Test partition scenarios
✅ Have fallback strategies
✅ Monitor system behavior during partitions

**Common Mistakes:**
❌ Assuming network is reliable
❌ Not testing partition scenarios
❌ Choosing wrong consistency model
❌ Not monitoring replication lag
❌ Ignoring partition tolerance

## Summary

- **CAP Theorem**: Can only guarantee 2 of 3 (Consistency, Availability, Partition Tolerance)
- **In practice**: Must choose between CP or AP (partitions will happen)
- **CP Systems**: Sacrifice availability for consistency (banks, inventory)
- **AP Systems**: Sacrifice consistency for availability (social media, catalogs)
- **PACELC**: Extends CAP to cover latency vs consistency trade-offs
- **No silver bullet**: Choose based on business requirements
- **Test partitions**: Chaos engineering to verify behavior
- Most modern systems are **tunable** - configure per operation

---
[← Back to SystemDesign](../README.md)
