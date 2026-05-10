---
up:
  - "[[003_skills/data-engineering/06 Spark/0603 SQL/060302 Operations/060302 Operations|060302 Operations]]"
down:
prev:
topic: false
question: How to read data from structured data source?
---
# How to read data from structured data source?


> [!Summary] Summary
> - Spark SQL handles reading of data using a unified interface `spark.read` 
> - We can use `.parquet()`, `.csv()`, `.json()` to read data from files and  `.table()`  to read from tables.
> - We can also use `.format()` to indicate the file format to read and `.options()` to apply additional option and `.load()` to give location of the file or table 


- Reading data from structured sources in Spark SQL is handled through a unified interface: the **`DataFrameReader`**. You access this through `spark.read`.
- Spark uses a common pattern for almost all structured sources:

```python
spark.read.format("...").option("...", "...").load("path")`
```

## 1. Common Structured Formats
Spark has built-in support for the most common big data formats.

### **Parquet (Recommended)**
Parquet is the default format for Spark. It is columnar and stores the schema within the file itself, making it incredibly fast.

```python
df = spark.read.parquet("path/to/data.parquet")
```

### **CSV**
Since CSVs are plain text and don't inherently store data types, you usually need to tell Spark to look for a header and infer the types.
```python
df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("path/to/data.csv")
```

### **JSON**
Spark can read multi-line JSON or JSON Lines (newline-delimited).
```python
df = spark.read.json("path/to/data.json")
```

## 2. Reading from Databases (JDBC)

To read from relational databases like MySQL, PostgreSQL, or Oracle, you use the JDBC connector. You will need the specific database driver `.jar` file in your Spark classpath.

```python
df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:mysql://localhost:3306/my_database") \
    .option("dbtable", "employees") \
    .option("user", "admin") \
    .option("password", "password123") \
    .load()
```

## 3. Reading from Hive Tables
If you have integrated Spark with the Hive Metastore, reading a table is as simple as referencing its name. Spark handles the location and format details automatically.
```python
df = spark.read.table("sales_db.orders")
# Or via SQL directly
df = spark.sql("SELECT * FROM sales_db.orders")
```

## 4. Fundamental Reading Concepts

### **Schema Inference vs. User-Defined Schema**
While `inferSchema` is convenient for ad-hoc analysis, it requires Spark to scan the data twice (once to figure out types, once to read). For production pipelines, it is best practice to define the schema manually.

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

schema = StructType([
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True)
])

df = spark.read.schema(schema).json("path/to/data.json")
```

### **Partition Discovery**
If your data is stored in a partitioned directory structure (e.g., `path/year=2023/month=01/`), Spark automatically recognizes these as columns. This is a key feature of distributed computing called **Partition Pruning**.

### **Lazy Evaluation**
Remember that reading data is "lazy." When you run `df = spark.read...`, Spark only checks the connection and schema. It doesn't actually pull the records into memory until you perform an **Action** (like `.show()`, `.count()`, or `.save()`).

| **Source**   | **Best for...**      | **Performance Note**                        |
| ------------ | -------------------- | ------------------------------------------- |
| **Parquet**  | Analytical workloads | Excellent (Columnar + Compression)          |
| **Avro**     | Row-based/Evolution  | Good for heavy write workloads              |
| **JDBC**     | Relational DBs       | Medium (Dependent on DB connection/network) |
| **CSV/JSON** | Interoperability     | Slowest (Requires parsing text)             |
|              |                      |                                             |
