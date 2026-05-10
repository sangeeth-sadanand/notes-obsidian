---
up:
  - "[[003_skills/data-engineering/05 Hive/05 Hive|05 Hive]]"
down:
prev:
topic: false
question: How does schema evolution handled in Hive?
---
# How does schema evolution handled in Hive?


> [!Summary] Summary
> - Hive follows a "schema on read" philosophy.
> - Data reside on HDFS as raw file and schema is applied at run of query
> - Meta data change is only updated on metastore
> - It does not rewrite existing files on disk which make schema change instantaneous.
> - Adding a columns, New rows will populate this column and old row will return null
> - hive allows data type promotion if change is safe if change is not compatible the hive will return null
> - renaming and reordering 
> 	- csv/text the order of column depends on position. if we shift schema but not files then data will shift into wrong columns
> 	- parquet/ORC they are mapped using name rather than index allowing safer reordering and renaming
> -	The partition change hive only changes to new data old will remain in same partition
> - Use cascade for to push down changes down to all existing partitions.




Hive handles **schema evolution** by allowing you to **add, change, or remove columns** in tables without breaking existing data.  
How it works depends on **file format**, **table type**, and **SERDE**.

## 1. What is Schema Evolution in Hive?

Schema evolution = ability to **change table structure** while keeping existing data intact.

Typical operations:

*   Add new columns
*   Change column order
*   Rename columns
*   Change data types (with limits)

## 2. How Hive Handles Schema Evolution

### A. For Text/CSV/TSV Tables (Row‑based formats)

Hive is **flexible and forgiving**.

#### Adding columns

```sql
ALTER TABLE emp ADD COLUMNS (salary INT);
```

Behavior:

*   Old data files do **not** contain the new column → Hive returns `NULL` for those files.
*   New data can include the new column.

#### Dropping/reordering columns

Hive does **not** enforce column order; it matches columns by position.

Drawback:

*   Schema mismatches may produce shifted/wrong data.

## 3. Schema Evolution with Columnar Formats (Parquet/ORC)

These formats support **true schema evolution** where metadata is stored in the file.

### A. ORC

Supports:

*   Add columns
*   Rename columns
*   Change column types (compatible types)

Hive reads ORC file footer metadata → makes schema evolution safe.

Example:

```sql
ALTER TABLE sales ADD COLUMNS (region STRING);
```

Reading old ORC file:

*   Hive sees old schema in ORC footer
*   Missing columns → returned as NULL

### B. Parquet

Supports:

*   Add optional columns
*   Remove optional columns
*   Rename (logical rename; physical name remains)
*   Type widening (INT → BIGINT)

Parquet stores schema in the file header so each file is self‑describing.

## 4. Internal vs External Tables

### Internal Tables

*   Schema stored in Hive Metastore
*   Data stored in ORC/Parquet/text
*   Schema evolution applies uniformly

### External Tables

*   Hive updates only metadata
*   Data files remain untouched
*   Parquet/ORC evolution works at the **file level**, so evolution must be compatible with file schemas

## 5. Partitioned Tables and Schema Evolution

When schema changes:

*   **New partitions** use the new schema
*   **Old partitions** keep old schema

Hive handles this by:

*   Reading partition schema → merging with table schema
*   Missing fields → returned as NULL

## 6. Type Evolution Rules

### Allowed

*   `INT → BIGINT`
*   `FLOAT → DOUBLE`
*   `STRING → VARCHAR`
*   `VARCHAR → STRING`
*   Adding columns

### Not Allowed (unsafe)

*   `BIGINT → INT`
*   `DOUBLE → FLOAT`
*   Complex type incompatible changes
*   Column deletion (without file rewrite)

## 8. How Schema Evolution Works Internally

1.  Hive metastore stores the *latest* schema.
2.  ORC/Parquet store *their own schema* in each file.
3.  During read:
    *   Hive compares metastore schema vs file schema.
    *   If columns missing → fill with NULL.
    *   If types differ → apply compatible conversion.
    *   If incompatible → throw error or fail silently (depending on format).





