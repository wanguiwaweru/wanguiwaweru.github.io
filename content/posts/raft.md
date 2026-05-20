---
title: "Raft"
date: "2026-05-18"
draft: false
---

# Raft

## TLDR: 

Raft is a consensus algorithm that ensures a cluster of nodes can agree on a sequence of commands, even when there are failures. Raft ensures all nodes apply the same sequence of commands to their state machines in the same order.
It achieves this through leader election, log replication, and safety guarantees. 

> Raft is designed to be understandable and practical for real-world systems.

Raft separates the problem into three subproblems:
1. **Leader election** — Choose one node to coordinate
2. **Log replication** — Replicate commands safely to all nodes
3. **Safety** — Ensure committed commands are never lost

---

## Fundamental Concepts

### Terms: Raft's Logical Clock

A **term** is Raft's way of marking time. Each term has a unique identity (just an integer: 0, 1, 2, 3, ...).

**Key properties:**
- Terms increment whenever a new election starts
- Each term has **at most one leader**
- A node never accepts commands from an old term
- Terms enforce causality such that if you see term T, you know at least that many elections have happened


**Terms matter because:**
- Old leaders might not know they've been replaced
- If an old leader crashes and recovers, it might try to send commands which are rejected by followers because its term is outdated.
- Terms prevent split-brain which occurs when a network partition disconnects a cluster into isolated subgroups. An old leader can't claim authority because its term is stale and a new leader has majority votes in a higher term.

### Node States: Leader, Follower, Candidate

Every node in Raft is in exactly one of three states at a given time:

#### Follower
- The default state. When a node joins a cluster, it starts as a follower.
- Doesn't initiate anything, it just responds to requests from leaders and candidates.
- A follower votes for candidates that ask for votes, but only once per term.
- If no leader heartbeat arrives for a timeout period, the node becomes a candidate and starts an election asking other nodes for votes

```
Follower state:
  - currentTerm: the last term I know about
  - votedFor: the candidate I voted for in currentTerm (null if none)
  - log: list of [term, command]
```

#### Candidate
- A node that wants to become leader
- Initiates an election by:
  1. Incrementing its term
  2. Voting for itself
  3. Sending RequestVote RPCs to all other nodes
  4. Waiting for votes
- If it gets majority votes: becomes leader
- If it gets AppendEntries from a new leader: reverts to follower
- If election timeout expires for instance when another node was also a candidate and none of them got a majority, the candidate increments term and starts a new election

#### Leader
- Elected by majority vote
- Sends AppendEntries RPCs to all followers (commands and heartbeats)
- Only node that accepts commands from clients and sends responses to the clients. The followers redirect requests sent to them by clients to the leader
- Tracks each follower's replication progress
- Commits entries and applies them to state machine
- Sends committed entries to followers

---

## State Transitions

```
Follower:
  - Doesn't receive heartbeat for election timeout
    → Becomes Candidate

Candidate:
  - Receives majority votes
    → Becomes Leader
  - Receives AppendEntries from leader (with term ≥ my term)
    → Becomes Follower
  - Election timeout expires
    → Increments term, starts new election (stays Candidate)

Leader:
  - Receives RequestVote from candidate with higher term
    → Becomes Follower
  - Receives AppendEntries from node with higher term
    → Becomes Follower
  - Discovers another leader
    → Becomes Follower
```

---

## Leader Election

### How Elections Start

**Election triggered when:**
1. System starts up since all nodes are followers
2. Current leader crashes
3. Follower's election timeout fires when they don't get a heartbeat from leader for 150-300ms

### Election Steps

**Step 1: Follower becomes candidate**
```
Follower detects timeout:
  currentTerm := currentTerm + 1
  votedFor := self
  Start election timer with random timeout (150-300ms)
  Send RequestVote RPC to all other nodes
```

**Step 2: Other nodes vote**
```
Node receives RequestVote(term, candidateId, lastLogIndex, lastLogTerm):
  
  If term < currentTerm:
    Return false (old term, ignore)
  
  If term > currentTerm:
    currentTerm := term
    votedFor := null
    Convert to follower
  
  If votedFor is null or votedFor is candidateId:
    AND candidate's log is at least as up-to-date as my log:
      votedFor := candidateId
      Return true
  
  Return false
```

Each node votes for **at most one candidate per term** to ensure a majority can only be achieved by one candidate.

**Step 3: Candidate counts votes**
```
If candidate receives votes from majority (including itself):
  Become leader
  Send AppendEntries heartbeats to all followers
  
If candidate receives AppendEntries from new leader:
  If term >= currentTerm:
    Recognize this node as leader, become follower
  
If election timeout expires without winner:
  Increment term again, start new election
```

