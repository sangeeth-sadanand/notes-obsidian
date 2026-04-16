# 03 How does map phase differ from reduce phase

The **Map phase** and **Reduce phase** are the two core stages of the **MapReduce** programming model.

## 1. Map Phase

### Purpose

The **Map phase** processes input data and converts it into **intermediate key–value pairs**.

### What it does

*   Takes raw input data (files, records, lines of text, etc.)
*   Breaks it into smaller pieces
*   Applies a **map function** to each piece independently
*   Produces intermediate results as `(key, value)` pairs

### Characteristics

*   Runs **in parallel** on multiple nodes
*   Performs **filtering, transformation, or extraction**
*   Does **not aggregate** data globally

### Example (Word Count)

Input:

    "big data big analytics"

Map output:

    (big, 1)
    (data, 1)
    (big, 1)
    (analytics, 1)

***

## 2. Reduce Phase

### Purpose

The **Reduce phase** aggregates and processes the intermediate key–value pairs produced by the Map phase.

### What it does

*   Receives grouped data from the Map phase
*   Applies a **reduce function** to each key
*   Combines values associated with the same key
*   Produces the **final output**

### Characteristics

*   Runs after the Map phase completes
*   Works on **grouped and sorted** data
*   Focuses on **aggregation, summarization, or computation**

### Example (Word Count)

Input to Reduce:

    (big, [1, 1])
    (data, [1])
    (analytics, [1])

Reduce output:

    (big, 2)
    (data, 1)
    (analytics, 1)

***

## 3. Key Differences at a Glance

| Aspect             | Map Phase                    | Reduce Phase                   |
| ------------------ | ---------------------------- | ------------------------------ |
| Main role          | Data transformation          | Data aggregation               |
| Input              | Raw input data               | Intermediate key–value pairs   |
| Output             | Intermediate key–value pairs | Final result                   |
| Execution          | Runs first                   | Runs after map phase           |
| Parallelism        | Highly parallel              | Parallel per key               |
| Data handling      | Independent processing       | Grouped by key                 |
| Typical operations | Filtering, parsing, mapping  | Summation, counting, averaging |

***

## 4. How They Work Together

1.  **Map phase** processes raw data and emits key–value pairs.
2.  **Shuffle & Sort** (automatic framework step) groups values by key.
3.  **Reduce phase** processes each group to generate final results.

***

# 04 Advantages and disadvantages of map-reduce


## Advantages of MapReduce

### 1. Scalability

*   Can process **huge volumes of data (TBs–PBs)** by distributing work across many machines
*   Easily scales horizontally by adding more nodes

***

### 2. Fault Tolerance

*   Automatically handles **node failures**
*   Re‑executes failed map or reduce tasks on other nodes
*   Data is replicated (e.g., in HDFS), preventing loss

***

### 3. Parallel Processing

*   Map tasks run independently and in parallel
*   Reduce tasks also run in parallel for different keys
*   Leads to faster processing of large datasets

***

### 4. Simple Programming Model

*   Developers focus only on **map() and reduce() logic**
*   No need to manage low‑level details like:
    *   Threading
    *   Synchronization
    *   Network communication

***

### 5. Data Locality

*   Computation is moved **closer to where data is stored**
*   Reduces network traffic and improves efficiency

***

### 6. Cost‑Effective

*   Runs on **commodity hardware**
*   Open‑source frameworks like Hadoop reduce overall cost

***

## Disadvantages of MapReduce

### 1. Not Suitable for Real‑Time Processing

*   Designed for **batch processing**
*   High latency makes it poor for:
    *   Real‑time analytics
    *   Interactive queries
    *   Streaming data

***

### 2. Disk I/O Overhead

*   Intermediate results are written to disk
*   Causes **slower performance** compared to in‑memory systems (e.g., Apache Spark)

***

### 3. Complex for Iterative Algorithms

*   Algorithms like machine learning and graph processing require repeated iterations
*   MapReduce reloads data in every iteration, making it inefficient

***

### 4. Rigid Programming Model

