# Consistency Models

## Overview
Consistency defines how and when changes made to a distributed system become visible to all nodes. Different consistency models offer different guarantees about the order and timing of updates, allowing systems to trade consistency for performance, availability, or partition tolerance.

## Consistency Spectrum

### Strong Consistency

All nodes see the same data at the same time. Read always returns the most recent write.

**Characteristics:**
- Linearizability: Operations appear atomic
- All clients see updates in the same order
- Highest correctness guarantee
- Lowest availability and performance

**Example:**
```
Time →
Client A writes X=1    ──────────────▶ All nodes immediately see X=1
Client B reads X       ──────────────▶ Always returns 1 (latest)
```

**Use Cases:**
- Financial transactions
- Inventory management
- Booking systems (seats, rooms)
- Authentication systems

**Systems:**
- Google Spanner
- CockroachDB
- etcd, ZooKeeper
- Traditional RDBMS with synchronous replication

### Eventual Consistency

All replicas will eventually converge to the same value if no new updates are made.

**Characteristics:**
- Updates propagate asynchronously
- Temporary inconsistency allowed
- High availability and performance
- No ordering guarantees

**Example:**
```
Time →
Client A writes X=1    ──▶ Node A: X=1
                            Node B: X=0 (stale)
                            Node C: X=0 (stale)

After replication delay:
                            Node A: X=1
                            Node B: X=1 (synced)
                            Node C: X=1 (synced)
```

**Use Cases:**
- Social media feeds
- Product catalogs
- Shopping carts
- DNS records
- Caching layers

**Systems:**
- Cassandra
- DynamoDB
- Riak
- Memcached, Redis (replicas)

### Causal Consistency

Causally related operations are seen in the same order by all nodes.

**Characteristics:**
- Preserves cause-effect relationships
- Independent operations can be seen in different orders
- Stronger than eventual, weaker than strong

**Example:**
```
User A posts comment    ──▶ All nodes see original first
User B replies to comment ──▶ Then see reply (causally related)

User C likes post (independent) ──▶ Can appear before/after reply
```

**Implementation:**
```javascript
class CausalConsistency {
  constructor() {
    this.vectorClock = {};
  }

  write(nodeId, data) {
    // Increment vector clock
    this.vectorClock[nodeId] = (this.vectorClock[nodeId] || 0) + 1;

    return {
      data,
      timestamp: { ...this.vectorClock }
    };
  }

  canRead(event1, event2) {
    // event2 can read event1 if event1 happened before event2
    return this.happensBefore(event1.timestamp, event2.timestamp);
  }

  happensBefore(t1, t2) {
    for (const nodeId in t1) {
      if (t1[nodeId] > (t2[nodeId] || 0)) {
        return false;
      }
    }
    return true;
  }
}
```

**Use Cases:**
- Comment threads
- Collaborative editing
- Messaging apps
- Version control systems

**Systems:**
- Cosmos DB (with causal consistency level)
- COPS (Clusters of Order-Preserving Servers)

### Read-Your-Writes Consistency

Users always see their own updates immediately.

**Characteristics:**
- Session consistency
- User-specific guarantee
- Other users may see stale data
- Good user experience

**Example:**
```
User A updates profile  ──▶ User A immediately sees update
                            User B might see old profile (eventually consistent)
```

**Implementation:**
```javascript
class SessionConsistency {
  constructor() {
    this.sessionVersions = new Map();
  }

  async write(userId, key, value) {
    const version = Date.now();

    // Write to database
    await db.write(key, value, version);

    // Track user's latest version
    this.sessionVersions.set(userId, version);

    return version;
  }

  async read(userId, key) {
    const userVersion = this.sessionVersions.get(userId) || 0;

    // Read from replica that has at least user's version
    const replicas = await this.getReplicasWithVersion(userVersion);

    if (replicas.length > 0) {
      return replicas[0].read(key);
    }

    // Wait for replication
    await this.waitForReplication(userVersion);
    return db.read(key);
  }
}
```

