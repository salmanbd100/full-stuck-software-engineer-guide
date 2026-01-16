# Transactions & ACID Properties

## 💡 **Overview**

Transactions are sequences of database operations that are treated as a single unit of work.

**What You'll Learn:**
- ACID properties and why they matter
- Transaction control commands (BEGIN, COMMIT, ROLLBACK)
- Isolation levels and concurrency control
- Locking mechanisms and deadlock prevention
- Real-world transaction patterns

**Why This Matters:**
- Ensures data integrity in multi-step operations
- Critical for financial transactions, inventory management, and concurrent operations
- Frequently tested in backend and database interviews
- Essential for building reliable, production-ready applications

---

## 💡 **What is a Transaction?**

A transaction is a sequence of one or more SQL operations treated as a single logical unit of work.

**How It Works:**

All operations in a transaction either:
- **Commit**: All changes are permanently saved
- **Rollback**: All changes are discarded

**Basic Syntax:**

```sql
-- Start transaction
START TRANSACTION;  -- or BEGIN;

-- Perform operations
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Save changes
COMMIT;

-- OR undo changes
ROLLBACK;
```

**Real-World Example: Money Transfer**

```sql
-- Transfer $100 from Account 1 to Account 2
START TRANSACTION;

-- Step 1: Deduct from sender
UPDATE accounts
SET balance = balance - 100
WHERE id = 1 AND balance >= 100;  -- Check sufficient funds

-- Step 2: Add to recipient
UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

-- Step 3: Record transaction
INSERT INTO transaction_history (from_account, to_account, amount, timestamp)
VALUES (1, 2, 100, NOW());

-- All succeed or all fail
COMMIT;  -- Save all changes
-- OR
ROLLBACK;  -- Undo all changes if something went wrong
```

**Why Transactions Are Critical:**

```
Without Transaction:
  Deduct $100 from Account 1 ✅
  [System crashes here] 💥
  Add $100 to Account 2 ❌
  Result: $100 lost forever!

With Transaction:
  START TRANSACTION
  Deduct $100 from Account 1 ✅
  [System crashes here] 💥
  Result: Automatic ROLLBACK - no money lost!
```

> **Key Insight:** Transactions ensure that either all operations succeed together, or none of them happen at all. This prevents partial updates that could corrupt your data.

---

## 💡 **ACID Properties**

ACID defines the key properties that guarantee reliable transaction processing.

### **A - Atomicity**

All operations succeed or all fail - no partial transactions.

**How It Works:**

Transaction is treated as a single atomic unit. If any operation fails, the entire transaction is rolled back.

**Example:**

```sql
START TRANSACTION;

INSERT INTO orders (user_id, total) VALUES (1, 100);
INSERT INTO order_items (order_id, product_id, quantity) VALUES (LAST_INSERT_ID(), 5, 2);

COMMIT;  -- Both inserts succeed together
```

**Real-World Scenario:**

| Scenario | Without Atomicity | With Atomicity |
|----------|-------------------|----------------|
| Create order + order items | Order created, items fail | Both succeed or both fail |
| Money transfer | Money deducted but not added | Both succeed or both fail |
| User signup + profile | User created, profile fails | Both succeed or both fail |

**Benefits:**
- ✅ Prevents orphaned records (order without items)
- ✅ Ensures data consistency
- ✅ Simplifies error handling

---

### **C - Consistency**

Database moves from one valid state to another, maintaining all constraints.

**How It Works:**

Transaction must satisfy all defined rules (constraints, triggers, cascades) or be rolled back.

**Examples:**

```sql
-- Constraint: Balance cannot be negative
CREATE TABLE accounts (
  id INT PRIMARY KEY,
  balance DECIMAL(10,2) CHECK (balance >= 0)  -- Enforces consistency
);

START TRANSACTION;
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
-- If balance would go negative, transaction fails (consistency maintained)
ROLLBACK;
```

**Consistency Rules:**

| Rule Type | Example | Enforcement |
|-----------|---------|-------------|
| **CHECK Constraints** | `balance >= 0` | Database rejects invalid values |
| **FOREIGN Keys** | `user_id` must exist in users | Prevents orphaned records |
| **UNIQUE Constraints** | Email must be unique | Prevents duplicates |
| **NOT NULL** | Required fields | Prevents incomplete data |
| **Triggers** | Update timestamps | Automatically maintains data |

**Real-World Example:**

