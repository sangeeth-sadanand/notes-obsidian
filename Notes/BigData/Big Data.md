# Big Data

1. Introduction
	 - [[Notes/BigData/The 5 V's of big data| The five V's of big data]]
	 - [[Notes/BigData/The problem of storing and processing petabytes of information|The problem of storing and processing petabytes of information.]]
	 - [[Notes/BigData/Evolution from traditional centralized systems to distributed architectures|Evolution from traditional centralized systems to distributed architectures.]]

2. Hadoop
	- [[Notes/BigData/Understanding the Hadoop Distributed File System|Understanding the Hadoop Distributed File System (HDFS).]]
	- [[Notes/BigData/Understanding the Hadoop Distributed File System#The Role of the Master (NameNode) and Slaves (DataNodes).|The Role of the Master (NameNode) and Slaves (DataNodes).]]
	- [[Notes/BigData/Understanding the Hadoop Distributed File System#HDFS Operational Workflow Write-Once-Read-Many access patterns.|HDFS Operational Workflow Write-Once-Read-Many access patterns]].
	- [[Notes/BigData/Understanding the Hadoop Distributed File System#Data Replication and Fault Tolerance strategies.| Data Replication and Fault Tolerance strategies]]
	- [[Notes/BigData/HDFS commands|Command]]

3. [[Notes/BigData/YARN|YARN]]
4. [[Notes/BigData/Map reduce|Map Reduce]]
5. [[Notes/BigData/Spark/Spark|Spark]]


















- **Chapter 9: Security, Failure Recovery, and Ethics**
    
    - Security Models: Kerberos authentication and shared secret RPC.
    - Failure Recovery: RDD Lineage vs. MapReduce Checkpointing.
    - Ethical Considerations: Privacy, accountability, and algorithmic fairness.
    - Impact of opaque automated decisions on vulnerable populations.
- **Chapter 10: Conclusion and Future Trends**
    
    - Hadoop as a storage engine (HDFS) versus Spark as a processing engine.
    - Cloud-Native HDFS and AI integration.
    - Final comparison: Cost-effectiveness of disk vs. the speed of RAM.
### **Book Table of Contents: Mastering Big Data, Spark Coding, and Advanced Optimization**







- **Chapter 8: Dealing with Data Skew: The "Salt" Technique**
    
    - Identifying Skew: Using the Spark Web UI to find "straggler" partitions.
    - **Key Salting**: Introducing randomness to "hot keys" to distribute data evenly across the cluster.
        
        ```
        from pyspark.sql.functions import concat, lit, rand, floor
        # Adding a salt column to redistribute skewed keys
        df_salted = df.withColumn("salt", floor(rand() * 10))
        df_composite = df_salted.withColumn("salted_key", concat(col("key"), lit("_"), col("salt")))
        ```
        
    
    .
    
- **Chapter 9: Storage Optimization and Data Skipping**
    
    - Efficient Formats: Why Parquet and ORC outperform CSV for analytical workloads.
    - **Z-Ordering**: Multi-dimensional data clustering to improve file skipping during queries.
    - **Liquid Clustering**: Next-generation spatial indexing to redefine clustering keys without data rewrites.
    - Coalesce vs. Repartition: When to avoid full shuffles.
- **Chapter 10: Spark in the Modern Ecosystem: Streaming and Cloud**
    
    - Structured Streaming: Real-time analytics using micro-batches and Kafka integration.
    - Spark on Kubernetes: Modern containerized deployment versus YARN.
    - Cost Optimization: Single-node cluster testing, spot instance utilization, and idle cluster termination.