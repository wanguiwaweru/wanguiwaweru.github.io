# Distributed Transactions

A **transaction** is a sequence of operations that must either all succeed or all fail together. In a distributed system, operations span multiple nodes, making transactions harder to guarantee.

The main challenge is how to ensure atomicity (all-or-nothing) across multiple independent systems that can fail, be slow, or be partitioned from each other?

In distributed transactions, you must choose between **strong consistency**, guaranteed correctness, but slow and **eventual consistency** fast, but temporarily inconsistent.

---

## ACID Properties

ACID defines what "correct" transactions look like. It's the promise traditional databases make.

### Atomicity: All-or-Nothing

A transaction either completes fully or not at all. There is no partial state.

**Example: Bank transfer**
```
Transaction: Transfer $100 from Account A to Account B

What atomicity means:
  A loses $100, B gains $100 ✓ (both happen)
  OR
  A keeps $100, B unchanged ✓ (neither happens)
  
NOT ALLOWED:
  A loses $100, B unchanged ✗ (partial state - money disappeared!)
```

**Atomicity is enforced by:**
1. **Write-ahead logging (WAL)**: Log every operation before applying it
2. **Rollback on failure**: If any operation fails, undo all previous operations
3. **Commit point**: Only after all operations succeed, mark as "committed" (safe from rollback)

```
Example execution:

BEGIN TRANSACTION
  1. Log: "Debit A by $100"
  2. Debit A by $100 in memory
  3. Log: "Credit B by $100"
  4. Credit B by $100 in memory
  5. Log: "COMMIT"
COMMIT
```

If crash occurs between step 3 and 5:
```
On recovery:
  Read log
  See "Debit A by $100" and "Credit B by $100"
  But no "COMMIT" mark
  Result: Rollback all operations, A and B return to original values
```

**In distributed systems:**
- Each node must log locally before committing
- If any node fails to commit, all nodes must rollback
- This is the **Two-Phase Commit (2PC)** problem

### Consistency: Valid State to Valid State

A transaction transforms the database from one valid state to another valid state. Invariants are always maintained.

**Example: Bank invariant**
```
Invariant: For any account, balance ≥ 0
           Total money in system is conserved

Valid state: A=$100, B=$200, Total=$300 ✓
Valid state: A=$50, B=$250, Total=$300 ✓

Invalid state: A=-$50, B=$200, Total=$150 ✗ (violates invariant)
```

**Transaction maintains invariant:**
```
Before: A=$100, B=$200, Total=$300, Invariant holds
Transfer $100 from A to B
After: A=$0, B=$300, Total=$300, Invariant still holds
```

> Consistency is the responsibility of both the **database** and the **application**.

Database provides:
- Type checking (amount is a number, not a string)
- Foreign key constraints (customer_id points to valid customer)
- Uniqueness constraints (email is unique)

