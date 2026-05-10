---
up:
  - "[[003_skills/data-engineering/05 Hive/05 Hive|05 Hive]]"
down:
prev:
topic: false
question: What is Hive?
---
# What is Hive?


> [!Summary] Summary
>  - Hive is an SQL interface to work with Hadoop without writing complex map reduce.
>  - It also provides database table interface to play with data.
>  - Hive bridges gap of writing complex map reduce by converting SQL queries into map-reduce or Spark Jobs.
>  - Hive parses Hive Query language (HQL) to MR or spark job
>  - It also stores table schema and metadata
>  - HQL, Metastore, driver. compiler, execution engine are core components
>  - It support both structured and semi structured data, schema on read
>  - It integrates with HDFS, YARN, Spark
>  - Supports partitions and bucketing
>  - Not suitable for OLAP (real time queries)
 
**Hive** provides a way for users—especially those familiar with SQL—to work with **big data** without writing complex MapReduce programs.

> In simple terms:  
> **Hive = SQL interface for Hadoop**

## Why Hive is Used

*   Traditional SQL databases cannot efficiently handle **very large datasets**
*   Writing **MapReduce code** is complex and time‑consuming
*   Hive bridges this gap by converting **SQL‑like queries into MapReduce or Spark jobs**

## How Hive Works

1.  User writes a query in **HiveQL**
2.  Hive parses and compiles the query
3.  The query is converted into:
    *   MapReduce jobs (earlier)
    *   Tez or Spark jobs (modern Hive)
4.  Jobs execute on Hadoop
5.  Results are returned to the user

## Key Components of Hive

*   **HiveQL (HQL)** – SQL‑like query language
*   **Metastore** – Stores table schemas and metadata
*   **Driver** – Manages query lifecycle
*   **Compiler** – Converts HQL into execution plan
*   **Execution Engine** – Runs tasks on Hadoop

## Features of Hive

*   SQL‑like querying (easy to learn)
*   Supports **structured and semi‑structured data**
*   Schema‑on‑read
*   Integrates with Hadoop ecosystem (HDFS, YARN, Spark)
*   Supports partitions and buckets for optimization
*   Supports UDFs (User Defined Functions)

## Limitations of Hive

*   Not suitable for **real‑time or OLTP queries**
*   High latency (batch processing)
*   Limited row‑level updates (though ACID support exists in newer versions)

## Hive vs Traditional Database

| Aspect      | Hive                  | RDBMS     |
| ----------- | --------------------- | --------- |
| Data size   | Very large (TBs, PBs) | Moderate  |
| Query type  | Batch processing      | Real‑time |
| Latency     | High                  | Low       |
| Updates     | Limited               | Full CRUD |
| Scalability | Horizontal            | Vertical  |

## Use Cases of Hive

*   Data warehousing
*   Log analysis
*   ETL processing
*   Business intelligence reporting
*   Offline analytics
