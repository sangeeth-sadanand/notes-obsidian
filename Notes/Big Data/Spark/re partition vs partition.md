# Re-partition vs partition

In Spark, both `repartition` and `coalesce` are used to change the number of partitions in an RDD or DataFrame, but they are used in very different scenarios to optimize performance.

The main difference lies in **shuffling**: `repartition` always shuffles data, while `coalesce` avoids it whenever possible.

##  Re-partition

`repartition` is used to either **increase** or **decrease** the number of partitions.
- **Full Shuffle:** it performs a "Full Shuffle" of the data across the network to create new, roughly equal-sized partitions.
- **Data Distribution:** It balances the data evenly across the new partitions, which is great for fixing **data skew** (where some partitions are much larger than others).
- **Performance Cost:** Because it involves a shuffle, it is an expensive operation in terms of network I/O and CPU.
## 2. coalesce

`coalesce` is optimized specifically for **decreasing** the number of partitions.
- **No Full Shuffle:** Instead of moving all data across the network, it simply collapses existing partitions into fewer ones on the same executors where possible.
- **Efficiency:** It is much faster than `repartition` because it minimizes data movement.
- **The Catch:** It cannot be used to _increase_ the number of partitions. If you try to increase partitions using `coalesce`, Spark will ignore the request unless you set the `shuffle` parameter to `true` (which effectively turns it into a `repartition`).
- **Risk of Skew:** Since it doesn't shuffle, you might end up with unevenly sized partitions if the original distribution was messy.
## Key Comparison Table

| **Feature**      | **repartition**                       | **coalesce**                                                 |
| ---------------- | ------------------------------------- | ------------------------------------------------------------ |
| **Primary Goal** | Change number of partitions (Up/Down) | Decrease number of partitions                                |
| **Shuffling**    | Always performs a Full Shuffle        | Avoids shuffle (unless forced)                               |
| **Performance**  | Slower (High Network/Disk I/O)        | Faster (Low Network/Disk I/O)                                |
| **Data Balance** | Excellent (Uniform distribution)      | May result in uneven partitions                              |
| **Use Case**     | Increasing parallelism or fixing skew | Saving data to fewer files (e.g., before writing to S3/HDFS) |
## When to use which?

- **Use `repartition`** when you have a small number of large partitions and you want to scale up the parallelism to use all available CPU cores. It’s also the right choice if your data is "skewed" and you need to redistribute it evenly to avoid some tasks taking way longer than others.
- **Use `coalesce`** right before saving a file to a data lake. If your processing created 1,000 small partitions but you only want to save 5 files, `coalesce(5)` is the most efficient way to merge them.
