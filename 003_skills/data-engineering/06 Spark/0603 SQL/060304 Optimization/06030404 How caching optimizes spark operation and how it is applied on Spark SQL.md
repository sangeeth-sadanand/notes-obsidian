---
up:
  - "[[003_skills/data-engineering/06 Spark/0603 SQL/060304 Optimization/060304 Optimization|060304 Optimization]]"
down:
prev:
topic: false
question: How caching optimizes spark operation and how it is applied on Spark SQL?
---
# How caching optimizes spark operation and how it is applied on Spark SQL?


> [!Summary] Summary
> - Caching breaks the lineage and skips recomputation, reduces network IO 
> - We can apply cache using -cachel) and • persiste) to persist in memory or disk 
> - We can use 'CACHE' key word to cache data into memory in SQL


Caching is one of the most powerful optimization techniques in Apache Spark. To understand why it is so effective, you first have to look at how Spark executes computations: **lazy evaluation**.
When you apply transformations (like `filter`, `select`, or `join`) to a DataFrame or RDD, Spark doesn't execute them immediately. Instead, it builds a "lineage graph" (a logical plan of instructions). Spark only executes this graph when you call an **action** (like `show()`, `count()`, or `write()`).
If you call multiple actions on the same DataFrame, Spark will re-evaluate the entire lineage graph from the original data source every single time. This is where caching steps in.

## **How Caching Optimizes Spark Operations**
1. **Breaks the Lineage and Skips Recomputation:** When you cache a DataFrame, Spark saves the fully computed data into memory (and/or disk) across the executor nodes after the first action is called. Subsequent actions on that DataFrame will read directly from this cache instead of re-reading the source data and re-executing all the previous transformations.
2. **Reduces Disk and Network I/O:** Reading from Spark's in-memory cache is orders of magnitude faster than fetching data over the network from remote storage (like Amazon S3, HDFS, or Azure Data Lake) or reading from a physical disk.
3. **Accelerates Iterative Algorithms:** Machine learning algorithms and interactive data exploration often require passing over the same dataset dozens or hundreds of times. Caching ensures these iterations happen at RAM speed.
4. **Preserves Fault Tolerance:** Caching does not compromise Spark's resilience. If a node crashes and loses a chunk of cached data, Spark simply uses the original lineage graph to recompute just the missing partition on another node.

## **How Caching is Applied in Spark SQL**
In Spark SQL, caching can be applied using either the DataFrame/Dataset API or pure SQL commands. Spark's Catalyst Optimizer automatically recognizes cached tables and swaps out the underlying physical plan with an `InMemoryTableScan`, effectively short-circuiting the computation.

### **1. Using the DataFrame API**
You can cache a DataFrame programmatically using two main methods:
- `cache()`: This is a convenience method. For DataFrames/Datasets, it caches the data using the default storage level: **MEMORY_AND_DISK** (meaning it will try to fit it in RAM, but spill to disk if it runs out of space).
- `persist()`: This gives you granular control over _where_ and _how_ the data is cached by allowing you to pass a specific `StorageLevel` (e.g., `MEMORY_ONLY`, `DISK_ONLY`, `MEMORY_AND_DISK_SER`).
    
```python
# Read massive dataset and apply heavy transformations
df = spark.read.parquet("s3://bucket/massive_data/")
transformed_df = df.filter("age > 18").groupBy("city").count()

# Mark the DataFrame for caching (Lazy - nothing happens yet)
transformed_df.cache()

# First action: Triggers computation and materializes the cache
transformed_df.count() 

# Second action: Lightning fast, reads directly from the cache
transformed_df.show() 
```

### **2. Using Pure SQL Commands**
If you are writing pure SQL queries, you can cache tables or views directly using SQL syntax.
```sql
-- Caches the table eagerly (computes and caches immediately)
CACHE TABLE user_metrics AS 
SELECT user_id, SUM(purchases) FROM transactions GROUP BY user_id;

-- Caches the table lazily (waits for the first action to compute)
CACHE LAZY TABLE user_metrics;

-- To remove the table from the cache to free up memory
UNCACHE TABLE user_metrics;
```

### **3. Impact on the Execution Plan (`explain()`)**
When you use Spark SQL to query a cached table, you can verify that the cache is being used by looking at the execution plan using `.explain()`.
Instead of seeing steps like `FileScan parquet` or `Exchange hashpartitioning` (shuffles), the physical plan will show:
```
== Physical Plan ==
*(1) InMemoryTableScan [user_id#12, sum(purchases)#34]
      +- InMemoryRelation [user_id#12, sum(purchases)#34], StorageLevel(disk, memory, deserialized, 1 replicas)
            +- *... (original lineage here)*
```

This `InMemoryTableScan` indicates that Spark SQL successfully bypassed the expensive computations and went straight to the executor memory.

> [!warning]
> - While caching is highly beneficial, it is not a silver bullet. 
> - Memory is a finite resource. 
> - If you cache too many DataFrames or cache massive datasets that don't fit into your execution memory, Spark will spend excessive time managing the cache, evicting old blocks, or spilling to disk, which can actually degrade your performance. 
> - You should only cache data that will be **reused multiple times** downstream in your application.