**Use Cases:**
- User profiles and settings
- Shopping cart updates
- Social media posts (own posts)
- Document editing (own changes)

### Monotonic Reads

Once a client reads a value, it will never read an older value.

**Characteristics:**
- Prevents reading stale data after reading fresh data
- Forward progress guarantee
- Can read same value multiple times

**Example:**
```
Time →
Read 1: X=5   ──▶ Node A
Read 2: X=7   ──▶ Node B (ok, newer)
Read 3: X=3   ──▶ ❌ Violation (older than X=7)
Read 3: X=7   ──▶ ✓ Ok (same or newer)
```

**Use Cases:**
- Feed readers
- News feeds
- Email inbox
- Notification systems

### Monotonic Writes

Client's writes are applied in the order they were issued.

**Characteristics:**
- Preserves write ordering from single client
- Different clients can have different orders
- Prevents write reordering

**Example:**
```
Client writes:
X=1  ──▶ Applied first
X=2  ──▶ Applied second
X=3  ──▶ Applied third

Never: X=2, X=1, X=3 (reordered)
```

## Consistency Protocols

### Two-Phase Commit (2PC)

Distributed transaction protocol ensuring atomic commits across multiple nodes.

**Phases:**

**Phase 1: Prepare (Voting)**
```
Coordinator → Participants: "Can you commit?"
Participants → Coordinator: "Yes" or "No"
```

**Phase 2: Commit (Decision)**
```
Coordinator → Participants: "Commit" or "Abort"
Participants: Execute and acknowledge
```

**Implementation:**
```javascript
class TwoPhaseCommit {
  async executeTransaction(participants, transaction) {
    // Phase 1: Prepare
    const votes = await Promise.all(
      participants.map(p => p.prepare(transaction))
    );

    const allVotedYes = votes.every(v => v === 'YES');

    if (allVotedYes) {
      // Phase 2: Commit
      await Promise.all(
        participants.map(p => p.commit(transaction))
      );
      return 'COMMITTED';
    } else {
      // Phase 2: Abort
      await Promise.all(
        participants.map(p => p.abort(transaction))
      );
      return 'ABORTED';
    }
  }
}

class Participant {
  async prepare(transaction) {
    try {
      // Validate transaction
      await this.validate(transaction);

      // Lock resources
      await this.lock(transaction);

      // Write to transaction log
      await this.log('PREPARED', transaction);

      return 'YES';
    } catch (error) {
      return 'NO';
    }
  }

  async commit(transaction) {
    // Apply changes
    await this.apply(transaction);

    // Release locks
    await this.unlock(transaction);

    // Log commit
    await this.log('COMMITTED', transaction);
  }

  async abort(transaction) {
    // Rollback changes
    await this.rollback(transaction);

    // Release locks
    await this.unlock(transaction);

    // Log abort
    await this.log('ABORTED', transaction);
  }
}
```

**Problems:**
- **Blocking**: If coordinator fails, participants wait indefinitely
- **Single Point of Failure**: Coordinator failure stops system
- **Performance**: Multiple round trips

**Solutions:**
- **Three-Phase Commit (3PC)**: Adds pre-commit phase
- **Timeouts**: Participants timeout and abort
- **Coordinator Replication**: Multiple coordinators

### Paxos

Consensus algorithm for agreeing on a single value in distributed systems.

**Roles:**
- **Proposer**: Proposes values
- **Acceptor**: Votes on proposals
- **Learner**: Learns chosen value

**Phases:**

**Phase 1: Prepare**
```
Proposer → Acceptors: Prepare(n)
Acceptors → Proposer: Promise(n, accepted_value) if n > previous_n
```

**Phase 2: Accept**
```
Proposer → Acceptors: Accept(n, value)
Acceptors → Learners: Accepted(n, value) if n >= promised_n
```