```sql
-- E-commerce: Stock must not go negative
START TRANSACTION;

-- Check stock before order
SELECT stock FROM products WHERE id = 1 FOR UPDATE;  -- Lock row

-- If stock >= quantity ordered, proceed
UPDATE products SET stock = stock - 5 WHERE id = 1 AND stock >= 5;

IF ROW_COUNT() = 0 THEN
  ROLLBACK;  -- Not enough stock, maintain consistency
ELSE
  COMMIT;
END IF;
```

---

### **I - Isolation**

Concurrent transactions don't interfere with each other.

**How It Works:**

Each transaction appears to execute in isolation, even when multiple transactions run simultaneously.

**Without Isolation:**

```
Transaction A: Read balance = 100
Transaction B: Read balance = 100
Transaction A: Deduct 50 → balance = 50
Transaction B: Deduct 30 → balance = 70
Result: Wrong! Should be 20, not 70
```

**With Isolation:**

```
Transaction A: Read balance = 100, Lock
Transaction A: Deduct 50 → balance = 50, Commit
Transaction B: Read balance = 50 (waits for lock)
Transaction B: Deduct 30 → balance = 20, Commit
Result: Correct! balance = 20
```

**Isolation Levels** (covered in detail below)

---

### **D - Durability**

Committed changes persist even after system failure.

**How It Works:**

Once a transaction is committed, changes are permanently recorded (written to disk), surviving crashes, power failures, etc.

**Example:**

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance + 1000 WHERE id = 1;
COMMIT;  -- Changes written to disk

[System crashes here] 💥

-- After restart: balance update still there!
```

**Durability Mechanisms:**

| Mechanism | Purpose |
|-----------|---------|
| **Write-Ahead Logging (WAL)** | Changes logged before being written to disk |
| **Transaction Log** | Records all changes for recovery |
| **Checkpoints** | Periodic flush of changes to disk |
| **Replication** | Copies to other servers for redundancy |

> **Key Insight:** ACID properties work together to ensure reliable, consistent data even in the face of failures, crashes, and concurrent operations.

---

## 💡 **Transaction Control Commands**

### **BEGIN / START TRANSACTION**

Starts a new transaction.

```sql
-- Both are equivalent
START TRANSACTION;
BEGIN;
```

### **COMMIT**

Permanently saves all changes made in the transaction.

```sql
START TRANSACTION;
UPDATE users SET status = 'active' WHERE id = 1;
COMMIT;  -- Changes are now permanent
```

### **ROLLBACK**

Discards all changes made in the transaction.

```sql
START TRANSACTION;
DELETE FROM users WHERE id = 1;
ROLLBACK;  -- User not deleted, changes discarded
```

### **SAVEPOINT**

Creates a point within a transaction to partially rollback.

```sql
START TRANSACTION;

UPDATE users SET name = 'John' WHERE id = 1;

SAVEPOINT before_delete;  -- Create savepoint

DELETE FROM orders WHERE user_id = 1;

ROLLBACK TO SAVEPOINT before_delete;  -- Undo delete, keep update

COMMIT;  -- Commits the update only
```

**Savepoint Use Cases:**

- ✅ Complex transactions with multiple steps
- ✅ Conditional rollback of partial changes
- ✅ Nested transaction-like behavior

---

## 💡 **Isolation Levels**

Isolation levels control the degree to which transactions are isolated from each other.

**Trade-off:**

```
Higher Isolation → More Consistency → Less Concurrency → Lower Performance
Lower Isolation → Less Consistency → More Concurrency → Higher Performance
```

### **Isolation Levels Comparison**

| Level | Dirty Reads | Non-Repeatable Reads | Phantom Reads | Concurrency | Use When |
|-------|-------------|---------------------|---------------|-------------|----------|
| **READ UNCOMMITTED** | ✅ Possible | ✅ Possible | ✅ Possible | Highest | Rarely (analytics only) |
| **READ COMMITTED** | ❌ Prevented | ✅ Possible | ✅ Possible | High | Most applications (default in PostgreSQL) |
| **REPEATABLE READ** | ❌ Prevented | ❌ Prevented | ✅ Possible | Medium | Financial transactions |
| **SERIALIZABLE** | ❌ Prevented | ❌ Prevented | ❌ Prevented | Lowest | Critical operations |

### **1. READ UNCOMMITTED** (Lowest Isolation)

Can read uncommitted changes from other transactions.

**Example Problem: Dirty Read**

```sql
-- Transaction A
START TRANSACTION;
UPDATE accounts SET balance = 1000 WHERE id = 1;
-- Not yet committed

