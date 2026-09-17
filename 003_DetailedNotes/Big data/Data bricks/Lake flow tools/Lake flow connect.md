---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Lake flow tools/Lake flow tools|Lake flow tools]]"
tags:
  - DataBricks/LakeFlow/connect
index: 2
type: topic
---
# Lake flow connect

```ad-summary
collapse: true 
title: Summary

- Databricks Lakeflow Connect serves as a native framework to efficiently ingest data from diverse sources directly into the Bronze layer of a Medallion Architecture.

## Lakeflow Connector Types

* **Upload Connectors:** Move local files into Databricks Volumes.
* **Standard Connectors:** Ingest data from cloud storage (ADLS Gen2, S3) using batch, incremental batch, or continuous streaming modes.
* **Managed Connectors:** Extract data directly from SaaS applications and external databases.

## Standard Ingestion Mechanisms for Cloud Storage
Depending on data volume and operational requirements, standard connectors utilize three primary methods to move data into Delta Lake:

### 1. CTAS (Create Table As Select)
A standard SQL command (`CREATE TABLE target AS SELECT * FROM source`) ideal for one-off data loads, raw file ingestion, or rapid exploration.

* **State Tracking:** CTAS performs a full data reload every time it runs. It does not track what has already been processed.
* **Error Handling:** You can define how bad data is treated using three modes: **Permissive** (default, saves bad data to a `_rescued_data` column), **DropMalformed** (silently discards errors), or **FailFast** (aborts the job immediately).

### 2. COPY INTO
A Databricks-native SQL command built for incremental batch ingestion of tens to thousands of files.

* **State Tracking:** It is highly idempotent and guarantees exactly-once processing. It automatically tracks ingested files and skips them on subsequent runs without requiring manual state management.
* **Configuration:** Allows schema evolution using `mergeSchema` (which can add new columns but cannot alter existing data types) and accepts specific format configurations (e.g., delimiters, headers).
* **Limitation:** Performance degrades when handling millions of files per hour or when attempting near real-time streaming.

### 3. Auto Loader
Databricks’ advanced ingestion engine designed to process massive file volumes either continuously or in scalable batches.

* **State Tracking:** Uses PySpark's `writeStream` API and maintains exactly-once semantics by tracking file metadata in a highly scalable RocksDB key-value store (the checkpoint location).
* **Flexibility:** Can operate continuously (`.trigger(processingTime="10 seconds")`) or process all available files and shut down for cost-effective batching (`.trigger(availableNow=True)`).

## Auto Loader Schema Management
Auto Loader samples up to 1,000 files or 50GB of data to infer schemas automatically. It allows you to provide schema hints (e.g., forcing an `order_id` to be a `BIGINT`), manually define strict schemas, or append system metadata (file paths, ingestion timestamps) for lineage.

When the source data introduces unexpected columns, Auto Loader dictates behavior through four **Schema Evolution Modes**:

* **addNewColumns:** (Default if no schema is provided). Automatically updates the schema with the new columns. The stream will briefly fail and gracefully restart with the new schema applied.
* **rescue:** Schema is never evolved. All unexpected columns are packed into a single rescued data column, allowing the pipeline to continue uninterrupted while preserving the raw data for later analysis.
* **failOnNewColumns:** Strict enforcement. The stream halts immediately upon detecting unfamiliar columns until the schema is manually updated.
* **none:** (Default if a manual schema is provided). The stream ignores any new columns entirely, simply dropping the unexpected data without failing.
  
  
> [!Note] 
> `addNewColumns` is default when schema is not defined and `none` is default mode when schema is defined

```
---
## Lakeflow Connect - Connector Types
Lakeflow Connect is Databricks' native data ingestion framework that enables building scalable, governed, and efficient ingestion pipelines directly within the Databricks platform.
Types of Connectors:
1. **Upload Connectors:** Upload files from local systems into Databricks volumes.
2. **Standard Connectors:** Ingest data from cloud storage into Delta Lake.
3. **Managed Connectors:** Ingest data from SaaS applications and databases.

