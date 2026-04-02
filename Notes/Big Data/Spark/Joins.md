# Join

- When PySpark executes a join, the **Catalyst Optimizer** evaluates your code (the logical plan) and translates it into a **physical execution plan**. 
- Because data is distributed across a cluster, the way Spark physically moves and matches the data heavily dictates the performance of your job.
## 1. Broadcast Hash Join (BHJ)

This is the fastest join strategy in Spark because it entirely avoids the most expensive operation in distributed computing: **the network shuffle**.

- **How it works:** Spark sends (broadcasts) a complete copy of the smaller table to every executor node. Each executor builds an in-memory hash table from this smaller dataset. Then, the executors read their partitions of the larger table and probe the hash table to find matches.
- **When it's used:** By default, Spark automatically uses this when one of the tables is smaller than `10 MB` (controlled by `spark.sql.autoBroadcastJoinThreshold`). It only works for equi-joins (e.g., `A.id = B.id`).
- **Pros:** Extremely fast; no network shuffle required for the larger table.
- **Cons:** Can cause OutOfMemory (OOM) errors on the driver or executors if the broadcasted table is larger than the available memory.
- **How to force it:** `df1.join(broadcast(df2), "key")` or using SQL hints `/*+ BROADCAST(df2) */`.
```python
# Wrap the smaller DataFrame in the broadcast() function
bhj_df = df_large.join(broadcast(df_small), "id", "inner")

# Verify the physical plan
bhj_df.explain() 
# Look for "BroadcastHashJoin" in the output
```

## 2. Sort Merge Join (SMJ)

This is Spark's robust, default workhorse for joining two large tables.

- **How it works:** It operates in three phases:
    1. **Shuffle:** Data from both tables is reshuffled across the cluster so that rows with the same join keys land on the same executor.
    2. **Sort:** The data within each partition is sorted by the join key.
    3. **Merge:** Spark iterates through the sorted data from both tables simultaneously, merging matches.
        
- **When it's used:** This is the default strategy for large tables in modern Spark versions. It requires equi-joins and join keys that can be sorted.
- **Pros:** Highly scalable. Because the data is sorted, Spark doesn't need to hold the entire partition in memory, making it highly resistant to OOM errors even with massive datasets.
- **Cons:** Shuffling and sorting are both heavy on CPU, Disk, and Network I/O.
- **How to force it:** SQL hint `/*+ MERGE(table) */` or `/*+ SHUFFLE_REPLICATE_NL(table) */`.

```python
# Disable auto-broadcast to force a shuffle/sort for our example
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)

# Join two "large" DataFrames
smj_df = df_large.alias("a").join(df_large.alias("b"), "id", "inner")
smj_df.explain()
# Look for "SortMergeJoin" in the output
```

## 3. Shuffle Hash Join (SHJ)

Think of this as a middle ground between Broadcast Hash Join and Sort Merge Join.
- **How it works:** 
	1. **Shuffle:** Like SMJ, data is shuffled across executors based on the join key.
    2. **Hash:** Instead of sorting, Spark takes the smaller partition on each node and builds an in-memory hash table.
    3. **Probe:** The larger partition is streamed through to probe the hash table for matches.
    
