---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Lake flow tools/Lake flow tools|Lake flow tools]]"
tags:
  - DataBricks/LakeFlow/declarativePipelines
index: 3
type: topic
---
# Lake flow declarative pipeline

```ad-summary
collapse: true 
title: Summary

**Lakeflow Declarative Pipelines** (formerly Delta Live Tables / DLT) is Databricks' declarative ETL framework—also contributed to Apache Spark—that allows data engineers to define _what_ data transformations to perform while the system automatically manages _how_ to execute, scale, orchestrate, and maintain them.

**Core Concept & Problem Solved**

- **Declarative Approach:** Eliminates manual low-level plumbing for complex patterns (e.g., SCD Type 2 / CDC merges) by letting the engine manage state, lineage, dependencies, and retry logic internally.
    
- **Solves Traditional ETL Challenges:** Automates infrastructure scaling, batch/streaming unification, data quality enforcement, error handling, and cross-environment deployment.

**Execution, Computes & Flow Types**

- **Triggers & Compute:** Supports **Continuous** or **Triggered** runs across **Development** or **Production** environments on **Serverless SQL Warehouses** or **Job Compute Clusters**.

- **Dataset Abstractions:**
    - **Streaming Table:** Delta table written by a stream for incremental ingestion where full recomputation is unnecessary.
        
    - **Materialized View:** Stores computed query results, automatically optimized for incremental updates.
        
    - **View:** Temporary dataset existing only within the execution pipeline run.
        
- **Data Quality (Expectations):**
    - `EXPECT` (Warning only)
    - `EXPECT ... ON VIOLATION DROP ROW` (Drops bad rows)
    - `EXPECT ... ON VIOLATION FAIL UPDATE` (Halts the pipeline)

**Medallion Architecture Flow**

1. **Bronze (Landing):** Ingests raw files incrementally using Auto Loader (`cloud_files`).
    
2. **Silver Cleaned:** Applies data quality constraints, schema selection, and row filtering.
    
3. **Silver Merged:** Performs deduplication and change tracking via built-in CDC/SCD (`APPLY CHANGES INTO` or `CREATE FLOW ... AS AUTO CDC` for SCD Type 1/2).
    
4. **Gold:** Aggregates and joins clean data into Materialized Views for BI and analytics.
    

**Enzyme Optimization Engine**

Enzyme is the built-in Incremental View Maintenance (IVM) engine tracked via `_enzyme_log`. It prevents expensive full recomputations on Materialized Views using four refresh strategies:

- **Monotonic Append:** Processes only newly appended rows.
- **Partition Recompute:** Recalculates only modified partitions.
- **Merge Updates:** Applies targeted upserts and deletes for complex CDC updates.
- **Full Compute:** Fallback full re-run when incremental updates are non-deterministic or structurally unfeasible.

~~~sql {4-12,20-22,52,53,57, 72} showLineNumbers
-- =========================================================
-- 1. BRONZE LAYER: Raw Data Ingestion via Auto Loader
-- =========================================================
CREATE STREAMING TABLE orders_bronze
AS
SELECT *,
  _metadata.file_name AS file_name,
  current_timestamp() AS load_time
FROM cloud_files(
  'abfss://retail@ttmystorageaccount001.dfs.core.windows.net/input/orders', 
  'csv', 
  map("cloudFiles.inferColumnTypes", "True")
);


-- ====================================================================
-- 2. SILVER LAYER: Data Quality Constraints & Schema Cleaning
-- ====================================================================
CREATE STREAMING TABLE orders_silver_cleaned (
  CONSTRAINT valid_order EXPECT (order_id IS NOT NULL) ON VIOLATION DROP ROW,
  CONSTRAINT valid_customer EXPECT (customer_id IS NOT NULL) ON VIOLATION DROP ROW,
  CONSTRAINT valid_amount EXPECT (total_amount > 0) ON VIOLATION FAIL UPDATE
)
AS
SELECT 
  orderid AS order_id,
  orderdate AS order_date,
  customerid AS customer_id,
  totalamount AS total_amount,
  status,
  file_name,
  load_time
FROM STREAM(orders_bronze);


-- ====================================================================
-- 3. SILVER MERGED LAYER: CDC & SCD Type Tracking
-- ====================================================================

-- 3A. Customers SCD Type 2 (Legacy DLT Syntax)
CREATE STREAMING TABLE customers_silver;

APPLY CHANGES INTO customers_silver
FROM STREAM(customers_silver_cleaned)
KEYS (customer_id)
SEQUENCE BY load_time
STORED AS SCD TYPE 2;

-- 3B. Orders SCD Type 2 (New Lakeflow Declarative Pipeline Syntax)
CREATE STREAMING TABLE orders_silver;

CREATE FLOW orders_silver_flow AS
AUTO CDC INTO orders_silver
FROM STREAM(orders_silver_cleaned)
KEYS (order_id)
SEQUENCE BY load_time
STORED AS SCD TYPE 2;

-- 3C. Orders CDC Merge / SCD Type 1 (New Lakeflow Declarative Pipeline Syntax)
CREATE STREAMING TABLE orders_silver;

CREATE FLOW orders_silver_flow AS 
AUTO CDC INTO orders_silver
FROM STREAM(orders_silver_cleaned)
KEYS (order_id)
SEQUENCE BY load_time;


-- ====================================================================
-- 4. GOLD LAYER: Aggregations & Business Reporting (Materialized View)
-- ====================================================================
CREATE MATERIALIZED VIEW city_wise_sales_gold
AS
SELECT 
  c.city, 
  SUM(o.total_amount) AS total_sales
FROM orders_silver o
JOIN customers_silver c
  ON o.customer_id = c.customer_id
GROUP BY c.city;
~~~
Python reference

~~~python
from pyspark import pipelines as dlt
from pyspark.sql.functions import col, current_timestamp, sum

# ====================================================================
# 1. BRONZE LAYER: Raw Data Ingestion via Auto Loader
# ====================================================================
@dlt.table(
    comment="Raw orders data ingested via Auto Loader"
)
def orders_bronze():
    return (
        spark.readStream.format("cloudFiles")
        .option("cloudFiles.format", "csv")
        .option("cloudFiles.inferColumnTypes", "true")
        .load("abfss://retail@ttmystorageaccount001.dfs.core.windows.net/input/orders")
        .withColumn("file_name", col("_metadata.file_name"))
        .withColumn("load_time", current_timestamp())
    )


# ====================================================================
# 2. SILVER LAYER: Data Quality Constraints & Schema Cleaning
# ====================================================================
@dlt.table(
    comment="Cleaned orders data with data quality checks"
)
@dlt.expect_or_drop("valid_order", "order_id IS NOT NULL")
@dlt.expect_or_drop("valid_customer", "customer_id IS NOT NULL")
@dlt.expect_or_fail("valid_amount", "total_amount > 0")
def orders_silver_cleaned():
    return (
        dlt.read_stream("orders_bronze")
        .select(
            col("orderid").alias("order_id"),
            col("orderdate").alias("order_date"),
            col("customerid").alias("customer_id"),
            col("totalamount").alias("total_amount"),
            col("status"),
            col("file_name"),
            col("load_time")
        )
    )


# ====================================================================
# 3. SILVER MERGED LAYER: CDC & SCD Type Tracking
# ====================================================================

# 3A. Customers SCD Type 2
dlt.create_streaming_table("customers_silver")

dlt.apply_changes(
    target="customers_silver",
    source="customers_silver_cleaned",
    keys=["customer_id"],
    sequence_by="load_time",
    stored_as_scd_type="2"
)

# 3B. Orders CDC Merge / SCD Type 1
dlt.create_streaming_table("orders_silver")

dlt.apply_changes(
    target="orders_silver",
    source="orders_silver_cleaned",
    keys=["order_id"],
    sequence_by="load_time",
    stored_as_scd_type="1"
)


# ====================================================================
# 4. GOLD LAYER: Aggregations & Business Reporting (Materialized View)
# ====================================================================
@dlt.table(
    comment="Aggregated sales per city for reporting"
)
def city_wise_sales_gold():
    orders = dlt.read("orders_silver")
    customers = dlt.read("customers_silver")
    
    return (
        orders.join(customers, on="customer_id", how="inner")
        .groupBy("city")
        .agg(sum("total_amount").alias("total_sales"))
    )
~~~

```
---