-- Transaction B (READ UNCOMMITTED)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT balance FROM accounts WHERE id = 1;  -- Reads 1000 (uncommitted!)

-- Transaction A
ROLLBACK;  -- Change discarded

-- Transaction B read invalid data!
```

**When to Use:** Almost never. Only for rough analytics where accuracy isn't critical.

---

### **2. READ COMMITTED** (Most Common)

Can only read committed data. Prevents dirty reads.

**Example:**

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

START TRANSACTION;
-- Will only see committed changes from other transactions
SELECT * FROM accounts WHERE id = 1;
COMMIT;
```

**Problem: Non-Repeatable Reads**

```sql
-- Transaction A
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1;  -- Reads 100

-- Transaction B
UPDATE accounts SET balance = 200 WHERE id = 1;
COMMIT;

-- Transaction A
SELECT balance FROM accounts WHERE id = 1;  -- Reads 200 (different!)
COMMIT;
```

**When to Use:**
- ✅ Most web applications
- ✅ When you can tolerate data changing between reads
- ✅ Default in PostgreSQL

---

### **3. REPEATABLE READ**

Ensures same query returns same result within a transaction.

**Example:**

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1;  -- Reads 100

-- Another transaction changes balance to 200

SELECT balance FROM accounts WHERE id = 1;  -- Still reads 100
COMMIT;
```

**Problem: Phantom Reads**

```sql
-- Transaction A
START TRANSACTION;
SELECT COUNT(*) FROM orders WHERE status = 'pending';  -- Returns 5

-- Transaction B
INSERT INTO orders (status) VALUES ('pending');
COMMIT;

-- Transaction A
SELECT COUNT(*) FROM orders WHERE status = 'pending';  -- Returns 6 (phantom!)
COMMIT;
```

**When to Use:**
- ✅ Financial transactions
- ✅ When consistent reads are critical
- ✅ Default in MySQL InnoDB

---

### **4. SERIALIZABLE** (Highest Isolation)

Transactions execute as if they run sequentially (one after another).

**Example:**

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

START TRANSACTION;
SELECT * FROM accounts WHERE balance > 1000 FOR UPDATE;
-- Locks all matching rows and prevents new rows matching condition
COMMIT;
```

**When to Use:**
- ✅ Critical operations (banking, financial)
- ✅ When absolute consistency is required
- ⚠️ Lower concurrency, higher chance of conflicts

---

### **Setting Isolation Levels**

```sql
-- MySQL
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
-- queries
COMMIT;

-- PostgreSQL
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- queries
COMMIT;

-- Session-level
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

---

## 💡 **Locking Mechanisms**

Locks prevent concurrent transactions from interfering with each other.

### **Lock Types**

| Lock Type | Purpose | Allows | Blocks |
|-----------|---------|--------|--------|
| **Shared Lock (S)** | Reading | Other reads | Writes |
| **Exclusive Lock (X)** | Writing | Nothing | Reads and writes |

### **Shared Lock (Read Lock)**

Allows multiple transactions to read, prevents writes.

```sql
-- PostgreSQL
SELECT * FROM accounts WHERE id = 1 FOR SHARE;

-- MySQL
SELECT * FROM accounts WHERE id = 1 LOCK IN SHARE MODE;
```

**Use Case:**
- ✅ Ensure data doesn't change while reading
- ✅ Allow multiple readers

---

### **Exclusive Lock (Write Lock)**

Blocks all other access (reads and writes).

```sql
-- Locks row for update
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- Now only this transaction can read or write this row
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;  -- Releases lock
```

**Use Case:**
- ✅ Prevent race conditions
- ✅ Ensure atomic read-modify-write operations

**Example: Preventing Double-Booking**

```sql
START TRANSACTION;

-- Lock the seat row
SELECT available FROM seats WHERE id = 1 FOR UPDATE;

-- Check availability
IF available = TRUE THEN
  UPDATE seats SET available = FALSE WHERE id = 1;
  INSERT INTO bookings (seat_id, user_id) VALUES (1, 123);
  COMMIT;
ELSE
  ROLLBACK;
END IF;
```

---

## 💡 **Deadlocks**

A deadlock occurs when two or more transactions wait for each other to release locks.

**How Deadlocks Happen:**

```
Transaction A: Locks Row 1, Waits for Row 2
Transaction B: Locks Row 2, Waits for Row 1
Result: Both wait forever! 💀
```

**Example:**

```sql
-- Transaction A
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Locks Row 1
-- Waiting for Transaction B to release Row 2
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Waits...