### Randomized Election Timeout?
Election timeouts are randomized to prevent split votes. If all nodes had the same timeout, they would all become candidates at the same time and split votes, causing repeated election failures.

Without randomization each node's timeout would fire simultaneously making all of them candidates at the same time, and they would all vote for themselves, resulting in no majority and a new election starting immediately. This would lead to continuous election failures and delays in electing a leader.
```
Time T=0: All followers' timeouts fire simultaneously
          All become candidates, all vote for themselves
          No one gets majority (everyone splits 1 vote)
          New election starts

This causes election failures and delays
```

With randomization typically ranging from 150-300ms one node's timeout will fire before the others, allowing it to become candidate and get votes before the others start their elections.
```
Time T=0: Node A's timeout fires first (150ms)
          A becomes candidate, sends RequestVote
          
Time T=0.05: Node B's timeout fires (155ms)
             B sends RequestVote
             But A already got B's vote in its first election
             
Time T=0.1: Node A gets majority (B, C voted for it)
            A becomes leader
            
Time T=0.15: Node B's and C's timeouts would fire
             But they already got AppendEntries from leader A
             Reset their timeouts
```

---

## Log Replication

### The Log

Each node maintains a **log** of [term, command] pairs:

```
Follower log:
  Index: 1     2     3     4
  Term:  1     1     2     2
  Cmd:   "x=1" "x=2" "y=3" "y=4"
  
Committed index: 2 (first 2 entries are safe)
Uncommitted: entries 3-4 (not yet majority replicated)
```

**Committed vs Uncommitted:**
- **Committed:** Entry is on a majority of nodes; safe to apply to state machine
- **Uncommitted:** Entry might be overwritten if leader crashes before replication

### How the Leader Replicates

**Step 1: Client sends command to leader**
```
Client: "execute x=1"
Leader: Append [currentTerm, "x=1"] to my log
        Log is now: [(1, "x=1")]
        Entry is uncommitted
```

**Step 2: Leader sends to all followers in parallel**
```
Leader sends AppendEntries(term, leaderId, prevLogIndex, prevLogTerm, entries, leaderCommit):
  
  prevLogIndex: index of entry before the ones I'm sending
  prevLogTerm: term of that entry
  entries: new entries to append
  leaderCommit: index up to which leader has committed
```

**Step 3: Follower receives AppendEntries**
```
Follower receives AppendEntries:
  
  If term < currentTerm:
    Return false (old leader)
  
  If term > currentTerm:
    currentTerm := term (accept new term)
  
  If log[prevLogIndex].term != prevLogTerm:
    Return false (log mismatch - my log differs)
  
  If new entries conflict with my log:
    Delete conflicting entries and all after
  
  Append new entries to log
  
  If leaderCommit > commitIndex:
    commitIndex := min(leaderCommit, last new entry index)
    Apply entries[lastAppliedIndex+1...commitIndex] to state machine
  
  Return true
```

**Step 4: Leader counts acknowledgments**
```
If majority of followers have replicated entry N:
  commitIndex := N
  Apply entries[lastAppliedIndex+1...commitIndex] to state machine
  Send updated commitIndex to followers (via next heartbeat)
```

### Example: Replicating a Single Entry

```
Initial state:
  Leader A: log=[1: "x=1", 1: "x=2"], commitIndex=1
  Follower B: log=[1: "x=1"], commitIndex=1
  Follower C: log=[1: "x=1"], commitIndex=1
  
Client sends "x=3" to leader A

A appends to log: log=[1: "x=1", 1: "x=2", 1: "x=3"]
A sends AppendEntries to B and C with new entries

B receives:
  prevLogIndex=2, prevLogTerm=1, entries=[1: "x=3"]
  B's log[2].term = 1 ✓ matches
  B appends: log=[1: "x=1", 1: "x=2", 1: "x=3"]
  B returns true

C receives: (same as B)
  C appends: log=[1: "x=1", 1: "x=2", 1: "x=3"]
  C returns true

A receives ACKs from B and C (majority including itself = 2/3 nodes)
A: commitIndex := 3
A applies entries[2...3] to state machine

A sends next heartbeat with leaderCommit=3 to B and C

B and C receive heartbeat:
  leaderCommit=3 > commitIndex=1
  commitIndex := min(3, 3) = 3
  Apply entries[2...3] to state machine

Final state: All nodes have same log and applied same entries
```

