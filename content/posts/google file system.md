---
title: "Google File System (GFS)"
date: "2026-05-20"
draft: false
---

# Google File System (GFS)

GFS is a distributed file system designed by Google to handle massive data processing workloads. The GFS paper published in 2003 was foundational to understanding how to build practical distributed systems at scale.

GFS prioritizes **reliability, availability, and throughput** over traditional file system which priotizes strict consistency, and low latency random writes. These are crucial design decisions that influence everything else.

---

## Design Goals & Assumptions

### What GFS Optimizes For

**1. Large-scale data processing**
- Gigabytes to terabytes of data
- Hundreds to thousands of machines
- High throughput rather than low latency

**2. Fault tolerance**
- Machines are expected to fail regularly and failure is not treated as an exception
- Network partitions happen
- The system must recover without user intervention

**3. Append-heavy workload**
- The system is optimized for workloads where most writes are append not overwrites
- Files are written once and read many times
- Sequential reads are the norm

**4. Throughput over latency**
- Optimize for **aggregate throughput**, not single-request latency
- A batch of 1000 slow requests completing is better than 1 fast request and 999 requests waiting.


The fundamental assumption that drives GFS design is

> Component failures are the norm rather than the exception.

This assumption means:
- **Replication is mandatory**, not optional
- **Failure detection must be automatic**, not manual
- **Recovery must be automatic**, not requiring operator intervention
- **Data integrity checks must be constant**, not occasional

---

## GFS Architecture 
[GFS architecture with Master, Chunkservers, and Clients](gfs_architecture_visualization.svg)

### Three Main Components

```
┌─────────────────────────────────────┐
│    GFS Master (1 per cluster)       
│  - Namespace management                 
│  - File system tree                     
│  - Chunk replica placement             
│  - Chunk lease management               
└─────────────────────────────────────┘
           ↓ heartbeat + commands
┌───────────────────────────────────┐
│    GFS Chunkservers (100s to 1000s)          
│  - Store chunks (64 MB each)                 
│  - Perform read/write operations             
│  - Report chunk inventory to master          
│  - Handle replication and deletion           
└───────────────────────────────────┘
           ↓ read/write
┌──────────────────────────────────┐
│           GFS Clients                        
│  - Contact master for metadata               
│  - Contact chunkservers for data             
│  - Cache metadata locally                    
└──────────────────────────────────┘
```

### The Master

The master is **not** a data node. It stores:
- **Namespace**  which is the file system tree
- **Access control lists** (ACLs) determining who can read/write files
- **File-to-chunks mapping** showing which chunks belong to which file
- **Chunk locations** showing where each chunk is replicated
- **Chunk leases** showing temporary write authority

It also controls system-wide activities such as chunklease
management, garbage collection of orphaned chunks, and
chunkmigration between chunkservers. The master periodically communicates with each chunkserver through `HeartBeat
messages` to give instructions and collect its state

All this information is stored in memory for fast retrieval. Latency for  metadata operations must be near-instant. The master never stores the actual data.

The master writes operation logs to disk and replicates them before applying them to ensure durability incase it crashes. When it recovers after a crash, it replays the log and restores its state.

### Chunkservers

Each chunkserver stores chunks which generally were 64 MB blocks of data in the original system. Systems like Hadoop Distributed File System (HDFS) which are inspired by GFS use 128 MB chunks.

When a client wants to read or write, it:
1. Asks the master for chunk locations
2. Contacts the nearest chunkserver directly
3. Reads/writes the data

Chunkservers don't know the file structure, they just store and serve chunks.

The chunkservers periodically send heartbeats
to the master, enabling the master to detect when new
chunkservers come online and when chunkservers fail.

Chunkservers are the source of truth for
the chunk handles that they store; the master requests
chunkservers that have just come online to send their list
of chunk handles.

### Clients

Clients contact the master node for **metadata only**:
- "Where is chunk 12345 of file /data/dataset.txt?"

Master responds with locations. Client then goes to chunkservers for data.

Clients cache metadata locally to reduce the load on the master which could become a bottleneck if every read/write request goes through it.

---

## The Chunk Abstraction

### Why 64 MB Chunks?

This is a deliberate design choice. Traditional file systems use 4 KB blocks but GFS uses 64 MB because:

1. **Reduced master load**: Fewer chunks = less metadata to manage
   - 1 TB file = 1024GB = 1048576MB = 16,384 chunks of 64 MB instead of 268,435,456 chunks with 4 KB blocks
   - Master overhead per file is minimal
   
2. **Reduced network traffic**: Fewer client-master interactions
   - One metadata request can cover a large portion of data
   
3. **Better for sequential access**: Most GFS workloads read sequentially
   - One chunk access can serve multiple GB of sequential reads
   
4. **Reduced metadata replication overhead**: Less metadata to replicate

**Costs of large chunks:**
1. **Internal fragmentation**: Small files waste space
   - A 1 MB file takes a full 64 MB chunk
   - GFS assumes large files, so this is acceptable
   
2. **Hot spots**: Popular chunks can become bottlenecks
   - If one chunk is read by many clients, that chunkserver gets hammered
   - GFS handles this with replication and spreading reads

