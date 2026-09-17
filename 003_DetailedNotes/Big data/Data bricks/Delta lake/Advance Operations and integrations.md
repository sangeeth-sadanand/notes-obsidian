---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Delta lake/Delta lake|Delta lake]]"
tags:
  - DataBricks/deltaLake
index: 6
type: topic
---
# Advance Operations and integrations

```ad-summary
collapse: true 
title: Summary
## 1. Upserts (MERGE Semantics)

Delta Lake handles **Upserts** (updating existing records and inserting new ones) in a single, atomic operation using the standard SQL `MERGE` command.
- **How it works:** You match a source dataset against your target Delta table using a unique key. Based on the match, you can define `WHEN MATCHED` (to update or delete) and `WHEN NOT MATCHED` (to insert) conditions. This eliminates the need for separate complex pipelines for updates and inserts.
## 2. Change Data Feed (CDF)

While the standard transaction log tracks changes at the _file_ level, **Change Data Feed (CDF)** tracks changes at the granular **row level**.

- **Row-Level Tracking:** When enabled, CDF records every row-level `insert`, `delete`, and `update`. For updates, it captures both the **pre-image** (state before) and **post-image** (state after).
- **Querying Changes:** You can query these changes between specific table versions using the `table_changes()` function.
- **Batch vs. Streaming:**
    - **Batch:** Requires custom logic to handle incremental updates and avoid duplicate processing.
    - **Streaming (Recommended):** By reading the CDF as a stream and using a **checkpoint location**, Spark automatically remembers what it has already processed. Using `trigger(availableNow=True)` is highly recommended here—it processes all new changes like a batch job and then automatically stops, making it perfect for scheduled, cost-effective incremental pipelines.

## 3. Cloning Tables

Delta Lake allows you to create point-in-time copies of your tables at specific versions without risking data corruption in production.

| Feature                       | Shallow Clone                               | Deep Clone                              |
| ----------------------------- | ------------------------------------------- | --------------------------------------- |
| **What is copied?**           | Metadata only                               | Both Metadata and underlying Data files |
| **Storage & Speed**           | Extremely fast, zero initial storage cost   | Slower, duplicates storage footprint    |
| **Data Independence**         | Points to the original source files         | Fully independent copy                  |
| **Impact of Dropping Source** | Breaks the clone (if it is a managed table) | Clone remains completely intact         |
| **Best Used For**             | Testing workflows on production data        | Archiving (e.g., monthly snapshots)     |

> [!Note]
> Neither type of clone natively pushes updates back to the source. 
> If you insert data into a Shallow Clone, it creates its own new Parquet files, branching away from the original table.


## 4. Delta Lake UniForm (Universal Format)

Historically, Delta Lake and Apache Iceberg both used Parquet files for data but had completely incompatible metadata layers. If an Iceberg client needed to read Delta data, you had to physically copy and rewrite the entire table.

**UniForm** solves this by maintaining a single copy of the underlying Parquet data while automatically generating both Delta _and_ Iceberg metadata.
- **The Benefit:** You avoid massive data duplication and storage costs, allowing Iceberg clients to natively and seamlessly read your Delta Lake tables.


```
---

## Upsert - MERGE Semantics

- **Upsert = Update + Insert** (implemented via `MERGE` in Delta Lake).
- **Core concept:** For each incoming record, match against target by key; update matched rows and insert unmatched rows in a single atomic operation.
- **SQL examples:**
    ```sql
    MERGE INTO target AS t
    USING source AS s
    ON t.key = s.key
    WHEN MATCHED THEN
      UPDATE SET t.col = s.col
    WHEN NOT MATCHED THEN
      INSERT (col1, col2) VALUES (s.col1, s.col2);
    ```
    
    Advanced example with deletes:
    ```sql
    MERGE INTO delta.`/path/to/table` AS dest
    USING source_table AS src
    ON dest.key = src.key
    WHEN MATCHED AND src.is_deleted THEN DELETE
    WHEN MATCHED THEN UPDATE SET *
    WHEN NOT MATCHED THEN INSERT *;
    ```

## Change Data Feed (CDF)

* Delta log tracks or deals with file or dataset level changes.
* We haven't understood about Row level changes (from this particular file, ignore the following Row).
* **CDF is used to track the row level changes between different versions** (All the operations that happened from one version to another).
* Operations that can happen at Row Level are:
	1. Inserts
	2. Deletes
	3. Updates
* **Pre-image:** The status of the Row before update.
* **Post-image:** The status of the Row after update.
### Why is This Needed?
For Granular level audit. If we want to provide only the latest data for the downstream teams (Incremental processing: Process only the latest Data).
### Demo
#### Step 1: Creating a Table Called Demo

```sql
CREATE TABLE demo (
  name STRING,
  id INT,
  amount DOUBLE
);

```

*(By default it is a Delta Table)*

#### Step 2: Insert Records into the Table

```sql
INSERT INTO demo VALUES (1, 'alice', 100.0), (2, 'bob', 200.0), (3, 'carol', 300.0);

```

#### Step 3: Enable Change Data Feed

