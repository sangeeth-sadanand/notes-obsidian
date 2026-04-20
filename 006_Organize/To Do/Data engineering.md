
## Spark
### Architecture
- [ ] What problems do spark solve compared to Hadoop map reduce?
- [ ] How does spark execution flow differ from map-reduce execution how?
- [ ] How does spark context act as entry point?
- [ ] Why spark is called unified execution engine?
- [ ] What is role of driver, cluster manager and executor manager
- [ ] What are main modules of spark eco-system?
### Data abstraction
- [ ] What is RDD?
- [ ] 40 How do spark ensures fault tolerance in RDD?
- [ ] What transformation and actions can be applied on RDD?
- [ ] Difference between wide and narrow transformations
- [ ] How does RDD compared to dataframes and dataset?
- [ ]  Difference between reduce and reduceBykey
- [ ] 35  000_+/060207 Difference between reduceBykey and groupBykey
- [ ] How broadcast variable and accumulator are used to share variable
- [ ] What is task, Jobs and stages in Spark
- [ ] What is role of data partitions
- [ ] What is difference between re-partition and coalesce?
- [ ] How and when to use cache in RDP

### SQL

#### Fundamentals
- [ ] 29 What are the fundamental principles of spark SQL?
- [ ] How does Spark SQL unify relational queries with distributed computing?
- [ ] How does spark SQL integrate with hive metastore?
- [ ] What is advantages of spark SQL over RDD?
#### Operations 

- [ ] 25 How to read data from structured data source
- [ ] What is difference between toDf and createDataFrame
- [ ] How can schema be inferred or explicitly defined in data frames
- [ ] Explain different modes in while reading file in spark
- [ ] What transformation operation can be performed on SQL
- [ ] 20 What are SQL action operations
- [ ] What are grouping aggregates
- [ ] How to perform windowing operation on SQL
- [ ] How to perform Joins on SQL
- [ ] What are common dataframe functions for column manipulations

#### Advance operator
- [ ] How do pivot operations reshape data frame
- [ ] Various methods to unpivot dataframes
- [ ] How do spark represent null values and how to handle null values
- [ ] What is user defined Functions UDF_How to use UDF in SQL
- [ ] What are Pandas UDF_How to use it

#### Optimization
- [ ] 10 What are the levels of optimization in spark code level and cluster level
- [ ] 9 What is thin and thick executors
- [ ] 8 Give an overview of spark memory split
- [ ] 7 How caching optimizes spark operation and how it is applied on Spark SQL
- [ ] 6 Memory tuning parameters
- [ ] 5 Describe different types of Join technique
- [ ] 4 How bucketing and partitioning is used to optimize joins
- [ ] 3 What is AQE?
- [ ] 2 What is catalyst optimizer
- [ ] 1 What is data skew? How does salting help in optimizing data skewing
