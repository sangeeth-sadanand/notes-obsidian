# Big Data

1. Introduction
	 - [[Notes/Big Data/The 5 V's of big data| The five V's of big data]]
	 - [[Notes/Big Data/The problem of storing and processing petabytes of information|The problem of storing and processing petabytes of information.]]
	 - [[Notes/Big Data/Evolution from traditional centralized systems to distributed architectures|Evolution from traditional centralized systems to distributed architectures.]]

2. Hadoop
	- [[Notes/Big Data/Understanding the Hadoop Distributed File System|Understanding the Hadoop Distributed File System (HDFS).]]
	- [[Notes/Big Data/Understanding the Hadoop Distributed File System#The Role of the Master (NameNode) and Slaves (DataNodes).|The Role of the Master (NameNode) and Slaves (DataNodes).]]
	- [[Notes/Big Data/Understanding the Hadoop Distributed File System#HDFS Operational Workflow Write-Once-Read-Many access patterns.|HDFS Operational Workflow Write-Once-Read-Many access patterns]].
	- [[Notes/Big Data/Understanding the Hadoop Distributed File System#Data Replication and Fault Tolerance strategies.| Data Replication and Fault Tolerance strategies]]
	- [[Notes/Big Data/HDFS commands|Command]]

3. [[Notes/Big Data/YARN|YARN]]
4. [[Notes/Big Data/Map reduce|Map Reduce]]
5. [[Notes/Big Data/Hive|Hive]]
6. [[Notes/Big Data/Spark/Spark|Spark]]
7. [[Notes/Big Data/File formats|File format]]
8. [[Notes/Big Data/Spark/re partition vs partition|re partition vs partition]]
9. [[Notes/Big Data/DataBricks/DataBricks|DataBricks]]























        
    
    .
    
- **Chapter 9: Storage Optimization and Data Skipping**
    
    - Efficient Formats: Why Parquet and ORC outperform CSV for analytical workloads.
    - **Z-Ordering**: Multi-dimensional data clustering to improve file skipping during queries.
    - **Liquid Clustering**: Next-generation spatial indexing to redefine clustering keys without data rewrites.
    - Coalesce vs. Repartition: When to avoid full shuffles.
