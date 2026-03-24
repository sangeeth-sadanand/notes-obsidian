# Core concepts

## What is a DataFrame in Spark
- A Spark DataFrame is a distributed collection of data organized into rows and columns, similar to a relational database table or an Excel spreadsheet, but optimized for big data processing across clusters.
- It provides a high-level API for structured data, built on top of RDDs, and integrates tightly with Spark SQL for querying.
### Key Characteristics of Spark DataFrames

- **Tabular structure**: Data is stored in a 2D format with rows and named columns.
- **Schema-aware**: Each column has a defined data type, allowing Spark to optimize queries.
- **Language support**: Available in **Python (PySpark)**, **Scala**, **Java**, and **R**.
- **Distributed processing**: DataFrames are partitioned across nodes in a cluster, enabling parallel computation.
- **Integration with SQL**: You can query DataFrames using SQL syntax via `spark.sql()` or by creating temporary views.
### Comparison: RDD vs DataFrame vs Dataset

|Feature|RDD|DataFrame|Dataset|
|---|---|---|---|
|**Structure**|Unstructured (no schema)|Structured (rows + columns)|Structured (rows + columns)|
|**Type Safety**|No|No (runtime checks only)|Yes (compile-time checks)|
|**Ease of Use**|Low (manual coding)|High (SQL-like operations)|Medium (typed API, more verbose)|
|**Performance**|Lower (no optimizations)|Higher (Catalyst optimizer)|Higher (Catalyst + type safety)|

### Why Use DataFrames?

- **Performance**: Uses Spark’s Catalyst optimizer and Tungsten execution engine for efficient query planning and execution.
- **Ease of use**: High-level operations like `select`, `filter`, `groupBy`, and `join` are intuitive.
- **Interoperability**: Can seamlessly switch between SQL queries and DataFrame API.
- **Scalability**: Handles massive datasets across distributed clusters.

```python
from pyspark.sql import SparkSession

# Initialize SparkSession
spark = SparkSession.builder.appName("DataFrameExample").getOrCreate()

# Create DataFrame from a list of tuples
data = [("Alice", 25), ("Bob", 30), ("Charlie", 35)]
columns = ["Name", "Age"]

df = spark.createDataFrame(data, columns)

# Show DataFrame
df.show()

# SQL-like operations
df.filter(df.Age > 28).select("Name").show()
```


## Schema definition and inference

- In Spark, a schema defines the structure of a DataFrame (column names, types, and nullability), and it can either be explicitly specified or automatically inferred using `inferSchema`. 
- Explicit schemas give you control and reliability, while inference is convenient but may misinterpret data types.**

### Schema Definition

- Schemas are defined using **`StructType`** and **`StructField`** objects in PySpark or Scala. 
- This ensures Spark knows exactly how to interpret each column.

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

schema = StructType([
    StructField("Name", StringType(), True),
    StructField("Age", IntegerType(), True),
    StructField("City", StringType(), True)
])

data = [("Alice", 25, "Mumbai"), ("Bob", 30, "Delhi")]
df = spark.createDataFrame(data, schema)
df.printSchema()
```

**Output:**

```
root
 |-- Name: string (nullable = true)
 |-- Age: integer (nullable = true)
 |-- City: string (nullable = true)
```

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("StringSchemaExample").getOrCreate()

# Sample data
data = [("Alice", 25, 50000.0), ("Bob", 30, 60000.5)]

# Define schema using string
schema = "Name STRING, Age INT, Salary DOUBLE"

# Create DataFrame
df = spark.createDataFrame(data, schema=schema)

df.printSchema()
df.show()

```
### Schema Inference

When reading external files (CSV, JSON, Parquet), Spark can **infer the schema automatically** if you set `inferSchema=True`.

- By default, Spark treats all columns as **strings** if no schema is provided.
- With `inferSchema`, Spark scans the data and assigns appropriate types (e.g., integers, doubles, booleans).


```python
df = spark.read.option("header", True).option("inferSchema", True).csv("people.csv")
df.printSchema()
```

If `people.csv` contains:

```
Name,Age,Salary
Alice,25,50000
Bob,30,60000
```

**Output:**

```
root
 |-- Name: string (nullable = true)
 |-- Age: integer (nullable = true)
 |-- Salary: integer (nullable = true)
```

### Comparison: Definition vs Inference

|Aspect|Explicit Schema Definition|Schema Inference|
|---|---|---|
|**Control**|Full control over column types|Relies on Spark’s guess|
|**Reliability**|Prevents misinterpretation|May misread mixed data|
|**Ease of Use**|Requires manual setup|Quick and automatic|
|**Performance**|Faster (no scanning)|Slower (scans data)|
|**Best Use Case**|Production pipelines|Data exploration|
## Execution Hierarchy in Spark

- **Driver**
        - The single process that runs your Spark application.
    - Responsible for creating the DAG, scheduling jobs, and coordinating executors.
    - There is always **one driver per Spark application**.
- **Job**
    - Triggered by an **action** (`count`, `collect`, `show`, `write`).
    - Each action = one job.
    - Example: calling `df.count()` and `df.show()` produces **two jobs**.
- **Stage**
    - A job is split into **stages** based on shuffle boundaries.
    - **Narrow transformations** (map, filter, select) stay in the same stage.
    - **Wide transformations** (groupBy, join, reduceByKey) require shuffles → new stage.
    - A job has at least **one stage**, but complex queries can have many.
- **Task**
    - The smallest unit of execution.
    - Each stage is divided into tasks, one per **data partition**.
    - Example: if a stage processes 200 partitions, Spark creates **200 tasks**.

### Relationship Overview

| Component  | Count                            | Depends On                                |
| ---------- | -------------------------------- | ----------------------------------------- |
| **Driver** | Always 1                         | One per Spark application                 |
| **Job**    | = Number of actions              | Each action triggers a job                |
| **Stage**  | ≥ 1 per job                      | Shuffle boundaries (wide transformations) |
| **Task**   | = Number of partitions per stage | Parallelism determined by partitions      |

### Example Walkthrough

Suppose you run:

```python
df = spark.read.csv("data.csv", header=True, inferSchema=True)
result = df.filter(df.age > 30).groupBy("city").count()
result.show()
```

- **Driver**: 1 (coordinates everything).
- **Job**: 1 (triggered by `show()`).
- **Stages**: 2
    - Stage 1: Read + filter (narrow transformations).
    - Stage 2: groupBy + count (shuffle boundary).
- **Tasks**: Equal to number of partitions (e.g., if CSV splits into 100 partitions → 100 tasks per stage).
### Key Considerations

- **Driver bottleneck**: If the driver is overloaded (too many jobs or large collect operations), the whole application slows down.
- **Stages matter for performance**: More shuffle boundaries = more stages = higher overhead.
- **Tasks scale with partitions**: Increasing partitions increases parallelism but also overhead.
### Best Practices

- Keep the **driver lightweight** (avoid large `collect()` calls).
- Minimize **wide transformations** to reduce shuffle stages.
- Tune **partition count** for optimal task parallelism (not too few, not too many).
- Use `df.explain()` or Spark UI to inspect jobs, stages, and tasks.
