---
up:
  - "[[000_+/0602 Data abstraction|0602 Data abstraction]]"
down:
prev:
topic: false
question: What is difference between re-partition and coalesce?
---
# What is difference between re-partition and coalesce?


> [!Summary] Summary
> Contents

| **Feature**           | **repartition(n)**                       | **coalesce(n)**                    |
| --------------------- | ---------------------------------------- | ---------------------------------- |
| **Primary Goal**      | Increase or Decrease partitions          | Decrease partitions only           |
| **Shuffle**           | Full Shuffle (Data moves across network) | No Shuffle (Minimal data movement) |
| **Performance**       | Slower / Resource Intensive              | Faster / Efficient                 |
| **Data Distribution** | Uniform (Equal size partitions)          | Can result in uneven partitions    |
| **Parallelism**       | Can increase parallelism                 | Cannot increase parallelism        |



While both `repartition` and `coalesce` are used to change the number of partitions in an RDD or DataFrame, they work very differently under the hood. 

## **1. repartition()**
`repartition` is used to increase or decrease the number of partitions. It performs a **Full Shuffle**.
- **How it works:** It redistributes data across the entire cluster to create a new set of partitions that are roughly equal in size.
- **Cost:** Very expensive. It moves all data across the network (shuffling), which consumes significant time and I/O.
- **When to use:** Use this when you want to **increase** the number of partitions or when your data is "skewed" (some partitions are much larger than others) and you need to balance the load across all executors.
    
## **2. coalesce()**
`coalesce` is a specialized version used strictly to **decrease** the number of partitions.
- **How it works:** It avoids a full shuffle by merging existing partitions on the same executor or nearby nodes. It simply "glues" existing partitions together.
- **Cost:** Much cheaper than repartition because it minimizes data movement across the network.
- **When to use:** Use this when you are **decreasing** partitions (e.g., after a heavy filter operation) to save on performance.

## **The "Pitfall" Example**

Suppose you have 1,000 partitions and you want to reduce them to 10 for saving a file:

- **Using `repartition(10)`:** Spark will shuffle 100% of your data across the network to create 10 new, perfectly balanced chunks.
    
- **Using `coalesce(10)`:** Spark will simply keep 10 partitions and "absorb" the other 990 into them without moving data across the network. It's nearly instantaneous by comparison.
    

> [!tip] 
> 
> If you try to use `coalesce` to _increase_ partitions (e.g., `coalesce(100)` on an RDD that has 50), Spark will silently ignore the request and keep it at 50. You **must** use `repartition` to go up.

> [!danger]
> When you do coalesce(1) it does not do all to all (shuffle) but does a many to one operation.

