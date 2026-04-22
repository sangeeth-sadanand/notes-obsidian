---
up:
  - "[[000_+/0602 Data abstraction|0602 Data abstraction]]"
down:
prev:
topic: false
question: Difference between wide and narrow transformations?
---
# Difference between wide and narrow transformations?


> [!Summary] Summary
> Contents

| **Feature**         | **Narrow Transformation**           | **Wide Transformation**                 |
| ------------------- | ----------------------------------- | --------------------------------------- |
| **Data Movement**   | No movement (No Shuffle)            | High movement (**Shuffle**)             |
| **Performance**     | Fast; executed in-memory            | Slower; involves Disk I/O and Network   |
| **Dependency Type** | Narrow Dependency                   | Shuffle Dependency                      |
| **Optimized by**    | Pipelining (Stitching operations)   | Breaking into new Stages                |
| **Partitioning**    | Child partition depends on 1 parent | Child partition depends on many parents |

- In Apache Spark, the distinction between **Wide** and **Narrow** transformations is the foundation of how Spark optimizes jobs and manages data movement across a cluster. 
- The core difference lies in whether data needs to cross the "network" (shuffle) to complete the operation.
## 1. Narrow Transformations
In a narrow transformation, all the data required to compute the records in a single partition resides in a **single partition** of the parent RDD. There is **no data movement between executors**.
- **Dependency:** Known as "One-to-One" or "Many-to-One" (where parents belong to one child).
- **Execution:** These are highly efficient because Spark can execute them in **parallel** across the cluster without waiting for other nodes.
- **Pipelining:** Spark can group multiple narrow transformations together into a single **Stage**.
- **Examples:** `map()`, `filter()`, `flatMap()`, `sample()`, `union()`.

## 2. Wide Transformations

Wide transformations require data from **multiple partitions** of the parent RDD to compute a single partition in the child RDD. This triggers a **Shuffle**.
- **Dependency:** Known as "Many-to-Many."
- **Execution:** Spark must write data to disk and move it across the network to group related keys together on the same executor. This is the most "expensive" operation in Spark.
- **Stage Boundaries:** A wide transformation always results in the creation of a **new Stage** in the Spark UI.
- **Examples:** `groupByKey()`, `reduceByKey()`, `join()`, `repartition()`, `distinct()`.
    
## Why does this matter?

When writing Spark code, your goal is often to **minimize wide transformations**. 
For instance, using `reduceByKey` is generally preferred over `groupByKey` because `reduceByKey` performs a local "combine" before shuffling, significantly reducing the amount of data sent over the network.

