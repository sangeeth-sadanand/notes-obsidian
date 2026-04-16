
## Processing layer
- [ ] What is the programming model of map-reduce?
- [ ] How does map phase differ from reduce phase?
- [ ] Advantages and disadvantages of map-reduce.
- [ ] What problem does YARN solve in hadoop?
- [ ] What is the use of resource manager and node manager?
## Hive
- [ ] What is hive?
- [ ] Difference between HQL and SQL
- [ ] What is role of meta store in hive?
- [ ] How does hive translate to MR and spark?
- [ ] Difference between Internal and external tables.
- [ ] How does schema evolution handled in Hive?
## Spark
### Architecture
- [ ] What problems do spark solve compared to hadoop map reduce?
- [ ] How does spark execution flow differ from map-reduce execution how?
- [ ] How does spark context act as entry point?
- [ ] Why spark is called unified execution engine?
- [ ] What is role of driver, cluster manager and executor manager
- [ ] What are main modules of spark eco-system?
### Data abstraction
- [ ] What is RDD?
- [ ] How do spark ensures fault tolerance in RDD?
- [ ] What transformation and actions can be applied on RDD?
- [ ] Difference between wide and narrow transformations
- [ ] How does RDD compared to dataframes and dataset?
- [ ] Difference between reduce and reduce By key
- [ ] Difference between reduce Bykey and group Bykey
- [ ] How broadcast variable and accumulator are used to share variable.
- [ ] What is task, Jobs and stages in Spark
- [ ] What is role of data partitions?
- [ ] What is difference between re-partition and coalesce?
- [ ] How and when to use cache in RDP?
### SQL
#### fundamentals
- [ ] What are the fundamental principles of spark SQL?
- [ ] How does Spark SQL unify relational queries with distributed computing?
- [ ] How does spark SQL integrate with hive metastore?
- [ ] What is advantages of spark SQL over RDD?
#### Operations
- [ ] How to read data from structured data source
- [ ] What is difference between to DFC) and create Data framel)
- [ ] How can schema be inferred or explicitly defined in data frames?
- [ ] Explain different modes in while reading file in spark
- [ ] What transformation operation can be performed on SQL?
- [ ] What are SQL action operations?
- [ ] What are grouping aggregates?
- [ ] How to perform windowing operation on SQL
- [ ] How to perform Joins on SQL
- [ ] What are common dataframe functions for column manipulations
#### Advance operator
- [ ] How do pivot operations reshape data frame?
- [ ] Various methods to unpirot dataframes
- [ ] How do spark represent null values and how to handle null values
- [ ] What is user defined Functions (UDF)? How to use UDF in SQL
- [ ] What are Pandas UDF? How to use it?
#### Optimization
- [ ] What are the levels of optimization in spark (code level and cluster level)
- [ ] What is thin and thick executors
- [ ] Give an overview of spark memory split.
- [ ] How caching optimises spark operation and how it is applied on Spark SQL
- [ ] Memory tuning parameters
- [ ] Describe different types of Join technique 7
- [ ] How bucketing and partitioning is used to optimise joins
- [ ] What is AQE?
- [ ] What is catalyst optimiser
- [ ] What is data skew? How does salting help in optimizing data skewing
