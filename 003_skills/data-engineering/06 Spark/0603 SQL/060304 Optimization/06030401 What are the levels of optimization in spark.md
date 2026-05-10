---
up:
  - "[[003_skills/data-engineering/06 Spark/0603 SQL/060304 Optimization/060304 Optimization|060304 Optimization]]"
down:
prev:
topic: false
question: What are the levels of optimization in spark?
---
# What are the levels of optimization in spark?


> [!Summary] Summary
> 1. ***Data level optimization*** 
> 	- Storing, partitioning and structure determine is vital in performance 
> 	- parquet/ORC file format perform well as compared to csv or json format. 
> 	- partitioning the files into chunks so that it can utilize column pruning or to many file reads 
> 	- Bucketing helps in distributing id's to same executor which helps in improving performance of  joins and filtering.
> 	- snappy compression technique is balance between compression ratio and speed on operation. 
> 
> 2. ***Execution level optimization*** 
> 	- The catalyst optimizer - analyse the query performs reordering of filters, push down predicates for an optimized plan 
> 	- Tungsten execution engine: Optimizes CPU and memory usage and collapses entire query into single optimized Java function. 
> 	- Caching and persistence help in storing intermediate value which are used multiple times can be store in memory (RAM and Disk I/O) 
> 	- Broadcasting helps in avoiding shuffle in joining small tables 
> 	- AQE handles skew-ness in data using salting technique 
> 	
> 3. ***Cluster level optimization*** 
> 	- Choosing a right level of resource is very important. 
> 	- F**at executors** lead to low parallelism and GC pauses 
> 	- **Thin executors** create less resource per executors 
> 	- The sweet spot is best for most of the application 
> 
> 4. Serialization & communication 
> 	- Disk IO and network IO are the bottle neck in spark execution 
> 	- Tuning shuffling partition is important as it affect memory required are roughly 100 MB to 200 MB.
> 	- Broadcast variables helps in send over the network 
> 5. ***Architecture*** 
> 	- Choose RDD for low level control ability and DF for high level optimized execution 
> 	- Use inbuilt function, use UDF only if required 

## 1. Data-Level Optimization
Data-level optimization is the foundation of a high-performance Spark application. Before you even touch your code or tune the cluster, how your data is **stored, partitioned, and structured** determines the "ceiling" of your performance.
- **Choosing the Right File Format (Parquet/ORC):** Standard text files (like CSV or JSON) are expensive to process. Modern data engineering relies on Columnar Storage like Parquet or ORC, which allow Spark to read only the required columns (**Column Projection**) and skip entire blocks of data using metadata (**Schema Evolution**).
- **Partitioning Strategy:** Dividing data into manageable chunks based on a column (e.g., `date`). This allows **Partition Pruning** (ignoring irrelevant folders). Aim for file sizes between **128MB and 1GB** to avoid the overhead of too many small files or the under-utilization of too few large files.
- **Bucketing:** Distributes data into a fixed number of "buckets" based on a hash of a high-cardinality column (e.g., `user_id`). It is specifically designed to optimize **Joins** by enabling a **Bucket Join**, which completely avoids expensive network shuffles.
- **Predicate Pushdown (Filter Pushdown):** Spark "pushes" the `WHERE` clause logic down to the data source (like Parquet or a SQL DB). The source only sends back matching rows, saving massive amounts of memory and network I/O.
- **Data Serialization:** When moving data across the network or to disk, prefer **Kryo Serialization** over the default Java serialization. Kryo is much faster and more compact (often 10x smaller).
- **Compression:** Compressed data moves faster across the network. **Snappy** is the industry standard for Spark/Parquet, offering a great balance between compression ratios and high decompression speeds.

## 2. Execution-Level Optimization
This level focuses on how your Spark code (Transformations and Actions) is translated into actual tasks and executed.
- **The Catalyst Optimizer:** When you write DataFrame/SQL code, this engine analyzes your query, reorders filters, pushes down predicates, and generates multiple physical plans, ultimately choosing the most efficient one.
- **Tungsten Execution Engine:** Optimizes CPU and memory efficiency using **Whole-Stage Code Generation**, which collapses the entire query into a single optimized Java function to reduce CPU overhead.
- **Caching and Persisting:** If you reuse a DataFrame multiple times, use `cache()` or `persist()` to keep it in memory. This prevents Spark from re-evaluating the entire lineage graph from the source data every time an action is called.
- **Broadcast Joins:** When joining a massive table with a very small table (e.g., a lookup table), use a Broadcast Hash Join. Spark sends a copy of the small table to every executor, completely eliminating the expensive network shuffle.
- **Handling Data Skew:** Skew happens when one partition is massively larger than the others, causing a "straggler" executor. Solve this using **Salting** (adding a random key to distribute the skewed column) or by leveraging Spark 3's **Adaptive Query Execution (AQE)**, which dynamically handles skew at runtime.
    
## 3. Cluster-Level Optimization
This level involves tuning the physical resources (CPU, Memory, and JVM) allocated to your Spark application.
- **Right-Sizing Executors:**
    - _Fat Executors_ (e.g., 32 cores) lead to terrible garbage collection (GC) pauses.
    - _Thin Executors_ (e.g., 1 core) prevent taking advantage of broadcasting and shared memory.
    - _The Sweet Spot:_ Usually 5 to 6 cores per executor, with 16GB - 32GB of memory.
        
- **Dynamic Resource Allocation:** Enable `spark.dynamicAllocation.enabled` so Spark can request more executors when there is a backlog of tasks and release them when idle, rather than statically locking up cluster resources.
- **Spark Memory Architecture Tuning:** Executor memory is divided into **Execution Memory** (for shuffles, joins) and **Storage Memory** (for caching). You can tune `spark.memory.fraction` to favor one over the other based on whether your workload is compute-heavy or cache-heavy.

## 4. Serialization & Communication Optimization
Because Spark is a distributed system, Network I/O and Disk I/O are almost always the biggest bottlenecks.
- **Tuning Shuffle Partitions:** By default, `spark.sql.shuffle.partitions` is set to 200. If you process terabytes of data, 200 partitions will be too massive and crash executors. If you process megabytes, 200 partitions causes scheduling overhead. Tune this so your shuffle partitions are roughly **100MB to 200MB each**.
- **Broadcast Variables:** If tasks require a large, read-only variable (like a ML model or configuration map), use `sc.broadcast()`. This ensures the data is sent over the network exactly _once per executor_, rather than serialized and sent with every single _task_.
- **Kryo for Shuffles:** Ensuring Spark uses Kryo (`org.apache.spark.serializer.KryoSerializer`) drastically reduces the payload size during wide transformations (shuffles).

## 5. Architectural & Governance-Level Optimization
This is the highest level, focusing on code design choices and data pipeline orchestration.
- **Choosing the Right API:** Avoid RDDs unless performing highly complex, low-level unstructured data manipulation. RDDs are opaque to Spark. Always use **DataFrames or Spark SQL** so the Catalyst Optimizer and Tungsten engine can automatically optimize your code.
- **Minimizing UDFs (User Defined Functions):** Standard Python or Scala UDFs force Spark to serialize data out of its optimized format, pass it to the native process, and serialize it back, which is incredibly slow. Use Spark's built-in SQL functions whenever possible. If you must use UDFs in Python, use **Pandas UDFs (Vectorized UDFs)**.
- **Observability & The Spark UI:** Continuous monitoring is vital. Master the Spark UI to look at the DAG (Directed Acyclic Graph) to identify excessive shuffles, check the "Event Timeline" for idle executors, and monitor garbage collection times.