*   Only supports **map and reduce operations**
*   Complex workflows require chaining multiple MapReduce jobs
*   Results in verbose and harder‑to‑maintain code

***

### 5. High Latency

*   Job startup time is significant
*   Not ideal for small or quick tasks

***

### 6. Debugging is Difficult

*   Runs across distributed systems
*   Tracking errors and performance issues is challenging

***

##  Summary Table

| Aspect            | MapReduce                         |
| ----------------- | --------------------------------- |
| Best for          | Large‑scale batch processing      |
| Scalability       | Excellent                         |
| Fault tolerance   | High                              |
| Speed             | Slower due to disk I/O            |
| Real‑time support | ❌ No                              |
| Iterative tasks   | ❌ Inefficient                     |
| Ease of use       | Simple concept, complex workflows |

***

## When to Use MapReduce

*   Log processing
*   Offline analytics
*   Data transformation (ETL)
*   Processing very large, static datasets

## When Not to Use MapReduce

*   Real‑time analytics
*   Interactive queries
*   Machine learning with many iterations
*   Streaming applications

# What problem does YARN solve in hadoop

**YARN (Yet Another Resource Negotiator)** was introduced in Hadoop to solve **resource management and scalability problems** present in earlier versions of Hadoop (Hadoop 1.x).

## Problem YARN Solves in Hadoop

### The Core Problem:

In **Hadoop 1.x**, the **JobTracker** was responsible for **too many tasks**, which caused major limitations.

## Problems Before YARN (Hadoop 1.x)

In Hadoop 1.x, the **JobTracker** handled:

1.  **Job scheduling**
2.  **Resource management**
3.  **Task monitoring**
4.  **Fault tolerance**

This design caused several issues:

### 1. Scalability Issue

*   One JobTracker managed the entire cluster
*   As cluster size increased, JobTracker became a **bottleneck**
*   Limited number of nodes and jobs could be supported

### 2. Single Point of Failure

*   If JobTracker failed, **all running jobs failed**
*   Entire cluster processing stopped

### 3. Inefficient Resource Utilization

*   Resources were fixed as **map slots and reduce slots**
*   Idle map slots couldn’t be used for reduce tasks and vice versa

### 4. Limited to MapReduce Only

*   Hadoop could not easily support other processing frameworks like:
    *   Spark
    *   Tez
    *   Storm

## ✅ How YARN Solves These Problems

YARN **separates resource management from data processing**, making Hadoop more powerful and flexible.

## Key Solutions Provided by YARN

### 1. Separates Responsibilities

YARN splits JobTracker functionality into:

*   **ResourceManager**
    *   Manages cluster-wide resources
*   **ApplicationMaster**
    *   Manages individual applications
*   **NodeManager**
    *   Manages resources on each node

 Result: **Better scalability and performance**

***

### 2. Dynamically Allocates Resources

*   Uses **containers** instead of fixed slots
*   Allocates CPU and memory as needed

✅ Result: **Efficient resource utilization**

***

### 3. Eliminates Single Point of Failure

*   Failure of one ApplicationMaster affects **only that application**
*   ResourceManager can be configured for high availability

✅ Result: **Improved fault tolerance**

***

### 4. Supports Multiple Processing Frameworks

YARN allows Hadoop to run:

*   MapReduce
*   Spark
*   Hive
*   HBase
*   Flink

✅ Result: Hadoop becomes a **general-purpose data processing platform**

***

### 5. Improves Cluster Utilization

*   Multiple applications share the same cluster resources
*   Better scheduling (FIFO, Capacity, Fair Scheduler)

✅ Result: **Cost-effective and optimized clusters**

***

## Summary: Problem vs Solution

| Problem in Hadoop 1.x   | How YARN Solves It                  |
| ----------------------- | ----------------------------------- |
| JobTracker bottleneck   | Distributed management              |
| Single point of failure | Application-level failure isolation |
| Fixed map/reduce slots  | Dynamic containers                  |
| MapReduce-only          | Multiple frameworks                 |
| Poor scalability        | Large cluster support               |



# What is the use of resource manager and node manager

