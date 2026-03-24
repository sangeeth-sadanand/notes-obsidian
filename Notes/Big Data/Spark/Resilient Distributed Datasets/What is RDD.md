# What is RDD ?

- **Definition**: An RDD is an **immutable distributed collection of objects** that can be processed in parallel across a cluster.
- **Origin**: Introduced as the **primary API in Apache Spark**, designed to overcome limitations of Hadoop MapReduce.
- **Properties**:
    - **Immutable**: Once created, it cannot be changed.
    - **Partitioned**: Data is split across nodes for parallelism.
    - **Fault-tolerant**: If a node fails, Spark can recompute lost partitions using lineage information.
- **Operations**:
    - **Transformations** (lazy): `map()`, `filter()`, `flatMap()`, etc.
    - **Actions** (eager): `collect()`, `count()`, `saveAsTextFile()`, etc.
- **Use Cases**:
    - Handling **unstructured data** (like logs, text).
    - When **fine-grained control** over transformations is needed.
    - For **low-level operations** where DataFrames/Datasets may not suffice

## Comparison: RDD vs DataFrame vs Dataset

| Feature         | RDD                       | DataFrame                   | Dataset                      |
| --------------- | ------------------------- | --------------------------- | ---------------------------- |
| **Type Safety** | No (works with objects)   | No (like SQL tables)        | Yes (strongly typed objects) |
| **Ease of Use** | Low (requires coding)     | High (SQL-like API)         | Medium (typed API)           |
| **Performance** | Lower (no optimization)   | Higher (Catalyst optimizer) | Higher (Catalyst optimizer)  |
| **Best For**    | Low-level transformations | Structured data analysis    | Type-safe structured data    |
