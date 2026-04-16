---
up:
  - "[[003_skills/data-engineering/02 Hadoop|02 Hadoop]]"
down:
prev:
topic: false
question: Describe tools used in Hadoop eco-system
---
# Describe tools used in Hadoop eco-system


> [!Summary] Summary
> 
> - Data processing engine - spark, Flink
> - Ingestion framework - Sqoop, Flume, Kafka
> - SQL on Hadoop - Hive, impala, Trino, presto
> - No SQL dB  Hbase
> - Management and Orchestration:- Zookeeper, Oozie, Airflow


The Hadoop ecosystem is a collection of various open-source tools and frameworks that extend the core capabilities of Hadoop (HDFS, YARN, and MapReduce).
These tools are categorized by their role in the data lifecycle: ingestion, storage, processing, and analysis.

## 1. Data Processing Engines
While MapReduce is the original engine, it is often replaced today by faster alternatives.
- **Apache Spark:** The current gold standard. It processes data "in-memory," making it up to 100x faster than MapReduce for certain tasks. It includes libraries for SQL, streaming, and machine learning (MLlib).
- **Apache Flink:** A "true" streaming engine. While Spark processes in tiny batches, Flink processes data as a continuous stream, making it ideal for real-time applications like fraud detection.
    
## 2. Data Ingestion (Getting Data In)
- **Apache Sqoop:** Designed for moving bulk data between Hadoop and **structured** databases (like MySQL or Oracle).
- **Apache Flume:** Used for collecting and moving large amounts of **streaming log data** into HDFS.
- **Apache Kafka:** A high-throughput distributed messaging system that acts as a "buffer" for real-time data streams before they are processed.

## 3. SQL-on-Hadoop (Analysis)
These tools allow users to query data using familiar SQL syntax instead of writing complex Java code.
- **Apache Hive:** A data warehouse layer. It turns SQL-like queries (HiveQL) into MapReduce or Spark jobs. Best for heavy, batch-oriented ETL.
- **Apache Impala:** A high-performance SQL engine. Unlike Hive, it bypasses MapReduce and uses parallel processing (MPP) for much faster, interactive queries.
- **Presto (Trino):** A distributed SQL engine designed for "federated" queries—it can query data across Hadoop, MySQL, and even NoSQL databases simultaneously.

## 4. NoSQL Databases (Storage)
- **Apache HBase:** A column-oriented NoSQL database that runs on top of HDFS. It provides real-time, random read/write access to billions of rows.
- **Apache Cassandra:** Often used alongside Hadoop for its high availability and scalability across multiple data centers.

## 5. Management & Orchestration

- **Apache ZooKeeper:** The "coordinator." It manages configuration and synchronization across the many different nodes in a cluster to ensure they don't step on each other's toes.
- **Apache Oozie:** A workflow scheduler. It allows you to chain various jobs (like a Sqoop import followed by a Hive query) into a single automated pipeline.
- **Apache Airflow:** A more modern, Python-based alternative to Oozie used to author, schedule, and monitor complex data workflows.
