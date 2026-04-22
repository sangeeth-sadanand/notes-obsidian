---
up:
  - "[[003_skills/data-engineering/0601 Architecture|0601 Architecture]]"
down:
prev:
topic: false
question: What are main modules of spark eco-system?
---
# What are main modules of spark eco-system


> [!Summary] Summary
> 
> The main modules in spark echo system are:
> 1) **Spark Core** - It has RDD at its core which is optimize to read and write and perform basic logic.
> 2) **Spark SQL** - It provide interface to perform SQL like high level abstraction.
> 3) **Streaming**:- this module enables scalable, high- throughput, fault-tolerant stream process of live data
> 4) **MLlib**:- It give basic classification , regression, clustering for ML
> 5) **GraphX** combines RDD with graph database 

- Apache Spark isn't just a single engine; it’s a massive ecosystem designed to handle everything from simple data processing to complex machine learning and real-time streaming.
## 1. Spark Core
This is the foundation of the entire project. It handles the "heavy lifting" like memory management, fault recovery, scheduling tasks on a cluster, and interacting with storage systems.
- **Key Concept:** It introduced the **RDD (Resilient Distributed Dataset)**, which is the basic logical data unit in Spark.

## 2. Spark SQL
This is the most popular module for data engineers. It allows you to use SQL queries to interact with your data.
- **DataFrames & Datasets:** It provides high-level abstractions that make it easier to work with structured and semi-structured data.
- **Interoperability:** You can read data from JSON, Parquet, Hive, and standard databases (via JDBC).

## 3. Spark Streaming
This module enables scalable, high-throughput, fault-tolerant stream processing of live data streams.
- **Sources:** It can ingest data from Kafka, Flume, or Kinesis.
- **Structured Streaming:** A newer approach built on the Spark SQL engine that treats live streams as a table that is continuously being appended.
    
## 4. MLlib (Machine Learning Library)
Spark’s scalable machine learning library. Because Spark runs in memory, these algorithms run significantly faster than traditional disk-based MapReduce implementations.
- **Tools:** Includes common algorithms for classification, regression, clustering, and collaborative filtering.
- **Pipelines:** Tools for constructing, evaluating, and tuning machine learning workflows.

## 5. GraphX
This is the API for graphs and graph-parallel computation. It simplifies tasks like social network analysis or fraud detection.
- **Functionality:** It combines the advantages of both RDDs and graph databases, allowing you to view the same data as both a collection and a graph.

### Summary Table

| **Module**    | **Primary Purpose** | **Best For...**                         |
| ------------- | ------------------- | --------------------------------------- |
| **Spark SQL** | Structured Data     | Analysts and SQL users                  |
| **Streaming** | Real-time Data      | Monitoring, fraud detection, IoT        |
| **MLlib**     | Machine Learning    | Predictive modeling and AI              |
| **GraphX**    | Graph Processing    | Social networks, recommendation engines |