-- Transaction B
START TRANSACTION;
UPDATE accounts SET balance = balance - 50 WHERE id = 2;   -- Locks Row 2
-- Waiting for Transaction A to release Row 1
UPDATE accounts SET balance = balance + 50 WHERE id = 1;   -- Waits...

-- DEADLOCK! 💀
```

### **Deadlock Prevention Strategies**

| Strategy | How It Works | Example |
|----------|--------------|---------|
| **Consistent Lock Order** | Always lock resources in same order | Always lock lower ID first |
| **Lock Timeout** | Automatic rollback after timeout | `innodb_lock_wait_timeout = 50` |
| **Shorter Transactions** | Reduce lock hold time | Keep transactions small and fast |
| **Avoid User Input** | Don't wait for user during transaction | Get user input before transaction |
| **Retry Logic** | Detect deadlock and retry | Catch deadlock exception and retry |

**✅ Good: Consistent Lock Order**

```sql
-- Always lock accounts in ascending ID order
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Lower ID first
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Higher ID second
COMMIT;
```

**❌ Bad: Inconsistent Lock Order**

```sql
-- Transaction A locks 1 then 2
-- Transaction B locks 2 then 1
-- Can cause deadlock!
```

### **Handling Deadlocks**

```javascript
async function transferWithRetry(fromId, toId, amount, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      await db.query('START TRANSACTION');

      // Perform operations
      await db.query('UPDATE accounts SET balance = balance - ? WHERE id = ?', [amount, fromId]);
      await db.query('UPDATE accounts SET balance = balance + ? WHERE id = ?', [amount, toId]);

      await db.query('COMMIT');
      return { success: true };

    } catch (error) {
      await db.query('ROLLBACK');

      // Deadlock detected (MySQL error code 1213)
      if (error.code === '1213' && attempt < maxRetries - 1) {
        console.log(`Deadlock detected, retrying (${attempt + 1}/${maxRetries})...`);
        await new Promise(resolve => setTimeout(resolve, 100));  // Wait before retry
        continue;
      }

      throw error;
    }
  }
}
```

---

## 📚 **Common Interview Questions**

### Q1: What is a transaction and why is it important?

**Answer:**

A transaction is a sequence of database operations treated as a single unit of work.

**Importance:**
- **Data Integrity**: Ensures all-or-nothing execution
- **Consistency**: Maintains database constraints
- **Concurrency**: Handles multiple simultaneous operations
- **Recovery**: Protects against system failures

**Example:**

```sql
-- Without transaction: Risk of partial update
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- [Crash here] 💥
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Never executes!

-- With transaction: Safe
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- Both succeed or both fail
```

---

### Q2: Explain ACID properties

**Answer:**

| Property | Meaning | Example |
|----------|---------|---------|
| **Atomicity** | All or nothing | Transfer money: deduct AND add, or neither |
| **Consistency** | Valid state to valid state | Balance never goes negative (CHECK constraint) |
| **Isolation** | Concurrent transactions don't interfere | Two transfers don't see each other's partial state |
| **Durability** | Committed changes persist | After COMMIT, changes survive crashes |

> **Key Insight:** ACID ensures reliable, consistent data even with failures and concurrent access.

---

### Q3: What are the different isolation levels?

**Answer:**

| Level | Prevents | Performance | Use Case |
|-------|----------|-------------|----------|
| **READ UNCOMMITTED** | Nothing | Fastest | Analytics (rarely used) |
| **READ COMMITTED** | Dirty reads | Fast | Most web apps |
| **REPEATABLE READ** | Dirty + non-repeatable reads | Medium | Financial systems |
| **SERIALIZABLE** | All anomalies | Slowest | Critical operations |

**Trade-off:** Higher isolation = Better consistency but lower concurrency.

---

### Q4: What causes deadlocks and how do you prevent them?

**Answer:**

**Cause:** Two transactions wait for each other's locks.

```
Transaction A: Locks Row 1 → Waits for Row 2
Transaction B: Locks Row 2 → Waits for Row 1
Result: Deadlock! 💀
```

**Prevention:**

1. **Consistent Lock Order**: Always lock resources in same order
2. **Short Transactions**: Release locks quickly
3. **Lock Timeout**: Automatic rollback after timeout
4. **Retry Logic**: Catch deadlock and retry
5. **Avoid User Input**: Don't hold locks while waiting for user

**Example:**

```sql
-- ✅ Good: Always lock lower ID first
UPDATE accounts SET balance = ... WHERE id = LEAST(?, ?);
UPDATE accounts SET balance = ... WHERE id = GREATEST(?, ?);
```

---

### Q5: When should you use transactions?

**Answer:**

**Always use transactions for:**
- ✅ Multi-step operations that must succeed/fail together
- ✅ Money transfers, order processing
- ✅ Operations affecting multiple tables
- ✅ When data consistency is critical

**Examples:**

```sql
-- ✅ Use transaction: Multiple related operations
START TRANSACTION;
INSERT INTO orders (user_id, total) VALUES (1, 100);
INSERT INTO order_items (order_id, product_id) VALUES (LAST_INSERT_ID(), 5);
UPDATE products SET stock = stock - 1 WHERE id = 5;
COMMIT;

