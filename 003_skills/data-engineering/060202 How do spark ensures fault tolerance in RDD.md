---
up:
  - "[[003_skills/data-engineering/0602 Data abstraction|0602 Data abstraction]]"
down:
prev:
topic: false
question: How do spark ensures fault tolerance in RDD?
---
# How do spark ensures fault tolerance in RDD?

> [!Summary] Summary
> - RDD, fault tolerance is ensured using lineage. lineage is an execution plan that records: how RDD is derived, transformation applied, and parent rold If a RDP partition is lost using this information it recompute the lost RDD. 
> - The immutability of RDD make it safe and deterministic to re-compute 
> - When a partition or RDD is lost. spark uses lineage to identify and recompute the partition or RDD. 
> - It recomputes only lost partitions. 
> - Lazy evaluation helps in recovery with less complication 
> - spark retries failed task and also launches duplicate task if a node is abnormally slow. 

Spark ensures **fault tolerance in RDDs** primarily through a mechanism called **lineage**, rather than data replication 

## What is Lineage?

**Lineage** is a **logical execution plan** that records:

- How an RDD was **derived**
- Which **transformations** were applied
- Which **parent RDDs** were used

Spark keeps this information in memory.

If data is lost, Spark **recomputes only the lost partitions**, instead of re-running the entire job.

## How Fault Tolerance Works Step‑by‑Step

### RDDs Are Immutable

- Once created, an RDD **cannot be changed**
- Any transformation produces a **new RDD**

This immutability makes re-computation **safe and deterministic**

### Lineage-Based Recovery (Main Mechanism)

If a node fails and loses an RDD partition:

1.  Spark detects the failure
2.  Uses lineage graph to:
    - Identify missing partitions
    - Trace back required transformations
3.  Recomputes **only the lost partitions**
4.  Continues execution


### Lazy Evaluation Helps Recovery

- Spark does **not execute transformations immediately**
- Builds the full lineage graph first

This allows Spark to:

- Optimize execution
- Recompute data accurately upon failure

### Task Retries

- If a task fails:
    - Spark automatically **retries it**
    - On the same or different executor
- Default retry count: **4**

Handles transient failures like network or memory issues

### Speculative Execution

- If a task runs abnormally slow:
    - Spark launches a **duplicate task**
- The result of the fastest task is used

Handles “slow nodes” (stragglers) gracefully

### Data Replication When Cached

When an RDD is **cached or persisted**:

```python
rdd.persist(StorageLevel.MEMORY_ONLY_2)
```

- Spark stores **replicas** across executors
- If one copy is lost, another is used
Reduces recomputation cost

### Checkpointing for Long Lineages

For very long lineage chains:

```python
sc.setCheckpointDir("/checkpoint")
rdd.checkpoint()
```

- RDD is saved to **HDFS**
- Lineage is **cut off**

Prevents expensive recomputation and stack overflow


## Narrow vs Wide Dependencies (Important)

### Narrow Dependencies

- Each partition depends on **one parent partition**
- Easy and fast to recompute

Examples:

- `map`
- `filter`


### Wide Dependencies

- Partition depends on **multiple parent partitions**
- Requires shuffle

Examples:

- `reduceByKey`
- `groupByKey`

Spark still recovers efficiently, but wide dependencies are costlier

## Comparison with MapReduce Fault Tolerance

| Feature           | MapReduce              | Spark                       |
| ----------------- | ---------------------- | --------------------------- |
| Fault tolerance   | Disk-based replication | Lineage-based recomputation |
| Intermediate data | Always on disk         | Mostly in memory            |
| Recovery cost     | High                   | Low                         |
| Iterative jobs    | Poor                   | Excellent                   |