### Log Matching Property

**Invariant:** If two nodes have the same [index, term] for an entry, all previous entries are identical.

Why this works:
- Follower rejects AppendEntries if prevLogIndex.term doesn't match (log mismatch)
- Leader retries with earlier entries until they match (decrement nextIndex)
- Once they match, leader sends all entries from that point forward

Example of log repair:
```
Leader: [1: "a", 1: "b", 2: "c", 2: "d", 3: "e"]
Follower: [1: "a", 2: "x", 2: "y"]

Leader tries AppendEntries(prevLogIndex=2, prevLogTerm=1, entries=[2: "c", 2: "d", 3: "e"])
Follower checks: log[2].term = 2, but prevLogTerm = 1 ✗ MISMATCH
Return false

Leader decrements nextIndex from 3 to 2
Leader tries AppendEntries(prevLogIndex=1, prevLogTerm=1, entries=[2: "c", 2: "d", 3: "e"])
Follower checks: log[1].term = 1 ✓ MATCH
Follower deletes entries 2+ and appends new ones
Follower: [1: "a", 2: "c", 2: "d", 3: "e"]
```

---

## Safety
Raft must ensure:

1. **Election safety:** At most one leader per term
2. **Leader append-only:** Leaders never overwrite or delete entries
3. **Log matching:** If two nodes have same [index, term], all previous entries are same
4. **Leader completeness:** Committed entries are on every new leader
5. **State machine safety:** Applied entries are applied in order and never change

### Election Safety Guarantee

**Rule:** Each node votes for **at most one candidate per term**.

This ensures at most one node can get majority votes in a term.

### Leader Completeness Guarantee

**Rule:** Only nodes with up-to-date logs can become leaders.

A log is "up-to-date" if:
- Its last term is higher, OR
- Its last term equals the candidate's, but its log is longer

This ensures the new leader has all committed entries.

Proof sketch:
```
Suppose entry X was committed, then leader crashes.
X must be on majority of nodes (by definition of committed).

For node to become new leader:
  - It must be voted on by majority
  - At least one voter must have X (since X is on majority)
  - Voter only votes for candidate with up-to-date log
  - So candidate's log must be at least as up-to-date as voter's log
  - So candidate must have the logs X had

Therefore, new leader has all committed entries.
```

### State Machine Safety

**Rule:** Apply entries in index order, only apply committed entries.

This ensures state machines on all nodes execute same commands in same order.

Proof:
```
Entry N is committed only when replicated to majority.
If entry N+1 gets committed, its leader must have entry N (log matching).
New leaders have all committed entries (leader completeness).

Therefore, each node applies entries in order, and all nodes apply same entries.
```

---

## Handling Failures

### Leader Crashes

```
Original cluster: A (leader), B, C

Time T=0: A crashes
Time T=0-0.3: B and C don't receive heartbeat
Time T=0.3: B's timeout fires → B becomes candidate
Time T=0.31: B gets votes from C (majority)
Time T=0.32: B becomes leader of term 2
Time T=0.33: C's timeout would fire, but gets heartbeat from B
Time T=0.5: A recovers
Time T=0.5+: A sends AppendEntries as old leader (term 1)
           B and C respond: "Your term (1) is old, current term is 2"
           A sees term 2, becomes follower
```

Key: A cannot claim it's still the leader because its term is outdated.

### Follower Crashes

```
Leader A continues sending AppendEntries to B and C
B crashes at T=0.5
Leader A: "B is not responding, it's dead"
A continues with C
Client requests are still replicated to majority (A, C = 2/3 nodes)

Later, B recovers
A receives its heartbeat response
A sends all missing log entries to B
B catches up
```

### Network Partition

```
Cluster: A, B, C
Partition: [A] vs [B, C]

Partition timeline:

A side (alone):
  No followers responding
  Eventually realizes it's lost quorum (< majority responding)
  Stops accepting client requests

B-C side:
  B's timeout fires → becomes candidate
  B-C vote for B
  B becomes leader of term 2
  B-C accept client requests
  B-C replicate entries

After partition heals:
  A: "I'm leader of term 1"
  B-C: "No, we're term 2, you're out of date"
  A: Updates term to 2, becomes follower
  A catches up from B

Result: No conflicting leaders, no split-brain
```

---

## Snapshots For Practical Systems

The Raft log grows indefinitely. For instance, after 1 year of operations:
  100,000 requests per second
  365 days * 86,400 seconds = 31,536,000 seconds
  31,536,000 * 100,000 = 3.15 trillion log entries
  