**Simplified Implementation:**
```javascript
class PaxosAcceptor {
  constructor() {
    this.promisedN = 0;
    this.acceptedN = 0;
    this.acceptedValue = null;
  }

  prepare(n) {
    if (n > this.promisedN) {
      this.promisedN = n;
      return {
        promise: true,
        acceptedN: this.acceptedN,
        acceptedValue: this.acceptedValue
      };
    }
    return { promise: false };
  }

  accept(n, value) {
    if (n >= this.promisedN) {
      this.promisedN = n;
      this.acceptedN = n;
      this.acceptedValue = value;
      return { accepted: true };
    }
    return { accepted: false };
  }
}

class PaxosProposer {
  constructor(acceptors) {
    this.acceptors = acceptors;
    this.n = 0;
  }

  async propose(value) {
    this.n++;

    // Phase 1: Prepare
    const promises = await Promise.all(
      this.acceptors.map(a => a.prepare(this.n))
    );

    const promiseCount = promises.filter(p => p.promise).length;
    const majority = Math.floor(this.acceptors.length / 2) + 1;

    if (promiseCount < majority) {
      throw new Error('Prepare phase failed');
    }

    // Use highest accepted value or proposed value
    const highestAccepted = promises
      .filter(p => p.acceptedValue !== null)
      .sort((a, b) => b.acceptedN - a.acceptedN)[0];

    const finalValue = highestAccepted ? highestAccepted.acceptedValue : value;

    // Phase 2: Accept
    const accepts = await Promise.all(
      this.acceptors.map(a => a.accept(this.n, finalValue))
    );

    const acceptCount = accepts.filter(a => a.accepted).length;

    if (acceptCount >= majority) {
      return finalValue;
    }

    throw new Error('Accept phase failed');
  }
}
```

**Use Cases:**
- Google Chubby
- Apache ZooKeeper (ZAB - similar to Paxos)
- Distributed lock services

### Raft

Easier-to-understand consensus algorithm similar to Paxos.

**Components:**
- **Leader**: Handles all client requests
- **Follower**: Passive, replicate leader's log
- **Candidate**: Follower trying to become leader

**Leader Election:**
```
1. Follower timeout → becomes Candidate
2. Candidate requests votes
3. Majority vote → becomes Leader
4. Leader sends heartbeats
```

**Log Replication:**
```
Client → Leader: Request
Leader → Followers: AppendEntries
Followers → Leader: Acknowledgment
Leader: Commit when majority acks
Leader → Client: Response
```

**Implementation:**
```javascript
class RaftNode {
  constructor(id, peers) {
    this.id = id;
    this.peers = peers;
    this.state = 'FOLLOWER'; // FOLLOWER, CANDIDATE, LEADER
    this.currentTerm = 0;
    this.votedFor = null;
    this.log = [];
    this.commitIndex = 0;
    this.lastApplied = 0;
  }

  startElection() {
    this.state = 'CANDIDATE';
    this.currentTerm++;
    this.votedFor = this.id;

    let votes = 1; // Vote for self
    const majority = Math.floor(this.peers.length / 2) + 1;

    this.peers.forEach(peer => {
      peer.requestVote(this.currentTerm, this.id, this.log.length - 1)
        .then(response => {
          if (response.voteGranted) {
            votes++;
            if (votes >= majority) {
              this.becomeLeader();
            }
          }
        });
    });
  }

  becomeLeader() {
    this.state = 'LEADER';
    console.log(`Node ${this.id} became leader for term ${this.currentTerm}`);

    // Send heartbeats
    this.sendHeartbeats();
  }

  async requestVote(term, candidateId, lastLogIndex) {
    if (term > this.currentTerm) {
      this.currentTerm = term;
      this.votedFor = null;
      this.state = 'FOLLOWER';
    }

    if (term < this.currentTerm) {
      return { voteGranted: false };
    }

    if (this.votedFor === null || this.votedFor === candidateId) {
      if (lastLogIndex >= this.log.length - 1) {
        this.votedFor = candidateId;
        return { voteGranted: true };
      }
    }

    return { voteGranted: false };
  }

  async appendEntries(term, leaderId, entries, leaderCommit) {
    if (term >= this.currentTerm) {
      this.currentTerm = term;
      this.state = 'FOLLOWER';

      // Append entries to log
      this.log.push(...entries);

      // Update commit index
      if (leaderCommit > this.commitIndex) {
        this.commitIndex = Math.min(leaderCommit, this.log.length - 1);
      }

      return { success: true };
    }

    return { success: false };
  }

  sendHeartbeats() {
    if (this.state !== 'LEADER') return;

    this.peers.forEach(peer => {
      peer.appendEntries(this.currentTerm, this.id, [], this.commitIndex);
    });

    // Schedule next heartbeat
    setTimeout(() => this.sendHeartbeats(), 150);
  }
}
```

