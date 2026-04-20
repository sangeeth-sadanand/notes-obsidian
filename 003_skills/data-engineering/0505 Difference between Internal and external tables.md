---
up:
  - "[[003_skills/data-engineering/05 Hive|05 Hive]]"
down:
prev:
topic: false
question: Difference between Internal and external tables.
---
# Difference between Internal and external tables

> [!Summary] Summary
> 
> They differ mainly in data ownership and life cycle management.
> 
> **Internal table**
> -	they are fully managed by hive, Hive controls both metadata as well as the data.
> -	Data is stored in default location
> -	Both metadata and data are deleted if table is dropped
> -	It is used for temporary data, hive only pipeline etc.
> -	truncate is allowed
> 
> **External table**
> -	hive only manages the metadata and not the data
> -	data is managed external to hive
> -	data is stored in custom location
> -	when a table is dropped only metadata is dropped
> -	Data can be shared across multiple tools.
> -	Truncate operation is not allowed
> 
> We can convert internal table to external table by Setting TBL PROPERTY ( 'EXTERNAL' = 'TRUE')
> 

In **Apache Hive**, **Internal (Managed) tables** and **External tables** differ mainly in **data ownership and lifecycle management**.  

## Internal (Managed) Table

### Definition
An **internal table** is **fully managed by Hive**.  
Hive controls **both the metadata and the actual data** stored in HDFS.

### Key Characteristics

*   Data stored under Hive’s warehouse directory:
        /user/hive/warehouse/<table_name>
*   Hive **owns the data**
*   Data lifecycle is tied to the table
### What Happens on DROP?

```sql
DROP TABLE employees;
```

 **Both table metadata AND data files are deleted**

### Example

```sql
CREATE TABLE employees (
  id INT,
  name STRING,
  dept STRING
);
```

*   Data location (default):
        /user/hive/warehouse/employees/

### When to Use Internal Tables
✔ Temporary data  
✔ ETL intermediate results  
✔ Hive-only pipelines  
✔ Data you don’t need after table deletion

## External Table

### Definition

An **external table** is where **Hive manages only metadata**,  
while the **data is stored and managed externally** (outside Hive).

### Key Characteristics

*   Data stored at a **custom location**
*   Hive does **NOT own** the data
*   Metadata lifecycle ≠ data lifecycle

### What Happens on DROP?

```sql
DROP TABLE ext_employees;
```

✅ **Only metadata is deleted**  
✅ **Data remains intact in HDFS**

### Example

```sql
CREATE EXTERNAL TABLE ext_employees (
  id INT,
  name STRING,
  dept STRING
)
LOCATION '/data/employees';
```

*   Data stays at:
        /data/employees/

### When to Use External Tables

✔ Shared data across tools (Spark, Impala, Presto)  
✔ Existing HDFS data  
✔ Production data  
✔ Data you must not accidentally delete

## Key Differences (Side‑by‑Side)

| Feature               | Internal Table      | External Table      |
| --------------------- | ------------------- | ------------------- |
| Data ownership        | Hive owns data      | User owns data      |
| Metadata              | Managed by Hive     | Managed by Hive     |
| Data deletion on DROP | ✅ Yes               | ❌ No                |
| Default location      | Hive warehouse      | Custom location     |
| Data safety           | Lower               | Higher              |
| Use case              | Temporary / staging | Shared / production |
## TRUNCATE Behavior

```sql
TRUNCATE TABLE table_name;
```

| Table Type | Result                           |
| ---------- | -------------------------------- |
| Internal   | ✅ Data deleted                   |
| External   | ❌ Not allowed (metadata remains) |

***

## Conversion Between Tables

### Internal → External

```sql
ALTER TABLE employees SET TBLPROPERTIES ('EXTERNAL'='TRUE');
```

### External → Internal

```sql
ALTER TABLE ext_employees SET TBLPROPERTIES ('EXTERNAL'='FALSE');
```

Use carefully—data ownership changes.


