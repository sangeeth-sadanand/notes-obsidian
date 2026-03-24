1. [[Notes/Big Data/Spark/SQL/Core concepts|Core concepts]]
2. [[Notes/Big Data/Spark/SQL/Creating Dataframes| Creating Dataframes]]
3. [[Notes/Big Data/Spark/SQL/Operations|Operations]]
4. 




---

## 🧩 Transformations vs Actions

- Transformations: select, filter, groupBy, join
- Actions: show, collect, count, save

---

## 📊 Advanced Features

- Pivot and unpivot operations
- Handling nulls (fillna, dropna, replace)
- User Defined Functions (UDFs) and Pandas UDFs
- SQL interoperability (`createOrReplaceTempView`)

---

## ⚡ Performance & Optimization

- Catalyst optimizer basics
- Tungsten execution engine
- Partitioning and bucketing
- Caching and persistence (`cache`, `persist`)
- Broadcast joins and `explain()`

---

## 📂 File Formats & Storage

- Reading/writing CSV, JSON, Parquet, ORC
- Partitioned data storage
- Schema evolution in Parquet
- Compression options

---

## 🛡️ Practical Considerations

- Error handling in DataFrames
- Logging and debugging with `explain()` and `printSchema()`
- Integration with MLlib and Spark SQL
- Best practices for production workflows