## Evolution & Core Concept
- Delta Live Tables (DLT) has been rebranded as **Lakeflow Declarative Pipelines**, introducing a new IDE to streamline ETL development. 
- The framework uses a _declarative approach_, allowing data engineers to specify _what_ needs to be done rather than _how_. 
- Databricks open-sourced this declarative framework and contributed it to the Apache Spark project.

## What is DLT?

- DLT is a framework to build ETL pipelines in a very simplified way using a declarative approach. 
- Simplifies the ETL process.

## What is Declarative Approach?

- An approach where you tell the system what needs to be done and not how it has to be done. 
- The system then uses the best way to get the task done.

**Example:**

- Manually implementing SCD Type 2 / xMerge strategy would involve a lot of complexity and might go wrong if not done the right way.
- It would be less prone to errors if it is handled by the system internally. 
- And we just tell the system to implement SCD type 2 without saying how it has to be done.
- i.e., no code for implementing SCD is provided. 
- The system itself does everything that is needed to get the task done.


## What is ETL and what are the challenges in traditional ETL?

In ETL, you need to take care of several things manually:
- How to "Extract the Data"
- How to "Transform the Data"
- How to "Load the Data"
- Need to setup an orchestration framework manually
- Infrastructure Management
- Scaling up and Scaling Down the infrastructure
- Manage Dependencies on your own.
- Track the lineage (what is the up-stream, down-stream)
- Handling Batch and streaming in the pipeline.
- Ensure Data Quality
- Handling Data failure and retries
- Monitor and Optimize pipeline
- Deploy in multiple environments

