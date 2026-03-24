# YARN

- Hadoop **YARN** (Yet Another Resource Negotiator) is a centralized resource management module within the Hadoop ecosystem designed to schedule tasks and allocate cluster resources. 
- Introduced in Hadoop version 2 (v2), it replaced the older, more inflexible JobTracker and TaskTracker architecture to better manage a pool of resources for diverse applications.

## Core Architecture Components

YARN operates through a master-slave architecture with the following primary components:

- **ResourceManager (Master):** The global authority that manages and distributes computational resources (CPU and memory) across the entire cluster.
- **NodeManager (Slave):** An agent running on each individual machine in the cluster that manages the storage and execution of that specific node and reports status to the ResourceManager.
- **ApplicationMaster:** A specific process spawned for every submitted job (e.g., a Spark or MapReduce job) that coordinates the execution of that specific application’s tasks and negotiates resources with the ResourceManager.
- **Containers:** Logical units of resources (Unix processes) that represent specific amounts of CPU and memory where individual tasks are executed.

## Operational Mechanism

When a cluster is launched, each node declares its resource capacities to the ResourceManager.

1. **Job Submission:** A client submits an application, and the ResourceManager associates it with a NodeManager to start an **ApplicationMaster** in a container.
2. **Resource Negotiation:** The ApplicationMaster requests the necessary containers from the ResourceManager.
3. **Task Execution:** Once allocated, the ApplicationMaster launches tasks within those containers on various worker nodes.

### Capabilities and Limitations

- **Multi-Framework Support:** Unlike its predecessor, YARN is a universal platform that can co-deploy multiple data processing frameworks, such as **Hadoop MapReduce**, **Apache Spark**, and **Apache Storm**, on the same hardware cluster.
- **Resource Utilization Issues:** A primary challenge in YARN is its "exclusive mode," where resources are exclusively allocated to a container until the task is complete. Because tasks often have fluctuating resource needs—such as a MapReduce reducer being idle during the "shuffle" phase while waiting for map outputs—this can lead to cluster underutilization.
- **Efficiency:** Despite high reservations (up to 80%), actual aggregated CPU utilization in production environments has been reported as low as 20-35% due to these idle occupied resources.