### Chunk Replication

The default is 3 replicas though this is configurable

The **placement strategy** is crucial for fault tolerance:
- 1 replica on the same node as the writer (if possible)
- 2 replicas on different racks
- Never put all 3 replicas on the same rack
This placement strategy ensures:
- Tolerates most common failures (single machine, single rack)
- Balances rack bandwidth usage with fault tolerance.
- Avoids correlated failures  such as all machines on one rack losing power simultaneously

---

## Read Path
**Step 1: Client asks master for metadata**

```
Client: "Where are the chunks for file /data/users.txt, starting at offset 0?"

Master response:
{
  "file": "/data/users.txt",
  "chunks": [
    {
      "chunk_handle": 1001,
      "version": 3,
      "locations": ["chunkserver_A:12345", "chunkserver_B:12345", "chunkserver_C:12345"]
    },
    {
      "chunk_handle": 1002,
      "version": 3,
      "locations": ["chunkserver_D:12345", "chunkserver_E:12345", "chunkserver_F:12345"]
    },
    ...
  ]
}

Client caches this metadata for 60 seconds (default TTL)
```

**What the client gets:**
- **Chunk handle**: A unique ID like "chunk 1001".
- **Version number**: Ensures all replicas have the same version of the chunk. If a replica is stale, it has a lower version number. Version increments each time the chunk changes.
- **Locations**: List of chunkservers holding this chunk. Always 3 (default replication factor).

**Step 2: Client reads from nearest chunkserver**

```
Client (running on host X in Rack A): "I need bytes 0-65536 from chunk 1001"

Client determines which replica is nearest by network topology:
  - Same machine? (never, clients and chunkservers are different)
  - Same rack? (preferred: inside-rack bandwidth is free)
  - Same building? (still in-DC, manageable)
  - Closest data center? (if cross-DC)

Client picks Rack A (same rack as itself)
  → Avoids inter-rack bandwidth (expensive)

Client sends read request directly to chunkserver:
{
  "chunk_handle": 1001,
  "byte_offset": 0,
  "byte_length": 65536,
  "chunk_version": 3
}

Chunkserver A responds:
{
  "data": [65536 bytes of actual file content]
}
```

Reading directly from chunkserver:
- Eliminates unnecessary hops
- Master only handles metadata making it fast
- Data I/O is between client and chunkserver (parallelizable)
- Enables data locality where tasks such as mapreduce jobs can be scheduled on nodes that have the data locally, reducing network traffic and improving performance.

**Step 3: If chunkserver fails or is slow, client retries**

```
Scenario 1: Replica is unreachable (dead)

Client tries Rack1 replica: [connection refused]

Client immediately tries Rack2 replica: [success, returns data]

[Master detects Rack1 is dead via heartbeat timeout (3+ min)]
[Master re-replicates chunk 1001 from Rack2 to new server]
```

```
Scenario 2: Replica is slow (but not dead)

Client tries Rack1 replica: [responding, but slow]

If read takes > 1 second:
  Client can start a "shadow read" to Rack2
  Whichever responds first is used
  (Optimization: don't wait for slow replicas)
```

### Read Consistency 

GFS makes the following guarantees about data visibility when you read:

**Guarantee 1: All clients see the same data after a write completes**

```
Time T=0: Client A writes "HELLO" to file F, offset 0
          Master grants lease to primary
          Primary buffers "HELLO", replicates to secondaries
          
Time T=1: Primary ACKs: "Write complete"

Time T=2: Client B reads from chunk at offset 0
          Gets "HELLO" from replica 1
          
Time T=2.5: Client C reads from chunk at offset 0
            Gets "HELLO" from replica 2
            
Result: Both B and C read the same data
```

**Guarantee 2: Writes are serialized by the primary**

```
Time T=0: Client A writes "HELLO" at offset 0
          Client B writes "WORLD" at offset 1000
          
          Both writes routed to the same primary
          Primary orders them:
            - Write 1: "HELLO" at offset 0
            - Write 2: "WORLD" at offset 1000
          
          Primary replicates in this exact order
          All replicas apply in same order
          
Time T=2: Client C reads offset 0: gets "HELLO"
          Client C reads offset 1000: gets "WORLD"
          [Same order across all reads]
```

**Guarantee 3: Data is NOT immediately durable**

```
Time T=0: Client A writes "HELLO" to primary
          Primary buffers in memory
          
Time T=0.5: Primary replicates to secondaries
            Secondaries buffer in memory (not persisted)
            
Time T=0.8: Primary ACKs to client: "Write complete"
            Client thinks write is done
            
Time T=0.9: Primary crashes before fsyncing
            [In-memory buffer lost]
            
Time T=1: Secondary has the write in its buffer
          Secondary crashes
          [Secondary buffer also lost]
          
Result: Write is lost!
        (This is rare but possible in GFS)
```

**No guarantee: Latest value is immediately visible**

