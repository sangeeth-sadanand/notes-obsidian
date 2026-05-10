---
up:
  - "[[003_skills/data-engineering/06 Spark/0603 SQL/060304 Optimization/060304 Optimization|060304 Optimization]]"
down:
prev:
topic: false
question: What is AQE?
---
# What is AQE?


> [!Summary] Summary
> - It coalesce smaller partition into larger dynamically 
> - It can switch to broadcast join if one dataframe is small 
> - It can apply salting dynamically if one partition is skewed
> 

**Adaptive Query Execution (AQE)** is one of the most significant performance optimizations introduced in Apache Spark (available from Spark 3.0 onwards, and enabled by default in Spark 3.2+).
To put it simply: **AQE allows Spark to change its mind and re-optimize the physical execution plan _during_ runtime based on actual data statistics.**

## **The Problem AQE Solves: Static Planning**
Before AQE, Spark's Catalyst Optimizer was purely **static**. When you submit a query, Spark builds an execution plan based on _estimates_ (e.g., table sizes, expected row counts after filters).
However, estimates are often wrong. For example, Catalyst might estimate that a `WHERE age > 18` filter will return 1 million rows, but in reality, it only returns 1,000 rows. Without AQE, Spark is locked into the execution plan it chose before the query even started, potentially executing a massive, slow shuffle join when a fast broadcast join would have sufficed.

## **How AQE Works**
AQE takes advantage of **"materialization points"** (specifically, shuffle boundaries). When Spark hits a shuffle, it has to write data to disk before sending it across the network. At this exact moment, Spark pauses, looks at the _exact size and shape_ of the data it just wrote, and asks: _"Knowing what I know now, can I make the rest of this query faster?"_
## **The 3 Key Features of AQE**
AQE performs three main dynamic optimizations:
### **1. Dynamically Coalescing Shuffle Partitions**
- **The Issue:** Spark's default shuffle partition count is 200 (`spark.sql.shuffle.partitions`). If your data is tiny, spreading it across 200 tasks creates massive overhead (spinning up 200 almost-empty tasks). If the data is huge, 200 partitions might cause OutOfMemory errors.
- **The AQE Fix:** Spark lets you set a relatively high initial partition count. At the shuffle boundary, AQE looks at the actual partition sizes. If it sees dozens of tiny partitions (e.g., 1MB each), it will **merge (coalesce)** them together into optimal, larger partitions (e.g., 100MB each) for the next stage. This dramatically reduces task overhead and scheduling delays.
    
### **2. Dynamically Switching Join Strategies**
- **The Issue:** Spark decides on a Sort Merge Join (SMJ) because it estimates both tables are larger than the 10MB Broadcast threshold. But after applying a filter, Table A shrinks to just 2MB. Because the plan is static, Spark still executes the heavy, slow SMJ.
- **The AQE Fix:** At the shuffle boundary, AQE checks the actual size of the filtered tables. It realizes Table A is now 2MB. Mid-flight, it cancels the Sort Merge Join, broadcasts Table A, and executes a lightning-fast **Broadcast Hash Join (BHJ)** instead.
    
### **3. Dynamically Optimizing Skew Joins**
- **The Issue:** Data skew happens when data is unevenly distributed (e.g., 90% of your sales data belongs to `country_id = 'US'`). During a shuffle join, all the 'US' data is routed to a single task on a single executor. That one task might take 5 hours, while all other tasks finish in 2 minutes. The entire job is bottlenecked by the slow task.
- **The AQE Fix:** AQE detects skewed partitions by looking at the shuffle statistics. If it sees one partition is wildly larger than the rest, it automatically **splits the skewed partition** into smaller sub-partitions. It then duplicates the corresponding data from the other table, allowing Spark to process the skewed key in parallel across multiple tasks.

