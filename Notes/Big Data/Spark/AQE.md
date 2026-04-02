# AQE

- **Catalyst** and **AQE (Adaptive Query Execution)** are the two primary engines that handle query optimization in **Apache Spark**. 
- Together, they act as the "brain" behind Spark SQL and DataFrame operations, ensuring your code runs as efficiently as possible.

## 1. The Catalyst Optimizer (The Planner)

Catalyst is Spark’s static query optimizer. When you submit a Spark SQL query or a DataFrame transformation, Catalyst evaluates the code _before_ it runs and creates the most efficient execution plan possible based on the initial data it has.
It works in four main phases:
- **Analysis:** It checks your code against the Catalog (metadata) to ensure the tables and columns you are referencing actually exist and that the data types match.
- **Logical Optimization:** It applies rule-based optimizations to simplify your query. Key techniques include:
    - **Predicate Pushdown:** Moving filters as close to the data source as possible so Spark reads less data.
    - **Column Pruning:** Dropping columns that aren't actively needed for the final output to save memory.
    - **Constant Folding:** Computing static expressions in advance (e.g., turning `col("price") * 1.0` into just `col("price")`).
- **Physical Planning:** Catalyst generates several possible physical execution strategies and uses a Cost-Based Optimizer (CBO) to pick the most efficient one based on estimated data sizes and cluster configurations.
- **Code Generation:** It generates optimized Java bytecode to execute the plan efficiently across the cluster.
**The Limitation:** Catalyst is strictly static. It relies on pre-computed estimates and statistics. If those estimates are wrong—or if the data changes drastically after a filter is applied—Catalyst’s chosen plan might end up being inefficient in practice.
## 2. Adaptive Query Execution / AQE (The Adapter)
- Introduced in Spark 3.0, AQE is a dynamic optimization framework. It addresses Catalyst's limitations by adjusting the query plan **at runtime** based on actual statistics gathered while the job is actively executing.
- When a Spark job runs, it is broken down into stages. At the boundary of these stages (specifically during a "shuffle," where data is moved across the network), AQE pauses, looks at the exact size and shape of the data that was just produced, and re-optimizes the next stage.

- AQE provides three major runtime optimizations:

	- **Dynamically Coalescing Shuffle Partitions:** If a shuffle results in hundreds of tiny, mostly empty partitions, AQE will combine (coalesce) them into fewer, optimal-sized partitions. This prevents Spark from wasting time spinning up tasks for tiny amounts of data.
	- **Dynamically Switching Join Strategies:** Spark usually defaults to a heavy "Sort-Merge Join" for large tables. However, if AQE notices that one of the tables has been filtered down to a very small size during execution, it will switch the strategy on the fly to a much faster "Broadcast Hash Join" (which avoids a network shuffle entirely).
	- **Optimizing Skewed Joins:** Data skew happens when one partition contains vastly more data than the others, causing one task to take forever while the rest sit idle. AQE detects this imbalance at runtime and splits the heavily skewed partition into smaller, evenly sized tasks so they can be processed in parallel.

## How They Work Together

Think of **Catalyst** as your GPS mapping out the best route before you start driving, based on historical traffic data.

Think of **AQE** as the live traffic updates on your dashboard. If a road suddenly closes or there's an unexpected accident on Catalyst's planned route, AQE steps in and reroutes you on the fly to ensure you still get to your destination as fast as possible.

You need Catalyst to build the foundational logic, and you need AQE to adapt to the unpredictable reality of big data processing.


## Salting

### The Problem: Data Skew

When you perform operations that require grouping data by a specific key (like a `JOIN` or `GROUP BY`), Spark sends all records with the same key to the same partition on the same worker node.

If your data is perfectly distributed, this is highly efficient. But in the real world, data is often skewed. For example, if you are joining a sales table by `city_id`, the key for "New York" might have 50 million records, while "Fargo" only has 5,000.

The worker node processing "New York" becomes a **straggler**. The rest of the cluster finishes its work in seconds and sits idle, waiting hours for the "New York" node to finish.

### The Solution: Salting

Salting is the process of artificially modifying the skewed keys to force Spark to distribute them across multiple partitions. You are essentially "tricking" Spark into splitting the massive workload.
#### How it works:
1. **Identify the Skew:** You figure out which key is causing the bottleneck (e.g., `city_id = 'NYC'`).
2. **Add the "Salt" (Large Table):** In your massive, skewed table, you append a random number (the salt) to the skewed key. If you choose a salt range of 0 to 9, `'NYC'` randomly becomes `'NYC_0'`, `'NYC_1'`, up to `'NYC_9'`.
    - _Result:_ The 50 million records are now split into 10 separate keys of ~5 million records each, allowing Spark to process them on 10 different nodes.
3. **Replicate the Keys (Small Table):** Because you changed the keys in the large table, they no longer match the small table. To fix this, you must **explode** (replicate) the corresponding row in the smaller table 10 times, appending the exact same salts (`'NYC_0'` through `'NYC_9'`).
4. **Perform the Join:** Now, Spark can join the tables in parallel.
5. **Clean Up:** After the join, you drop the salt from the key so your final output remains clean.

### Why do this if we have AQE?

As we discussed, Spark 3.0's **Adaptive Query Execution (AQE)** can automatically handle skew for **Joins**. However, salting is still your "break glass in case of emergency" tool for two reasons:
- **Aggregations:** AQE does not automatically fix skew during a `groupBy().count()` or other non-join aggregations.
- **Extreme Skew:** Sometimes the skew is so massive that AQE’s default thresholds don't trigger correctly, and a manual salt is more reliable.

|**Feature**|**AQE Skew Optimization**|**Manual Salting**|
|---|---|---|
|**Effort**|Automatic (Set-it-and-forget-it)|Manual (Requires code changes)|
|**Supported Operations**|Joins only|Joins and Aggregations|
|**Complexity**|Low|High (Exploding tables can increase memory)|
