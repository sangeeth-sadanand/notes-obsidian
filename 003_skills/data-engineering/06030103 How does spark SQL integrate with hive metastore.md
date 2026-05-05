---
up:
  - "[[003_skills/data-engineering/060301 Fundamentals|060301 Fundamentals]]"
down:
prev:
topic: false
question: How does spark SQL integrate with hive metastore?
---
# How does spark SQL integrate with hive metastore?


> [!Summary] Summary
> - Hire metastore stores the table schema, partitions and locations 
> - Spark SQL gets the catalog information from Hive External Catalog 
> - Spark acts as hive client to communicate with metastore 
> - Spark only uses meta data of hive 
> - Spark uses its optimized native reader to read data. 
> - if data is not in spark supported file format the it fall backs to Hive SerDe to read records 
>
> 
> |**Feature**|**Without Hive Metastore**|**With Hive Metastore**|
> |---|---|---|
> |**Table Persistence**|Tables disappear after the Spark session ends.|Tables are permanent and accessible by other users.|
> |**Schema Management**|You must define the schema every time you read.|Schema is stored in the Metastore; just use `spark.table("name")`.|
> |**Interoperability**|Limited to Spark.|Works with Hive, Impala, Presto, and BI tools.|
> |**Partition Discovery**|Manually point to subfolders.|Spark automatically knows about partitions via HMS.|
> 

- Spark SQL's integration with the **Hive Metastore (HMS)** is a cornerstone of modern data lakehouse architectures. 
- It allows Spark to act as a powerful, distributed compute engine while using Hive as the central "librarian" for data definitions.

## 1. The Role of the Metastore
The Hive Metastore is a central repository that stores **metadata**—the data about your data. It doesn't store the actual files (which live in S3, HDFS, or ADLS), but it tracks:
- **Table Schemas:** Column names and data types.
- **Partitions:** Where specific subsets of data are located.
- **Storage Location:** The physical path to the underlying files.

## 2. Shared Metadata via Catalog
When Spark SQL integrates with HMS, it uses the **HiveExternalCatalog**. This allows for a "write once, read anywhere" workflow:
- **Persistent Tables:** If you create a table in Hive, Spark SQL can immediately query it.
- **Cross-Engine Compatibility:** A table created by a Spark job can be queried by Presto, Trino, or Hive itself, because they all reference the same HMS.
- **Centralized Governance:** Security and access policies defined at the Metastore level can be enforced across different compute engines.

## 3. How the Integration Works
The integration happens through a client-side configuration. Spark acts as a **Hive Client** to communicate with the Metastore service.
### Configuration (`hive-site.xml`)
To connect, Spark needs access to the `hive-site.xml` file, which typically contains the `hive.metastore.uris` (the address of the HMS). In your Spark code, you enable this by calling `.enableHiveSupport()` when building the SparkSession:
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("HiveIntegration") \
    .config("spark.sql.warehouse.dir", "/user/hive/warehouse") \
    .enableHiveSupport() \
    .getOrCreate()
```

## 4. Native Spark Execution vs. Hive SerDe
Even though Spark uses Hive for metadata, it doesn't necessarily use Hive for **execution**:
- **Metadata Only:** Spark asks HMS: _"Where is the 'orders' table?"_ HMS responds: _"It's at s3://my-bucket/orders and it's in Parquet format."_
- **Native Execution:** Once Spark has the location and schema, it uses its own highly optimized **native readers** (like the vectorized Parquet reader) to pull data directly from storage, bypassing Hive’s slower execution engine (MapReduce or Tez).
- **SerDe Compatibility:** If the data is in a format Spark doesn't natively support, it can fall back to using Hive's **SerDe** (Serializer/Deserializer) to read the records.

