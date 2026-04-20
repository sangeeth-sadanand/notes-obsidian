---
up:
  - "[[003_skills/data-engineering/05 Hive|05 Hive]]"
down:
prev:
topic: false
question: What is role of meta store in hive?
---
# What is role of meta store in hive?

> [!Summary] Summary
> - Metastore stores the metadata about the data.
> - It stores database, table name; columns and its datatypes; table location; partitioning and bucketing info, serialization & deserialization info, table properties & statistics.
> - Metastore is important to maintain hive schema, Query execution, Data location, Meta data sharing, partition pruning.
> - Metastore can be embedded, local metastore or in a separate database (production)


In **Apache Hive**, the **Metastore** plays a crucial role by managing **metadata information** about the data stored in Hive.

## Role of Metastore in Hive

The **Hive Metastore** is a **central repository** that stores information *about* the data, not the actual data itself.

## What Metadata Does It Store?

The Metastore stores details such as:

*   Database names
*   Table names
*   Column names and data types
*   Table location in HDFS
*   Partitions and buckets information
*   Table type (managed or external)
*   Serialization/deserialization (SerDe) information
*   Table properties and statistics

## Why Metastore is Important

### 1. **Schema Management**

*   Maintains the schema of Hive tables
*   Enables Hive to apply **schema‑on‑read** when querying data

### 2. **Query Execution Support**

*   When an HQL query is submitted:
    *   Hive consults the Metastore to understand table structure
    *   Determines where the data is stored in HDFS
*   Without Metastore, Hive cannot process queries

### 3. **Data Location Tracking**

*   Keeps track of **where data files are physically located**
*   Helps Hive access the correct HDFS paths

### 4. **Metadata Sharing**

*   Other tools like **Spark, Presto, Impala** can use the same Metastore
*   Enables consistent metadata across the Hadoop ecosystem

### 5. **Partition Pruning & Optimization**

*   Stores partition information
*   Helps Hive skip unnecessary data scans, improving performance

## Metastore Architecture (Brief)

*   Stored in an **RDBMS** (MySQL, PostgreSQL, Oracle, etc.)
*   Accessed via **Metastore Service**
*   Supports:
    *   Embedded mode
    *   Local metastore
    *   Remote metastore (recommended for production)

## What Metastore Does NOT Store

❌ Actual data files  
✅ Only metadata about the data
