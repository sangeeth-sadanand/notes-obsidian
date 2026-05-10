---
up:
  - "[[003_skills/data-engineering/06 Spark/0603 SQL/060304 Optimization/060304 Optimization|060304 Optimization]]"
down:
prev:
topic: false
question: How bucketing and partitioning is used to optimize joins?
---
# How bucketing and partitioning is used to optimize joins?


> [!Summary] Summary
> | **Feature**            | **Partitioning**                                                                | **Bucketing**                                                                                |
> | ---------------------- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
> | **Physical Layout**    | Folders/Directories (`/key=value/`)                                             | Files (Hash-based `part-0001`, `part-0002`)                                                  |
> | **Best Column Choice** | Low cardinality columns (e.g., Date, Country, Status)                           | High cardinality columns (e.g., UserID, TransactionID)                                       |
> | **Join Optimization**  | **Data Reduction:** Skips reading irrelevant data.                              | **Shuffle Elimination:** Skips network data movement.                                        |
> | **Risk**               | "Small Files Problem" if you partition on a column with too many unique values. | Requires careful planning (both tables must have the same bucket count to skip the shuffle). |
> 


- To understand how partitioning and bucketing optimize joins, you have to look at the biggest bottleneck in distributed computing: **The Shuffle**.
- When Spark performs a heavy join (like a Sort Merge Join), it must physically move data across the network (shuffle) to ensure that rows with the same join keys end up on the same executor.
- **Partitioning** and **bucketing** are data layout strategies applied _when you write the data to disk_. By organizing the data strategically on disk beforehand, you can drastically reduce or completely eliminate the need to shuffle that data later when you join it.

### **1. Partitioning (Optimizes by Data Reduction)**
Partitioning divides your data into separate **directories** (folders) based on the unique values of a specific column (e.g., `date`, `country`, `department`).
- **How it is written:** If you partition by `country`, Spark creates folders like `/country=US/`, `/country=UK/`, and places the respective data files inside them.
- **How it optimizes Joins:** It enables **Partition Pruning**. If your join query includes a filter on the partitioned column, Spark will completely ignore the directories that don't match the filter.
- **The Join Impact:** Partitioning _does not_ prevent a shuffle. However, by skipping irrelevant directories, you read vastly less data from disk into memory, which means you have significantly less data to shuffle across the network during the join.

```python
# Query: Join sales and users for the US only
# Because sales is partitioned by country, Spark ONLY reads the US folder.
spark.sql("""
    SELECT s.item, u.name 
    FROM sales s 
    JOIN users u ON s.user_id = u.user_id 
    WHERE s.country = 'US' 
""")
```

### **2. Bucketing (Optimizes by Eliminating Shuffles)**
While partitioning is directory-based, bucketing is **file-based**. It uses a hash function to distribute rows evenly across a fixed number of files (buckets) based on a specific column (the bucket key).
- **How it is written:** If you bucket a table by `user_id` into 100 buckets, Spark runs a hash function on every `user_id`. `hash(user_id) % 100` determines exactly which of the 100 files that row goes into. Spark can also _sort_ the data within those buckets.
- **How it optimizes Joins:** It enables **Colocated Joins** (Zero-Shuffle Joins). Because the data is already physically grouped by the hash of the join key on disk, it is effectively "pre-shuffled."
- **The Join Impact:** If you join Table A and Table B on `user_id`, and _both_ tables are bucketed by `user_id` into the _same number of buckets_, Spark knows that all `user_id = 5` records for Table A are in Bucket 3, and all `user_id = 5` records for Table B are _also_ in Bucket 3.
    - Spark skips the network shuffle phase entirely.
    - If the buckets are also pre-sorted, Spark skips the sort phase.
    - The Sort Merge Join becomes a lightning-fast "Merge Only" join.