**Advantages over Paxos:**
- Easier to understand and implement
- Leader-based (clearer roles)
- Better performance in practice

**Use Cases:**
- etcd (Kubernetes)
- Consul
- CockroachDB

## Conflict Resolution

### Vector Clocks

Track causality between events in distributed systems.

**How it works:**
```
Each node maintains a vector: [Node1: n, Node2: n, Node3: n]

Event happens:
1. Increment own counter
2. Send vector with event
3. On receive: merge vectors (take max of each)
```

**Implementation:**
```javascript
class VectorClock {
  constructor(nodeId, nodeCount) {
    this.nodeId = nodeId;
    this.clock = new Array(nodeCount).fill(0);
  }

  increment() {
    this.clock[this.nodeId]++;
  }

  update(otherClock) {
    for (let i = 0; i < this.clock.length; i++) {
      this.clock[i] = Math.max(this.clock[i], otherClock[i]);
    }
    this.increment();
  }

  happensBefore(otherClock) {
    let atLeastOneLess = false;

    for (let i = 0; i < this.clock.length; i++) {
      if (this.clock[i] > otherClock[i]) {
        return false;
      }
      if (this.clock[i] < otherClock[i]) {
        atLeastOneLess = true;
      }
    }

    return atLeastOneLess;
  }

  concurrent(otherClock) {
    return !this.happensBefore(otherClock) &&
           !new VectorClock().happensBefore.call({ clock: otherClock }, this.clock);
  }
}

// Usage
const node1 = new VectorClock(0, 3); // [0, 0, 0]
const node2 = new VectorClock(1, 3); // [0, 0, 0]

node1.increment(); // [1, 0, 0]
node2.increment(); // [0, 1, 0]

node1.update(node2.clock); // [1, 1, 0] (merge)
```

**Use Cases:**
- Detecting concurrent updates
- Conflict resolution in Dynamo-style databases
- Version control systems

### Last-Write-Wins (LWW)

Simple conflict resolution based on timestamps.

**Implementation:**
```javascript
class LWWRegister {
  constructor() {
    this.value = null;
    this.timestamp = 0;
  }

  write(value) {
    const now = Date.now();
    if (now > this.timestamp) {
      this.value = value;
      this.timestamp = now;
    }
  }

  merge(otherValue, otherTimestamp) {
    if (otherTimestamp > this.timestamp) {
      this.value = otherValue;
      this.timestamp = otherTimestamp;
    } else if (otherTimestamp === this.timestamp) {
      // Tie-breaker: use value comparison
      if (otherValue > this.value) {
        this.value = otherValue;
      }
    }
  }
}
```

**Problems:**
- **Clock skew**: Different server times
- **Data loss**: Loses concurrent writes

**Solutions:**
- Use **hybrid logical clocks**
- Combine with node ID for tie-breaking

### Application-Level Conflict Resolution

Let application decide how to merge conflicts.

**Example: Shopping Cart**
```javascript
class ShoppingCart {
  constructor() {
    this.items = new Map();
  }

  merge(otherCart) {
    // Union of items (additive merge)
    for (const [itemId, quantity] of otherCart.items) {
      const currentQty = this.items.get(itemId) || 0;
      this.items.set(itemId, currentQty + quantity);
    }
  }
}

// User adds item on mobile: [Item1: 1]
// User adds item on web:    [Item2: 1]
// Merge result:              [Item1: 1, Item2: 1]
```

