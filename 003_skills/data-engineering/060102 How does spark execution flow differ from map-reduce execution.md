---
up:
  - "[[003_skills/data-engineering/0601 Architecture|0601 Architecture]]"
down:
prev:
topic: false
question: How does spark execution flow differ from map-reduce execution
---
# How does spark execution flow differ from map-reduce execution


> [!Summary] Summary
> - In Map reduce to achieve a task entire task is divided into multiple jobs intermediate results of the Job are saved and read from disk. In a Job MR follows strict pattern of map, shuffle and reduce phase.
>
> - In spark, the entire task is a Job. Each job is divided into stages and intermediate results are stored in-memory instead of disk. It create a DAG when a transform is applied and it only executes when an action is performed DAG can be optimized with predicate push down, join strategies, etc. based on the type and size of data.


- The fundamental difference in execution flow comes down to how each engine views a "job." 
- Hadoop MapReduce views a job as a rigid, two-step process (Map, then Reduce), while Spark views a job as a fluid, multi-step graph called a **DAG (Directed Acyclic Graph)**.

## 1. Hadoop MapReduce: The Rigid "Stop-and-Go" Flow
In MapReduce, the execution flow is linear and strictly segmented. Even if your task requires five steps, you must chain multiple MapReduce jobs together.
1. **Read:** Data is read from the HDFS (Disk).
2. **Map:** Data is filtered or sorted.
3. **Shuffle & Sort:** Data is moved across the network to the correct reducers.
4. **Reduce:** Data is aggregated.
5. **Write:** The final result **must** be written back to the Disk.

**The Bottleneck:** If you need to do another calculation on that result, the next job must start over at step 1. This "Read-Write-Read" cycle creates massive latency.

## 2. Apache Spark: The Fluid DAG Flow
Spark doesn't force a "Map" and "Reduce" structure. Instead, it builds a **DAG**—a logical map of all the transformations you want to apply to your data.
1. **Lazy Evaluation:** When you tell Spark to filter or map data, it doesn't do it immediately. It just adds that instruction to the DAG.
2. **The Action Trigger:** Execution only starts when you call an "Action" (like `count()` or `save()`).
3. **Stages and Pipelining:** Spark’s scheduler looks at the DAG and collapses multiple steps into a single "Stage" to avoid moving data.
4. **In-Memory Persistence:** Unlike MapReduce, Spark keeps the data in RAM between these stages. It only writes to the disk at the very end (or if it runs out of memory).

## Key Differences in Flow Logic

| **Feature**           | **MapReduce Flow**                | **Spark Flow**                                  |
| --------------------- | --------------------------------- | ----------------------------------------------- |
| **Structure**         | Linear (Map $\rightarrow$ Reduce) | Graph-based (DAG)                               |
| **Data Movement**     | Frequent "Checkpoints" to Disk    | Data stays in RAM across stages                 |
| **Optimization**      | Minimal (Sequential)              | High (Optimizer reorders tasks)                 |
| **Intermediate Data** | Stored on HDFS                    | Stored in Memory (RDDs/DataFrames)              |
| **Fault Tolerance**   | Re-runs the failed task from Disk | Uses the DAG to "re-compute" only the lost data |
