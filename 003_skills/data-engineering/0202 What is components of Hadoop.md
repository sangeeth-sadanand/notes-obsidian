---
up:
  - "[[003_skills/data-engineering/02 Hadoop|02 Hadoop]]"
down:
prev:
topic: false
question: What is components of Hadoop?
---
# What is components of Hadoop?


> [!Summary] Summary
>  There are basically three component of Hadoop system that enable distributed computing.
> 1. **HDFS** - for distributed storage
> 2. **MapReduce**-for processing the datal in distributed fashion
> 3. **YARN**- This component is responsible to manage resource on the host machine.

While people often use "Hadoop" to describe the whole ecosystem, the core framework actually consists of four primary components. 

## 1. HDFS (Hadoop Distributed File System)
This is the **Storage Layer**. HDFS is designed to store massive amounts of data across clusters of standard servers.
- **How it works:** It breaks large files into smaller pieces (blocks) and distributes them across different machines.
- **Key Feature:** **Replication**. It makes multiple copies of every block. If one server hardware fails, the data is still safe on another node.
## 2. MapReduce

This is the **Processing Layer**. It is the original programming model used to process vast amounts of data in parallel.
- **Map Phase:** Filters and sorts the data (e.g., counting words in a million documents).
- **Reduce Phase:** Aggregates the results from the Map phase into a final output.
- **The Logic:** Instead of moving data to the code, MapReduce moves the **code to the data**, which is much faster when dealing with petabytes of information.

## 3. YARN (Yet Another Resource Negotiator)
This is the **Management Layer**. Introduced in Hadoop 2.0, YARN acts as the operating system for the cluster.
- **Resource Management:** It tracks how much CPU and memory each node has available.
- **Scheduling:** It decides which jobs get priority and allocates the necessary "containers" (resources) to get the work done.
- **Flexibility:** It allows other engines (like Spark or Flink) to run on Hadoop alongside MapReduce.

## 4. Hadoop Common
This is the **Utility Layer**. It refers to the collection of common utilities and Java libraries that support the other three modules.
- It provides the necessary Java Archive (JAR) files and scripts needed to start Hadoop.
- It handles the basic system-level details like the FileSystem abstraction.

### How they work together:
1. **HDFS** holds the data.
2. **YARN** allocates the brainpower (CPU/RAM).
3. **MapReduce** (or Spark) performs the actual calculation.
4. **Hadoop Common** provides the underlying tools to make the communication possible.
