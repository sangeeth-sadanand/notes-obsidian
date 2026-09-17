---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Delta lake/Delta lake|Delta lake]]"
tags:
  - DataBricks/deltaLake
index: 3
type: topic
---
# Concurrency control and schema management

```ad-summary
collapse: true 
title: Summary

## Data Integrity and Concurrency

- **Write-Time Constraints:** Delta enforces constraints (like `NOT NULL` or `CHECK`) at the moment data is written. If a write violates the rule, the entire operation fails, protecting data quality.
    
- **Optimistic Concurrency Control (OCC):** Delta guarantees ACID properties without slowing down the system with traditional database locks. Writers read a snapshot version of the data, make their changes, and attempt to commit. If another writer has updated the table in the meantime (changing the version), the commit fails and prompts a retry. This ensures strong consistency in highly scalable, distributed environments.
    

## Schema Management

- **Schema Enforcement:** By default, Delta strictly enforces the table's schema. Any incoming data that doesn't match is rejected, preventing accidental schema drift and corruption.
    
- **Schema Evolution:** When you intentionally want to change the structure, Delta allows for safe evolution. You can add new columns dynamically on write (using `mergeSchema`) or safely widen data types (e.g., from `INT` to `LONG`). Narrowing data types (e.g., `STRING` to `INT`) is not done automatically.
    

## Column Mapping (Renaming & Dropping)

- **The Parquet Problem:** Standard Parquet files embed column names directly into the file schema. Traditionally, renaming or deleting a single column meant entirely rewriting all the underlying files—an incredibly expensive and slow process.
    
- **The Delta Solution:** Delta Lake solves this using **Column Mapping**, which separates the logical column name from the physical one. Once enabled, renaming or dropping a column becomes a lightning-fast, metadata-only operation inside the transaction log.
    
- **How Deletes Work:** When a column is dropped, Delta simply marks it as logically deleted in the metadata. Queries will immediately stop surfacing that column, even though the historical physical data remains untouched in the Parquet files (which also preserves your ability to time travel).

```
---
## Constraints, ACID, and Concurrency Control

- **Constraints:** Delta supports constraints (e.g., `NOT NULL`, `CHECK`) enforced at write time. Violations cause the write to fail.
    ```sql
    ALTER TABLE <table-name> ADD CONSTRAINT chk_qty_price CHECK (qty > 0 AND price > 0);
    ```
    
- **ACID & isolation:** Delta provides ACID guarantees using **Optimistic Concurrency Control (OCC)**:
    - Writers read a snapshot (a version), perform changes, and attempt to commit.
    - If the table changed since the snapshot, the commit fails and must be retried.
- **Example (version-based conflict):**
    - Table at version 3. 
    - Writer A reads v3 and commits → table becomes v4. 
    - Writer B, still on v3, tries to commit → conflict detected → retry required.
- **Why OCC works:** No locking overhead, scalable for distributed workloads, and provides strong consistency.
## Schema Enforcement and Schema Evolution
- **Schema enforcement:** Incoming data must match the table schema; non-conforming writes are rejected to prevent schema drift and data corruption.
- **Schema evolution:** Delta can evolve schema safely (e.g., add columns, widen types).
    - **Add column (merge schema):**
        ```python
        df.write.option("mergeSchema", "true").format("delta").save("/path/to/table")
        ```
        
    - **Type widening:** Allowed (e.g., `INT` → `LONG`, `FLOAT` → `DOUBLE`) when enabled:
        ```sql
        SET TBLPROPERTIES ('delta.enableTypeWidening' = 'true');
        ```
    - Narrowing conversions (e.g., `STRING` → `INT`) are not applied automatically.

## Column Rename and Delete (Column mapping)
- **Problem with Parquet:** Parquet files embed column names in file schema; renaming or dropping a column traditionally requires rewriting all files.
- **Solution — Column mapping:** Delta separates **logical** and **physical** column names. Renames and deletes can be metadata-only operations (no parquet rewrite).
    - **Benefits:** Fast, scalable, avoids expensive rewrites, preserves historical data for time travel.
- **Operations:**
    - Enable name-based column mapping:
        ```sql
        ALTER TABLE <table-name> SET TBLPROPERTIES ('delta.columnMapping.mode' = 'name');
        ```
        
    - Rename column (metadata-only):
        ```sql
        ALTER TABLE <table-name> RENAME COLUMN old_name TO new_name;
        ```
        
    - Drop column: Delta marks the column logically deleted in metadata; physical data may remain in parquet files but queries no longer surface the column.