Storing all entries is not feasible and replaying them to catch up a follower is slow.

### Snapshot Process

Periodically for example every 1000 entries, the node takes snapshot:

  1. Serialize state machine state (the result of applying entries 1-N)
  2. Persist to disk: [lastIncludedIndex=N, lastIncludedTerm=term of N, state=...]
  3. Discard all log entries up to index N
  
Log now looks like:
  [Snapshot: 1-N] [Log entries N+1...]
  
If node crashes and recovers:
  - Load snapshot
  - Apply log entries N+1...
  - Restore to current state


### Sending Snapshots to Followers
when a new node joined cluster or fell far behind, the leader can send it a snapshot instead of log entries.


The Leader detects nextIndex[follower] is before my first log entry and:
```
Leader sends InstallSnapshot RPC:
  term: current term
  leaderId: who I am
  lastIncludedIndex: snapshot covers 1-N
  lastIncludedTerm: term of entry N
  offset: byte offset of this chunk (for large snapshots)
  data: chunk of snapshot data
  done: true if this is last chunk

Follower:
  Receives snapshot chunks, accumulates them
  When done==true, applies entire snapshot
  Discards any log entries up to lastIncludedIndex
  Continues with log entries after lastIncludedIndex
```

---

## Practical Considerations

### Election Timeouts

Typical values:
- Heartbeat interval: 50-100ms
- Election timeout: 150-300ms (randomized)

If election timeout too short:
  - False positives: healthy leaders deposed due to delays
  - Frequent elections = instability
  
If election timeout too long:
  - Takes long time to detect failure
  - High unavailability after leader crash
  
election timeout >> heartbeat interval
 
  - Gives leader time to send heartbeats before timeout fires
  - But quick enough to detect failure


### Log Compaction via Snapshots
When to take snapshots:
- Every N entries (e.g., every 10,000 entries)
- When log size exceeds threshold (e.g., 1 GB)

Trade-offs:

Frequent snapshots:
  + Smaller logs, faster follower catch-up
  - More CPU/disk I/O for snapshots
  
Infrequent snapshots:
  + Less overhead
  - Larger logs, slower to catch up followers

### Read-only Queries

Executing queries like commands that don't modify state (e.g., "What's the value of x?") can be cause problems because of stale leaders.

Problem:
```
Old leader (partitioned) still thinks it's leader
Client queries old leader: "What's the value of x?"
Old leader returns stale value (new leader has newer value)
```

Solution: **Leader must confirm majority before responding to reads**

```
Client sends read query to leader
Leader:
  Send AppendEntries (empty) to all followers
  Wait for majority ACK (confirms leader still has quorum)
  Execute read from state machine
  Return result
```

This is **read-only optimization** for linearizability which means the clients always see the latest committed data, even if they query an old leader.

---

## Common Implementation Details

### Persistent State (must survive crashes)

```
persistentState:
  currentTerm: latest term node has seen
  votedFor: which candidate received vote in currentTerm
  log: entries of [term, command]
  
These must be persisted on disk (persisted to battery-backed RAM)
Restoring from disk after crash ensures safety
```

### Volatile State

```
volatileState:
  commitIndex: index of highest entry committed
  lastApplied: index of highest entry applied to state machine
  
These can be lost; they're recomputed from persistent state
```

### Leader State (volatile, reset after election)

```
leaderState (volatile, only on leader):
  nextIndex[i]: next log index to send to follower i
  matchIndex[i]: highest log index on follower i
  
Reset to initial values when node becomes leader
```

### Implementation Pitfalls

1. **Persisting before responding:** Write to disk before ACK, not after
2. **Applying entries in order:** Don't apply entry N+2 before N+1
3. **Reverting term:** Update currentTerm immediately, even if not candidate
4. **Duplicate handling:** Client must provide unique IDs; leader deduplicates
5. **RPC failures:** Retry forever on failures; eventually works
6. **Stale leaders:** Check term before every RPC response; revert if stale

---

## Real-World Usage

### etcd (Kubernetes configuration)
- Uses Raft for replicating key-value store
- Every write goes through Raft consensus
- Critical for Kubernetes cluster state

### Consul (Service discovery)
- Raft-based cluster coordination
- Elects leader for writes
- Followers serve reads (eventually consistent)

---


# References
- [In Search of an Understandable Consensus Algorithm
(Extended Version)](https://raft.github.io/raft.pdf) by Diego Ongaro and John Ousterhout
- [In Search of an Understandable Consensus Algorithm](https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14.pdf)
