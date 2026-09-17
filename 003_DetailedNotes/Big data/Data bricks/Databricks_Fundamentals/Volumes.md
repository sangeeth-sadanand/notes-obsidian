---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Databricks_Fundamentals/Databricks_Fundamentals|Databricks_Fundamentals]]"
tags:
  - DataBricks/fundamentals
index: 5
type: topic
---
# Volumes

```ad-summary
collapse: true 
title: Summary

- Volumes are Unity Catalog objects designed to govern and manage file-based, non-tabular datasets (such as CSVs, JSON, PDFs, and images).
- They are the recommended replacement for legacy DBFS mount points because they provide centralized governance, fine-grained access control, auditing, and secure access without exposing raw cloud storage paths.
## Managed vs. External Volumes
Databricks supports two distinct types of volumes depending on your data lifecycle needs:

| Feature             | Managed Volumes                                                                     | External Volumes                                                                                     |
| ----------------------- | ----------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Storage & Lifecycle** | Fully managed by Databricks; data is deleted when the volume is dropped.                  | Kept in customer-managed cloud storage; underlying data persists even if the volume is deleted.           |
| **Access Control**      | Strict isolation; direct access via cloud URIs is not allowed.                            | Governed by Unity Catalog internally, but external systems can still access data using direct cloud URIs. |
| **Best For**            | Internally generated data like ML artifacts, intermediate datasets, and application logs. | Legacy data, shared datasets, and workloads that mix Databricks with external systems.                    |
## The Role of External Locations
- External volumes require an **External Location**, which defines a trusted root path in your cloud storage (e.g., an ADLS Gen2 URI).
- This creates a key conceptual distinction: the external location dictates _where_ Databricks can access data overall, while the external volume dictates _who_ can access specific subfolders within it, enabling highly granular governance.

## Quick Reference: Key Operations

Based on your provided syntax, here are the most common commands for interacting with volumes:

- **Create a Volume:** Use `CREATE VOLUME IF NOT EXISTS catalog_name.schema_name.volume_name;` to initialize it.
- **Write Data:** Save DataFrames directly using `df.write.format("delta").mode("overwrite").save("path_to_volume")`.
- **Query Data:** Read from the volume path using `SELECT * FROM delta.\`volume_path`;`.
- **View Metadata:** Check history and formats using `DESCRIBE DETAIL 'volume_path';` or `DESCRIBE HISTORY table_name;` for time travel.

```
---
## What are volumes?
- Volumes are Unity Catalog objects in Databricks that provide governance, access control, and management over non-tabular datasets. 
- While tables are used to govern tabular data, volumes are used to govern file-based, non-tabular data of any format. 
- Databricks strongly recommends using Volumes to govern all non-tabular data instead of directly accessing cloud storage paths.

## Types of Data Governed by Volumes
Volumes can store and govern:
- **Structured data** (CSV, Parquet files not registered as tables)
- **Semi-structured** data (JSON, XML, Avro)
- **Unstructured data** (images, PDFs, audio, video, text files, logs)

Any file-based data that is not represented as a table should be governed using a volume.

## Why Use Volumes?
Volumes provide the following benefits:
- Centralized governance through Unity Catalog
- Fine-grained access control
- Auditing and lineage support
- Secure access without exposing cloud storage paths
- Seamless integration with Databricks notebooks, jobs, and workflows

## Types of Volumes
Databricks supports two types of volumes:
### 1. Managed Volumes
- Managed volumes are fully managed by Databricks. 
- Databricks controls the storage location and lifecycle of the data. 
- If a volume is dropped, the data is dropped after the retention period.

**Create a volume**
```SQL
create volume catalog_name.schema_name.volume_name
```
 
 By default creates a managed volume

**Create a directory under volume**
```sql
%fs  
mkdir /Volumes/catalog_name/schema_name/volume_name/directory_name
```
 
**Read a csv file**

```SQL
%sql  
select * from csv.`<path to csv in volume>`
```

```python
spark.read.format("csv").option("header", "true").load("<path to volume csv>")
```
 

**Key Characteristics:**
- No cloud storage path needs to be specified
- Storage and folder structure are managed by Databricks
- Strong governance by default
- Ideal for internally generated data

**Typical Use Cases:**
- Application-generated files
- Machine learning artifacts
- Intermediate datasets
- Logs and checkpoint files

### 2. External Volumes
External volumes reference an existing cloud storage location while enabling governance through Unity Catalog.
**Key Characteristics:**
- Data remains in customer-managed cloud storage
- No data movement required
- Unity Catalog controls access and permissions
- Suitable for legacy or shared datasets
    
