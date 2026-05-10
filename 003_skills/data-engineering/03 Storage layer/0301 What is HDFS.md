---
up:
  - "[[003_skills/data-engineering/03 Storage layer/03 Storage layer|03 Storage layer]]"
down:
prev:
topic: false
question: What is HDFS? What is role of data node and name node? How does HDFS stores and read files?
---
# What is HDFS?

What is role of data node and name node?
How does HDFS stores and read files?

> [!Summary] Summary
> 
> - HDFS is a file system designed to store data on multiple nodes
> - It split large files into small chunk called block (Generally 128 MB)
> - These block are stored across multiple machine with a replication factor (3 by default) to ensure fault tolerance
> - key features of HDFS - Fault tolerance, scalability High throughput, master-slave architecture
> - HDFS acts as master-slave architecture, name node acts as the master and data node as slave.
> - The name node store the metadata and know the files index. While the data nodes stores the actual data
> 
> ---
> Storing data
> - When a file is uploaded to HDFS then the following operation are performed
> 	- **Data splitting** - files are split into blocks, 
> 	- then it **contact name node** for permission and also ask for which data node should receive the block. 
> 	- To create a **replication** a replication pipeline is initiated. 
> 		- The client sends the block to one data node. 
> 		- That data node then copies it to second node which copies to third
> 	- Once data node copies the file data node send **acknowledgement** to name node
> ---
> Reading operations
> -	When the data is requested .
> -	The request is sent to name node for specific location
> -	The name node returns list of data node containing those blocks usually sorted by proximity. then the client reads the data directly from data node
> -	As data is read HDFs verifies checksum to ensure data is not corrupted.
> 
> 

- HDFS, or the **Hadoop Distributed File System**, is the primary storage system used by Apache Hadoop applications. 
- Think of it as a giant, virtual hard drive that spans across hundreds or thousands of ordinary servers, allowing you to store massive datasets (petabytes of data) that wouldn't fit on a single machine.
- It is designed to be **fault-tolerant** and optimized for **streaming data access**, meaning it’s built to handle hardware failures gracefully while reading large files very quickly.

## 1. The Architecture: Who’s in Charge?
HDFS operates on a **Master/Slave** architecture consisting of two main components:
- **NameNode (The Master):** This is the brain. It doesn't store the actual data bits; instead, it maintains the **metadata**. It knows which files exist, how they are split into blocks, and which specific servers those blocks are stored on.
- **DataNodes (The Slaves):** These are the workhorses. They store the actual data blocks on their local disks and perform read/write requests as told by the NameNode.
    
## 2. How HDFS Stores Files (The Write Process)
When you upload a file to HDFS, it doesn't just "plop" the file onto a disk. It goes through a specific sequence:
1. **Data Striping:** The file is split into large chunks called **Blocks** (default is usually 128MB or 256MB).
2. **Contacting NameNode:** The HDFS client asks the NameNode for permission to write and asks which DataNodes should receive the blocks.
3. **Replication:** To prevent data loss, HDFS usually makes **three copies** of every block.
4. **Pipeline Setup:** The client sends the block to the first DataNode. That DataNode then copies it to the second, which copies it to the third. This is called a **Replication Pipeline**.
5. **Acknowledgment:** Once all three nodes have the block, they send a "success" signal back to the NameNode.

## 3. How HDFS Reads Files (The Read Process)
Reading is designed to be fast by bringing the "computation to the data."
1. **Metadata Request:** The client asks the NameNode for the location of the blocks that make up a specific file.
2. **Proximity Search:** The NameNode returns a list of DataNodes containing those blocks, usually sorted by **proximity** (the node closest to the client in the network rack).
3. **Parallel Reading:** The client connects directly to the DataNodes and reads the blocks in parallel.
4. **Checksum Verification:** As the data is read, HDFS verifies a "checksum" to ensure the data hasn't been corrupted while sitting on the disk.

## Key Technical Specifications

| **Feature**               | **Description**                                                                                                                    |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **Fault Tolerance**       | If a DataNode crashes, the NameNode notices and automatically replicates the lost blocks to a new node.                            |
| **Data Locality**         | HDFS tries to run calculations on the physical node where the data lives to avoid moving huge files over the network.              |
| **Write Once, Read Many** | HDFS is optimized for files that are written once and then read many times. You cannot easily "edit" the middle of a file in HDFS. |
| **Block Size**            | Large blocks (e.g., 128MB) minimize the overhead of seeking a disk, which is vital for Big Data processing.                        |