**Example: Collaborative Document**
```javascript
class CRDTText {
  constructor() {
    this.characters = [];
  }

  insert(position, char, timestamp, nodeId) {
    this.characters.push({
      position,
      char,
      timestamp,
      nodeId,
      deleted: false
    });
    this.characters.sort((a, b) => {
      if (a.position !== b.position) return a.position - b.position;
      if (a.timestamp !== b.timestamp) return a.timestamp - b.timestamp;
      return a.nodeId - b.nodeId;
    });
  }

  delete(position) {
    const char = this.characters.find(c => c.position === position);
    if (char) char.deleted = true;
  }

  getText() {
    return this.characters
      .filter(c => !c.deleted)
      .map(c => c.char)
      .join('');
  }
}
```

## Quorum-Based Consistency

### Quorum Reads and Writes

Ensures consistency through overlapping read and write sets.

**Formula:**
```
R + W > N

R = Read quorum
W = Write quorum
N = Total replicas

Example:
N = 3 replicas
W = 2 (write to 2 replicas)
R = 2 (read from 2 replicas)
R + W = 4 > 3 ✓ (guarantees overlap)
```

**Implementation:**
```javascript
class QuorumStore {
  constructor(replicas, readQuorum, writeQuorum) {
    this.replicas = replicas;
    this.R = readQuorum;
    this.W = writeQuorum;
    this.N = replicas.length;

    // Ensure R + W > N
    if (this.R + this.W <= this.N) {
      throw new Error('Invalid quorum: R + W must be > N');
    }
  }

  async write(key, value) {
    const version = Date.now();
    const writes = this.replicas.map(replica =>
      replica.write(key, value, version)
    );

    // Wait for W successful writes
    const results = await Promise.allSettled(writes);
    const successful = results.filter(r => r.status === 'fulfilled');

    if (successful.length >= this.W) {
      return { success: true, version };
    }

    throw new Error(`Write failed: only ${successful.length} of ${this.W} succeeded`);
  }

  async read(key) {
    const reads = this.replicas.map(replica =>
      replica.read(key)
    );

    // Wait for R successful reads
    const results = await Promise.race([
      this.waitForQuorum(reads, this.R),
      this.timeout(5000)
    ]);

    // Return value with highest version
    const latest = results.reduce((max, current) =>
      current.version > max.version ? current : max
    );

    return latest.value;
  }

  async waitForQuorum(promises, count) {
    const results = [];

    for (const promise of promises) {
      try {
        const result = await promise;
        results.push(result);

        if (results.length >= count) {
          return results;
        }
      } catch (error) {
        // Continue waiting
      }
    }

    throw new Error('Quorum not reached');
  }
}
```

**Tuning Consistency:**

**Strong Consistency:**
```
R = W = N
Read and write from all replicas
Highest consistency, lowest availability
```

**Eventual Consistency:**
```
R = 1, W = 1
Read/write from any single replica
Highest availability, lowest consistency
```

**Balanced:**
```
R = W = (N/2) + 1
Example: N=3, R=2, W=2
Good balance of consistency and availability
```

## Interview Questions

**Q: What's the difference between strong consistency and eventual consistency?**

A:
- **Strong Consistency**: All reads return the most recent write. All nodes see the same data at the same time.
  - Example: Bank balance must be accurate
  - Trade-off: Lower availability, higher latency
  - Systems: Google Spanner, traditional RDBMS

- **Eventual Consistency**: All replicas will converge to the same value eventually, but may be temporarily inconsistent.
  - Example: Social media feed can show slightly stale data
  - Trade-off: Higher availability, lower latency
  - Systems: Cassandra, DynamoDB

**Q: Explain the CAP theorem and how it relates to consistency.**

A: CAP theorem states you can only have 2 of 3:
- **C**onsistency: All nodes see same data
- **A**vailability: System always responds
- **P**artition tolerance: Works despite network failures

Since network partitions **will** happen, you must choose:
- **CP**: Strong consistency, sacrifice availability (banking)
- **AP**: Eventual consistency, sacrifice consistency (social media)