## Execution & Deployment Modes

- **Trigger Types**: Continuous or Triggered.
- **Environments**: Development or Production.
- **Execution Computes**: Serverless SQL Warehouse or Job Compute Clusters.

## Core Abstractions & Data Quality

- **Pipeline Flow Types**:
    - **Streaming Table**: Delta table written by a stream, used for incremental reloads where full recomputation is expensive.
    - **Materialized View**: Results of a query stored in a Delta table (full load / complete recomputation), optimized via the **Enzyme** engine (`_enzyme_log`) using monotonic appends, partition recomputation, merge updates, or full compute.
    - **View**: Exists only for the duration/span of the pipeline run.
        
- **Data Quality Expectations (Constraints)**:
    - **`EXPECT` (Allow)**: Logs violations as warnings and records the metric without dropping rows.
    - **`EXPECT ... ON VIOLATION DROP ROW`**: Silently drops violating records from the target dataset.
    - **`EXPECT ... ON VIOLATION FAIL UPDATE`**: Halts the entire pipeline update if a constraint fails.

## Medallion Architecture & Implementation Syntax

**1. Landing to Bronze (Ingestion with Auto Loader)** Loads raw data incrementally into Delta format.

- **SQL Example**:

```SQL
CREATE STREAMING TABLE orders_bronze
AS
SELECT *,
  _metadata.file_name AS file_name,
  current_timestamp() AS load_time
FROM cloud_files('abfss://retail@ttmystorageaccount001.dfs.core.windows.net/input/orders', 'csv', map("cloudFiles.inferColumnTypes", "True"));
```

- **Python Example**:

