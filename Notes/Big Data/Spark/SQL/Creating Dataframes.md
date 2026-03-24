# Creating Dataframes

## From Collections (lists, dicts, etc.)

Useful for small datasets or testing.

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("DFExamples").getOrCreate()

# From list of tuples
data = [("Alice", 25), ("Bob", 30)]
columns = ["Name", "Age"]
df = spark.createDataFrame(data, columns)
df.show()

```

## From Files (CSV, JSON, Parquet, ORC, Avro)

Spark can read structured/semi-structured files directly.

```python
# CSV
df_csv = spark.read.option("header", True).option("inferSchema", True).csv("people.csv")

# JSON
df_json = spark.read.json("people.json")

# Parquet
df_parquet = spark.read.parquet("people.parquet")

# ORC
df_orc = spark.read.orc("people.orc")

# Avro (requires spark-avro package)
df_avro = spark.read.format("avro").load("people.avro")
```

## From External Databases (JDBC)

Connect to relational databases like MySQL, PostgreSQL, Oracle, etc.

```python
df_jdbc = spark.read.format("jdbc").options(
    url="jdbc:mysql://localhost:3306/testdb",
    driver="com.mysql.cj.jdbc.Driver",
    dbtable="employees",
    user="root",
    password="password"
).load()
```

---

## Programmatic Schema Definition (`StructType` and `StructField`)

Explicit schema ensures correct data types and avoids inference overhead.

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

schema = StructType([
    StructField("Name", StringType(), True),
    StructField("Age", IntegerType(), True),
    StructField("City", StringType(), True)
])

data = [("Alice", 25, "Mumbai"), ("Bob", 30, "Delhi")]
df_schema = spark.createDataFrame(data, schema)
df_schema.printSchema()
df_schema.show()
```


## RDD to DataFrame with Inferred Schema

If your RDD is a collection of tuples, you can directly convert it by providing column names.

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("RDDtoDF").getOrCreate()

# Create RDD
rdd = spark.sparkContext.parallelize([("Alice", 25), ("Bob", 30)])

# Convert to DataFrame with column names
df = rdd.toDF(["Name", "Age"])
df.show()
```

**Output:**

```
+-----+---+
| Name|Age|
+-----+---+
|Alice| 25|
|  Bob| 30|
+-----+---+
```

### RDD to DataFrame with Explicit Schema

For more control, define a schema using `StructType` and `StructField`.

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

# Define schema
schema = StructType([
    StructField("Name", StringType(), True),
    StructField("Age", IntegerType(), True)
])

# Apply schema to RDD
df_schema = spark.createDataFrame(rdd, schema)
df_schema.printSchema()
df_schema.show()
```

### RDD of Row Objects

You can also convert an RDD of `Row` objects into a DataFrame.

```python
from pyspark.sql import Row

# Create RDD of Row objects
rdd_row = spark.sparkContext.parallelize([
    Row(Name="Alice", Age=25),
    Row(Name="Bob", Age=30)
])

# Convert to DataFrame
df_row = spark.createDataFrame(rdd_row)
df_row.show()
```

#TODO Read modes, Nested schema