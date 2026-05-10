---
up:
  - "[[003_skills/data-engineering/06 Spark/0602 Data abstraction/0602 Data abstraction|0602 Data abstraction]]"
down:
prev:
topic: false
question: What is role of data partitions?
---
# What is role of data partitions?


> [!Summary] Summary
> - Data partition help in parallelism. 
> - If a data is partitioned into 10 partitions then 10 tasks are created and executed in parallel 
> - The size of partition is 128 MB by default 
> - if partition size increases then memory error occurs 
> - If size is too low then large number of partition are created.
> 
> | **Concept**             | **Description**                                                                                         |
> | ----------------------- | ------------------------------------------------------------------------------------------------------- |
> | **Default Parallelism** | Usually based on the number of cores in your cluster.                                                   |
> | **repartition()**       | Increases or decreases partitions by performing a full shuffle. Useful for balancing data.              |
> | **coalesce()**          | Efficiently decreases partitions by merging them (minimizes shuffling).                                 |
> | **Data Skew**           | When one partition is much larger than others, causing one "straggler" task to slow down the whole job. |
> 

- **Data Partitions** are the fundamental unit of parallelism. 
- They are the logical chunks that your large dataset is divided into so that multiple cores can process the data simultaneously.

## **1. Enabling Parallelism**
The most critical role of partitions is to determine how many tasks can run in parallel.
- **1 Partition:** Even if you have 100 CPU cores, only **one** core will work while the other 99 sit idle.
- **100 Partitions:** Spark can launch 100 tasks simultaneously, utilizing the full power of your cluster.

## **2. Managing Memory and Stability**
Partitions help keep the data "digestible" for the executors.
- **Size Matters:** Ideally, each partition should be  **128MB**.
- **The Risk:** If partitions are too large (e.g., several GBs), you will run into `OutOfMemoryError` because a single task cannot fit its assigned data into the executor's RAM.
- **The Overhead:** If partitions are too small (e.g., 1KB), the "scheduling overhead" (the time it takes Spark to manage and launch a task) will be longer than the time it takes to actually process the data.

## **3. Minimizing Data Shuffling**
Partitions play a major role in how data moves during joins or aggregations. 
By using **Partitioners** (like Hash or Range partitioning), you can ensure that data with the same key ends up in the same partition.
- **Co-location:** If two datasets are partitioned the same way on the same key, Spark can perform a "Shuffle-less Join," which is drastically faster because no data needs to travel across the network.