### Standard Connectors - Ingestion Modes
Standard connectors support:
* **Batch ingestion:** Re-ingests full data on every run
* **Incremental batch ingestion:** Ingests only new data; skips already ingested records
* **Incremental streaming ingestion:** Continuously ingests data in near real time

## Data Ingestion from Cloud Storage
This section explains the approaches used to ingest data from cloud storage into Delta Lake using Lakeflow Connect standard connectors.
Data ingestion can be performed from: Azure Data Lake Storage Gen2, Amazon S3, Databricks Volumes.
The ingested data is stored in Delta Lake, typically in the Bronze layer of the Medallion Architecture.

Databricks provides multiple ingestion mechanisms for cloud storage data:

### 1. CTAS (Create Table As Select)
CTAS is a SQL-based ingestion mechanism used to create a Delta table directly from source data.
```sql
CREATE TABLE target_table AS SELECT * FROM source;
```
* The table is created if it does not already exist
* Data is loaded immediately during table creation

CTAS is commonly used when:
* Performing initial data ingestion
* Loading raw files from cloud storage
* Creating Delta tables quickly for exploration or downstream processing
* Data volume is manageable and full reloads are acceptable (batch ingestion)

**CTAS Ingestion Behavior**
To re-run, we need to drop the table first. It will reload everything.
```sql
%sql
use catalog catalog_name;
Create table <table_name> as
select * from read_files('path_to_volumes', format = 'csv')
```
With schema:
```sql
create table orders as
select * from read_files(
  'path-volume',
  format => 'csv',
  schema => 'order_id BIGINT',
  header => True,
  mode => 'PERMISSIVE',
  columnNameOfCorruptRecord => '_rescued_data'
)
```

**Full Data Reload**
* All data is re-ingested every time the CTAS statement is executed
* There is no built-in tracking of previously ingested data
* Existing tables are either overwritten or recreated
* *Implication:* CTAS is not suitable for recurring ingestion of large datasets where incremental loading is required.

**Handling Bad or Malformed Records in CTAS**
* **Permissive Mode:** Default behavior. Job continues even if malformed records are encountered. Bad records are stored in the rescued_data column. Recommended for initial ingestion.
* **DropMalformed Mode:** Malformed records are silently dropped. Job continues without failure. Recommended for non-critical data.
* **FailFast Mode:** Job fails immediately upon encountering the first malformed record. Ensures strict data correctness.

### 2. COPY INTO
COPY INTO is a Databricks-native command used to load data from cloud file locations into Delta Lake tables. It is designed for reliable, incremental batch ingestion and addresses the limitations of full reload approaches such as CTAS.

**Key Characteristics of COPY INTO**
* Loads data from cloud storage into Delta Lake
* Idempotent and retriable operation
* Guarantees exactly-once processing
* Automatically skips files that have already been ingested
* Optimized for ingestion of up to thousands of files

**Incremental Batch Ingestion Behavior**
COPY INTO follows an incremental batch ingestion model:
* Only new files in the source location are ingested
* Files that have already been loaded are automatically skipped
* No manual tracking of processed files is required

**Table Creation Options**
* **Creating an Empty Table Without Schema:** Schema is inferred from the source data.
  ```sql
  %sql
  drop table if exist orders;
  create table orders;
  
  COPY INTO catalog.schema.table
  from 'Path'
  FILEFORMAT = CSV
  FORMAT_OPTIONS (
    'header' = 'true',
    'infer_schema' = 'true',
    'delimiter' = ','
  )
  COPY_OPTIONS ('mergeSchema' = 'true')
  ```
  *Note: mergeSchema does not modify data type; it can only add new columns.*
* **Creating a Table With Schema:** Enforces schema consistency, prevents unexpected drift.

**COPY INTO Configuration Options**
* `format_options()`: Define how source files are parsed and interpreted (e.g., Delimiter, Header, Date formats).
* `copy_options()`: Control the behavior of the COPY INTO operation itself (e.g., Schema evolution support, handling of schema changes).

**When to Use COPY INTO**
* Data arrives periodically in cloud storage
* Incremental ingestion is required
* File counts range from tens to thousands
* Streaming ingestion is not necessary
* Reliability and idempotency are important