In **Hadoop YARN**, the **ResourceManager** and **NodeManager** work together to manage cluster resources and application execution.

***

## ✅ ResourceManager (RM)

### **Use / Role**

The **ResourceManager** is the **master daemon** responsible for **cluster‑wide resource management and scheduling**.

### **Main Functions**

*   Manages **CPU and memory resources** across the entire Hadoop cluster
*   Decides **which application gets how many resources**
*   Schedules applications using schedulers like:
    *   FIFO
    *   Capacity Scheduler
    *   Fair Scheduler
*   Tracks availability of resources on all nodes
*   Communicates with **ApplicationMasters** and **NodeManagers**

### **Key Point**

> ResourceManager **does not execute tasks** — it only **allocates resources**

***

## ✅ NodeManager (NM)

### **Use / Role**

The **NodeManager** is a **slave daemon** that runs on **each node** in the cluster and manages **node‑level resources**.

### **Main Functions**

*   Manages **containers** on its node
*   Launches and monitors containers for applications
*   Monitors:
    *   CPU usage
    *   Memory usage
    *   Disk
*   Reports node status and health to the ResourceManager
*   Kills containers if they exceed allocated resources

### **Key Point**

> NodeManager **executes and monitors tasks** on a specific node

***

## 🔁 How They Work Together

1.  A job is submitted to YARN
2.  **ResourceManager** allocates resources
3.  **NodeManager** launches containers on assigned nodes
4.  NodeManager monitors execution and reports back to ResourceManager

***

## 📊 Comparison Table

| Feature           | ResourceManager                  | NodeManager                      |
| ----------------- | -------------------------------- | -------------------------------- |
| Level             | Cluster‑wide                     | Node‑level                       |
| Type              | Master daemon                    | Slave daemon                     |
| Responsibility    | Resource allocation & scheduling | Container execution & monitoring |
| Runs on           | Master node                      | Every worker node                |
| Resource handling | Global view of resources         | Local resource usage             |
| Task execution    | ❌ No                             | ✅ Yes                            |

***

## ✅ Short Exam‑Ready Answer

**The ResourceManager manages and allocates resources across the Hadoop cluster, while the NodeManager runs on each node to execute tasks, monitor containers, and report resource usage back to the ResourceManager.**



# What is Hive?

**Hive** provides a way for users—especially those familiar with SQL—to work with **big data** without writing complex MapReduce programs.

> In simple terms:  
> **Hive = SQL interface for Hadoop**

***

## 🔍 Why Hive is Used

*   Traditional SQL databases cannot efficiently handle **very large datasets**
*   Writing **MapReduce code** is complex and time‑consuming
*   Hive bridges this gap by converting **SQL‑like queries into MapReduce or Spark jobs**

***

## 🧩 How Hive Works

1.  User writes a query in **HiveQL**
2.  Hive parses and compiles the query
3.  The query is converted into:
    *   MapReduce jobs (earlier)
    *   Tez or Spark jobs (modern Hive)
4.  Jobs execute on Hadoop
5.  Results are returned to the user

***

## 🏗️ Key Components of Hive

*   **HiveQL (HQL)** – SQL‑like query language
*   **Metastore** – Stores table schemas and metadata
*   **Driver** – Manages query lifecycle
*   **Compiler** – Converts HQL into execution plan
*   **Execution Engine** – Runs tasks on Hadoop

***

## ✅ Features of Hive

*   SQL‑like querying (easy to learn)
*   Supports **structured and semi‑structured data**
*   Schema‑on‑read
*   Integrates with Hadoop ecosystem (HDFS, YARN, Spark)
*   Supports partitions and buckets for optimization
*   Supports UDFs (User Defined Functions)

***

## ❌ Limitations of Hive

*   Not suitable for **real‑time or OLTP queries**
*   High latency (batch processing)
*   Limited row‑level updates (though ACID support exists in newer versions)

***

## 📊 Hive vs Traditional Database

| Aspect      | Hive                  | RDBMS     |
| ----------- | --------------------- | --------- |
| Data size   | Very large (TBs, PBs) | Moderate  |
| Query type  | Batch processing      | Real‑time |
| Latency     | High                  | Low       |
| Updates     | Limited               | Full CRUD |
| Scalability | Horizontal            | Vertical  |

