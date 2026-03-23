# Map reduce

- Hadoop MapReduce is a Java-based distributed computing programming model designed to process large data sets across clusters of commodity hardware by dividing tasks into smaller, parallelizable units.

### Core Architecture and Components

- **Centralized Management (YARN):** MapReduce version 2 (YARN) uses a **ResourceManager** to manage global cluster resources and **NodeManagers** on individual machines to handle specific tasks.
- **Job Coordination:** An **ApplicationMaster** is spawned for every job to manage its specific execution flow and negotiate resources.
- **Storage Integration:** It operates primarily on the **Hadoop Distributed File System (HDFS)**, which splits data into blocks (typically 128 MB or 256 MB) to facilitate parallel processing.

### The MapReduce Execution Flow

- **Input Splitting:** Data is divided into splits based on block size; a mapper task is created for each split.
- **Map Phase:** The user-defined map function processes input records to produce intermediate **key-value pairs**.
- **Circular Buffer and Spilling:** Mapper output is written to a circular memory buffer. When the buffer reaches a threshold, data is sorted and "spilled" to the local disk.
    - `mapreduce.task.io.sort.mb`: Controls the buffer size.
    - `mapreduce.map.sort.spill.percent`: Defines the threshold for disk spilling.
- **Shuffle Phase:** This critical step sorts and redistributes the intermediate data so that all values associated with a specific key are moved to the same reducer node.
- **Merge Phase:** Spilled files are merged into a single output file per mapper.
    - `mapreduce.task.io.sort.factor`: Controls the number of streams merged at once.
- **Reduce Phase:** Reducers fetch their specific partitions from various mappers via HTTP, perform a final merge, and execute the user’s reduction logic (e.g., aggregation) before writing the final output back to HDFS.
    - `mapreduce.job.reduce.slowstart.completedmaps`: Determines when reducers start fetching data from mappers.

### Performance Characteristics and Limitations

- **Batch Processing Efficiency:** MapReduce is ideal for non-time-sensitive batch jobs where data size exceeds available memory, as it relies on disk read/write operations for every stage.
- **Iterative Bottlenecks:** It is significantly slower than Apache Spark for iterative algorithms (like machine learning) because it must write intermediate results to disk between every job, leading to high I/O overhead.
- **Resource Utilization:** YARN historically uses an "exclusive mode" where resources are tied to a container until completion. This can cause underutilization, particularly during the shuffle phase when CPU usage is low.
- **Fault Tolerance:** MapReduce is highly resilient. If a node fails, it can resume tasks from where they left off because states are persisted to disk.

### Key Configuration Knobs for Tuning

|Parameter|Function|
|:--|:--|
|`dfs.block.size`|Controls the size of HDFS data blocks.|
|`mapreduce.job.reducers`|Sets the number of reducer tasks for a job.|
|`mapred.compress.map.output`|Enables compression of intermediate data to reduce network/disk load.|
|`mapreduce.reduce.shuffle.input.buffer.percent`|Fraction of memory for storing data fetched from mappers.|
|`mapreduce.reduce.merge.inmem.threshold`|The number of map outputs required to trigger an in-memory merge.|