---
up:
  - "[[003_skills/data-engineering/01_Introduction|01_Introduction]]"
down:
prev:
topic: false
question: What is difference between monolithic Vs distributed system?
---
# What is difference between monolithic Vs distributed system?


> [!Summary] Summary
> | **Feature**         | **Monolithic System**                | **Distributed System (Big Data)**      |
> | ------------------- | ------------------------------------ | -------------------------------------- |
> | **Scaling**         | **Vertical** (Scale-up)              | **Horizontal** (Scale-out)             |
> | **Cost**            | High (Proprietary high-end hardware) | Low to Medium (Commodity hardware)     |
> | **Complexity**      | Simple to develop and deploy         | Complex (requires networking and sync) |
> | **Reliability**     | Low (Single point of failure)        | High (Redundancy and replication)      |
> | **Data Throughput** | Limited by single-node I/O           | High (Parallel I/O across nodes)       |
> | **Example**         | Traditional SQL Database   | Apache Spark, Hadoop, NoSQL            |
> 

## Monolithic Systems
A **monolithic system** is a single-tier software application in which different components are combined into a single program from a single platform. In Big Data, this usually refers to a single, powerful machine (a mainframe or a high-end server) attempting to process an entire dataset.
- **Vertical Scaling:** To handle more data, you must buy a faster CPU or more RAM for that specific machine. This is expensive and eventually hits a hardware limit.
- **Centralized Storage:** All data resides on one local disk array.
- **Single Point of Failure:** If the server hardware fails, the entire processing pipeline stops.
- **Data Limit:** Monoliths struggle when the volume of data exceeds the memory and storage capacity of a single node (the "Big Data" threshold).

## Distributed Systems

A **distributed system** consists of multiple independent computers (nodes) that appear to the user as a single coherent system. This is the foundation of modern Big Data frameworks like Hadoop, Spark, and Cassandra.
- **Horizontal Scaling:** To handle more data, you simply add more "commodity" servers to the cluster. This is theoretically infinite.
- **Distributed Storage:** Data is partitioned across many disks (e.g., HDFS). If you have 100 nodes, you have the aggregate I/O speed of 100 disks.
- **Fault Tolerance:** If one node fails, the system redistributes the task to another node. The process continues without data loss.
- **Parallelism:** Tasks are broken into sub-tasks. Instead of one CPU processing 1 TB of data, 100 CPUs process 10 GB each simultaneously.
