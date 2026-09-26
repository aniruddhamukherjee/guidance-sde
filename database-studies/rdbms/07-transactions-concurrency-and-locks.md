# Module 07: Transactions, Concurrency & Locks

## 1. Transactions & Savepoints

A transaction controls a sequence of operations executed as a single atomic unit.

```sql
BEGIN;

-- Deduct funds
UPDATE accounts SET balance = balance - 500 WHERE id = 1;

-- Create intermediate rollback checkpoint
SAVEPOINT transfer_initiated;

-- Attempt secondary charge
UPDATE accounts SET balance = balance - 50 WHERE id = 1;

-- If secondary charge fails, rollback only to the savepoint without losing the first update
ROLLBACK TO SAVEPOINT transfer_initiated;

-- Finalize valid operations
COMMIT;
```

---

## 2. Transaction Isolation Levels & Concurrency Anomalies

SQL standards define 4 isolation levels to balance concurrency and consistency. PostgreSQL supports 3:

| Isolation Level | Dirty Reads | Non-Repeatable Reads | Phantom Reads | Serialization Anomaly |
|---|---|---|---|---|
| **Read Committed** *(PostgreSQL Default)* | ❌ Prevented | ⚠️ Allowed | ⚠️ Allowed | ⚠️ Allowed |
| **Repeatable Read** | ❌ Prevented | ❌ Prevented | ❌ Prevented *(in Postgres)* | ⚠️ Allowed |
| **Serializable** | ❌ Prevented | ❌ Prevented | ❌ Prevented | ❌ Prevented |

### Setting Isolation Level:
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- or
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### Explanation of Anomalies:
1. **Dirty Read**: Transaction A reads data modified by Transaction B that has not yet committed. (Impossible in PostgreSQL in any level).
2. **Non-Repeatable Read**: Transaction A reads a row, Transaction B updates/deletes that row and commits; Transaction A re-reads the same row and sees different data.
3. **Phantom Read**: Transaction A queries rows matching a condition, Transaction B inserts new rows matching that condition and commits; Transaction A re-runs the query and sees "phantom" rows.
4. **Serialization Anomaly**: The outcome of concurrent committed transactions could not have occurred if they ran serially one after another.

---

## 3. Explicit Row-Level Locking (Pessimistic Locking)

To prevent race conditions (e.g. ticket booking, balance deduction, queue workers), use explicit locking:

### 1. `FOR UPDATE` (Exclusive Row Lock)
Blocks other transactions from updating, deleting, or locking the selected rows until transaction commits:

```sql
BEGIN;

-- Lock the row exclusively
SELECT balance 
FROM accounts 
WHERE id = 1 
FOR UPDATE;

UPDATE accounts 
SET balance = balance - 100 
WHERE id = 1;

COMMIT;
```

### 2. High-Throughput Job Queue Pattern (`FOR UPDATE SKIP LOCKED`)
Ideal for message queues and background workers to avoid lock contention:

```sql
-- Worker retrieves 1 pending job without blocking or waiting on other workers
BEGIN;

SELECT job_id, payload 
FROM background_jobs 
WHERE status = 'pending' 
ORDER BY priority DESC, created_at ASC 
LIMIT 1 
FOR UPDATE SKIP LOCKED;

-- Process the job...
UPDATE background_jobs 
SET status = 'completed', processed_at = clock_timestamp() 
WHERE job_id = 42;

COMMIT;
```

---

## 4. Deadlocks & Resolution

A **deadlock** occurs when two transactions wait on locks held by each other.

```
Transaction 1: Holds Lock on Row A ───► Waiting for Row B (held by Tx 2)
                               ▲                 │
                               │                 ▼
Transaction 2: Waiting for Row A ◄─── Holds Lock on Row B
```

### Deadlock Prevention Best Practices:
1. **Consistent Lock Order**: Always acquire locks on rows or tables in the same predefined order (e.g., sort IDs ascending before locking).
   ```sql
   -- Lock multiple accounts in ascending ID order:
   SELECT * FROM accounts WHERE id IN (2, 5, 8) ORDER BY id FOR UPDATE;
   ```
2. **Keep Transactions Short**: Perform non-DB processing outside the transaction.
3. **Set Lock Timeouts**: Avoid indefinite hangs:
   ```sql
   SET lock_timeout = '3s';
   SET statement_timeout = '10s';
   ```
4. **PostgreSQL Deadlock Detection**: PostgreSQL detects deadlocks automatically after `deadlock_timeout` (default 1 second) and aborts one of the transactions with error code `40P01 (deadlock_detected)`.
