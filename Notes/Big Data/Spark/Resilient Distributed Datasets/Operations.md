# Operations
## Read (Creating RDDs)

- `sc.parallelize([1,2,3])` → create RDD from local collection.
- `sc.textFile("path")` → read text file into RDD.
- `sc.wholeTextFiles("path")` → read directory of files (filename, content).

```python
from pyspark import SparkContext
sc = SparkContext("local", "RDD_Example")

# Create RDD from collection
rdd1 = sc.parallelize([1, 2, 3, 4, 5])

# Read RDD from file
rdd2 = sc.textFile("data.txt")

```

## Transformations (Lazy)

Transformations build new RDDs without executing immediately.

### Narrow Transformations
- Narrow transformations are operations where each partition of the parent RDD is used by **at most one** partition of the child RDD.
	- **Data Movement:** No data shuffling is required across the network.
	- **Execution:** These can be executed in a single "pipeline" on a single executor node. If a node fails, only the missing partition needs to be recomputed.
	- **Performance:** High; they are generally very fast because they avoid the overhead of network I/O.
- Examples:
    - `map(func)`
    - `filter(func)`
    - `flatMap(func)`
    - `union(rdd)`
    - `sample(withReplacement, fraction)`
```python
# map → element-wise transformation
mapped = rdd1.map(lambda x: x * 2)

# filter → keep elements satisfying condition
filtered = rdd1.filter(lambda x: x % 2 == 0)

# flatMap → multiple outputs per input
flatmapped = rdd1.flatMap(lambda x: (x, x*x))

# union → combine two RDDs
unioned = rdd1.union(sc.parallelize([6, 7]))

# Sample without replacement (approx 40% of elements)
sample1 = rdd.sample(False, 0.4)
print("Sample without replacement:", sample1.collect())
```
### Wide Transformations

- Wide transformations (also known as **Shuffle Transformations**) occur when multiple child partitions may depend on data from a single parent partition.
	- **Data Movement:** Requires a **Shuffle** operation. Data must be reorganized, partitioned, and sent across the network to different nodes.
	- **Execution:** Spark breaks the job into a new **Stage** whenever a wide transformation occurs.
	- **Performance:** Expensive; they involve disk I/O, data serialization, and network latency.
- Examples:
    - `groupByKey()`
    - `reduceByKey(func)`
    - `join(rdd)`
    - `distinct()`
    - `cogroup(rdd)`

```python
pairs = sc.parallelize([("a", 1), ("b", 2), ("a", 3)])

# groupByKey → groups values by key
grouped = pairs.groupByKey()

# reduceByKey → aggregation by key
reduced = pairs.reduceByKey(lambda x, y: x + y)

# join → join two RDDs by key
rddA = sc.parallelize([("a", 1), ("b", 2)])
rddB = sc.parallelize([("a", 3), ("b", 4)])
joined = rddA.join(rddB)

# distinct → remove duplicates
distincted = rdd1.distinct()

# Groups the values of two (or more) RDDs sharing the same key.
cogrouped = rdd1.cogroup(rdd2)

# Collect results
for key, (vals1, vals2) in cogrouped.collect():
    print(key, list(vals1), list(vals2))
```


## Actions (Eager)

Actions trigger execution and return results or write output.

- `collect()` → return all elements to driver.
- `count()` → number of elements.
- `first()` → first element.
- `take(n)` → first _n_ elements.
- `reduce(func)` → aggregate elements.
- `foreach(func)` → apply function to each element.
```python
print("Collect:", mapped.collect())       # → [2,4,6,8,10]
print("Count:", rdd1.count())             # → 5
print("First:", rdd1.first())             # → 1
print("Take:", rdd1.take(3))              # → [1,2,3]
print("Reduce:", rdd1.reduce(lambda x,y: x+y))  # → 15

# foreach → apply function (side effects only)
rdd1.foreach(lambda x: print(x))

```

##  Write (Persisting Results)

- `saveAsTextFile("path")` → store as text.
- `saveAsSequenceFile("path")` → Hadoop sequence format.
- `saveAsObjectFile("path")` → serialized objects.

```python
# Save as text file
mapped.saveAsTextFile("output/mapped")

# Save as Sequence file (Hadoop format)
pairs.saveAsSequenceFile("output/seq")

# Save as object file
rdd1.saveAsObjectFile("output/obj")

```



## Caching vs. Persistence

While often used interchangeably, there is a technical distinction:
- **`cache()`**: This is the "easy button." It is a shorthand for `persist()` using the default storage level (**MEMORY_ONLY**).
- **`persist()`**: This is the flexible version. It allows you to specify exactly where and how you want the data stored by passing a `StorageLevel`.
### Storage Levels

When using `persist()`, you can choose how Spark handles the data. This is a classic trade-off between **CPU speed** and **Memory usage**.

