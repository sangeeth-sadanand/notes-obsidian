---
up:
  - "[[000_+/060302 Operations|060302 Operations]]"
down:
prev:
topic: false
question: How to read data from structured data source?
---
# How to read data from structured data source?


> [!Summary] Summary
> Contents11

Reading data from structured sources in Spark SQL is handled through a unified interface: the **`DataFrameReader`**. You access this through `spark.read`.

Spark uses a common pattern for almost all structured sources:

`spark.read.format("...").option("...", "...").load("path")`

---

## 1. Common Structured Formats

Spark has built-in support for the most common big data formats.

### **Parquet (Recommended)**

Parquet is the default format for Spark. It is columnar and stores the schema within the file itself, making it incredibly fast.

Python

```
df = spark.read.parquet("path/to/data.parquet")
```

### **CSV**

Since CSVs are plain text and don't inherently store data types, you usually need to tell Spark to look for a header and infer the types.

Python

```
df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("path/to/data.csv")
```

### **JSON**

Spark can read multi-line JSON or JSON Lines (newline-delimited).

Python

```
df = spark.read.json("path/to/data.json")
```

---

## 2. Reading from Databases (JDBC)

To read from relational databases like MySQL, PostgreSQL, or Oracle, you use the JDBC connector. You will need the specific database driver `.jar` file in your Spark classpath.

Python

```
df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:mysql://localhost:3306/my_database") \
    .option("dbtable", "employees") \
    .option("user", "admin") \
    .option("password", "password123") \
    .load()
```

---

## 3. Reading from Hive Tables

If you have integrated Spark with the Hive Metastore, reading a table is as simple as referencing its name. Spark handles the location and format details automatically.

Python

```
df = spark.read.table("sales_db.orders")
# Or via SQL directly
df = spark.sql("SELECT * FROM sales_db.orders")
```

---

## 4. Fundamental Reading Concepts

### **Schema Inference vs. User-Defined Schema**

While `inferSchema` is convenient for ad-hoc analysis, it requires Spark to scan the data twice (once to figure out types, once to read). For production pipelines, it is best practice to define the schema manually.

Python

```
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

---

| **Source**   | **Best for...**      | **Performance Note**                        |
| ------------ | -------------------- | ------------------------------------------- |
| **Parquet**  | Analytical workloads | Excellent (Columnar + Compression)          |
| **Avro**     | Row-based/Evolution  | Good for heavy write workloads              |
| **JDBC**     | Relational DBs       | Medium (Dependent on DB connection/network) |
| **CSV/JSON** | Interoperability     | Slowest (Requires parsing text)             |
|              |                      |                                             |

Are you connecting to a specific cloud storage like S3 or ADLS, or are you working with local files on your machine?