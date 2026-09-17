---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Delta lake/Delta lake|Delta lake]]"
tags:
  - DataBricks/deltaLake
index: 4
type: topic
---
# Performance Optimization and Data layout

```ad-summary
collapse: true 
title: Summary
 
## Deletion Vectors (Efficient Deletes & Updates)

Historically, changing a single row meant rewriting an entire massive Parquet file. Delta Lake solves this using **Deletion Vectors**—compressed bitmaps that mark specific rows as deleted so they are filtered out during reads, leaving the original file intact.

- **For Deletes:** Delta creates a Deletion Vector, logs a transaction to logically remove the original Parquet file, and immediately re-adds it alongside a metadata pointer to the Deletion Vector.
    
- **For Updates:** Delta creates a Deletion Vector for the old rows and writes the updated rows to a _brand-new_ Parquet file. The transaction log removes the original file, re-adds it with the Deletion Vector pointer, and adds the newly created Parquet file.

> [!Note]
> If a delete or update affects multiple Parquet files, Delta creates a separate Deletion Vector for each affected file.
   
## Managing File Sizes

Frequent data ingestion creates thousands of small files, which degrades I/O performance. Delta Lake resolves this by compacting small files into larger, optimized files (ideally between 16MB and 1GB) using a bin-packing algorithm.

- **Manual Optimize:** You can explicitly run the `OPTIMIZE <table_name>;` command to combine files.
- **Auto Optimize:** Enabled by default, this automatically triggers file compaction in the background based on minimum and maximum file count thresholds.

## Data Skipping and Layout Strategies

To execute queries quickly, Delta minimizes resource usage by skipping irrelevant files. It offers three distinct strategies for organizing data physically:

| Strategy    | Best For                                   | How It Works                                                                                                | Limitations / Drawbacks                                                                                      |
| ---------------- | ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **Partitioning** | Low-cardinality columns (e.g., `country`, `year`) | Physically separates data into Hive-style sub-directories based on column values.                                 | Can cause severe data skew, creates small file issues, and requires a full table rewrite if query patterns change. |
| **Z-Ordering**   | High-cardinality columns (e.g., `order_id`)       | Co-locates similar data within files to create non-overlapping min/max ranges, improving read-time data skipping. | Does not adapt automatically; requires running `OPTIMIZE ... ZORDER BY`.                                           |
|**Liquid Clustering**|Changing or unpredictable query patterns|Incrementally optimizes data layout on _new_ data without rewriting history. Automatically handles file sizing.|This is the modern standard designed to completely replace Partitioning and Z-Ordering.|

```
---
## Deletion Vectors (Efficient Updates & Deletes)
Historically, Parquet files could contain millions of rows, and deleting a single row required rewriting the entire file.
* **How it works:** Instead of rewriting the massive Parquet files, Delta creates a **Deletion Vector** marking the specific rows that should be excluded during read times.
* **Benefit:** Only metadata is updated—not the full file. This significantly improves the performance of `DELETE`, `UPDATE`, and `MERGE` operations. While deleting records, it adds a record in the Delta table indicating the deletion vector rather than deleting the Parquet file entirely.

When a delete operation is applied on delta lake it does the following operation
	1. **Generates a Deletion Vector:** A compressed bitmap is created to track the exact row indices of the deleted records.
    2. **Creates a New Delta Log JSON:** A new transaction commit file is written to the `_delta_log` directory.
	    a. **Logs a `RemoveFile` Entry:** The JSON file logically removes the original, unmodified Parquet file.
	    b. **Logs an `AddFile` Entry:** The JSON file immediately re-adds the exact same Parquet file, but now includes a metadata pointer to the Deletion Vector for read-time filtering.
![[005_assets/Big_Data/Delta_delete_operation.drawio.svg|700]]

When a update operation is applied then
1. **Generates a Deletion Vector:** A compressed bitmap is created to mark the original, pre-update rows as deleted.
2. **Writes a New Parquet File:** The updated versions of the rows are physically written to a brand-new Parquet data file.
3. **Creates a New Delta Log JSON:** A new transaction commit file is written to the `_delta_log` directory containing three key actions.
	a. **Logs a `RemoveFile` Entry:** The JSON logically removes the original Parquet file that contains the pre-update rows.
	b. **Logs an `AddFile` Entry (Original File + DV):** The JSON re-adds the original Parquet file, but now includes a metadata pointer to the Deletion Vector so the old rows are filtered out on read.
	c. **Logs an `AddFile` Entry (New Data):** The JSON adds the brand-new Parquet file that contains the newly updated rows.