| **Storage Level**   | **Meaning**                                                                                                                                  |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **MEMORY_ONLY**     | Stores RDD as deserialized Java objects in the JVM. If it doesn't fit, it re-computes the missing parts on the fly. (Default for `cache()`). |
| **MEMORY_AND_DISK** | Stores in memory, but spills to disk if the RDD is too large. This is usually the "safest" bet.                                              |
| **DISK_ONLY**       | Stores the RDD only on disk. Slower to read than RAM, but still faster than re-running complex transformations.                              |
| **MEMORY_ONLY_SER** | Stores RDD as **serialized** (compact) byte arrays. Saves space but costs CPU cycles to "unzip" the data.                                    |

> **Note:** Most levels have a `_2` variant (e.g., `MEMORY_ONLY_2`). This replicates the data on two cluster nodes for fault tolerance, so if one node crashes, you don't lose the cache.

```python
from pyspark import StorageLevel

# This stores data on Disk if it doesn't fit in Memory
filtered_rdd.persist(StorageLevel.MEMORY_AND_DISK)
```

### When should you cache?

Don't cache everything! Caching consumes the same memory that Spark needs for joins and aggregations. Use it when:

1. **Iterative Algorithms:** Like Machine Learning (e.g., Logistic Regression) where the same data is looped over multiple times.
2. **Multiple Actions:** When you plan to run a `count()`, then a `saveAsTextFile()`, then a `sum()` on the same transformed dataset.
3. **Heavy Computation:** If an RDD took 20 minutes to filter and map, you definitely want to save that state.

```python
import time
from pyspark import SparkContext

sc = SparkContext("local", "Caching Example")

# 1. Create a large RDD
raw_data = sc.parallelize(range(1, 10000001))

# 2. Perform a "heavy" transformation (e.g., filtering even numbers)
# This isn't actually executed yet due to Lazy Evaluation
filtered_rdd = raw_data.filter(lambda x: x % 2 == 0)

# --- SCENARIO A: WITHOUT CACHING ---
start_time = time.time()
print(f"First Count: {filtered_rdd.count()}") 
print(f"Time taken (First Action): {time.time() - start_time:.2f} seconds")

start_time = time.time()
print(f"Second Count: {filtered_rdd.count()}") 
print(f"Time taken (Second Action - Recomputing): {time.time() - start_time:.2f} seconds")

# --- SCENARIO B: WITH CACHING ---
filtered_rdd.cache()  # Mark it for caching

# The first action after cache() still takes time because Spark has to build the cache
start_time = time.time()
print(f"Third Count (Building Cache): {filtered_rdd.count()}") 
print(f"Time taken: {time.time() - start_time:.2f} seconds")

# The fourth action will be blazing fast!
start_time = time.time()
print(f"Fourth Count (Using Cache): {filtered_rdd.count()}") 
print(f"Time taken: {time.time() - start_time:.2f} seconds")
```

### How to release memory

Spark automatically monitors cache usage and drops old data using a **Least Recently Used (LRU)** algorithm. However, if you want to be a good "cluster citizen," you can manually clear an RDD from memory when you're done with it:

```python
rdd.unpersist()
```



## Shared Variable

### 1. Broadcast Variables (Read-Only)

A **Broadcast Variable** allows you to keep a read-only cache of a variable on each **machine** (executor) rather than shipping a copy with every task.
- **Best for:** Large lookup tables or configuration parameters.
- **Key Rule:** They are immutable (read-only) once broadcasted.

```python
# A small dictionary we want to use across the cluster
country_map = {"US": "United States", "IN": "India", "UK": "United Kingdom"}

# Broadcast it!
broadcast_countries = sc.broadcast(country_map)

# Use it inside an RDD transformation
rdd = sc.parallelize(["US", "IN", "UK", "US"])
result = rdd.map(lambda x: broadcast_countries.value.get(x)).collect()
# Output: ['United States', 'India', 'United Kingdom', 'United States']
```

### 2. Accumulators (Write-Only)

Accumulators are Spark's version of a "global counter." While workers can "add" to them, they **cannot read them**. Only the Driver program can read the final value.

This is perfect for debugging or counting specific events (like "how many rows had errors?") without the overhead of a full RDD reduction.
- **Best for:** Counters, sums, or diagnostic metrics.
- **Key Rule:** Only the Driver can call `.value`.

```python
# Initialize an accumulator
error_count = sc.accumulator(0)

def validate_data(x):
    global error_count
    if x < 0:
        error_count += 1  # Workers can add to it
        return False
    return True

data = sc.parallelize([10, -1, 20, -5, 30])
valid_data = data.filter(validate_data).collect()

print(f"Total errors found: {error_count.value}") 
# Output: Total errors found: 2
```

### Quick Comparison

| **Feature**          | **Broadcast Variables**                    | **Accumulators**                                    |
| -------------------- | ------------------------------------------ | --------------------------------------------------- |
| **Primary Purpose**  | Efficiently sharing large, read-only data. | Aggregating information from workers to the driver. |
| **Worker Access**    | Read-only (`.value`).                      | Write-only (`+=`).                                  |
| **Driver Access**    | Read/Write.                                | Read-only (`.value`).                               |
| **Typical Use Case** | Join optimization, lookup tables.          | Row counters, error logging, sum of values.         |
