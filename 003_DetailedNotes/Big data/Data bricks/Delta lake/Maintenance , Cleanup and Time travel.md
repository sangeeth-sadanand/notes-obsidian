---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Delta lake/Delta lake|Delta lake]]"
tags:
  - DataBricks/deltaLake
index: 5
type: topic
---
# Maintenance , Cleanup and Time travel

```ad-summary
collapse: true 
title: Summary

## Time Travel and Restoration
Because Delta Lake retains old Parquet files and transaction logs, you can easily query or restore previous table states.

- **Querying History:** You can view older versions of your data using SQL or Spark by specifying a version number (e.g., `VERSION AS OF 2`) or a specific time (e.g., `TIMESTAMP AS OF '2024-01-01T12:00:00'`).
- **Restoring:** If data gets corrupted or accidentally deleted, you can roll back the entire table using the `RESTORE TABLE` command.
## VACUUM (File Cleanup)
While Time Travel is powerful, keeping every historical file forever will drastically inflate storage costs.
- **How it Works:** The `VACUUM` command physically deletes old Parquet files that are no longer referenced by the current table state **and** are older than your configured retention period (the default is 7 days).
- **Safety First:** Always use `VACUUM <table-name> DRY RUN` to preview what will be deleted.

> [!Danger] 
> Never run `VACUUM ... RETAIN 0 HOURS` in a production environment. 
> It deletes unreferenced files immediately, permanently breaking your ability to Time Travel and risking irreversible data loss.

## Managing Logs and Deletion Vectors

As your table processes updates and deletes, metadata can pile up and slow down read performance.
- **Log Compaction:** Thousands of small JSON transaction logs create read overhead. Delta uses **Checkpoints** to periodically compact these JSON logs into single, larger files to speed up scanning.
- **Reclaiming Space (REORG):** While Deletion Vectors are great for fast writes, accumulating too many of them degrades query performance. To fix this, run `REORG TABLE <table-name> APPLY (PURGE)`. This physically rewrites the files without the deleted data. Afterward, run `VACUUM` to permanently delete the old, bloated files.
## Production Best Practices
- **Retention:** Stick to the default 7-day retention window unless you have a specific, well-understood reason to change it.
- **Maintenance:** Regularly run `OPTIMIZE` (for file sizing), `REORG` (to clear Deletion Vectors), and checkpoints to keep query speeds fast.
- **Schema Changes:** Be cautious with automatic schema evolution (`mergeSchema`). For critical data pipelines, it is much safer to make explicit schema changes.
- **Concurrency:** Always design your write pipelines to automatically retry if they encounter an Optimistic Concurrency Control (OCC) conflict.
```
---

## Time Travel and File Retention
- **Purpose:** Retain removed parquet files so Delta Lake can reconstruct table state for any historical version or timestamp.
- **Version & time model:** Versions V1,V2,V3... map to commit timestamps T1,T2,T3.... To read a past state, Delta Lake scans logs up to the requested version or timestamp and includes only files active at that point.
- **How to query historical state:**
    - SQL (version):
        ```sql
        SELECT * FROM orders VERSION AS OF 2;
        ```
    - SQL (timestamp):
        ```sql
        SELECT * FROM orders TIMESTAMP AS OF 'yyyy-MM-ddTHH:mm:ss';
        ```
    - Spark API:
        ```python
        df = spark.read.option("versionAsOf", 2).table("orders")
        df = spark.read.option("timestampAsOf", "2024-01-01T12:00:00").table("orders")
        ```
        
- **History and restore:**
    - View history: `DESCRIBE HISTORY <table-name>;`
    - Restore: `RESTORE TABLE <table-name> TO VERSION AS OF 1;` 
	    - this creates a restore log entry and requires both parquet files and JSON logs to be present.


## VACUUM (File cLeanup) and retention
- **Purpose:** Remove data files that are no longer referenced by any valid table version and are older than the configured retention period (default **7 days**). Benefits include reduced storage cost and compliance with retention policies.
- **Basic commands:**
    - Dry run (preview):
        ```sql
        VACUUM <table-name> DRY RUN;
        ```
    - Actual run (example):
        ```sql
        VACUUM <table-name> RETAIN 168 HOURS; -- 7 days
        ```
    - Aggressive (not recommended in production):
        ```sql
        VACUUM <table-name> RETAIN 0 HOURS;
        ```
        
> [!Warning]
> RETAIN 0 HOURS` removes files immediately, breaks time travel, and can cause irreversible data loss.
        
- **Retention configuration (table-level):**
    ```sql
    ALTER TABLE <table-name> SET TBLPROPERTIES ('delta.deletedFileRetentionDuration' = '30 days');
    ```
    
- **Safety check:** Delta enforces a retention-duration safety check. It can be disabled (not recommended) via Spark config:
    ```python
    spark.conf.set("spark.databricks.delta.retentionDurationCheck.enabled", "false")
    ```
    
- **Folder cleanup behavior:** VACUUM may remove files and leave empty directories; subsequent runs can remove empty directories.


## Delta Log Files, Checkpoints, and Read Performance
- **Log growth problem:** Many small commits create many JSON log files; scanning many logs can slow reads.
- **Mitigations:**
    - **Checkpoints** and **log compaction** combine multiple JSON logs into fewer files (e.g., `000001.json`), reducing read overhead.
    - Delta configuration properties:
        - `delta.logRetentionDuration` — how long JSON log files are kept.
        - `delta.deletedFileRetentionDuration` — how long deleted parquet files are kept.
            
- **Deletion vectors:**
    - Without deletion vectors, updates/deletes rewrite entire parquet files (expensive).
    - With deletion vectors, Delta records deletions in metadata (smaller writes, faster). Over time many deletion vectors can accumulate and impact performance.
## Reclaiming Space and Removing Deletion Vectors
- **When deletion vectors accumulate:** Performance may degrade. `OPTIMIZE` may not remove deletion vectors. Use `REORG` to rewrite files and purge deleted data:
    ```sql
    REORG TABLE <table-name> APPLY (PURGE);
    -- APPLY (UPGRADE) may rewrite all files
    ```
    
- After `REORG ... APPLY (PURGE)`, run `VACUUM` to physically delete old files.


## Practical Recommendations

- **Retention & VACUUM**
    - Keep a safe retention window (default 7 days) unless you fully understand the risks of immediate deletion.
    - Use `VACUUM ... DRY RUN` before actual deletion.
    - Avoid `RETAIN 0 HOURS` in production.
        
- **Performance**
    - Regularly compact logs and run checkpoints to reduce JSON log scan overhead.
    - Use `OPTIMIZE` and `REORG` (with `APPLY (PURGE)`) after large deletes to reclaim space and remove deletion vectors.
    - 
- **Schema**
    - Use `mergeSchema` and type-widening cautiously; prefer explicit schema changes for critical pipelines.
        
- **Concurrency**
    - Design clients to retry commits on OCC conflicts.