```python
from pyspark import pipelines as dp
from pyspark.sql.functions import current_timestamp, col

@dp.table(
    comment="Raw orders data ingested via Auto Loader"
)
def orders_bronze():
    return (
        spark.readStream.format("cloudFiles")
        .option("cloudFiles.format", "csv")
        .option("cloudFiles.inferColumnTypes", "true")
        .load("abfss://retail@ttmystorageaccount001.dfs.core.windows.net/input/orders")
        .withColumn("file_name", col("_metadata.file_name"))
        .withColumn("load_time", current_timestamp())
    )
```

**2. Bronze to Silver Cleaned (Data Quality Constraints)** Applies validation rules and schema filtering to clean bronze data.

- **SQL Example**:
```sql {2-4}
CREATE STREAMING TABLE orders_silver_cleaned (
  CONSTRAINT valid_order EXPECT (order_id IS NOT NULL) ON VIOLATION DROP ROW,
  CONSTRAINT valid_customer EXPECT (customer_id IS NOT NULL) ON VIOLATION DROP ROW,
  CONSTRAINT valid_amount EXPECT (total_amount > 0) ON VIOLATION FAIL UPDATE
)
AS
SELECT 
  orderid AS order_id,
  orderdate AS order_date,
  customerid AS customer_id,
  totalamount AS total_amount,
  status,
  file_name,
  load_time
FROM STREAM(orders_bronze);
```

```python {7-9}
from pyspark import pipelines as dlt
from pyspark.sql.functions import col

@dlt.table(
    comment="Cleaned orders data with data quality checks"
)
@dlt.expect_or_drop("valid_order", "order_id IS NOT NULL")
@dlt.expect_or_drop("valid_customer", "customer_id IS NOT NULL")
@dlt.expect_or_fail("valid_amount", "total_amount > 0")
def orders_silver_cleaned():
    return (
        dlt.read_stream("orders_bronze")
        .select(
            col("orderid").alias("order_id"),
            col("orderdate").alias("order_date"),
            col("customerid").alias("customer_id"),
            col("totalamount").alias("total_amount"),
            col("status"),
            col("file_name"),
            col("load_time")
        )
    )
```

**3. Silver Cleaned to Silver Merged (Deduplication & CDC / SCD Type 2)** Eliminates duplicates or tracks historical changes.
- **SQL Example (Auto CDC / SCD Type 2)**:
```sql {7}
-- Customers SCD Type 2
CREATE STREAMING TABLE customers_silver;

APPLY CHANGES INTO customers_silver
FROM STREAM(customers_silver_cleaned)
KEYS (customer_id)
SEQUENCE BY load_time
STORED AS SCD TYPE 2;

-- New syntax

CREATE FLOW orders_silver_flow AS
AUTO CDC INTO orders_silver
FROM STREAM(orders_silver_cleaned)
KEYS (order_id)
SEQUENCE BY load_time
STORED AS SCD TYPE 2;


-- Orders CDC Merge
CREATE STREAMING TABLE orders_silver;

CREATE FLOW orders_silver_flow AS AUTO CDC
INTO orders_silver
FROM STREAM(orders_silver_cleaned)
KEYS (order_id)
SEQUENCE BY load_time;
```

- **Python Example (Auto CDC / SCD Type 2)**:
```python
from pyspark import pipelines as dlt

# Customers SCD Type 2
dlt.create_streaming_table("customers_silver")

dlt.apply_changes(
    target="customers_silver",
    source="customers_silver_cleaned",
    keys=["customer_id"],
    sequence_by="load_time",
    stored_as_scd_type="2"
)

# Orders Merge (SCD Type 1)
dlt.create_streaming_table("orders_silver")

dlt.apply_changes(
    target="orders_silver",
    source="orders_silver_cleaned",
    keys=["order_id"],
    sequence_by="load_time",
    stored_as_scd_type="1"
)
```

