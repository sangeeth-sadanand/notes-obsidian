---
up:
  - "[[003_skills/data-engineering/06 Spark/0601 Architecture/0601 Architecture|0601 Architecture]]"
down:
prev:
topic: false
question: How does spark session act as entry point?
---
# How does spark Session act as entry point?


> [!Summary] Summary
> -	Spark session is unified entry point for spark 2+ version which encapsulated the three context Spark context - for RDD operation, SQL context- for Spark SQL and data frames, Hive Context for work with Hive tables
> -	Spark Session provides a single entry point that handles all these functionalities, making the code cleaner
> -	Through spark session spark functionality work in sync we can access data frames, execute SQL queries as well access underlying Spark context.
> -	It allows to configure all the run time configs directly. It propagates to respective context and cluster manager
> -	Spark session uses a builder pattern so that only one session is created across one application
> -	We can use following code snippet for spark session



- While the **SparkContext** was the original entry point, **SparkSession** (introduced in Spark 2.0) acts as a "unified" entry point. 
- It simplifies the developer experience by wrapping several distinct contexts into a single, cohesive interface.

## Why SparkSession replaced SparkContext

Before Spark 2.0, developers had to manage different "Contexts" depending on what they were doing:
- **SparkContext:** For core RDD operations.
- **SQLContext:** For Spark SQL and DataFrames.
- **HiveContext:** For working with Hive tables.

The **SparkSession** provides a single point of entry that handles all of these functionalities, making the code cleaner and reducing the overhead of managing multiple objects.

## Key Responsibilities of SparkSession

### 1. Unified API Access

Through a SparkSession, you can access all of Spark’s functional silos. You can create DataFrames, execute SQL queries, and access the underlying `SparkContext` for RDD-level operations if needed.

### 2. Configuration Management

It allows you to set runtime configurations (like executor memory or shuffle partitions) directly. These settings are then propagated to the underlying SparkContext and Cluster Manager.

### 3. Metadata Catalog
SparkSession provides access to the **Catalog**, which is a high-level API for managing metadata. You can use it to:
- List databases and tables.
- Check if a table exists.
- Cache tables in memory.

### 4. Builder Pattern Integration

SparkSession uses the **Builder Pattern**, which makes it very easy to instantiate. It ensures that if a session already exists, it retrieves the current one rather than trying to create a conflicting second session in the same JVM.

## How it looks in Code

In a modern PySpark application, this is typically the first thing you write:

```python
from pyspark.sql import SparkSession

# Creating the SparkSession
spark = SparkSession.builder \
    .appName("MyDataApp") \
    .config("spark.some.config.option", "some-value") \
    .getOrCreate()

# Accessing other contexts via the session
sc = spark.sparkContext  # Underlying SparkContext
sql_context = spark._wrapped  # Internal SQLContext
```


## Comparison: SparkContext vs. SparkSession

|**Feature**|**SparkContext**|**SparkSession**|
|---|---|---|
|**Era**|Spark 1.x (Legacy)|Spark 2.x / 3.x (Modern)|
|**Primary Data Unit**|RDDs|DataFrames and Datasets|
|**Scope**|Low-level cluster resource management|High-level API for SQL, Hive, and Streaming|
|**User Experience**|Required managing multiple objects|Single, unified interface|