Application provides:
- Business logic (don't allow negative balances)
- Derived values (total = sum of all amounts)

**In distributed systems:**
- Each node must validate operations locally
- Replicas must apply same operations in same order to reach same state
- This requires **consensus** (use Raft or Paxos) to agree on operation order

### Isolation: Concurrent Transactions Don't Interfere

Transactions execute concurrently but appear to execute serially to each other.

**Problem: Without isolation**
```
Account A starts with $100

Transaction 1: Transfer $50 to B
  T=0: Read A = $100
  T=1: (slow network, pause)
  T=5: Subtract $50, A = $50
  T=6: Write A = $50

Transaction 2: Transfer $30 to C (concurrent)
  T=2: Read A = $100 (T1 hasn't written yet)
  T=3: Subtract $30, A = $70
  T=4: Write A = $70

Final state: A = $70, but should be A = $20
Result: $30 disappeared (lost update)
```

**With isolation:**
```
Same operations, but isolated:

Transaction 1 locks A
Transaction 2 waits for lock
Transaction 1 completes: A = $50
Transaction 2 acquires lock, reads A = $50
Transaction 2 completes: A = $20

Final state: A = $20 ✓
```

**Isolation Levels** define how strict the isolation is. 
- Read uncommitted: No isolation (dirty reads) ie read uncommitted data
- Read committed: Avoid dirty reads by only reading committed data
- Repeatable read: Avoid non-repeatable reads by holding locks on read rows
- Serializable: Full isolation (equivalent to serial execution) where transactions appear to run one at a time but can execute concurrently with locking to ensure correctness.


**In distributed systems:**
- Locking is expensive because it requires coordination
- Optimistic concurrency where conflicts are detected at commit are faster but you sacrifice some isolation guarantees

> Higher isolation lead to fewer anomalies but has more locking leading to slower performance.

#### Isolation Levels

When multiple transactions execute concurrently, you must prevent interference. Different isolation levels offer different guarantees and costs.

### Read Uncommitted: No Isolation

**Guarantee:** None. Transactions can read each other's uncommitted changes.

**Problems allowed:**
- **Dirty reads**: Read data written by uncommitted transaction
- **Non-repeatable reads**: Read different values for same row in one transaction
- **Phantom reads**: Different rows appear/disappear during transaction

**Example: Dirty read**
```
Account has $100

Transaction 1: Tries to transfer $50
  T=0: Debit A by $50, A = $50 (in memory, uncommitted)
  
Transaction 2: Reads A (concurrent)
  T=1: Read A = $50 (sees uncommitted data!)
  
Transaction 1: Crashes, rollback
  A reverts to $100
  
Transaction 2: But it already read A = $50
  Inconsistency!
```

**Performance:** Fastest (no locks, no coordination)

**Use case:** Analytics/reporting where rough numbers are ok; not for financial transactions

### Read Committed: Avoid Dirty Reads

**Guarantee:** Only read committed (finalized) data.

**Problems still allowed:**
- **Non-repeatable reads**: Same row might change between reads
- **Phantom reads**: Rows might appear/disappear

**Example: Non-repeatable read**
```
Transaction 1 (repeatable read):
  T=0: Read A = $100
  T=5: (slow operation)
  T=10: Read A again = $90 (changed during transaction!)
```

**How it works:**
- Readers don't wait for writers
- Readers get latest committed version
- Writers hold locks on modified rows

**Performance:** Moderate (some locking)

**Use case:** Most web applications; balance correctness with performance

### Repeatable Read: Avoid Non-Repeatable Reads

**Guarantee:** Once you read a value, reading it again in the same transaction returns the same value.

**Problems still allowed:**
- **Phantom reads**: New rows matching WHERE clause might appear

**Example: Phantom read**
```
Transaction: Sum salary of all employees in department "Engineering"

T=0: Query returns 4 employees, total = $400k
T=5: New employee added to Engineering (by different transaction)
T=10: Query again (same WHERE clause), returns 5 employees, total = $450k

Phantom! New row appeared.
```

**How it works:**
- Hold locks on all read rows
- Prevents changes to those rows
- Doesn't prevent new rows matching WHERE clause

**Performance:** Slower (more locks held)

**Use case:** Some financial systems; some databases (PostgreSQL) implement as snapshot isolation

### Serializable: Full Isolation

**Guarantee:** Transactions are equivalent to some serial (one-at-a-time) execution order. No dirty reads, no non-repeatable reads, no phantom reads.

**How it works:**
- Lock entire tables or ranges
- Hold locks for entire transaction duration
- Ensure total order of execution

**Example: Fully serializable**
```
Transaction 1: Sum salaries in Engineering
  T=0: Lock all Engineering rows
  T=1: Read sum = $400k
  T=5: Unlock
  
Transaction 2 (concurrent): Add new Engineering employee
  T=2: Try to insert Engineering row
  T=2: Blocked (T1 holds lock)
  T=5: T1 finishes, lock released
  T=6: T2 acquires lock, inserts row
  
Result: T1 sees 4 employees, T2 sees 5 (serialized order)
```

**Performance:** Slowest due to many locks, and limited parallelism

**Use case:** Critical financial systems; regulatory requirements (HIPAA, SOX)

### Snapshot Isolation: A Practical Middle Ground

Not a standard SQL level, but widely used (PostgreSQL, MySQL 5.7+, Oracle).

**Guarantee:** Each transaction sees a consistent snapshot of the database at the moment the transaction started.

**How it works:**
```
MVCC (Multi-Version Concurrency Control):
  Each transaction gets a version number
  Transaction sees all committed versions ≤ its version
  
Timeline:
  T=0: Version 1 commits (A=$100, B=$200)
  T=1: Transaction 2 starts, version=2, reads snapshot (A=$100, B=$200)
  T=2: Transaction 3 starts, version=3
  T=3: Version 2 modifies B=$300 (new version created)
  T=4: Version 3 reads B (gets $200 from version 1, not $300 from version 2)
  T=5: Version 2 commits
  T=6: Version 3 reads B again (still gets $200, same snapshot)
```

**Problems prevented:**
- No dirty reads (only see committed versions)
- No non-repeatable reads (same snapshot throughout)

**Problems allowed:**
- Phantom reads (if implementation doesn't protect against it)
- Write conflicts (if two transactions modify same row)

**Performance:** Very good (no locks for reads; only check conflicts on write)

**Use case:** Modern databases that prioritize availability (Amazon, Google Cloud SQL)

---

### Durability: Committed Data Survives Failures

Once a transaction is committed, the data persists even if the system crashes, loses power, or fails.

**How it's enforced:**
1. **Write-ahead logging**: Write to persistent storage (disk, battery-backed RAM)
2. **fsync**: Force OS to write to disk (not just RAM)
3. **Replication**: Write to multiple nodes
4. **Backup**: Archive data regularly

```
Durability levels:

No durability:
  Data only in RAM
  Crash → all data lost
  
Single disk durability:
  Write to local disk
  Crash → disk survives, data recovered
  Disk failure → data lost
  
Replicated durability:
  Write to several replicas' disks
  Can tolerate simultaneous failures of some replicas
  Crash → data survives on other replicas
  
Archive durability:
  Regular backups to tape/cloud
  Can recover from almost anything
```

**Durability vs. Speed tradeoff:**
```
No fsync (fast):
  Write to RAM buffer
  Return ACK immediately
  fsync to disk in background
  Risk: crash before fsync = data lost

Immediate fsync (slow):
  Write to disk
  Wait for fsync to complete
  Return ACK
  Safe: crash after ACK, data survives
  Cost: 10-100x slower
```

**In distributed systems:**
- Single fsync on one node: not durable (node can crash)
- fsync on majority of replicas: durable (survives most failures)
- This is the **replication + durability** problem

---


## Two-Phase Commit (2PC): Distributed Transactions

### The Problem

How do you commit a transaction across multiple databases/services?

```
Transaction: Transfer $100 from Bank A to Bank B

Node A: Debit account
Node B: Credit account

If one succeeds and other fails:
  Result: Partial transaction (money disappeared or duplicated)
```

### The Solution: Two-Phase Commit

**Phase 1: Prepare (Voting)**
```
Coordinator: "Can you commit the following operations?"
  Debit account at Node A by $100
  Credit account at Node B by $100

Node A: 
  Check if operation is valid
  Lock the account (prevent other changes)
  Write to WAL (persistent log)
  Return "YES, I can commit"

Node B:
  Same checks, lock, WAL write
  Return "YES, I can commit"

Coordinator:
  Waits for YES from ALL participants
  If any says NO or times out:
    Send ABORT to all nodes
    All rollback, unlock
    Transaction fails
```

**Phase 2: Commit (Execute)**
```
Coordinator: "All votes received. COMMIT!"
  Send COMMIT to all participants

Node A:
  fsync to disk (make permanent)
  Unlock the account
  Return "COMMITTED"

Node B:
  fsync to disk
  Unlock
  Return "COMMITTED"

Coordinator:
  All participants committed
  Transaction is durable and atomic
```

### 2PC Timeline

```
T=0: Client sends transaction to coordinator
T=1: Coordinator sends Prepare to A and B (in parallel)

T=2: A and B receive Prepare
     A: Lock, log, YES
     B: Lock, log, YES
     
T=3: Coordinator receives both YES
     Sends COMMIT to A and B

T=4: A and B receive COMMIT
     A: fsync, unlock
     B: fsync, unlock
     
T=5: Coordinator receives both COMMITTED
     Returns success to client
     
Total time: 5-10ms (4-6 network round trips)
```

### 2PC Problems

**Blocking**
```
Scenario: Node B crashes during Phase 1

T=0: Prepare sent to A and B
T=1: A locks and responds YES
T=2: B crashes (before responding)
T=3: Coordinator waits for B's response (forever)

Result: A's account is locked indefinitely
        Other transactions trying to access A's account are blocked
        This cascades: if other transactions wait for A, they block too
        
Could last for hours if no timeout
```

**2. Lack of Availability**
```
During coordinator crash or network partition:
  Participants are stuck with locks held
  No way to know if transaction should commit or abort
  
Recovery procedure:
  Participants must contact any other participant to ask
  "Did the transaction commit?"
  If no one knows, participants wait for coordinator recovery
  
Result: High unavailability when failures occur
```

**3. Slow**
```
Synchronous operation:
  Client waits for all nodes to prepare and commit
  Network round trips add latency
  10ms per round trip × 6 rounds = 60ms minimum
  
If one node is slow:
  All other nodes wait
  Limited by slowest participant
```

### When 2PC Works

2PC is acceptable when:
- Failures are rare
- All participants are reliable (same data center, good network)
- Transaction volume is low
- Latency is not critical

2PC fails when:
- Distributed across multiple data centers (high latency)
- One participant is frequently down/slow
- High availability is required
- Real-time requirements

---

## Alternative: Eventual Consistency

Instead of atomic distributed transactions, accept temporary inconsistency.

```
Write to local node immediately:
  Client: "Add $100 to my account"
  Local node: Updates balance immediately, returns success
  
Replicate asynchronously:
  Background job: "Send update to other replicas"
  Other nodes catch up over seconds/minutes
  
Trade-off:
  Benefit: Immediate response, high availability
  Cost: Other clients might temporarily see old balance
```

### Event Sourcing Pattern

Instead of storing current state, store immutable events.

```
Traditional approach:
  Account state: balance = $500
  Update: Set balance = $600
  Problem: Concurrent updates conflict
  
Event sourcing:
  Store events: [
    {type: "opened", amount: 500},
    {type: "deposit", amount: 100},
    {type: "withdrawal", amount: 50}
  ]
  
  Replay events to get current state:
    500 + 100 - 50 = $550
  
  New transaction: {type: "deposit", amount: 50}
  Append to event log (append is atomic)
  All replicas eventually apply same events in same order
```

**Benefits:**
- Immutable event log is easy to replicate
- Eventual consistency is natural
- No 2PC needed

**Challenges:**
- Complex queries (must replay events)
- Events might conflict (what if withdrawal happens while balance is unknown?)
- Requires careful ordering of events

### Saga Pattern

Break distributed transaction into local transactions with compensations.

```
Transaction: Transfer $100 from Bank A to Bank B

Instead of 2PC:

Step 1: Debit Bank A
  Bank A: Debit account locally
  If fails: Stop, rollback local change
  
Step 2: Credit Bank B
  Bank B: Credit account locally
  If fails: Compensation! Credit Bank A back

Result: Either both succeed, or B fails and A compensates
        Similar to atomicity but not distributed

Timeline:
  T=0: Debit A (succeeds)
  T=1: Credit B (fails)
  T=2: Compensation: Credit A (reverses debit)
  Final: A unchanged, B unchanged
```

**Benefits:**
- No blocking (each step is local)
- Highly available

**Challenges:**
- Not truly atomic (partial visible state during compensation)
- Complicated (must define compensations for each step)
- Debugging difficult (non-obvious failure modes)

---

## Consistency Models

| Model | Guarantee | Latency | Failures | Use Case |
|-------|-----------|---------|----------|----------|
| **Strict Consistency** | All replicas same immediately | High (2PC) | Low availability | Finance, legal |
| **Linearizability** | Writes appear atomic | Moderate (consensus) | Medium | Databases, caching |
| **Causal Consistency** | Causally related events ordered | Low | Good | Social media, comments |
| **Eventual Consistency** | All replicas converge | Very low | Very high | Analytics, caching |

---

## Practical Implementation Patterns

### Pattern 1: Optimistic Concurrency with Retries

Assume no conflicts. Check at commit. Retry on conflict.

```
Client 1:
  Read balance = $100
  Compute new balance = $150
  Try to update: "IF balance == $100, SET balance = $150"
  
Client 2 (concurrent):
  Read balance = $100
  Compute new balance = $120
  Try to update: "IF balance == $100, SET balance = $120"
  
Execution:
  Client 1 updates: balance == $100? YES → balance = $150
  Client 2 tries: balance == $100? NO → FAIL, retry
  
  Client 2 retries:
    Read balance = $150
    Compute new balance = $170
    Try update: balance == $150? YES → balance = $170
```

**Benefits:**
- No locks (fast)
- Works well when conflicts are rare

**Challenges:**
- Retry loops under high contention
- Livelocks if conflicts are frequent

### Pattern 2: Pessimistic Locking

Lock before modifying.

```
Client 1:
  Lock row for update
  Read balance = $100
  Compute new balance = $150
  Write new balance
  Unlock
  
Client 2 (concurrent):
  Try to lock same row
  Blocked (Client 1 holds lock)
  Wait...
  Lock released, Client 2 acquires lock
  Read balance = $150
  Compute new balance = $170
  Write
  Unlock
```

**Benefits:**
- Guaranteed no conflicts (serialized access)

**Challenges:**
- Locks hold resources
- Deadlocks are possible (A locks X then Y; B locks Y then X)

### Pattern 3: Snapshot Isolation with Write Tracking

Use MVCC(Multi-Version Concurrency Control) and detect write conflicts.

```
Transaction 1:
  snapshot_id = 1
  Read A (version 0, value 100)
  Read B (version 0, value 200)
  Compute C = A + B = 300
  On commit: "Write version 1: C = 300"
  Check: Did anyone else write A or B since my snapshot?
    No → Commit success
    Yes → Conflict detected, abort
```

**Benefits:**
- No locks for reads
- Parallelism for read-only transactions

**Challenges:**
- Detects conflicts late (at commit)
- Must be idempotent (retry might re-execute)

---

## Tradeoffs

### Strong Consistency vs. Availability

```
Strong consistency (2PC):
  ✓ Guaranteed correctness
  ✗ Slow (2PC latency)
  ✗ Low availability (blocking)
  ✗ Cannot tolerate partitions 

Eventual consistency:
  ✓ High availability
  ✓ Fast (local writes)
  ✓ Tolerates partitions
  ✗ Temporary inconsistency
  ✗ Eventual, not immediate
```

### Isolation Level vs. Performance

```
Read uncommitted:
  ✓ Fast (no locks)
  ✗ Dirty reads possible
  
Read committed:
  ~ Moderate speed
  ~ Some guarantees
  
Repeatable read:
  ~ Slower
  ~ More guarantees
  
Serializable:
  ✗ Slow (many locks)
  ✓ Complete isolation
```

### Durability vs. Latency

```
No durability:
  ✓ Fast (RAM only)
  ✗ Data lost on crash
  
Single machine durability:
  ~ Moderate (fsync)
  ~ Survives local crash
  ✗ Lost if machine dies
  
Replicated durability:
  ~ Slower (replicate to multiple machines)
  ~ Survives most failures
  
Archive durability:
  ✗ Very slow (tape backup)
  ✓ Survives almost anything
```

---

## Real-World Examples

### Google Spanner: Strong Consistency at Scale

**Approach:** Consensus and atomic clocks (TrueTime)

- Uses Paxos (Raft equivalent) for replication
- Synchronized clocks with bounded error (TrueTime)
- Can determine if one event happened before another
- Supports ACID transactions across continents
- Cost: ~$millions per cluster, complex infrastructure

**Trade-off:** Achieves consistency but requires special hardware (atomic clocks)

### Amazon DynamoDB: Eventual Consistency at Scale

**Approach:** Eventual consistency + quorum reads

- All writes eventually replicate to all replicas
- Strong read consistency requires quorum reads (majority)
- Eventually consistent reads are cheap (any replica)
- No 2PC; no distributed transactions
- Cost: Low; simple; scales to any size

**Trade-off:** No distributed transactions, but massive scale and availability

### PostgreSQL: Local Transactions Only

**Approach:** ACID within one database; WAL + MVCC for isolation

- Full ACID guarantees for local transactions
- Snapshot isolation (MVCC) for concurrency
- For distributed transactions: application must implement saga pattern
- Cost: Moderate; well-understood

**Trade-off:** Strong guarantees locally, but application complexity for distributed systems

---

---

## Decision Tree for System Design

```
Question 1: Do all nodes need to agree on every write?
  YES → Use 2PC or consensus-based replication
  NO → Use eventual consistency
  
Question 2 (if 2PC): Are all nodes in same data center?
  YES → 2PC is acceptable
  NO → High latency, consider eventual consistency
  
Question 3 (if eventual consistency): Is temporary inconsistency acceptable?
  YES → Proceed with eventual consistency
  NO → Must use strong consistency (2PC, consensus)
  
Question 4: How often do conflicts occur?
  Rare → Optimistic locking
  Common → Pessimistic locking or serializable isolation
  
Question 5: What isolation level is needed?
  Financial → Serializable
  Web application → Repeatable read or snapshot isolation
  Analytics → Read committed
  Reporting → Read uncommitted
```