![[005_assets/Big_Data/Deltalake Update flow.drawio.svg]]

> [!TIP]
> A Deletion Vector is created **for each affected Parquet file**
> **Example:** If your `DELETE` statement removes rows that happen to be spread across 3 different Parquet files, Delta Lake will generate 3 separate Deletion Vectors (one for each file) and update the transaction log with 3 `RemoveFile`/`AddFile` pairs.

## Handling Small Files

* **Problem:** Processing 10,000 files of 1MB each introduces massive I/O overhead compared to 100 files of 100MB each. Frequent updates (e.g., hourly) create small files.
### 1. Manual Optimize
Combines small files into larger files.

```sql
-- Disable auto optimize during table creation to test
TBLPROPERTIES (
  'delta.autoOptimize.optimizeWrite' = 'false',
  'delta.autoOptimize.autoCompact' = 'false'
);

-- Check history
DESCRIBE HISTORY orders_managed;

-- Execute manual optimize
OPTIMIZE orders_managed;

```
* **Ideal Target File Size:** 16MB to 1GB.
* Target file size setting (Classic compute only):
```sql
SET spark.databricks.delta.optimize.maxFileSize = 1GB;

```

### Bin Packing Algorithm:
Given files: `100MB, 300MB, 300MB, 100MB, 600MB, 300MB`
1. Sort descending: `600MB, 300MB, 300MB, 300MB, 100MB, 100MB`
2. Distribute into bins (Target ~1GB):
* **Bin 1:** `600MB + 300MB + 100MB = 1000MB (1GB)`
* **Bin 2:** `300MB + 300MB + 100MB = 700MB`
### 2. Auto Optimize
Enabled by default; triggers compacting operations automatically based on file thresholds:
* `spark.databricks.delta.autoCompact.minNumFiles` (Minimum file count to trigger)
* `spark.databricks.delta.autoCompact.maxNumFiles`

## Delta Lake Data Skipping & Partitioning
* **Data Skipping:** Pruning/skipping irrelevant files to minimize resource usage during query execution.
* Achieved via **Data Layout** (Partitioning / Z-Ordering).
### Partitioning (Hive-Style Partitioning)

Organizes data into sub-directories/folders based on column values.
```sql
CREATE TABLE IF NOT EXISTS orders_managed_partitioned (
  order_id BIGINT,
  sku STRING,
  product_name STRING,
  product_category STRING,
  qty INT,
  unit_price DECIMAL (10,2),
  country STRING
)
USING DELTA
PARTITIONED BY (country);

INSERT INTO orders_managed_partitioned
SELECT * FROM orders_managed;

```

### Query Optimization Example:
```sql
SELECT product_category, SUM(qty) AS total_qty
FROM orders_managed_partitioned
WHERE country = 'IN'
GROUP BY product_category;

```

*(Scans only the `country=IN` folder and skips all other country folders.)*

### Challenges with Partitioning:
* Works **only** for **low cardinality** columns.
* Queries filtering on non-partitioned columns derive **no** pruning benefits.
* Changing usage patterns requires rewriting the whole table.
* Can cause **small file problems** or severe **data skewness** if data is unequally distributed across partition values.

## Z-Ordering

* Used for **high cardinality** columns where partitioning fails.
* Co-locates similar data within the same files to minimize data range overlap.
### Example:

Without Z-ordering, trip distance ranges across 5 files overlap heavily (`0–500`, `2–400`, `10–450`, etc.).
With Z-ordering, ranges are sorted into non-overlapping bins:

* File 1: `0 - 30`
* File 2: `31 - 80`
* File 3: `81 - 200`
* File 4: `201 - 400`
* File 5: `401 - 500`

### Syntax:

```sql
OPTIMIZE <table_name> ZORDER BY (<column_name>);

```

## Liquid Clustering

Replaces partitioning and Z-ordering to resolve their limitations (e.g., choosing columns, table rewrites on access pattern changes).
### Key Mechanism:
* Incrementally optimizes data layout **only on newly arriving data** without rewriting historical data.
* Automatically resizes files to an ideal size (combining small files, splitting overly large files).
### Enable Liquid Clustering:

```sql
ALTER TABLE <table_name> CLUSTER BY (<column_name>);

```
### Remove Liquid Clustering:

```sql
ALTER TABLE <table_name> CLUSTER BY NONE;

```