***

## ✅ Use Cases of Hive

*   Data warehousing
*   Log analysis
*   ETL processing
*   Business intelligence reporting
*   Offline analytics


# Difference between HQL and SQL


## ✅ Difference Between HQL and SQL

### 🔹 HQL (Hive Query Language)

*   Used in **Apache Hive**
*   Designed for querying **big data stored in Hadoop (HDFS)**
*   SQL‑like, but optimized for **batch processing**

### 🔹 SQL (Structured Query Language)

*   Used in **relational databases (RDBMS)** like MySQL, Oracle, PostgreSQL
*   Designed for **structured data** with **real‑time querying**

***

## 📊 Comparison Table

| Aspect            | HQL                                       | SQL                            |
| ----------------- | ----------------------------------------- | ------------------------------ |
| Full form         | Hive Query Language                       | Structured Query Language      |
| Used in           | Apache Hive                               | RDBMS (MySQL, Oracle, etc.)    |
| Data storage      | HDFS                                      | Tables in databases            |
| Processing type   | Batch processing                          | Real‑time / interactive        |
| Latency           | High                                      | Low                            |
| Query execution   | Converted to MapReduce / Spark / Tez jobs | Executed directly by DB engine |
| Schema            | Schema‑on‑read                            | Schema‑on‑write                |
| Transactions      | Limited (ACID support added later)        | Full ACID support              |
| Updates & deletes | Limited, not row‑level friendly           | Fully supported                |
| Scalability       | Horizontal (scale‑out)                    | Mostly vertical (scale‑up)     |
| Use case          | Big data analytics, data warehousing      | OLTP, transactional systems    |

***

## 🧠 Key Differences Explained

### 1. **Performance**

*   **SQL** is faster because it works on indexed, structured data
*   **HQL** is slower because it processes massive datasets in batch mode

***

### 2. **Data Size**

*   **SQL** handles GBs of data efficiently
*   **HQL** is designed for **TBs or PBs of data**

***

### 3. **Query Execution**

*   **SQL** runs queries immediately
*   **HQL** converts queries into distributed jobs, which increases latency

***

### 4. **Flexibility**

*   **HQL** handles structured and semi‑structured data
*   **SQL** requires strictly structured schemas

***

## 📝 Example

### SQL Query

```sql
SELECT name FROM employees WHERE salary > 50000;
```

### HQL Query

```sql
SELECT name FROM employees WHERE salary > 50000;
```

✅ Syntax looks similar  
❌ Execution and performance are very different

***

# What is role of meta store in hive

In **Apache Hive**, the **Metastore** plays a crucial role by managing **metadata information** about the data stored in Hive.

***

## ✅ Role of Metastore in Hive

The **Hive Metastore** is a **central repository** that stores information *about* the data, not the actual data itself.

***

## 🧩 What Metadata Does It Store?

The Metastore stores details such as:

*   Database names
*   Table names
*   Column names and data types
*   Table location in HDFS
*   Partitions and buckets information
*   Table type (managed or external)
*   Serialization/deserialization (SerDe) information
*   Table properties and statistics

***

## 🔍 Why Metastore is Important

### 1. **Schema Management**

*   Maintains the schema of Hive tables
*   Enables Hive to apply **schema‑on‑read** when querying data

***

### 2. **Query Execution Support**

*   When an HQL query is submitted:
    *   Hive consults the Metastore to understand table structure
    *   Determines where the data is stored in HDFS
*   Without Metastore, Hive cannot process queries

***

### 3. **Data Location Tracking**

*   Keeps track of **where data files are physically located**
*   Helps Hive access the correct HDFS paths

***

### 4. **Metadata Sharing**

*   Other tools like **Spark, Presto, Impala** can use the same Metastore
*   Enables consistent metadata across the Hadoop ecosystem

***

### 5. **Partition Pruning & Optimization**

*   Stores partition information
*   Helps Hive skip unnecessary data scans, improving performance

