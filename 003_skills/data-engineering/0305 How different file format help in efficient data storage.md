---
up:
  - "[[003_skills/data-engineering/03 Storage layer|03 Storage layer]]"
down:
prev:
topic: false
question: How different file format help in efficient data storage?
---
# How different file format help in efficient data storage?


> [!Summary] Summary
> -	File system in a distributed system is very important. It helps in reduction in traffic Ex. Using a columnar format, we can send only the columns that are required.
> -	**Splitability**. Since the HDFS Store the data in block a file format that is easily splitable is very important.
> -	**Lower storage cost**. csv take more space as compare to parquet hence using parquet will help in lowering storage cost.
> 
> 
| **Format**   | **Type** | **Compression** | **Schema Evolution** | **Best Use Case**                   |
| ------------ | -------- | --------------- | -------------------- | ----------------------------------- |
| **CSV/JSON** | Text     | Poor            | Limited              | Quick debugging, small data.        |
| **Avro**     | Row      | Moderate        | **Excellent**        | Data Ingestion, Kafka, "Hot" data.  |
| **Parquet**  | Columnar | **High**        | Good                 | Spark, Analytics, "Cold" data.      |
| **ORC**      | Columnar | **Highest**     | Moderate             | Hive, Large-scale Data Warehousing. |



- In HDFS, the choice of file format is often more important than the hardware itself. 
- Because HDFS is designed for massive datasets, using the wrong format (like raw Text/CSV) can lead to "storage bloat" and painfully slow queries.
- Different formats optimize for two main things: **Storage Space** (compression) and **I/O Speed** (how fast the CPU can find the data it needs).

## 1. Row-Based vs. Columnar Storage
The biggest performance differentiator is how the data is physically laid out on the disk.
### Row-Based (e.g., Avro, CSV)
Data is stored row by row. To read one column, the system must read the entire row.
- **Pros:** Fast at writing data; great if you need to retrieve an entire record (like a single user profile).
- **Cons:** Terrible for analytical queries where you only need 2 columns out of 100.

### Columnar-Based (e.g., Parquet, ORC)
Data is stored column by column. All values for "Price" are stored together, then all values for "Date."
- **Pros:** Massive compression (similar data types compress better) and "Column Projection" (only reading the columns you need).
- **Cons:** Slower to write because the data must be buffered and rearranged.

## 2. Key File Formats in HDFS
### Apache Parquet (The Industry Standard)
Parquet is a columnar format designed for the entire Hadoop ecosystem (Spark, Hive, Impala).
- **Efficiency:** It uses **Dictionary Encoding** and **Run-Length Encoding** to shrink data. For example, if a column has the word "USA" 1 million times, Parquet stores "USA" once and a small marker for its positions.
- **Predicate Pushdown:** It allows the engine to skip entire blocks of data if they don't match your query filters (e.g., `WHERE year = 2026`).

### Apache ORC (Optimized Row Columnar)

Highly popular in the Hive community, ORC is similar to Parquet but often provides even better compression.
- **Efficiency:** It includes "indexes" within the file itself (min/max/sum) so the system can decide whether to skip a stripe of data without even opening it.
- **Best For:** Heavy Hive/Presto users and long-term data archiving.

### Apache Avro
A row-based format that stores the **Schema** (the data definition) inside the file itself.
- **Efficiency:** Because the schema is built-in, Avro files are "self-describing." If you change your table structure (Schema Evolution), Avro handles it gracefully without breaking old data.
- **Best For:** Real-time data streaming (Kafka) and "Write-heavy" landing zones.

### SequenceFiles & MapFiles
These are older, binary formats used primarily to solve the **"Small Files Problem"** in HDFS.
- **Efficiency:** They act as a container to wrap thousands of tiny files into one large binary block, making it much easier for the NameNode to manage metadata.

## 3. Why this matters for HDFS Performance
1. **Reduced Network Traffic:** Since columnar formats only read the columns required, less data is sent from the DataNodes to the NameNode/Client.
2. **Splittability:** Modern formats (Parquet/Avro/ORC) are designed to be "splittable." This means HDFS can break a single file into blocks and process them in parallel across 100 different servers.
3. **Lower Storage Costs:** Switching from CSV to Parquet can often reduce storage requirements by **up to 75%** due to efficient encoding.