*Drawback:* It becomes inefficient or impractical when new files arrive every few minutes/seconds, millions of files arrive every hour, or near real-time ingestion is required.

### 3. Auto Loader
Auto Loader is an advanced ingestion mechanism in Databricks designed to incrementally and efficiently process new data files as they arrive in cloud storage.

**Key Capabilities of Auto Loader:**
* Incrementally processes newly arriving files
* Supports incremental batch and streaming ingestion
* Can backfill or migrate billions of files
* Scales to ingest millions of files per hour
* Guarantees exactly-once semantics
* Handles failures and restarts gracefully

**Supported Cloud Storage Sources:** Amazon S3, Azure Data Lake Storage Gen2, Google Cloud Storage (GCS), Databricks Volumes.

**Auto Loader Ingestion Modes**
Unified API: `writeStream` is used for both batch and streaming ingestion.
* **Streaming Mode:** Continuously processes new files as they arrive. `.trigger(processingTime="10 seconds")`
* **Batch-Style Mode (Available Now):** Processes all available files and then stops. `.trigger(availableNow=True)`. Used for backfills or large migrations.

**Example Code:**
```python
orders_df = (
  spark.readStream
  .format("cloudFiles")
  .option("cloudFiles.format", "csv")
  .option("cloudFiles.inferSchema", "true")
  .option("cloudFiles.inferColumnTypes", "true")
  .option("cloudFiles.schemaLocation", checkpoint_path)
  .load(orders_data)
)

(orders_df.writeStream
  .format("delta")
  .option("checkpointLocation", checkpoint_path)
  .option("mergeSchema", "true")
  .outputMode("append")
  .trigger(processingTime='10 seconds') # stream mode
  # .trigger(availableNow=True) # batch mode
  .toTable("table_path")
)
```

**How Auto Loader Tracks Ingestion Progress**
Auto Loader tracks ingestion progress using a checkpoint mechanism.
* **Metadata Tracking:** As files are discovered, their metadata is stored in a scalable key-value store implemented using RocksDB, persisted in the checkpoint location.
* **Exactly-Once Semantics:** Each file is processed only once. In case of job failure, Auto Loader resumes from the last processed state.

## Auto Loader - Schema Management
Auto Loader provides flexible mechanisms to infer schema automatically, manually define schema, control schema evolution behavior, capture unexpected data, and add metadata columns.

### 1. How Schema Inference Works Internally
Auto Loader infers schema based on a sample of data.
**Sampling Limits for Schema Inference:**
Schema inference stops when either condition is met first:
* 1000 files
* 50 GB of data

**Configuration Options:**
* `cloudFiles.schemaInference.sampleSize.numBytes`: Controls the maximum number of bytes scanned.
* `cloudFiles.schemaInference.sampleSize.numFiles`: Controls the maximum number of files scanned.

### 2. Manually Defining the Schema
In some scenarios, schema inference may not be desirable (strict enforcement required). You can explicitly define the schema.
* **Important Behavior Change:**
  * When a schema is **not** provided, Auto Loader defaults to "add new columns" mode.
  * When a schema **is** provided, Auto Loader defaults to "none" mode.

### 3. Schema Hints
Schema hints allow you to partially control schema inference without fully defining the schema (e.g., enforcing correct data types for specific columns).
```python
.option("cloudFiles.schemaHints", "order_id BIGINT")
```

### 4. Adding New Metadata Columns
Auto Loader allows adding system-generated metadata columns during ingestion (File path, File name, File modification time, Ingestion timestamp). Useful for auditing and lineage.

## Schema Evolution Modes in Auto Loader
Schema evolution determines how Auto Loader reacts when new columns appear in incoming data.

**1. none**
* **Behavior:** Schema is not evolved. New columns are ignored. Data is not rescued unless `rescuedDataColumn` is explicitly set. Stream does not fail due to schema changes.
* **Use Case:** Strict schema enforcement. (Default mode when schema is defined).
* `option("cloudFiles.schemaEvolutionMode", "none")`

