# Memory Optimization

## Spark Memory Split Overview

### 1. On-Heap Memory
- Managed by JVM garbage collector.
- Divided into:
    - **Execution Memory**: For shuffles, joins, sorts, aggregations.
    - **Storage Memory**: For caching RDDs/DataFrames.
- Configs:
    - `spark.executor.memory`: Base JVM heap size.
    - `spark.memory.fraction`: Fraction of heap for unified memory (default 0.6).
    - `spark.memory.storageFraction`: Fraction of unified memory for storage (default 0.5).

### 2. Off-Heap Memory
- Allocated outside JVM heap, reducing GC overhead.
- Useful for **Tungsten project**, shuffle buffers, and native libraries.
- Configs:
    - `spark.memory.offHeap.enabled` (default: false).
    - `spark.memory.offHeap.size`: Explicit off-heap allocation size.
    - `spark.executor.memoryOverhead`: Extra memory for native overheads (default 10% of executor memory, min 384 MB).
- Total off-heap = `spark.executor.memoryOverhead + spark.memory.offHeap.size`
### Key Spark memory configs (concise table)

| **Config**                      | **Purpose**                            | **Default**                | **Notes**                         | **Example** |
| ------------------------------- | -------------------------------------- | -------------------------- | --------------------------------- | ----------- |
| `spark.executor.memory`         | JVM heap per executor                  | user set                   | excludes overhead                 | `8g`        |
| `spark.memory.fraction`         | Fraction of heap for unified memory    | `0.6`                      | controls execution+storage        | `0.6`       |
| `spark.memory.storageFraction`  | Fraction of unified memory for storage | `0.5`                      | storage vs execution split        | `0.5`       |
| `spark.executor.memoryOverhead` | Native/container overhead              | `10%` of executor (≥384MB) | includes off‑heap needs           | `1g`        |
| `spark.memory.offHeap.enabled`  | Toggle off‑heap allocation             | `false`                    | must be true to use off‑heap pool | `true`      |
| `spark.memory.offHeap.size`     | Size of off‑heap pool                  | `0`                        | used when off‑heap enabled        | `2g`        |
![[Notes/Big Data/Spark/media/Spark_memory.png]]

## In-memory caching

###  Key Concepts of In-Memory Caching in Spark
- **Definition**: Data is stored in **RAM** instead of disk, enabling faster access and computation.
- **Use Cases**:
  - Iterative algorithms (e.g., machine learning training loops).
  - Interactive queries where the same dataset is reused.
  - Complex pipelines with repeated transformations.
- **Benefits**:
  - **Reduced latency**: Avoids recomputation and disk I/O.
  - **Improved performance**: Especially for large-scale analytics.
  - **Cost efficiency**: Memory is cheaper than repeated compute cycles.
### How to Use Caching
- **DataFrame/Dataset API**:
  - `df.cache()` → Stores data in memory (default storage level).
  - `df.persist(StorageLevel.MEMORY_AND_DISK)` → More control over storage.
- **SQL API**:
  - `spark.catalog.cacheTable("tableName")` → Caches a table in memory.
- **Uncaching**:
  - `df.unpersist()` → Removes cached data to free memory.

### Storage Levels in Spark

| Storage Level | Description | Use Case |
|---------------|-------------|----------|
| MEMORY_ONLY | Stores RDD/DataFrame in RAM only | Fastest, but may fail if data doesn’t fit |
| MEMORY_AND_DISK | Keeps in RAM, spills to disk if needed | Safer for large datasets |
| MEMORY_ONLY_SER | Serialized format in RAM | More space-efficient |
| MEMORY_AND_DISK_SER | Serialized in RAM, spills to disk | Balance between efficiency and reliability |
| DISK_ONLY | Stores only on disk | Rarely used; slowest |
### Trade-offs & Risks
- **Memory Pressure**: Large datasets may exceed RAM, causing eviction or spilling to disk.
- **Serialization Overhead**: Serialized caching saves space but adds CPU cost.
- **Cluster Resource Management**: Over-caching can starve other jobs of memory.

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("CachingExample").getOrCreate()

df = spark.read.csv("bigdata.csv", header=True, inferSchema=True)

# Cache DataFrame in memory
df.cache()

# Trigger action to materialize cache
df.count()

