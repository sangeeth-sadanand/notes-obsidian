# Spark

1. Apache Spark Architecture
	- [[Notes/Big Data/Spark/The Spark Revolution|The Spark Revolution]]
	- [[Notes/Big Data/Spark/The Spark Revolution#Unified Analytics Engine|Unified Analytics Engine]]
	- [[Notes/Big Data/Spark/The Spark Revolution#Core components|Core components]]
	- [[Notes/Big Data/Spark/The Spark Revolution#Spark Context as the entry point for functionality|Spark Context as the entry point for functionality]]
	- [[Notes/Big Data/Spark/The Spark Revolution#Execution flow|Execution flow]]
2.   [[Notes/Big Data/Spark/The Spark Ecosystem Modules|The Spark Ecosystem Modules]]
3.  [[Notes/Big Data/Spark/Resilient Distributed Datasets/Resilient Distributed Datasets|Resilient Distributed Datasets]]
4. [[Notes/Big Data/Spark/SQL/SQL|SQL]]
5. 
















- **Chapter 7: Performance Tuning and Optimization**
    
    - In-Memory Caching: Reducing latency by keeping data in RAM.
    - Memory Management: Tuning On-Heap and Off-Heap memory.
        - Example Configuration: `spark.executor.memory 4g`.
        - Example Configuration: `spark.memory.offHeap.enabled true`.
    - Optimizing Shuffle Operations and Reducing Data Movement.
        - Example Configuration: `spark.sql.shuffle.partitions 200`.
    - Enhancing Data Locality: Moving computation close to data.
    - Leveraging the Spark Web UI for bottleneck identification.
- **Chapter 5: Optimizing Joins and Shuffling**
    
    - Shuffle Sort Merge Join: The default mechanism for large-scale distributed joins.
    - **Broadcast Hash Join**: The fastest join for small-to-large table matching.
    - Bucketing: Pre-partitioning data to eliminate costly shuffles during joins.

- **Chapter 8: Dealing with Data Skew: The "Salt" Technique**
    
    - Identifying Skew: Using the Spark Web UI to find "straggler" partitions.
    - **Key Salting**: Introducing randomness to "hot keys" to distribute data evenly across the cluster.
        
- **Chapter 6: Advanced Query Optimization: Catalyst and AQE**
    
    - The Catalyst Optimizer: From Analysis to Physical Planning.
    - Rule-Based Optimization: Constant Folding, Filter Pushdown, and Projection Pruning.
    - **Adaptive Query Execution (AQE)**: Re-optimizing query plans at runtime based on intermediate statistics.
- 
- **Chapter 7: High-Performance Memory and CPU Management**
    
    - **Project Tungsten**: Improving efficiency through bytecode generation and cache-aware algorithms.
    - Unified Memory: Balancing Execution Memory (shuffles) versus Storage Memory (caching).
    - On-Heap vs. Off-Heap: Reducing Garbage Collection (GC) overhead.