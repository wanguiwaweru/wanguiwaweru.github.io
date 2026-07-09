# MapReduce
MapReduce is a programming model and implementation for processing large datasets in parallel across a distributed cluster. It's the foundation of Hadoop and was published by Google in 2004. The framework automatically handles parallelization, fault tolerance, and data distribution.

**Tldr**: Break a computation into independent tasks (Map) and then aggregate the results (Reduce).

---

## Fundamental Concepts

### The MapReduce Pipeline

The entire process follows this flow:

1. **Input** → Raw data in storage
2. **Map** → Process each record in parallel, emit key-value pairs
3. **Shuffle & Sort** → Group all values by key
4. **Reduce** → Aggregate values for each key
5. **Output** → Final results written to storage

### Example: Word Count

Let's say you have three text files and want to count word frequency.

**Input:**
```
File 1: "hello world hello"
File 2: "world foo"
File 3: "hello foo foo"
```

**Map Phase:**
Each mapper processes one file and emits (word, 1) for each word:

```
Mapper 1 → (hello, 1), (world, 1), (hello, 1)
Mapper 2 → (world, 1), (foo, 1)
Mapper 3 → (hello, 1), (foo, 1), (foo, 1)
```

**Shuffle & Sort:**
Group by key:

```
hello → [1, 1, 1]
foo → [1, 1, 1]
world → [1, 1]
```

**Reduce Phase:**
Sum all values for each key:

```
Reducer → (hello, 3), (foo, 3), (world, 2)
```

**Output:**
```
hello 3
foo 3
world 2
```

Here is the complete [MapReduce flow](./mapreduce_wordcount_example.svg) for counting words across multiple files.

---

### The Map Phase
The **Map function** is applied to each input record independently. It transforms a single record into zero or more intermediate key-value pairs.

```
map(key_in, value_in) → list<key_intermediate, value_intermediate>
```

- **Independent execution**: Each mapper operates on different input splits with no knowledge of other mappers. Mapper operating on file1 does not know what file2 or file3 contains.
- **Parallelism**: The entire dataset is divided into splits (typically 64-256 MB each). Each split is processed by a separate mapper task.
- **Data locality**: MapReduce tries to run the mapper on the same node where the input data is stored, minimizing network I/O.
- **Intermediate output**: Written to local disk on the mapper node, not on disk for efficiency.

### Example: Word Count Mapper

```python
def map(key_in, value_in):
    # key_in: file name or offset (ignored)
    # value_in: line of text
    words = value_in.split()
    for word in words:
        emit(word, 1)  # Emit (word, 1) for each word 
    return
```

For input `"hello world hello"`, the mapper emits:
- (hello, 1)
- (world, 1)
- (hello, 1)

---

### The Shuffle & Sort Phase
This connects mappers to reducers. Three sub-phases occur:

1. **Partition**: Intermediate pairs are partitioned by key (using a hash function)
2. **Sort**: All pairs within a partition are sorted by key
3. **Transfer**: Partitions are transferred from mappers to reducers over the network

- **Automatic grouping**: All values with the same key are guaranteed to reach the same reducer
- **Ordering**: Keys are sorted, so the reducer processes keys in a deterministic order
- **Network transfer**: This is the most expensive phase (network I/O). Reducing the data volume here is critical for performance

### Partitioning Strategy

By default, MapReduce uses hash partitioning:
```
partition = hash(key) % num_reducers
```

This distributes keys across reducers evenly. You can override this for custom partitioning.

### Example: Word Count Shuffle & Sort

Intermediate output from all mappers:
```
(hello, 1), (world, 1), (hello, 1), (world, 1), (foo, 1), (hello, 1), (foo, 1), (foo, 1)
```

After shuffle & sort (with 2 reducers):

**Reducer 0** (gets keys hashing to 0):
```
(foo, [1, 1, 1])
```

**Reducer 1** (gets keys hashing to 1):
```
(hello, [1, 1, 1])
(world, [1, 1])
```

---

### The Reduce Phase
The **Reduce function** processes each key with all its values and produces zero or more output records.

```
reduce(key, list<values>) → list<key_out, value_out>
```
- **One key at a time**: The reducer processes keys sequentially. It receives all values for a key before moving to the next key
- **Aggregation**: Typical operations include sum, min, max, concatenation, filtering
- **Output**: Written directly to storage for durability

### Example: Word Count Reducer

```python
def reduce(key, values):
    total = sum(values)  # Sum all counts for this word
    emit(key, total)     # Emit (word, total_count)

```

For input `(hello, [1, 1, 1])`, the reducer outputs:
- (hello, 3)

---


[MapReduce Hadoop Pipeline](./mapreduce_hadoop_pipeline.svg) 


## MapReduce Optimization Techniques

### Combiner
A **combiner** is an optional function that runs on each mapper node, performing a partial reduce on the mapper's output *before* data is shuffled to reducers.

Using a combiner can drastically reduce the amount of data transferred over the network, improving performance.
**Key constraint**: The combiner must be **idempotent and commutative** (like sum, min, max). It cannot be used for operations that depend on order or full context (like average, median).

**Without combiner:**
```
Mapper output: (hello, 1), (hello, 1), (world, 1), (hello, 1), ...
  → Shuffle sends lots of records over the network
```

