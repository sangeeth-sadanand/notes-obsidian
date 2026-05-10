---
up:
  - "[[003_skills/data-engineering/05 Hive/05 Hive|05 Hive]]"
down:
prev:
topic: false
question: How does hive translate to MR and spark?
---
# How does hive translate to MR and spark?

> [!Summary] Summary
> - Hive does not execute queries by itself.
> - It parses HQL and translate it into execution plan that run on MR, Apache spark or Tez
> - When a hive query is submitted, it is parsed and an AST is generated
> - This AST is compiled and optimized and a physical plan is generated (MR or spark) based on the config. then this plan is executed on the YARN
> - In MR Job, one or more Map reduce Job is created with heavy I/O job.
> - In Spark Job, It create one job with multiple stages based on wide transforms
> - We can set the config using
> ```hive
> -- MapReduce
> SET hive.execution.engine=mr;
> -- Spark
> SET hive.execution.engine=spark;
> ```




- Apache **Hive doesn’t execute queries by itself**. 
- Instead, it **parses HiveQL and translates it into execution plans** that run on **MapReduce (MR)**, **Apache Spark**, or **Tez**.  
## High‑level Flow (Same for MR & Spark)

1.  **HiveQL submitted**
2.  **Parser** → builds AST (Abstract Syntax Tree)
3.  **Compiler & Optimizer**
    *   Predicate pushdown
    *   Column pruning
    *   Join reordering
4.  **Physical plan generation**
    *   MR jobs **OR** Spark jobs
5.  **Execution via YARN**

The **difference** lies in **Step 4: physical execution plan**.

## Hive on MapReduce (Classic)

### Architecture

*   Each Hive query → **one or more MapReduce jobs**
*   Each job has:
    *   Map phase
    *   Optional Reduce phase
*   Heavy disk I/O between stages (HDFS read/write)

### Example Query

```sql
SELECT dept, COUNT(*)
FROM employees
GROUP BY dept;
```


### How Hive Translates This to MapReduce

#### **Map Phase**

*   Reads rows from HDFS
*   Emits:

```text
(dept, 1)
```

#### **Shuffle & Sort**

*   Groups all rows with same `dept`

#### **Reduce Phase**

*   Reducer receives:

```text
(dept, [1,1,1,1...])
```

*   Emits:

```text
(dept, total_count)
```

**1 MR job (Map → Reduce)**

### Join Example (MR)

```sql
SELECT e.name, d.dept_name
FROM emp e JOIN dept d ON e.dept_id = d.id;
```

#### Translation

*   If **large tables**:
    *   MR Job 1: shuffle both tables by join key
    *   Reducers perform join
*   If **small table detected**:
    *   Map‑side join (Distributed Cache)

✅ Usually **2–3 MR jobs**

### Characteristics of Hive on MapReduce

| Aspect          | MapReduce   |
| --------------- | ----------- |
| Execution       | Batch       |
| Disk I/O        | Heavy       |
| Latency         | High        |
| Fault tolerance | Very strong |
| Startup time    | Slow        |

## Hive on Spark (Modern)

### Architecture

*   Hive generates a **Spark DAG**
*   Executes operations **in memory**
*   Uses **RDDs / DataFrames**

### Same Query

```sql
SELECT dept, COUNT(*)
FROM employees
GROUP BY dept;
```

***

### Translation to Spark

#### Logical Plan

*   Scan → Project → GroupBy → Aggregate

#### Physical Spark Plan

```text
HDFS Read → map(dept,1)
        → reduceByKey(sum)
        → Store/Return result
```

✅ Runs as **a single Spark job with multiple stages**

### Join Example (Spark)

```sql
SELECT e.name, d.dept_name
FROM emp e JOIN dept d ON e.dept_id = d.id;
```

#### Spark Execution

*   **Broadcast join** (if small table)
*   Or **Shuffle hash join**

✅ Done **within a single Spark DAG**

### Characteristics of Hive on Spark

| Aspect               | Spark         |
| -------------------- | ------------- |
| Execution            | In-memory DAG |
| Disk I/O             | Minimal       |
| Latency              | Low           |
| Iterative processing | Efficient     |
| Startup time         | Much faster   |

## Execution Plan Comparison

| Feature           | Hive on MR         | Hive on Spark                 |
| ----------------- | ------------------ | ----------------------------- |
| Query → Jobs      | Many MR jobs       | Single DAG                    |
| Intermediate data | Written to HDFS    | Memory (spill if needed)      |
| Joins             | Reduce-side joins  | Broadcast / Hash joins        |
| Performance       | Slower             | 5–50× faster                  |
| Best for          | Massive batch jobs | Interactive & mixed workloads |

## How to See the Translation

### Explain Plan

```sql
EXPLAIN
SELECT dept, COUNT(*)
FROM employees
GROUP BY dept;
```

### Detailed Plan

```sql
EXPLAIN EXTENDED
SELECT ...
```

### Spark Physical Plan

```sql
SET hive.execution.engine=spark;
EXPLAIN FORMATTED SELECT ...
```


## Configuration Switch

```sql
-- MapReduce
SET hive.execution.engine=mr;

-- Spark
SET hive.execution.engine=spark;
```

***