**4. Silver to Gold Layer (Materialized Views & Aggregation)** Aggregates transformed data for downstream consumption and BI reporting.
- **SQL Example**:
```sql
CREATE MATERIALIZED VIEW city_wise_sales_gold
AS
SELECT 
  c.city, 
  SUM(o.total_amount) AS total_sales
FROM orders_silver o
JOIN customers_silver c
  ON o.customer_id = c.customer_id
GROUP BY c.city;
```

- **Python Example**:
```sql
from pyspark import pipelines as dlt
from pyspark.sql.functions import sum

@dlt.table(
    comment="Aggregated sales per city for reporting"
)
def city_wise_sales_gold():
    orders = dlt.read("orders_silver")
    customers = dlt.read("customers_silver")
    
    return (
        orders.join(customers, on="customer_id", how="inner")
        .groupBy("city")
        .agg(sum("total_amount").alias("total_sales"))
    )
```

## Modern Implementation: Unity Catalog vs. Legacy Hive Metastore

| Feature Area            | Legacy Approach (Hive Metastore)                        | Modern Approach (Unity Catalog)                                                              |
| ----------------------- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| **Storage Governance**  | Local to workspace; relies on mount points (`/mnt/...`) | Centralized repository; governed via **External Locations** and **Volumes** (`/Volumes/...`) |
| **File Identification** | `input_file_name()`                                     | `_metadata.file_name`                                                                        |
| **CDC API**             | `APPLY CHANGES INTO ...`                                | `CREATE FLOW ... AS AUTO CDC`                                                                |
| **Compute Options**     | Classic clusters (serverless unsupported)               | **Serverless Compute** (instant provisioning, Enzyme enabled by default)                     |
| **Namespace Syntax**    | Required `live.<table_name>` keyword                    | 3-level namespace (`catalog.schema.table`) without `live` keyword requirement                |

## New Lakeflow Pipeline IDE Structure

- **Pipeline Assets Root Folder**: Contains subfolders for pipeline logic.
    - `explorations/`: Houses ad-hoc, experimental analysis files that are omitted during automated pipeline execution.
    - `transformations/`: Contains production pipeline transformation code executed during pipeline runs.
    - `README.md`: Pipeline documentation.
        
- **Modular Notebook Execution**: Multiple notebooks can be linked in the pipeline path; cross-notebook dependencies are automatically managed by the framework graph.


## Enzyme Optimization in Materialized View 

- **Enzyme** is the incremental view maintenance (IVM) optimization engine in Databricks designed specifically for **Materialized Views** in Lakeflow Declarative Pipelines / DLT. 
- Instead of executing expensive full table recomputes on every pipeline refresh, Enzyme determines the most cost-effective way to incrementally process only updated or newly appended data.

**How Enzyme Works**
- **`_enzyme_log` Directory**: When a Materialized View is created, an internal metadata directory named `_enzyme_log` is generated to track changes and maintain execution state between updates.

- **Partial vs. Full Recomputation**: Enzyme analyzes upstream source changes and performs targeted partial recomputations to generate exact query results while significantly reducing CPU cycles and runtime.
    
- **Serverless Default**: Enzyme is enabled by default when running DLT on **Serverless Compute**.
    

### The 4 Core Enzyme Refresh Strategies
- **1. Monotonic Append**:
    - Used when new records are strictly added (appended) to upstream datasets without modifying existing historical records.
    - Enzyme processes only the newly appended batch and adds the computed results directly to the Materialized View.
        
- **2. Partition Recompute**:
    - Used when data in specific partitions of the source table is updated or modified.
    - Enzyme isolates and recomputes only the affected partitions rather than recalculating the entire historical table.
        
- **3. Merge Updates**:
    - Used for complex updates, upserts, or CDC scenarios where changes affect existing rows across the table.
    - Enzyme generates an internal delta plan to update, delete, and merge changed records efficiently into the backing dataset.
        
- **4. Full Compute**:
    - The fallback strategy used when non-deterministic operations occur, table definitions change, or when incremental computation is estimated to be more costly than a fresh calculation.
    - The entire query logic is evaluated from scratch across all data.