To track Row Level Changes, enable 'Change Data Feed' feature. This can be done at two levels:
1. While table creation
2. By Altering the existing table

```sql
ALTER TABLE demo SET TBLPROPERTIES (delta.enablechangedatafeed = true);
```

#### Perform a Set of Operations after Enabling the Change Data Feed:

* **Update:**
```sql
UPDATE demo SET amount = amount * 1.1 WHERE id = 2;
```

* **Insert additional:**
```sql
INSERT INTO demo VALUES (4, 'dan', 400.0);
```

* **Delete:**
```sql
DELETE FROM demo WHERE id = 1;
```

* **Describe History:**
```sql
DESCRIBE HISTORY demo;
```

After performing the above operations, there will be 5 entries:

1. Update
2. Write (for insert)
3. Optimize (automatically optimized)
4. Delete
5. Optimize (automatically optimized)

```sql
SELECT * FROM table_changes('demo', 2);
```

* `table_changes` is a function that gives change data feed.
* This displays all the row level changes that have happened after version 2.
* There will be pre-image and post-image entries for the Update operation.
* This doesn't display the granular level changes entries for version 1 where the change data feed was not enabled.
* We can give starting and end version (e.g., `table_changes("demo", 2, 4)`).

### Example Scenario

Downstream team requests for all the deleted records.
```sql
CREATE TABLE IF NOT EXISTS demo_deleted_data_batch (
  id INT,
  name STRING,
  amount DOUBLE,
  _change_type STRING,
  _commit_version LONG,
  _commit_timestamp TIMESTAMP
) USING delta;
```

#### Batch Approach

```sql
INSERT INTO demo_deleted_data_batch
SELECT id, name, amount, _change_type, _commit_version, _commit_timestamp
FROM table_changes('demo', 2)
WHERE _change_type = 'delete';
```

#### Streaming Approach

Instead of taking as a batch, the data needs to be read as a stream of continuous data.

```python
from pyspark.sql.functions import col

(spark.readStream.format("delta")
  .option("readChangeFeed", "true") # Tells use of change data feed
  .option("startingVersion", 2)
  .table("demo")
  .filter(col("_change_type") == "delete")
  .select("id", "name")
  .writeStream
  .outputMode("append")
  .option("checkpointLocation", "/volumes/...") # Stores incremental metadata
  .trigger(availableNow=True) # Run, process available data, and stop
  .table("demo_deleted_data_streaming")
)
```

> Whenever streaming, we need to provide the checkpoint location so that it can remember whatever was processed earlier and process incrementally.

* `trigger(availableNow=True)` indicates $\rightarrow$ Run, process the data and stop. No need to run it continuously.

#### Advantage of Streaming Approach:

* In the case of batch, if you want the data to be updated incrementally, then we need to develop our own logic. By default, in batch there will be duplicates and it needs processing from the beginning again.
* In the case of streaming, the operations performed earlier are stored in the checkpoint location.
* Since all activities are saved, the next processing starts from the point where it left off earlier and doesn't have to re-do everything from the beginning again.

> [!NOTE]
> * In case of serverless, the microbatches style of processing is not supported.
> * For example, using `trigger(processingTime="2 seconds")` is not supported in serverless.
> * We need to start a compute cluster to execute the above.
> * Processing Time = "2 seconds" implies treating it like a microbatch of 2 seconds (i.e., run every 2 seconds).
> * Whenever there are changes in the source table, it will be reflected to the downstream team as well in case of streaming approach.

### Additional CDF Configurations

#### Enabling Change Data Feed during Table Creation:

```sql
CREATE TABLE demo (
  id INT,
  name STRING,
  amount DOUBLE
)
USING DELTA
TBLPROPERTIES (delta.enableChangeDataFeed = true);

```

#### Enabling Change Data Feed at Spark Session Level:

```sql
SET spark.databricks.delta.properties.defaults.enableChangeDataFeed = true;

```

> [!NOTE]
> This works for classic compute cluster and not for serverless compute.

### Properties & Details

* **`availableNow = True`**:
	* Process all the data that is currently available and stop.
	* It runs as a finite job (like a batch job) and stops automatically.
	* In case of batch processing, we have to implement incremental logic, but in the case of `availableNow`, it is automatically done.
	* If it is re-run later with the same checkpoint, it will pick up only the new changes.
* **When is it best to use `availableNow`?**
	* Best for serverless as continuous triggers are not supported in serverless.
	* If there is no requirement to analyze it in real-time.
	* If required on a daily or hourly basis, periodic jobs can be scheduled.
	* Schedule a Notebook as a job; this doesn't require the cluster to be alive always.

## Cloning

* **CDF:** Tracks row level changes between two versions.

* **Cloning:** Creating a copy of the delta table at whichever version is required.

### Types of Clones

1. **Shallow Clone:** Only metadata is copied. The copied table points/references the source data.

2. **Deep Clone:** Copies both data and metadata.

---

### Shallow Cloning Demo Steps:

1. **Connect to the catalog:**
```sql
USE CATALOG deltalake_catalog;

```

