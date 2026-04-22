---
up:
  - "[[003_skills/data-engineering/0601 Architecture|0601 Architecture]]"
down:
prev:
topic: false
question: What problems do spark solve compared to Hadoop map reduce?
---
# What problems do spark solve compared to Hadoop map reduce?


> [!Summary] Summary
> -	Hadoop map reduce solved the big data problem of distributed computing but was not a speed demon
> -	It writes the data back to storage unit after every job which created a I/o bottleneck. 
> - On the other hand spark created one job with multiple stages and intermediate data in-memory
> -	Iterative algorithms which needed same data was handled in memory instead of reading it from disk every time
> -	Spark supported both batch and streaming process
> -	spark reduced the complexity for boiler plate code
> -	Spark provided all in one stack- Spark SQL, MLlib, graphx spark steaming



- While Hadoop MapReduce revolutionized big data processing by allowing us to crunch massive datasets on commodity hardware, it wasn't exactly a "speed demon." 
- Apache Spark was essentially built to fix the specific bottlenecks and frustrations that made MapReduce difficult to work with.

## 1. The "I/O Bottleneck" (Speed)

The biggest issue with MapReduce is its reliance on the disk. After every **Map** and **Reduce** step, the data must be written back to the physical hard drive (HDFS). This creates a massive amount of "disk I/O" overhead.

- **Spark's Solution:** Spark performs **In-Memory Processing**. It keeps data in RAM across various stages of a job. By minimizing the need to read/write to the disk constantly, Spark can run programs up to **100x faster** in memory than MapReduce.
    

## 2. Iterative Algorithms (Machine Learning)

MapReduce is built for "one-pass" jobs. If you have an algorithm that needs to look at the data 20 times (like most Machine Learning models), MapReduce has to restart a brand-new job for every single pass, loading the data from the disk every time.

- **Spark's Solution:** Because Spark keeps data in memory, it excels at **Iterative Processing**. You load the data once and run multiple passes over it instantly. This makes Spark the industry standard for large-scale ML.
    

## 3. High Latency (Real-Time vs. Batch)

MapReduce is strictly a **batch processing** engine. You give it a mountain of data, wait 20 minutes (or hours), and get an answer. It cannot handle data as it arrives.

- **Spark's Solution:** Spark provides a unified engine. It handles batch processing, but also includes **Spark Streaming**. This allows it to process "micro-batches" of data in near real-time (latency in seconds or milliseconds), solving the need for live analytics.
    

## 4. Complexity and "Boilerplate" Code

Writing a simple word-count program in MapReduce often requires dozens of lines of Java code, custom Writable types, and complex configurations. It’s notoriously difficult to develop and maintain.

- **Spark's Solution:** Spark offers a much higher level of abstraction with its **RDD (Resilient Distributed Dataset)** and **DataFrame** APIs.
    
    - **Conciseness:** A task that takes 50 lines in MapReduce might take 2 or 3 lines in Spark.
        
    - **Language Support:** Spark supports Python, Scala, Java, and R, making it accessible to data scientists, not just specialized Java engineers.
        

## 5. The "Swiss Army Knife" Problem

In the early Hadoop ecosystem, if you wanted to do SQL, you needed **Hive**. For streaming, you needed **Storm**. For graph processing, you needed **Giraph**. Managing all these separate tools was a DevOps nightmare.

- **Spark's Solution:** Spark is an **All-in-One Stack**. It includes:
    - **Spark SQL** (for database queries)
    - **MLlib** (for Machine Learning)
    - **GraphX** (for graph processing)
    - **Spark Streaming**
        
### Comparison Summary

|**Feature**|**Hadoop MapReduce**|**Apache Spark**|
|---|---|---|
|**Primary Storage**|Disk (HDFS)|Memory (RAM)|
|**Performance**|Slower (High Disk I/O)|Very Fast (In-memory)|
|**Complexity**|High (Verbose code)|Low (Simple APIs)|
|**Real-time**|No (Batch only)|Yes (Streaming)|
|**Iterative Work**|Poor|Excellent|
