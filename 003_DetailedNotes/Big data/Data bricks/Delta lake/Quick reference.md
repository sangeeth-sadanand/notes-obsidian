---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Delta lake/Delta lake|Delta lake]]"
tags:
  - DataBricks/deltaLake
index: 100
type: topic
---
# Quick reference

```ad-summary
collapse: true 
title: Summary
 
 1. **Time Travel**-Allows you to view a historical snapshot of your data.
2. **Restore**-Reverts the entire Delta table back to a previous state
3. **Optimize**-combine many small, inefficient Parquet files into optimally sized larger files
4. `ZORDER BY` sorts and co-locates the data within those files based on the specified column
5. **VACUUM**-Cleans up your storage to save costs and enforce data retention policies.
6. **Reorg**-Reclaims physical storage space and cleans up fragmented data after extensive `DELETE` or `UPDATE` operations.
7. **Upsert (Merge)**- Performs an (Update + Insert) in a single, atomic operation.
 
 
# Quick Reference- Useful Commands

~~~sql
-- Time travel
SELECT * FROM table VERSION AS OF 2;
SELECT * FROM table TIMESTAMP AS OF '2024-01-01T12:00:00';

-- History and restore
DESCRIBE HISTORY table_name;
RESTORE TABLE table_name TO VERSION AS OF 1;

--OPTIMIZE 
-- Basic compaction
OPTIMIZE my_db.my_table;

-- Compact only recent partitions (recommended)
OPTIMIZE my_db.my_table WHERE date >= current_date() - INTERVAL 7 DAYS;

-- Compact + Z‑Order
OPTIMIZE my_db.my_table ZORDER BY (customer_id, order_date);

-- Full rewrite (recompress or recluster)
OPTIMIZE my_db.my_table FULL;

-- VACUUM
VACUUM table_name DRY RUN;
VACUUM table_name RETAIN 168 HOURS;

-- REORG to remove deletion vectors
REORG TABLE table_name APPLY (PURGE);

-- MERGE (upsert)
MERGE INTO target AS t
USING source AS s
ON t.key = s.key
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
~~~
```