**2. failOnNewColumns**
* **Behavior:** Stream fails immediately when new columns are detected. Stream does not restart unless schema is updated manually or the offending file is removed.
* **Use Case:** Strong data quality guarantees.
* `option("cloudFiles.schemaEvolutionMode", "failOnNewColumns")`

**3. rescue**
* **Behavior:** Schema is never evolved. Stream does not fail due to schema changes. All new or unexpected columns are captured in the rescued data column.
* **Use Case:** Ingest first, analyze schema changes later.
* `option("cloudFiles.schemaEvolutionMode", "rescue")`

**4. addNewColumns (default)**
* **Behavior:** Stream detects new columns and adds them to the schema automatically. Existing columns do not evolve their data types. Stream may briefly fail and then restart with updated schema.
* **Use Case:** Rapidly evolving data sources. (Default mode when no schema is provided).
* `option("cloudFiles.schemaEvolutionMode", "addNewColumns")`


> [!Note]
> `addNewColumns` is default when schema is not defined and `none` is default mode when schema is defined



## Managed Connectors

- Managed connectors are a Lakeflow Connect capability used to ingest data from databases and SaaS applications into Databricks.
- They abstract away connection handling, incremental ingestion logic, and orchestration, making database ingestion simpler and more reliable.
### Change Detection Mechanisms
- Managed connectors support database-level mechanisms to detect and ingest only changed
data.

1. Change Tracking (CT)
	- Tracks rows that have changed since the last reload
	- Lightweight and efficient
	- Does not store full historical changes
2. Change Data Capture (CDC)
	- Captures insert, update, and delete operations
	- Maintains detailed change history
	- Suitable for advanced replication and auditing use cases

### High-Level Ingestion Flow
1. Data is extracted from the source database using a secure connection
2. Extracted data is written to a Databricks Volume (staging area)
3. Data from the volume is ingested into Delta Lake tables

### Supported Azure Databases
Lakeflow Connect supports a wide range of database deployments, including:

* **Azure SQL Database**
* **Microsoft SQL Server** running on Azure VMs
* **On-premises SQL Server** accessed through Azure ExpressRoute
* **PostgreSQL** and **MySQL**

### Key Architecture Components
Database ingestion via Lakeflow Connect involves three primary components to capture continuous changes:

* **Ingestion Gateway:** A continuous task running in its own job that connects to the source database, extracts snapshots, and continuously captures change data from the source.
* **Ingestion Pipeline:** Powered by serverless compute, this pipeline replicates the CDC/CT data source and handles schema evolution events. It only brings in new data or updates, making ingestion fast and cost-efficient.
* **Destination Tables:** The streaming tables in Unity Catalog where the pipeline writes the ingested data.

**Prerequisites for SQL Server Ingestion**
To configure ingestion from an Azure SQL Database or SQL Server, you must ensure:

* **SQL Version:** SQL Server 2012 or later is required for change tracking, though 2016 or newer is highly recommended. Enterprise edition is required if using CDC on versions older than 2016.
* **Change Tracking:** Microsoft Change Tracking (CT) or Change Data Capture (CDC) must be enabled on the source database to track inserts, updates, and deletes.
* **Service Account:** A dedicated database service account must be configured with the necessary privileges for Databricks to access the database.

**How to Configure Ingestion**
You can set up a Lakeflow Connect database pipeline using the Databricks UI or Databricks Asset Bundles (DABs):

1. **Add Data:** In the Databricks UI, select **Add or Upload data**, then choose the SQL Server or target database connector.
2. **Set up Gateway:** Provide a pipeline name and specify a Unity Catalog Volume location for staging.
3. **Configure Pipeline:** Create the connection to your Azure SQL database using the required authentication.
4. **Select Tables:** Choose specific tables or an entire schema to ingest. Databricks recommends splitting large schemas across multiple pipelines for optimal performance.
5. **Set Destination:** Select the target Unity Catalog catalog and schema.
6. **Schedule:** Set the frequency for the pipeline to run (e.g., every 6 hours) and configure success/failure notifications.