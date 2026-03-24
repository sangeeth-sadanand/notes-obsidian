## Immutability in RDD

- **Definition**: RDD (Resilient Distributed Dataset) is an _immutable_ distributed collection of objects.
- **Why immutable?**
    - **Consistency**: Prevents accidental modification of data across nodes in a cluster.
    - **Parallelism**: Multiple transformations can safely run in parallel without conflicts.
    - **Determinism**: Guarantees that transformations always produce the same result given the same input.
- **How it works**: Instead of modifying existing RDDs, Spark creates new RDDs after each transformation (e.g., `map`, `filter`).

Example:

```python
rdd1 = sc.parallelize([1,2,3,4])
rdd2 = rdd1.map(lambda x: x*2)  # rdd1 remains unchanged
```

Here, `rdd1` is immutable; `rdd2` is a new dataset.

## Resilience in RDD

- **Resilient = Fault Tolerant**
- If a node fails, Spark **recomputes lost partitions** using _lineage_ (the record of transformations applied).
- **Lineage Graph**: Each RDD keeps track of how it was derived from previous RDDs.
- **Automatic Recovery**: Spark doesn’t need to replicate data across nodes; it simply replays transformations to rebuild missing data.

Example:

- Suppose a cluster node holding part of an RDD crashes.
- Spark uses the lineage graph to reapply transformations on the original dataset to regenerate the lost partition.


## Lazy Evaluation

- **Definition**: Transformations on RDDs are not executed immediately.
- **How it works**:
    - Spark records transformations in a **lineage graph**.
    - Actual computation happens **only when an action** (like `collect()`, `count()`, `saveAsTextFile()`) is called.
- **Benefits**:
    - **Optimization**: Spark can analyze the entire job before execution.
    - **Efficiency**: Avoids unnecessary work by combining transformations.
    - **Fault tolerance**: Lineage allows recomputation if data is lost.

Example:

```python
rdd = sc.textFile("data.txt")
words = rdd.flatMap(lambda line: line.split(" "))   # transformation (lazy)
pairs = words.map(lambda word: (word, 1))           # transformation (lazy)
counts = pairs.reduceByKey(lambda x, y: x + y)      # transformation (lazy)

results = counts.collect()  # action → triggers execution
```

## DAG (Directed Acyclic Graph)

- **Definition**: Spark represents computations as a **DAG of stages**.
- **Structure**:
    - **Vertices** → RDDs.
    - **Edges** → transformations between RDDs.
- **Stages**:
    - **Narrow transformations** → pipelined into the same stage (no shuffle).
    - **Wide transformations** → require shuffle, creating stage boundaries.
- **Execution**:
    - When an action is triggered, Spark’s DAG Scheduler breaks the lineage into stages and tasks.
    - Tasks are distributed across the cluster for parallel execution.

## Data partitions

- **Partitioning** = dividing an RDD into smaller chunks (partitions) that can be processed in parallel across cluster nodes.
- Each partition is a **logical division of data**, not necessarily tied to physical storage.
- Spark distributes partitions across executors for parallel computation.

###  Why Partitioning Matters

- **Parallelism** → tasks run independently on partitions.
- **Performance** → reduces shuffle and network overhead.
- **Data locality** → tasks run closer to where data resides.
- **Scalability** → large datasets handled efficiently.

### Default Partitioning

- When reading from a file (`textFile`), Spark decides partitions based on **HDFS block size** or input splits.
- When creating via `parallelize`, you can specify partitions:
    
    ```python
    rdd = sc.parallelize(range(1, 11), 4)  # 4 partitions
    ```
    
### Custom Partitioning

- **HashPartitioner** → distributes keys by hash value.
- **RangePartitioner** → distributes keys into ranges.
- Example:
    
    ```python
    from pyspark import SparkContext
    from pyspark.rdd import RDD
    
    sc = SparkContext("local", "PartitionExample")
    rdd = sc.parallelize([("a",1),("b",2),("c",3),("d",4)], 2)
    
    # Repartition using hash partitioner
    partitioned = rdd.partitionBy(3)
    print(partitioned.glom().collect())  # view partition contents
    ```
    

### Partition Operations

- `repartition(n)` → increases/decreases partitions (shuffle involved).
- `coalesce(n)` → reduces partitions without shuffle (efficient).
- `partitionBy(n)` → partitions pair RDDs by key.
- `glom()` → collects elements per partition into lists.

