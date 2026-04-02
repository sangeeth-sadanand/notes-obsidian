# Hive

## Managed vs External table

- In Apache Hive, the distinction between **Internal** (or Managed) and **External** tables boils down to one main question: **Who owns the data?**
- Understanding this determines how Hive handles your files when you run a `DROP TABLE` command.
### 1. Internal Tables (Managed)
When you create an internal table, Hive assumes full responsibility for the data. It manages both the table metadata (the schema) and the actual physical data files on HDFS.
- **Storage Location:** By default, data is stored in the Hive warehouse directory (usually `/user/hive/warehouse/`).
- **The "Drop" Behavior:** If you drop an internal table, Hive deletes **both** the metadata and the actual data files from HDFS.
- **Best Use Case:** When you want Hive to manage the lifecycle of the data entirely, or for temporary/intermediate tables.
### 2. External Tables
External tables are essentially a pointer to data that exists outside of Hive's direct control. Hive only manages the metadata.
- **Storage Location:** You must specify a `LOCATION` in HDFS where the data already lives (or will live).
- **The "Drop" Behavior:** If you drop an external table, Hive deletes **only** the metadata. The actual data files remain untouched on HDFS.
- **Best Use Case:** When the data is shared across multiple tools (like Spark, Pig, or MapReduce) or when you want to ensure the data isn't accidentally deleted if someone drops the table schema.
### Key Comparison Table

|**Feature**|**Internal (Managed) Table**|**External Table**|
|---|---|---|
|**Keyword**|`CREATE TABLE ...`|`CREATE EXTERNAL TABLE ...`|
|**Data Ownership**|Managed by Hive|Managed by HDFS/User|
|**Default Location**|`/user/hive/warehouse/`|Defined by the `LOCATION` clause|
|**On `DROP TABLE`**|Deletes Metadata **AND** Data|Deletes **ONLY** Metadata|
|**Data Sharing**|Not ideal for sharing files|Perfect for sharing across tools|
|**ACID Support**|Supports ACID/Transactions|Limited/No ACID support|
### How to tell the difference?

If you aren't sure which type a table is, you can check it using this command in your Hive CLI:

SQL

```
DESCRIBE FORMATTED table_name;
```

## Schema evolution

- Schema evolution in Hive refers to the ability to modify the structure of a table over time—such as adding, renaming, or changing the data types of columns—without needing to recreate the table or rewrite the existing data.
- How Hive handles these changes depends heavily on the **File Format** (e.g., Parquet, ORC, Avro, or Text) and the **Table Type** (Internal vs. External).
### 1. Common Schema Evolution Operations
Here are the standard SQL commands used to evolve a schema:
- **Adding Columns:** `ALTER TABLE table_name ADD COLUMNS (new_col INT COMMENT 'New column');`
- **Renaming/Changing Columns:** `ALTER TABLE table_name CHANGE old_col new_col STRING;`
- **Replacing Columns (Drop/Restructure):** `ALTER TABLE table_name REPLACE COLUMNS (col1 INT, col2 STRING);`
### 2. How Different Formats Handle Evolution

Not all file formats are created equal when it comes to schema changes. This is where the underlying storage engine matters most.

| **Format**   | **Support Level** | **Behavior**                                                                                                                                                               |
| ------------ | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Avro**     | **Excellent**     | Best for evolution. It stores the schema in the file header. If you change the schema in the Metastore, Avro maps old data to the new schema seamlessly using field names. |
| **ORC**      | **Good**          | Supports adding columns. It uses **column indexes**. If you add a column, Hive simply returns `NULL` for that column when reading older files that don't have it.          |
| **Parquet**  | **Good**          | Similar to ORC, it uses **name-based** or **index-based** mapping. It is quite flexible with adding new columns at the end of the schema.                                  |
| **CSV/Text** | **Poor**          | Based purely on position. If you add a column in the middle of the schema, your data will shift and become corrupted (misaligned). You should only add columns to the end. |

### 3. The "Metadata vs. Data" Conflict
A common point of confusion is why data doesn't "update" immediately.
- **Metastore Update:** When you run `ALTER TABLE`, Hive only updates its **Metastore** (the catalog). It does _not_ go back and rewrite the existing physical files on HDFS.
- **Lazy Evolution:** Hive handles the discrepancy at **read-time**. When you query the table, Hive looks at the current schema in the Metastore and tries to map the old files to it.
- **Partitioned Tables:** This is a major "gotcha." If you add a column to a partitioned table, the change only applies to **new partitions** created after the command. To fix old partitions, you must use the `CASCADE` keyword:
    
    `ALTER TABLE table_name ADD COLUMNS (c1 INT) CASCADE;`
### 4. Key Limitations & Best Practices
- **Data Type Compatibility:** You can usually evolve a type "upwards" (e.g., `INT` to `BIGINT`), but "downwards" (e.g., `STRING` to `INT`) will likely result in `NULL` values if the data cannot be cast.
- **Positioning:** Avoid inserting columns into the middle of a table schema if you are using Text or Sequence files, as Hive reads these by position.
- **Use Avro for Buffering:** If your schema changes weekly, Avro is generally the industry standard for the landing zone because of its robust schema-matching logic.