- **When it's used:** Used when tables are large (so they must be shuffled), but one table is significantly smaller than the other (at least 3x smaller), and the average partition size is small enough to fit in memory.
- **Pros:** Faster than Sort Merge Join because it skips the expensive "Sort" phase.
- **Cons:** Highly susceptible to OOM errors if your data is skewed (i.e., if one join key is heavily repeated, resulting in a massive partition that won't fit into the memory of a single executor).
- **How to force it:** SQL hint `/*+ SHUFFLE_HASH(table) */`.
```python
shj_df = df_large.join(
	df_small.hint("shuffle_hash"), 
	"id", 
	"inner" 
)
```

## 4. Broadcast Nested Loop Join (BNLJ)

This is a fallback strategy used when Spark has to evaluate complex, non-equi join conditions (e.g., `<, >, <=, >=, BETWEEN`).

- **How it works:** The smaller table is broadcast to all executors. Then, Spark performs a nested loop—for every single row in the larger table, it iterates through every single row in the broadcasted small table to check the join condition.
- **When it's used:** Non-equi joins where one table is small enough to broadcast.
- **Pros:** Highly flexible; can evaluate complex, non-standard join conditions.
- **Cons:** Extremely slow. The time complexity is $O(N \times M)$.
- **How to force it:** Automatically triggered on non-equi joins if one table is broadcasted.
```python
# Note the non-equi join condition: df_large.id > df_small.id
bnlj_df = df_large.join(
    broadcast(df_small), 
    df_large.id > df_small.id, 
    "inner"
)

bnlj_df.explain()
# Look for "BroadcastNestedLoopJoin" in the output
```
## 5. Cartesian Product Join (Cross Join)

The most expensive join operation, matching every row to every other row.
- **How it works:** It shuffles the data to ensure every row in Table A is compared against every row in Table B.
- **When it's used:** When you explicitly call `.crossJoin()` or when you do a standard join without providing any join conditions.
- **Pros:** Useful only when you explicitly need all possible combinations of two datasets (e.g., generating a grid of dates and stores).
- **Cons:** Causes massive data explosion. Two tables of 1 million rows will output 1 trillion rows.
## Summary Comparison

|**Strategy**|**Needs Shuffle?**|**Needs Sort?**|**Speed**|**Best For**|**Risk**|
|---|---|---|---|---|---|
|**Broadcast Hash**|No|No|Fastest|1 Small Table, 1 Large Table|OOM on Driver/Executors|
|**Sort Merge**|Yes|Yes|Slow/Steady|2 Large Tables|High Disk/Network I/O|
|**Shuffle Hash**|Yes|No|Medium|1 Medium Table, 1 Large Table|OOM on Executors (Skew)|
|**Broadcast Nested Loop**|No|No|Very Slow|Complex non-equi conditions|High CPU time|
|**Cartesian Product**|Yes|No|Slowest|Explicit Cross Joins|Massive data explosion|

##  Partitioning and Bucketing

- In the world of big data—specifically within frameworks like Apache Hive, Spark, and Hadoop—**Partitioning** and **Bucketing** are the two primary techniques used to optimize query performance and organize massive datasets.
### 1. Partitioning

Partitioning organizes data into a **hierarchical directory structure** based on the values of a specific column (the partition key).
- **How it works:** It creates a physical sub-folder for every unique value in the partition column. For example, if you partition by `Country`, all data for "India" goes into one folder, and "USA" goes into another.
- **Best for:** Columns with **low cardinality** (a limited number of unique values), such as Date, Country, or Department.
- **Main Benefit:** **Partition Pruning.** When you run a query with a `WHERE` clause on the partition column, the engine skips all other folders and only reads the relevant data, drastically reducing I/O.
### 2. Bucketing (Clustering)

Bucketing decomposes data into a **fixed number of files** (buckets) based on a hash function of a column.
- **How it works:** You specify the number of buckets (e.g., 16 or 32). The engine applies a hash function to the bucketing column: $hash(value) \pmod{N}$, where $N$ is the number of buckets. This ensures that the same value always ends up in the same bucket.
- **Best for:** Columns with **high cardinality** (thousands or millions of unique values) like `User_ID` or `Transaction_ID`, where partitioning would create too many tiny folders.
- **Main Benefit:** **Optimized Joins.** If two tables are bucketed on the same column with the same number of buckets, the engine can perform a "Bucket Map Join," which is much faster than a standard shuffle join because it knows exactly which files to compare.
### Comparison Table

|**Feature**|**Partitioning**|**Bucketing**|
|---|---|---|
|**Organization**|Creates sub-directories.|Creates fixed-size files within a directory.|
|**Logic**|Based on literal column values.|Based on a hash function of the value.|
|**Cardinality**|Use for low cardinality (Date, State).|Use for high cardinality (User ID, Email).|
|**File Management**|Can lead to "Small File Problem" if over-partitioned.|Keeps the number of files constant and predictable.|
|**Primary Use Case**|Speeding up filters (`WHERE` clauses).|Speeding up Joins and Sampling.|
### Can you use both?

Yes! This is a common practice in production environments. You can **Partition by Date** (to narrow down the time range) and then **Bucket by User_ID** within each partition (to allow for fast joins and prevent skewed data).

### How Bucketing is Created

Bucketing is created during the data insertion process. It uses a **Hash Function** and the **Modulo Operator** to determine which file a specific row belongs to.
1. **Select Column & Number of Buckets:** You choose a column (e.g., `user_id`) and a fixed number of buckets (e.g., $N = 4$).
2. **Calculate the Hash:** For every row, the engine calculates $Hash(user\_id)$.
3. **Apply Modulo:** It then calculates $Hash(user\_id) \pmod N$.
4. **Write to File:** The result (0, 1, 2, or 3) determines which of the 4 files the row is written to.
Because this logic is deterministic, the same `user_id` will **always** end up in the same bucket file every time you run the process.
```sql
CREATE TABLE users_bucketed (
    user_id INT,
    name STRING
)
CLUSTERED BY (user_id) 
INTO 4 BUCKETS;
```

### How Bucketing Optimizes Joins

In a standard Join (like a **Shuffle Hash Join**), the system has to move data across the network so that rows with matching keys from two different tables end up on the same worker node. This "Shuffle" is the #1 bottleneck in Spark/Hive.

#### The "Bucket Map Join" Advantage

If you have two tables (e.g., `Orders` and `Users`) that are both bucketed on `user_id` with the **same number of buckets**, the engine can perform a **Bucket Map Join**:
- **No Shuffling:** The engine already knows that `user_id = 101` in the `Orders` table is in `Bucket_1`, and `user_id = 101` in the `Users` table is also in `Bucket_1`.
- **Local Processing:** It only needs to join `Orders_Bucket_1` with `Users_Bucket_1`.
- **Reduced Complexity:** Instead of comparing every row of Table A against Table B, it only compares corresponding files.
#### Key Requirements for Optimization

To trigger this optimization, three conditions must usually be met:
1. **Same Join Key:** Both tables must be bucketed on the column you are joining.
2. **Same Number of Buckets:** If Table A has 4 buckets and Table B has 8, a shuffle might still occur (though some engines can handle multiples, it's less efficient).
3. **Sorted Buckets:** Ideally, the data within the buckets should also be sorted (`SORTED BY`), which allows for a **Sort-Merge Join** without the "Sort" step.

Summary Table: Shuffle Join vs. Bucket Join

| **Feature**       | **Standard Shuffle Join**           | **Bucket Map Join**             |
| ----------------- | ----------------------------------- | ------------------------------- |
| **Data Movement** | Massive (Network Shuffle)           | Zero (Local Read)               |
| **CPU Overhead**  | High (Hashing & Sorting on the fly) | Low (Data is pre-organized)     |
| **Performance**   | Slower, scales poorly with size     | Extremely fast, scales linearly |
|                   |                                     |                                 |