-- ❌ No transaction needed: Single, simple operation
UPDATE users SET last_login = NOW() WHERE id = 1;
```

---

## ✅ **Best Practices**

### **Do's**

1. **✅ Keep Transactions Short**
   ```sql
   -- Good: Quick transaction
   START TRANSACTION;
   UPDATE accounts SET balance = balance - 100 WHERE id = 1;
   COMMIT;

   -- Bad: Long transaction
   START TRANSACTION;
   -- Process for 10 seconds
   -- Hold locks for too long
   COMMIT;
   ```

2. **✅ Lock in Consistent Order**
   ```sql
   -- Always lock resources in same order (ascending ID)
   UPDATE accounts SET ... WHERE id = LEAST(id1, id2);
   UPDATE accounts SET ... WHERE id = GREATEST(id1, id2);
   ```

3. **✅ Use Appropriate Isolation Level**
   ```sql
   -- READ COMMITTED for most cases
   SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

   -- SERIALIZABLE for critical operations
   SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
   ```

4. **✅ Handle Errors and Rollback**
   ```javascript
   try {
     await db.query('START TRANSACTION');
     // operations
     await db.query('COMMIT');
   } catch (error) {
     await db.query('ROLLBACK');
     throw error;
   }
   ```

5. **✅ Use SELECT FOR UPDATE**
   ```sql
   -- Lock rows you plan to update
   SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
   UPDATE accounts SET balance = balance - 100 WHERE id = 1;
   ```

### **Don'ts**

1. **❌ Don't Hold Locks Long**
   ```sql
   -- Bad: Waiting for user input during transaction
   START TRANSACTION;
   SELECT * FROM seats WHERE id = 1 FOR UPDATE;
   -- Wait for user to click "Confirm"... ❌
   UPDATE seats SET booked = TRUE WHERE id = 1;
   COMMIT;
   ```

2. **❌ Don't Nest Transactions Unnecessarily**
   ```sql
   -- Most databases don't support nested transactions
   START TRANSACTION;
     START TRANSACTION;  -- ❌ Not supported in most DBs
     COMMIT;
   COMMIT;
   ```

3. **❌ Don't Forget Error Handling**
   ```javascript
   // ❌ Bad: No error handling
   await db.query('START TRANSACTION');
   await db.query('UPDATE...');
   await db.query('COMMIT');  // What if UPDATE fails?

   // ✅ Good: Proper error handling
   try {
     await db.query('START TRANSACTION');
     await db.query('UPDATE...');
     await db.query('COMMIT');
   } catch (error) {
     await db.query('ROLLBACK');
     throw error;
   }
   ```

4. **❌ Don't Use Wrong Isolation Level**
   ```sql
   -- ❌ READ UNCOMMITTED for financial data
   -- ✅ SERIALIZABLE for financial data
   ```

---

## 🎯 **Summary**

**Key Concepts:**
- **Transactions** ensure all-or-nothing execution of database operations
- **ACID** properties guarantee reliability, consistency, and durability
- **Isolation levels** balance consistency vs. performance
- **Locking** prevents concurrent transaction conflicts
- **Deadlocks** occur when transactions wait for each other; prevent with consistent lock order

**Transaction Commands:**
- `START TRANSACTION` / `BEGIN` - Start transaction
- `COMMIT` - Save changes permanently
- `ROLLBACK` - Discard all changes
- `SAVEPOINT` - Create rollback point

**When to Use:**
- ✅ Multi-step operations
- ✅ Money transfers, order processing
- ✅ Operations affecting multiple tables
- ✅ Critical data consistency requirements

**Best Practices:**
- Keep transactions short
- Lock resources in consistent order
- Use appropriate isolation level
- Handle errors with rollback
- Avoid user interaction during transactions

---

[← Previous: Indexes](./03-indexes.md) | [Next: PostgreSQL →](./05-postgresql.md)
