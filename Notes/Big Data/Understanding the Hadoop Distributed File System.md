# Understanding the Hadoop Distributed File System (HDFS).

## Core Architecture

- **Master/Slave Model:** HDFS operates with a single **NameNode** (master) and multiple **DataNodes** (slaves).
- **NameNode (Master):** This node manages the file system namespace, maintains metadata (such as file-to-block mappings and access permissions), and coordinates cluster-level operations like opening or renaming files.
- **DataNodes (Slaves):** These nodes manage the actual physical storage on the machines they run on, serving read and write requests from clients and performing block creation and replication under the NameNode's instruction.
- **Metadata Persistence:** The NameNode uses an **EditLog** to record every change to the metadata and an **FsImage** to store a snapshot of the entire namespace.
- **Secondary name node** is not a backup for the NameNode. Instead, it acts as a helper process that periodically merges the NameNode’s edit logs with the filesystem image (FsImage) to keep metadata manageable and prevent the NameNode from being overloaded.

### Data Organization and Block Splitting

- **Data Blocks:** HDFS splits large files into smaller, manageable chunks called "blocks".
- **Default Block Size:** The default block size is typically 128 MB, though this is configurable.

### Fault Tolerance and Reliability

- **Data Replication:** By default, HDFS maintains three copies (replicas) of every data block across different nodes to prevent data loss during hardware failure.
- **Rack Awareness:** The NameNode uses a "rack awareness" policy to ensure replicas are placed on different physical racks, protecting data even if an entire rack switch fails.
- **Heartbeats:** DataNodes send periodic "heartbeat" signals to the NameNode; if a heartbeat is missed, the NameNode marks the DataNode as dead and initiates re-replication of its blocks to healthy nodes.
- **Integrity Checks:** HDFS uses checksums to verify data integrity; if a client detects a corrupted block, it fetches a healthy replica from another node.
### Advantages and Limitations

- **Advantages:**
    - **Fault Tolerance:** Automatically heals itself through replication.
    - **Scalability:** Can scale horizontally to thousands of nodes and petabytes of data.
    - **Cost-Effectiveness:** Designed to run on low-cost commodity hardware.
- **Limitations:**
    - **Small File Inefficiency:** Storing millions of tiny files can overwhelm the NameNode's memory.
    - **High Latency:** It is not optimized for real-time applications requiring low-latency data access.

## The Role of the Master (NameNode) and Slaves (DataNodes).

### **Role of the Master (NameNode)**

The NameNode acts as the central coordinator and repository for all metadata within the cluster. Its primary responsibilities include:

- **Namespace Management:** It manages the traditional hierarchical file organization, allowing users to create, delete, move, or rename files and directories.
- **Metadata Storage:** It records vital information such as file names, permissions, and file-to-block mappings. This metadata is stored in memory for high performance.
- **Block Mapping:** It determines the mapping of specific data blocks to the DataNodes that will host them.
- **Access Control:** The NameNode regulates file access by clients, validating permissions before allowing read or write operations.
- **Persistence Mechanisms:** It uses two primary files to ensure metadata persists across restarts:
    - `EditLog`: A transaction log that records every change made to the namespace.
    - `FsImage`: A persistent system image containing the entire namespace and block mapping.
- **Replication Orchestration:** It makes all decisions regarding the replication of blocks, tracking which blocks need to be copied to maintain the specified replication factor.
- **Safemode:** Upon startup, the NameNode enters a special "Safemode" state where it does not replicate blocks until a configurable percentage of data blocks are reported as safe by DataNodes.

### **Role of the Slaves (DataNodes)**

DataNodes are responsible for the physical storage and serving of data within the cluster. Their key functions are:

- **Actual Data Storage:** They store HDFS data in separate files within their local OS file system, typically running on commodity Linux machines.
- **Serving Client Requests:** DataNodes interact directly with clients to perform read and write operations, which prevents the NameNode from becoming a bottleneck for data flow.
- **Block Operations:** Under instruction from the NameNode, they perform block creation, deletion, and replication tasks.
- **Status and Health Reporting:** They maintain constant communication with the NameNode through:
    - `Heartbeat`: Periodic messages (e.g., every 3 seconds) that signal the DataNode is functioning properly.
    - `Blockreport`: A comprehensive list of all data blocks currently residing on the DataNode.
- **Pipelining:** During write operations, DataNodes participate in a pipeline where they receive data in small 4 KB portions, write it locally, and simultaneously forward it to the next DataNode in the replication chain.

### **Key Interaction and Fault Tolerance**

- **Separation of Concerns:** Because user data never flows through the NameNode, the system can scale to handle tens of millions of files across thousands of nodes.
- **Handling Node Failure:** If the NameNode stops receiving heartbeats from a DataNode, it marks that node as dead and instructs other DataNodes to replicate the blocks that were lost to ensure data availability.
- **Configuration:** Key parameters governing this relationship, such as block size (default `128 MB` or `64 MB`), are often configured in `hdfs-site.xml` using settings like `dfs.block.size`.