**With combiner:**
```
Mapper output: (hello, 1), (hello, 1), (world, 1), (hello, 1), ...
  → Combiner reduces to: (hello, 3), (world, 1), ...
  → Shuffle sends only fewer records over the network
```

The combiner and reducer functions can be the same function especially in cases where they achieve the same goal.

---

### Data Locality
MapReduce applies the principle of **data locality** where you move computation to data, not the data to computation. When scheduling a mapper task, it is scheduled to run on one of the nodes that stores that input block.

- **Network is expensive**: Transferring data across the network is 100x slower than reading from local disk
- **Hadoop's advantage**: Map Reducee is used in Hadoop and HDFS replicates each block to 3 nodes. Hadoop's scheduler can almost always find a mapper node that has the data

On Hadoop, the process is as follows:
1. Hadoop checks which nodes have the block from HDFS metadata
2. It submits the mapper task to one of those nodes
3. The mapper reads data from the local disk therefore there isno network transfer

---

### Fault Tolerance
MapReduce is resilient to failures through several mechanisms:

**1. Task-level recovery:**
- If a mapper task fails, Hadoop re-runs it on a different node
- Failed reducers are also re-run
- Intermediate data from failed mappers can be regenerated since it's on local disk

**2. Data replication:**
- HDFS replicates each block to 3 nodes
- If a node dies, data is still available on replicas
- Final reducer output is written to HDFS which is replicated, so it survives node failures

**3. Speculative execution:**
- If a task is running slow, Hadoop may start a duplicate task on another node
- Whichever finishes first is used and the other is killed
- This handles stragglers without waiting ensuring they don't slow down the entire job.

### Limitations

- **In-flight task failures**: If a task fails mid-execution, it must be re-run (expensive)
- **Mapper failures after intermediate data**: The intermediate data is lost and must be regenerated
- **Network partition**: If part of the cluster disconnects, tasks on both sides may fail

---

## Common MapReduce Patterns

### 1. Aggregation (Counting, Summing)

**Use case**: Count occurrences, sum values

**Pattern:**
- Map: Emit (key, 1) for each occurrence
- Reduce: Sum all 1s
- Combiner: Also sums (identical to reduce)

**Example**: Word count, counting unique users

### 2. Filtering

**Use case**: Extract records matching a condition

**Pattern:**
- Map: Emit record only if it matches the filter
- Reduce: Identity function (pass through)
- Combiner: Not needed

**Example**: Extract logs for errors only, filter users by location

### 3. Transformation

**Use case**: Restructure or parse data

**Pattern:**
- Map: Transform each record to a new format
- Reduce: Identity or light aggregation
- Combiner: Usually not needed

**Example**: Converting CSV to JSON, extracting fields

### 4. Grouping & Aggregation

**Use case**: Group by a key and compute statistics

**Pattern:**
- Map: Emit (grouping_key, data)
- Reduce: Aggregate data for each group
- Combiner: Often beneficial for partial aggregation

**Example**: Average salary by department, sales per region

### 5. Joining (Reduce-side Join)

**Use case**: Join two datasets by key

**Pattern:**
- Map: Emit (join_key, (source_id, value)) for both datasets
- Reduce: Collect all values by key, join records from both sources
- Combiner: Not applicable

**Example**: Join users with their orders

---

## When to Use MapReduce

### Good Fit

- **Batch processing**: Non-real-time jobs that can tolerate minutes/hours of latency
- **Massive scale**: Gigabytes to terabytes of data
- **Simple operations**: Counting, aggregation, filtering, transformations
- **Offline analytics**: Historical analysis, log processing, data warehousing
- **Fault tolerance critical**: You need guaranteed fault recovery (no partial results)

### Poor Fit

- **Real-time**: Anomaly detection, live dashboards, instant feedback (use Spark Streaming, Kafka)
- **Complex algorithms**: Machine learning, graph processing (use Spark MLlib, Pregel)
- **Iterative jobs**: Requires multiple MapReduce passes (expensive). Use Spark instead
- **Small datasets**: Overhead of job setup, data replication not worth it (just use a single machine)
- **Unstructured queries**: Ad-hoc analytics (use Hive/Pig for SQL-like interface)

## Real-World Example: Inverted Index

**Problem**: Build an inverted index from a corpus of documents (needed for search engines).

**Input**: Documents with IDs
```
doc1: "the quick brown fox"
doc2: "quick fox runs"
doc3: "the running fox"
```

**Desired output**: For each word, list all documents that contain it
```
brown: [doc1]
fox: [doc1, doc2, doc3]
quick: [doc1, doc2]
runs: [doc2]
the: [doc1, doc3]
...
```

**MapReduce solution:**

**Mapper:**
```
for each document (doc_id, content):
  for each word in content:
    emit (word, doc_id)
```

Output:
```
(the, doc1), (quick, doc1), (brown, doc1), (fox, doc1)
(quick, doc2), (fox, doc2), (runs, doc2)
(the, doc3), (running, doc3), (fox, doc3)
```

**Reducer:**
```
for each word and its list of doc_ids:
  emit (word, [unique doc_ids sorted])
```

Output:
```
brown [doc1]
fox [doc1, doc2, doc3]
quick [doc1, doc2]
running [doc3]
runs [doc2]
the [doc1, doc3]
```

---