***

## 🏗️ Metastore Architecture (Brief)

*   Stored in an **RDBMS** (MySQL, PostgreSQL, Oracle, etc.)
*   Accessed via **Metastore Service**
*   Supports:
    *   Embedded mode
    *   Local metastore
    *   Remote metastore (recommended for production)

***

## 📌 What Metastore Does NOT Store

❌ Actual data files  
✅ Only metadata about the data

***

# How does hive translate to MR and spark

Apache **Hive doesn’t execute queries by itself**. Instead, it **parses HiveQL and translates it into execution plans** that run on **MapReduce (MR)**, **Apache Spark**, or **Tez**.  
Below is a **clear, step‑by‑step explanation of how Hive translates queries to MapReduce and Spark**, with practical examples.

***

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

***

# 1️⃣ Hive on MapReduce (Classic)

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

***

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

✅ **1 MR job (Map → Reduce)**

***

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

***

### Characteristics of Hive on MapReduce

| Aspect          | MapReduce   |
| --------------- | ----------- |
| Execution       | Batch       |
| Disk I/O        | Heavy       |
| Latency         | High        |
| Fault tolerance | Very strong |
| Startup time    | Slow        |

***

# 2️⃣ Hive on Spark (Modern)

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

***

### Join Example (Spark)

```sql
SELECT e.name, d.dept_name
FROM emp e JOIN dept d ON e.dept_id = d.id;
```

#### Spark Execution

*   **Broadcast join** (if small table)
*   Or **Shuffle hash join**

✅ Done **within a single Spark DAG**

***

### Characteristics of Hive on Spark

| Aspect               | Spark         |
| -------------------- | ------------- |
| Execution            | In-memory DAG |
| Disk I/O             | Minimal       |
| Latency              | Low           |
| Iterative processing | Efficient     |
| Startup time         | Much faster   |

***

# 3️⃣ Execution Plan Comparison

| Feature           | Hive on MR         | Hive on Spark                 |
| ----------------- | ------------------ | ----------------------------- |
| Query → Jobs      | Many MR jobs       | Single DAG                    |
| Intermediate data | Written to HDFS    | Memory (spill if needed)      |
| Joins             | Reduce-side joins  | Broadcast / Hash joins        |
| Performance       | Slower             | 5–50× faster                  |
| Best for          | Massive batch jobs | Interactive & mixed workloads |

***

# 4️⃣ How to See the Translation

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

***

# 5️⃣ Configuration Switch

```sql
-- MapReduce
SET hive.execution.engine=mr;

-- Spark
SET hive.execution.engine=spark;
```

***


# Difference between Internal and external tables

In **Apache Hive**, **Internal (Managed) tables** and **External tables** differ mainly in **data ownership and lifecycle management**.  

***

## 1️⃣ Internal (Managed) Table

### Definition

An **internal table** is **fully managed by Hive**.  
Hive controls **both the metadata and the actual data** stored in HDFS.

***

### Key Characteristics

*   Data stored under Hive’s warehouse directory:
        /user/hive/warehouse/<table_name>
*   Hive **owns the data**
*   Data lifecycle is tied to the table

***

### What Happens on DROP?

```sql
DROP TABLE employees;
```

✅ **Both table metadata AND data files are deleted**

***

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

***

### When to Use Internal Tables

✔ Temporary data  
✔ ETL intermediate results  
✔ Hive-only pipelines  
✔ Data you don’t need after table deletion

***

## 2️⃣ External Table

### Definition

An **external table** is where **Hive manages only metadata**,  
while the **data is stored and managed externally** (outside Hive).

***

### Key Characteristics

*   Data stored at a **custom location**
*   Hive does **NOT own** the data
*   Metadata lifecycle ≠ data lifecycle

***

### What Happens on DROP?

```sql
DROP TABLE ext_employees;
```

✅ **Only metadata is deleted**  
✅ **Data remains intact in HDFS**

***

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

***

### When to Use External Tables

✔ Shared data across tools (Spark, Impala, Presto)  
✔ Existing HDFS data  
✔ Production data  
✔ Data you must not accidentally delete