**Typical Use Cases:**
- Pre-existing cloud storage data
- Data shared across multiple platforms
- External systems writing directly to cloud storage

#### External Volume Prerequisite: External Location

An external location defines a trusted cloud storage path that Unity Catalog can access. Example (Azure Data Lake Storage Gen2)
- External Location Name: externaldata
- Container / Folder: ext_volume
- URI: `abfss://externaldata@ttmystorageaccount001.dfs.core.windows.net/ext_volume`
    
**Create External volumes**
1. Create a ADLS gen 2 storage
2. Create external Location
```sql
 %sql  
 CREATE EXTERNAL volume cat_name.sche_name.vol_name  
 LOCATION 'vol location'
```

3. Create dir
```SQL
 %fs  
 mkdir /Volumes/cat_name/schema_name/vol_name/folder
```

> [!Warning]
> _If the volume is deleted then the underlying volume data is not deleted_

Once the external location is configured, an external volume can be created on top of a subfolder within that location.

**Key Characteristics of External Volumes**
- Data physically resides in customer-managed cloud storage
- Unity Catalog governs access permissions
- No data movement is required
- External tools and systems can still access data using direct URIs
- Enables controlled access at subfolder level

## Managed vs External Volumes

|Feature|Managed Volume|External Volume|
|---|---|---|
|Storage Location|Databricks-managed|Customer-managed cloud storage|
|Data Lifecycle|Managed by Databricks|Managed externally|
|Access Control|Fully enforced by Unity Catalog|Governed by UC, but external access possible|
|External Tool Access|Not allowed|Allowed via direct URIs|
|Typical Use Case|Databricks-only workloads|Mixed Databricks + external systems|

### Access Control Differences

**Managed Volumes**
- All access goes through Unity Catalog
- No direct access using cloud storage URIs
- Strongest governance and isolation

**External Volumes**
- Unity Catalog controls access within Databricks
- External systems can still access data using cloud URIs
- Useful when data must be shared across platforms
    
### Why External Volumes when External locations already exist?

This is a key conceptual distinction.
- **External Locations:** Grant access to a root cloud storage path. Permissions apply to the entire location. Coarse-grained access control.
- **External Volumes:** Are created within external locations. Allow governance at subfolder level. Enable granular access control.
    
In short:
- External locations control _where_ Databricks can access data.
- External volumes control _who_ can access which part of that data. This enables fine-grained governance without exposing the entire storage path.

## DBFS Mount Points vs Unity Catalog Volumes

Historically, Databricks used DBFS mount points to access external cloud storage. Databricks is now transitioning from DBFS mount points to Unity Catalog volumes because volumes provide:
- Centralized governance
- Fine-grained access control
- Better auditing and lineage
- Native integration with the Unity Catalog security model
- Stronger alignment with the broader Databricks ecosystem

Volumes are the recommended replacement for DBFS mount points.

## Quick reference - Working with Volumes
**Create a Volume:**
```sql
-- Create volume inside a specific catalog and schema
CREATE VOLUME IF NOT EXISTS catalog_name.schema_name.volume_name;
```

**Create a Folder Inside a Volume:**
```python
# Using dbutils
dbutils.fs.mkdirs('volumes/catalog_name/schema_name/volume_name/folder_name')
```


```sql
# Using magic commands
%fs mkdirs /volumes/catalog_name/schema_name/volume_name/folder_name
```

**Write a DataFrame into a Volume:**
```python
df.write.format("delta").mode("overwrite").save("path_to_volume")
```

**Query Volume Data / External Data:**
```sql
SELECT * FROM delta.`volume_path`;
```

## Table Metadata & History
**Describe Details and Format:**
```sql
DESCRIBE DETAIL 'volume_path';
DESCRIBE FORMATTED table_name;
DESCRIBE EXTENDED table_name;
DESCRIBE HISTORY table_name; -- Views the transaction history (Time Travel)
```

**View Underlying Files:**
```python
display(dbutils.fs.ls("path_to_table"))
```

## Reading JSON Logs
```python
log_file_path = "path/to/_delta_log/00000000000000000000.json"
log_df = spark.read.json(log_file_path)
display(log_df)
```
```sql
SELECT * FROM json.`path/to/_delta_log/00000000000000000000.json`;
```

## Creating an External Table
```sql
CREATE OR REPLACE TABLE table_name (
    -- schema definition --
) 
USING DELTA 
LOCATION 'path/to/external/storage';
```