**Q: How does Raft ensure consistency?**

A: Raft uses leader-based replication:

1. **Single Leader**: All writes go through leader
2. **Log Replication**: Leader replicates log entries to followers
3. **Majority Commit**: Entry committed when majority acknowledges
4. **Term-based Election**: Higher term wins leadership
5. **Log Matching**: Ensures all committed entries are durable

This guarantees **linearizability** (strong consistency) because:
- Leader serializes all writes
- Commits only after majority replication
- New leader has all committed entries

**Q: What are vector clocks and when would you use them?**

A: Vector clocks track causality in distributed systems. Each node maintains a counter for itself and all other nodes.

**When to use:**
- Detecting concurrent updates
- Resolving conflicts in multi-master replication
- Version control systems
- Systems: Riak, Voldemort

**Example:**
```
Node A writes: [A:1, B:0, C:0]
Node B writes: [A:0, B:1, C:0]

These are concurrent (neither happened before the other)
Application must resolve conflict
```

**Q: How do you achieve read-your-writes consistency?**

A: Several strategies:

1. **Read from Leader**: Direct reads to master (always up-to-date)
   ```javascript
   if (recentWrite) {
     return readFromLeader(key);
   }
   ```

2. **Track Version**: Store user's last write version
   ```javascript
   const userVersion = session.getLastWriteVersion(userId);
   return readFromReplicaWithVersion(key, userVersion);
   ```

3. **Session Affinity**: Route user to same replica
   ```javascript
   const replicaId = hash(userId) % replicaCount;
   return replicas[replicaId].read(key);
   ```

4. **Quorum Reads**: Read from majority (includes latest write)
   ```javascript
   return quorumRead(key, replicas, quorum=2);
   ```

**Q: Compare Two-Phase Commit and Raft.**

A:

**Two-Phase Commit (2PC):**
- **Purpose**: Atomic distributed transactions
- **Coordinator-based**: Single coordinator
- **Blocking**: Participants block if coordinator fails
- **Use case**: Database distributed transactions

**Raft:**
- **Purpose**: Consensus and replication
- **Leader-based**: Leader elected by majority
- **Non-blocking**: New leader elected if current fails
- **Use case**: Replicated state machines

**Key difference**: 2PC ensures atomicity (all-or-nothing), Raft ensures consistency (agreement on order).

## Best Practices

**Choosing Consistency Model:**
✅ Use **strong consistency** for:
- Financial transactions
- Inventory management
- Booking systems
- Authentication

✅ Use **eventual consistency** for:
- Social media feeds
- Product catalogs
- Recommendation engines
- Analytics data

✅ Use **causal consistency** for:
- Comment threads
- Messaging apps
- Collaborative editing

✅ Use **read-your-writes** for:
- User profiles
- Settings
- Personal data

**Implementation:**
✅ Monitor replication lag
✅ Set appropriate timeouts
✅ Use quorum for critical operations
✅ Implement conflict resolution
✅ Test partition scenarios
✅ Document consistency guarantees

**Common Mistakes:**
❌ Assuming strong consistency by default
❌ Ignoring replication lag
❌ Not handling concurrent updates
❌ Using LWW without understanding data loss
❌ Not testing network partitions
❌ Forgetting to version data

## Summary

- **Consistency models** trade correctness for performance and availability
- **Strong consistency** (linearizability) provides highest correctness but lowest availability
- **Eventual consistency** provides highest availability but temporary inconsistency
- **Causal consistency** preserves cause-effect relationships
- **Session consistency** (read-your-writes) ensures good user experience
- **2PC** provides atomic commits but can block
- **Paxos** achieves consensus but is complex
- **Raft** is easier to understand than Paxos with similar guarantees
- **Vector clocks** track causality and detect conflicts
- **Quorum-based systems** tune consistency with R + W > N
- Choose consistency model based on business requirements
- Always test consistency under partition and failure scenarios

---
[← Back to SystemDesign](../README.md)