***

## 3️⃣ Key Differences (Side‑by‑Side)

| Feature               | Internal Table      | External Table      |
| --------------------- | ------------------- | ------------------- |
| Data ownership        | Hive owns data      | User owns data      |
| Metadata              | Managed by Hive     | Managed by Hive     |
| Data deletion on DROP | ✅ Yes               | ❌ No                |
| Default location      | Hive warehouse      | Custom location     |
| Data safety           | Lower               | Higher              |
| Use case              | Temporary / staging | Shared / production |

***

## 4️⃣ TRUNCATE Behavior

```sql
TRUNCATE TABLE table_name;
```

| Table Type | Result                           |
| ---------- | -------------------------------- |
| Internal   | ✅ Data deleted                   |
| External   | ❌ Not allowed (metadata remains) |

***

## 5️⃣ Conversion Between Tables

### Internal → External

```sql
ALTER TABLE employees SET TBLPROPERTIES ('EXTERNAL'='TRUE');
```

### External → Internal

```sql
ALTER TABLE ext_employees SET TBLPROPERTIES ('EXTERNAL'='FALSE');
```

⚠️ Use carefully—data ownership changes.

#  How does schema evolution handled in Hive

Hive handles **schema evolution** by allowing you to **add, change, or remove columns** in tables without breaking existing data.  
How it works depends on **file format**, **table type**, and **SERDE**.

Below is the clear, interview‑ready explanation.

***

# 1. What is Schema Evolution in Hive?

Schema evolution = ability to **change table structure** while keeping existing data intact.

Typical operations:

*   Add new columns
*   Change column order
*   Rename columns
*   Change data types (with limits)

***

# 2. How Hive Handles Schema Evolution

## A. For Text/CSV/TSV Tables (Row‑based formats)

Hive is **flexible and forgiving**.

### Adding columns

```sql
ALTER TABLE emp ADD COLUMNS (salary INT);
```

Behavior:

*   Old data files do **not** contain the new column → Hive returns `NULL` for those files.
*   New data can include the new column.

### Dropping/reordering columns

Hive does **not** enforce column order; it matches columns by position.

Drawback:

*   Schema mismatches may produce shifted/wrong data.

***

# 3. Schema Evolution with Columnar Formats (Parquet/ORC)

These formats support **true schema evolution** where metadata is stored in the file.

## A. ORC

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

***

## B. Parquet

Supports:

*   Add optional columns
*   Remove optional columns
*   Rename (logical rename; physical name remains)
*   Type widening (INT → BIGINT)

Parquet stores schema in the file header so each file is self‑describing.

***

# 4. Internal vs External Tables

## Internal Tables

*   Schema stored in Hive Metastore
*   Data stored in ORC/Parquet/text
*   Schema evolution applies uniformly

## External Tables

*   Hive updates only metadata
*   Data files remain untouched
*   Parquet/ORC evolution works at the **file level**, so evolution must be compatible with file schemas

***

# 5. Partitioned Tables and Schema Evolution

When schema changes:

*   **New partitions** use the new schema
*   **Old partitions** keep old schema

Hive handles this by:

*   Reading partition schema → merging with table schema
*   Missing fields → returned as NULL

***

# 6. Type Evolution Rules

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

***

# 7. Practical Example

### Step 1: Original Table

```sql
CREATE TABLE users (
  id INT,
  name STRING
)
STORED AS ORC;
```

### Step 2: Schema Change

```sql
ALTER TABLE users ADD COLUMNS (email STRING);
```

### Reading Old Data

| id | name | email |
| -- | ---- | ----- |
| 1  | John | NULL  |

Hive returns `NULL` for the new column because old ORC files don’t have that field.

***

# 8. How Schema Evolution Works Internally

1.  Hive metastore stores the *latest* schema.
2.  ORC/Parquet store *their own schema* in each file.
3.  During read:
    *   Hive compares metastore schema vs file schema.
    *   If columns missing → fill with NULL.
    *   If types differ → apply compatible conversion.
    *   If incompatible → throw error or fail silently (depending on format).