```
Time T=0: Client A writes to file F, get ACK
Time T=0.05: Client B reads from file F
             B might read from slow replica that hasn't applied write yet
             B sees OLD data (not the write from A)
             
This is a key tradeoff:
  - No immediate visibility requires waiting for latest replica
  - Waiting for latest replica = higher latency
  - GFS chooses availability over immediate consistency
```

The weak consistency model is acceptable for GFS's target workloads (batch processing) because:
1. **Avoids expensive coordination**: No need for consensus protocols like Paxos or Raft for every write. This keeps latency low and throughput high. Requiring `strict consistency` would make reads slow and the master a bottleneck as every read would have to check with the master and wait for all replicas to sync.
2. **Fits the use case perfectly**: Most GFS workloads are batch jobs that write large files and then read them later.
3. **High availability**:  Clients can read from any replica immediately, even if it's stale. This is better than blocking reads until the latest replica is available.

---

## Write Path

Writing in GFS is less about a single clean write operation and more about orchestrating a distributed update under the assumption that failures are ordinary rather than exceptional. 

When a client wants to modify a chunk, it first asks the master for the chunk’s metadata and the **current lease holder**, which is the replica temporarily authorized to act as the primary for that chunk. That primary becomes the **serialization point** for the write: it accepts the update, orders it, and forwards the data to the other replicas so they can apply the same change in the same sequence. 

This design is a deliberate compromise. Instead of paying the cost of full consensus for every write, GFS lets one replica hold temporary authority over a chunk, which keeps the common case fast and avoids the coordination overhead that would make throughput collapse under heavy load. 

The primary buffers the incoming data, replicates it to the secondaries, and only then acknowledges the client, which means the system is optimized for high-throughput batch workloads rather than for strict transactional durability. 

If a replica is slow or crashes, the write can still proceed with the remaining copies, and the master will eventually repair the replication gap in the background. If the primary itself fails mid-flight, the update may be lost, but that risk is considered acceptable in a system designed around massive-scale data processing where retries and redundancy are part of the operating model.

### Write Consistency Guarantees

The guarantees GFS provides are therefore pragmatic rather than idealized.
 - All replicas apply writes in the same order, so once the write has propagated, the chunk state is consistent across copies even if the replicas were temporarily out of sync during the process. 
 - Temporary divergence is possible while the replication is still in flight, which means a client might read slightly stale data from one replica while another has already observed the newer version. This is a conscious tradeoff where GFS favors availability and throughput over immediate, globally synchronized visibility. 
 - Durability is also delayed rather than immediate, because data is only safely persistent after the replicas eventually move it to stable storage. The result is a system that is excellent for large, append-heavy batch jobs, but not for workloads that require tight transactional guarantees or instant consistency across every replica.

---

### GFS's Consistency Guarantees

GFS does **not** provide **strict consistency**. Instead, it provides:

| State | Guarantee |
|-------|-----------|
| **After write completes** | All replicas have the same data (serialized by primary) |
| **Immediate after write** | Data is in replica buffers, not yet on disk |
| **After all replicas fsync** | Data is durable and crash-safe |
| **During replica divergence** | Clients might see different data from different replicas though rare |

### Defined vs. Undefined Regions

GFS distinguishes between two states of a file region:

**Defined region:**
- The data written is exactly what the client wrote
- All replicas have the same bytes

**Undefined region:**
- The data written may contain:
  - Duplicates where same write is applied multiple times
  - Padding where null bytes from failed writes are present
  - Interleaved writes where multiple writes are mixed

When can undefined regions happen?

**Example: Primary crashes during replication**

Suppose a client writes the bytes "HELLO" to offset 0, and the primary replica buffers the update and sends it to the secondaries, but one replica never receives it because of a network failure and the primary crashes before acknowledging the client. When the client retries, it contacts the master, which grants a new lease to another replica, and the write is resent, but now one replica has the update while the other does not, leaving the affected region in an undefined state where different copies of the chunk disagree about what data should be present.

GFS handles this with:
1. **Checksums** at the chunk level
2. **Application-level checksums** by writing checksums with data
3. **Deduplication** to detect and ignore duplicate writes
4. **Idempotent operations** which are safe to retry without corrupting state

---

## Comparison with Traditional File Systems

| Property | Traditional FS | GFS |
|----------|---|---|
| **Consistency** | Strict (after fsync) | Eventual |
| **Replication** | Manual backup | Automatic, default 3x |
| **Fault handling** | Operator intervention | Automatic |
| **Chunk size** | 4 KB | 64 MB |
| **Master** | N/A | Single, in-memory |
| **Write semantics** | Random writes | Append-only |
| **Use case** | General purpose | Large-scale data processing |

---

### Modern Variants

- **HDFS** (Hadoop): Open-source GFS clone
- **Bigtable**: Builds on GFS for structured data
- **Spanner**: Google's next-generation system (adds stronger consistency)
- **Colossus**: Google's next-gen file system (improves on GFS design)

## Resources
- [MIT GFS lecture](https://www.youtube.com/watch?v=EpIgvowZr00&list=PLrw6a1wE39_tb2fErI4-WkMbsvGQk9_UB&index=3)
- [The Google File System](https://static.googleusercontent.com/media/research.google.com/en//archive/gfs-sosp2003.pdf)