## HDFS Operational Workflow: Write-Once-Read-Many access patterns.

### Core Concept: Write-Once-Read-Many (WORM)

- **Definition:** HDFS files are strictly "write-once" and have only one writer at any given time.
- **Stability:** Once a file is closed, it need not be changed, though there are future plans to support appending-writes.
- **Target Use Case:** This model is ideal for large-scale batch processing, such as MapReduce applications or web crawlers, which process massive datasets (gigabytes to petabytes) in parallel.

### HDFS Write Operation (The "Write-Once")

The write process involves the NameNode (metadata management) and DataNodes (actual storage) working in a pipelined fashion:

1. **Request:** The client establishes a connection with the Distributed FileSystem and requests the NameNode to create a new file.
2. **Verification:** The NameNode validates the client's permissions and checks if the file already exists.
3. **Metadata Allocation:** If successful, the NameNode records the new file in the `FsImage`/`EditLog` and provides the client with the IP addresses of the available DataNodes for the first block.
4. **Staging:** Data is initially cached in a temporary local file on the client side until it reaches the block size (default 128 MB or 64 MB).
5. **Replication Pipelining:**
    - The client flushes the block to the first DataNode in small 4 KB portions.
    - The first DataNode receives data, writes it to its local repository, and simultaneously forwards it to the second DataNode.
    - This continues until all replicas (defined by the `replication factor`) are successfully written.
6. **Completion:** Once all replicas are written, the client receives an acknowledgment and informs the NameNode that the file is closed.

### HDFS Read Operation (The "Read-Many")

The read workflow is optimized for streaming data at high bandwidth:

1. **Request:** The client contacts the NameNode to get the locations of the data blocks for a specific file.
2. **Locality Check:** The NameNode provides the addresses of DataNodes containing replicas, often prioritizing the nearest node via **rack awareness** to minimize network congestion.
3. **Streaming:** The client opens a connection to the nearest DataNode and reads the data blocks directly via an `FSDataInputStream`.
4. **Verification:** The client software implements checksum checking; if a block is found to be corrupted, the client attempts to read from a different replica.
5. **Closure:** After reading the required blocks, the input stream is closed.

### Operational Benefits of the WORM Model

- **Simple Coherency:** Because files do not change, there is no need for complex locking mechanisms across the distributed cluster.
- **High Throughput:** The model focuses on the speed at which data can be read sequentially (streaming access), making it highly effective for Big Data analytics.
- **Fault Tolerance:** Reliability is maintained through block replication; if a DataNode fails, the NameNode automatically triggers re-replication from surviving copies.

### Key Configuration Parameters

- `DFS.block.size`: Defines the size of each block (default is typically 128 MB).
- `dfs.replication`: Determines how many copies of each block are stored across the cluster.
- `bin/hadoop dfs -cat /path/file.txt`: A standard shell command used to read (many times) the content of a file once it has been written.

## Data Replication and Fault Tolerance strategies

### HDFS Data Replication and Rack Awareness

- **Block-Based Replication**: HDFS stores files as sequences of blocks (default 128 MB), which are replicated across multiple DataNodes to ensure reliability and fault tolerance.
- **Configurable Replication Factor**: The number of replicas is managed by the NameNode and can be configured per file using the `dfs.replication` parameter.
- **Rack Awareness Placement Policy**: To optimize performance and availability, HDFS follows a specific placement rule:
    - The first replica is placed on a node in the local rack.
    - The second replica is placed on a node in a different (remote) rack.
    - The third replica is placed on a different node within that same remote rack.
- **Reliability Benefits**: This strategy ensures that data remains accessible even if an entire rack fails or experiences a network switch issue.
- **Read Locality**: To minimize latency and global bandwidth consumption, HDFS satisfies read requests from the replica physically closest to the reader, prioritizing nodes on the same rack.

### **HDFS Fault Tolerance Strategies**

- **Automatic Re-replication**: DataNodes send periodic `Heartbeat` and `Blockreport` messages to the NameNode. If a DataNode fails (marked dead after missing heartbeats) or a replica becomes corrupted, the NameNode automatically triggers the replication of those blocks to other healthy nodes to maintain the required factor.
- **Data Integrity**: HDFS implements checksum checking; when a client retrieves a block, it verifies it against a stored checksum file. If corruption is detected (e.g., due to disk or network faults), the client retrieves that block from a different replica.
- **Safemode**: On startup, the NameNode enters a read-only state where replication is paused until a configurable percentage of data blocks are safely accounted for.
- **High Availability (HA)**: To eliminate the NameNode as a single point of failure, HDFS supports configurations with **Active/Standby NameNodes** and **Shared Edit Logs**.
- **Erasure Coding**: For cold or archival data, HDFS provides **Erasure Coding** as a storage-efficient alternative to traditional replication.