2. **Create a new managed table named `orders_managed`:**
```sql
DROP TABLE IF EXISTS orders_managed;
CREATE OR REPLACE TABLE orders_managed (
  order_id BIGINT,
  sku STRING,
  product_name STRING,
  product_category STRING,
  qty INT,
  unit_price DECIMAL (10,2)
);

```

3. **Add a check constraint:**
```sql
ALTER TABLE orders_managed ADD CONSTRAINT valid_qty CHECK (qty > 0);

```

4. **Insert records:**
*(New parquet and JSON files will be created upon insertion.)*

5. **Delete a record:**
*(Deletion vector will be created and Optimize will run, consolidating data into one parquet file.)*

6. **Update a record:**
*(One Deletion vector will be created and Optimize automatically consolidates into one parquet file.)*

* **Check dependent files:**
```sql
DESCRIBE DETAIL <table_name>;

```

#### Performing a Shallow Clone:

```sql
DROP TABLE IF EXISTS orders_managed_sclone;
CREATE OR REPLACE TABLE orders_managed_sclone SHALLOW CLONE orders_managed;
DESCRIBE DETAIL orders_managed_sclone;

```

*(By default, if you just mention `CLONE`, it performs a Deep Clone.)*

#### Notes on Shallow Clone:

* Only metadata for the latest versions will be copied.

* Refers to the latest files of the source table.

* Both source and shallow cloned tables have exact same data initially as they point to the same parquet files.

#### Shallow Clone Scenarios:

* **Inserting records into source table:** Changes will **not** reflect in the cloned table (copy is frozen at that specific version).

* **Inserting records into cloned table:** Changes will **not** show in the source table. A new parquet file is created for the cloned table, making them independent.

* **Deleting records from cloned table:** Creates a new file followed by an auto-optimize activity.

* **Deleting source records and executing aggressive `VACUUM retain 0 hours`:** The files referenced by the shallow clone are **not** deleted, while unreferenced files are removed.

```sql
DELETE FROM orders_managed_sclone;

ALTER TABLE orders_managed_sclone
SET TBLPROPERTIES ('delta.deletedFileRetentionDuration' = 'interval 0 hours');

VACUUM orders_managed_sclone DRY RUN;

```

* **Restoring a table to a specific version:**
```sql
RESTORE TABLE orders_managed VERSION AS OF 8;

```

*(Vacuum works only if files are not referenced by source or cloned tables.)*


### Deep Cloning

* Copies **both Data and Metadata**.
* It is storage and time-consuming, process-heavy compared to Shallow Clone.
* If you drop the source table, the deep clone retains its data *(Note: for shallow clone managed tables, dropping source breaks target, though data remains accessible for 7 days)*.

```sql
DROP TABLE IF EXISTS orders_managed_dclone;
CREATE OR REPLACE TABLE orders_managed_dclone DEEP CLONE orders_managed;
DESCRIBE DETAIL orders_managed_dclone;

```

* Deep Clone copies the required data of the latest versions from where it was cloned.
* **Modifications:** Changes to source or clone do not affect each other.
* **Cloning vs. CTAS (`CREATE TABLE AS SELECT`):**
* CTAS does **not** save metadata like constraints.
* Deep clones preserve state including metadata for archival purposes.

#### Use Cases:

1. **Archival:** Take monthly production table snapshots:
```sql
CREATE OR REPLACE TABLE archive_table CLONE my_prod_table;

```

2. **Testing Workflows:** Create a shallow clone of a prod table to test workflows without corrupting production data.

#### Rules:

* Shallow clones on external tables should be external tables; shallow clones on managed tables should be managed tables.
* Dropping source managed table breaks shallow clone target (data retrievable within 7 days).
* Shallow clones on external tables are not impacted by dropping source.
* Clones can be created for any specific version:
```sql
CREATE TABLE table_shallow SHALLOW CLONE source_table VERSION AS OF <version>;

```

## Delta Lake UniForm (Universal Format)

* Both Delta Lake and Iceberg store data underlying as Parquet files.
* The key difference is how they maintain and structure **Metadata**.
* **Iceberg clients cannot natively read Delta metadata**.
### Approaches to Share Delta Data with Iceberg Clients:

1. **Approach 1 (Copy/Rewrite):** Rewrite Delta table as Iceberg table.
	* *Disadvantages:* Data duplication, high storage costs, slow write times.
2. **Approach 2 (UniForm - Ideal):** Maintain one copy of Parquet data files with separate metadata files for each format.

### Enabling UniForm Demo:

```sql
USE CATALOG deltalake_catalog;

DROP TABLE IF EXISTS orders_managed;
CREATE OR REPLACE TABLE orders_managed (
  order_id BIGINT,
  sku STRING,
  product_name STRING,
  product_category STRING,
  qty INT,
  unit_price DECIMAL (10,2)
) TBLPROPERTIES (
  'delta.columnMapping.mode' = 'name',
  'delta.enableIcebergCompactV2' = 'true',
  'delta.universalFormat.enabledFormats' = 'iceberg'
);

```

* After inserting records, checking `DESCRIBE TABLE` displays `# Delta Uniform Iceberg`.
* Iceberg metadata files are automatically generated alongside Delta metadata.

---
