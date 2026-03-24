# Evolution from traditional centralized systems to distributed architectures.

The evolution of big data has shifted from traditional centralized storage to highly efficient distributed processing architectures. 
### **The Era of Traditional Centralized Systems**

- **Pre-2000s Limitations:** Organizations relied on single standalone machines. These faced scalability bottlenecks because it was difficult to obtain more RAM and CPU power for a single machine to handle parallel jobs efficiently.
- **The Problem:** Exponential growth in data volume (Big Data) became a major problem as it exceeded the capacity of traditional databases and processing systems.

### **The Rise of the Hadoop Ecosystem (2000s - 2013)**

- **Distributed Storage (HDFS):** Hadoop was developed to solve the storage problem using the Hadoop Distributed File System (HDFS).
    - **Mechanism:** It splits large files into blocks (default 128 MB) and distributes them across a cluster of commodity machines.
    - **Distributed Processing (MapReduce):** Hadoop introduced MapReduce, a two-phase (Mapper and Reducer) linear dataflow structure for parallel processing.
- **2013:** Hadoop adoption became widespread, with over 50% of Fortune 50 companies using the framework.

### **The Spark Revolution (2009 - 2015)**

- **2009:** Apache Spark was started by Matei Zaharia at UC Berkeley's AMPLab to address MapReduce's reliance on slow disk I/O.
- **2010:** Spark was open-sourced under a BSD license.
- **2012:** Spark's core architecture—**Resilient Distributed Datasets (RDDs)**—was developed, enabling in-memory computation that is up to 100x faster than Hadoop for certain workloads.
- **2013:** The Spark project was donated to the Apache Software Foundation.
- **2014:** Spark became a Top-Level Apache Project.
    - **May 2014:** Spark 1.0 was released.
- **2015:** **Project Tungsten** was launched to bring Spark closer to "bare metal" efficiency by optimizing memory management and code generation.

### **Modern Distributed Architectures (2016 - Present)**

- **2016:** **Spark 2.0** was released, introducing the **Spark Session** as a unified entry point and encouraging the high-level Dataset/DataFrame APIs over low-level RDDs.
- **2018:** Spark 2.3 introduced **Spark on Kubernetes**, allowing big data jobs to run in containerized environments alongside online business applications.
- **2020:** **Spark 3.0** introduced the **Adaptive Query Execution (AQE)** framework, enabling the system to re-optimize query plans at runtime based on live data statistics.
- **2021:** Spark on Kubernetes reached General Availability (GA) with Spark 3.1.
- **2025 - 2026:**
    - **Delta Lake 3.0:** Introduced technologies like **Delta Universal Format (UniForm)** and **Liquid Clustering** to enable interoperability between different table formats (Iceberg, Hudi) and flexible data layout.
    - **Spark 4.x:** Current stable releases (e.g., 4.0.1 in Sept 2025) continue to refine unified analytics for batch, stream, and AI workloads.