# Later free memory
df.unpersist()
```

## On-Heap vs. Off-Heap Memory in Spark

|Aspect|On-Heap Memory|Off-Heap Memory|
|---|---|---|
|**Location**|Inside JVM heap|Outside JVM heap|
|**Management**|Controlled by JVM Garbage Collector|Managed by Spark (via Tungsten project)|
|**Performance**|Can suffer from GC pauses|Reduces GC overhead, faster for large data|
|**Serialization**|Objects stored in JVM format|Data stored in serialized binary format|
|**Use Cases**|Small to medium workloads|Large datasets, iterative ML workloads|
### Tuning Parameters

#### On-Heap Memory

- **`spark.executor.memory`** → Defines total JVM heap size per executor.
- **`spark.driver.memory`** → Memory allocated to the driver JVM.
- **`spark.memory.fraction`** (default: 0.6) → Fraction of heap for execution + storage.
- **`spark.memory.storageFraction`** (default: 0.5 of above) → Portion reserved for cached data.

#### Off-Heap Memory

- **Enable Off-Heap**:
    
    ```bash
    spark.memory.offHeap.enabled=true
    ```
    
- **Size Allocation**:
    
    ```bash
    spark.memory.offHeap.size=4g
    ```
    
- **Benefit**: Reduces GC overhead, improves performance for large iterative workloads.

### Trade-offs & Risks

- **On-Heap**: Easier to configure but prone to GC pauses when handling large datasets.
- **Off-Heap**: Faster but requires careful tuning; improper sizing can cause OOM errors.
- **Serialization Costs**: Off-heap requires serialization/deserialization, adding CPU overhead.
- **Cluster Resource Balance**: Over-allocating memory can starve other jobs.

### Best Practices

- Use **on-heap memory** for smaller workloads or when JVM GC overhead is manageable.
- Enable **off-heap memory** for large-scale analytics, ML pipelines, or when GC pauses are frequent.
- Monitor memory usage via **Spark UI → Executors tab**.
- Combine with **efficient serialization** (Kryo vs. Java serializer) for better performance.
- Always **benchmark workloads** before deciding on off-heap tuning.
###  Example Configuration

```bash
# Executor memory (on-heap)
--conf spark.executor.memory=8g

# Enable off-heap memory
--conf spark.memory.offHeap.enabled=true
--conf spark.memory.offHeap.size=4g

# Adjust fractions
--conf spark.memory.fraction=0.7
--conf spark.memory.storageFraction=0.4
```

## Shuffle operations

### What is a Shuffle in Spark?

- **Definition**: A shuffle occurs when Spark redistributes data across partitions/nodes (e.g., during `groupByKey`, `reduceByKey`, or joins).
- **Cost Drivers**:
    - **Disk I/O**: Intermediate files are written and read.
    - **Network I/O**: Data moves between executors.
    - **CPU overhead**: Sorting and serialization.
### Techniques to Optimize Shuffle Operations

#### 1. **Choose Efficient Transformations**

- Prefer **`reduceByKey`** or **`aggregateByKey`** over `groupByKey` (less data shuffled).
- Use **map-side combine** to reduce data before shuffle.

#### 2. **Optimize Joins**

- **Broadcast joins**: Use `broadcast()` for small tables to avoid shuffling large datasets.
- **Skew handling**: Use salting techniques or `skewed join hints` to balance uneven key distribution.

#### 3. **Partition Management**

- **Repartitioning**: Use `repartition()` to increase parallelism when needed.
- **Coalesce**: Reduce partitions without shuffle when shrinking dataset size.
- Align partitioning strategy with downstream operations to avoid repeated reshuffles.

#### 4. **Serialization & File Formats**

- Use **Kryo serialization** for faster shuffle data handling.
- Store intermediate data in **Parquet/ORC** for efficient reads.

#### 5. **Resource Tuning**

- Increase **shuffle buffer size** (`spark.shuffle.file.buffer`).
- Tune **parallelism** (`spark.sql.shuffle.partitions`) to match cluster size.

### Comparison of Join Strategies

| Strategy            | Shuffle Impact                     | Best Use Case                   |
| ------------------- | ---------------------------------- | ------------------------------- |
| Sort-Merge Join     | High shuffle                       | Large datasets with sorted keys |
| Shuffle Hash Join   | Moderate shuffle                   | Medium datasets                 |
| Broadcast Hash Join | No shuffle (small table broadcast) | Small dimension tables          |

### Risks & Trade-offs

- **Over-partitioning** → Too many small tasks, overhead increases.
- **Under-partitioning** → Large tasks, risk of skew and OOM errors.
- **Broadcast joins** → Fail if broadcast table exceeds memory limits.

### Best Practices

- Cache reused datasets to avoid repeated shuffles.
- Monitor shuffle metrics in **Spark UI → SQL tab**.
- Use **adaptive query execution (AQE)** in Spark 3.x to dynamically optimize shuffle partitions.
- Benchmark workloads before tuning parameters.


```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import broadcast

spark = SparkSession.builder.appName("ShuffleOptimization").getOrCreate()

# Broadcast join to avoid shuffle
large_df = spark.read.parquet("large_dataset.parquet")
small_df = spark.read.parquet("small_lookup.parquet")

optimized_join = large_df.join(broadcast(small_df), "id")
```
