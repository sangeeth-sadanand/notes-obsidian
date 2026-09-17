---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Delta lake/Delta lake|Delta lake]]"
tags:
  - DataBricks/deltaLake
index: 1
type: topic
---
# Overview

```ad-summary
collapse: true 
title: Summary

## Data Lakes overview 
- They are centralized, highly scalable storage repositories (like Amazon S3, Azure ADLS Gen2, or Google Cloud Storage) designed to hold massive volumes of raw data—whether structured, unstructured, or semi-structured—in its native format.

### Key Advantages
- **Cost-Effective & Scalable:** Cheaper than traditional data warehouses for bulk storage and capable of handling petabytes of data.
- **Highly Flexible:** Accepts any data format without requiring a predefined schema before writing.
- **Decoupled Architecture:** Storage and compute scale independently, optimizing resource usage.

### The Core Challenge: Lack of ACID Guarantees
Traditional data lakes struggle because they do not enforce **ACID properties** (Atomicity, Consistency, Isolation, Durability) like standard databases do. This limitation leads to four main failure points:
1. **Incomplete Appends:** If a job crashes while adding data, partial files are left behind, ruining consistency.
2. **Destructive Overwrites:** A failed overwrite can delete original data while only partially writing new data, violating atomicity and durability.
3. **Read/Write Interference:** Readers might accidentally process incomplete data if a write job is happening at the exact same time (poor isolation).
4. **Schema Mismatches:** Because there is no strict schema enforcement, evolving data structures can silently corrupt data quality over time.
```
---

# Data Lake Overview
A Data Lake is a centralized storage system designed to hold large volumes of raw data in its native format. 
* **Supported Data Types:** It stores structured, unstructured, and semi-structured data natively.
* **Storage Solutions:** Common cloud storage solutions utilized as data lakes include Amazon S3, Azure ADLS Gen2, and Google Cloud Storage (GCS).

## Advantages of Data Lakes
* **Cost-effective:** Generally cheaper for bulk storage than traditional data warehouses.
* **Highly Scalable:** Can easily scale to handle petabytes of data.
* **Flexibility:** Supports any form of data without enforcing a schema on write.
* **Decoupled Architecture:** Storage and compute are decoupled, allowing them to scale independently.


# Challenges with Traditional Data Lakes
Traditional data lakes fail to provide database-style **ACID guarantees**, leading to several fundamental problems:
* **ACID Properties Defined:**
  * **Atomicity:** All steps in a transaction must succeed; otherwise, none should apply (all-or-nothing).
  * **Consistency:** Data should remain valid and uncorrupted before and after applying transactions.
  * **Isolation:** Concurrent operations should not interfere with one another or reveal partial results.
  * **Durability:** Once a transaction is applied, it must persist even in the event of system failure.

## Why Do Traditional Data Lakes Fail?
* **Failed Append Operations:** If a job fails during an append, partially written files remain, leading to data inconsistency.
* **Failed Overwrite Operations:** When a write job fails during an overwrite, the original data is lost, and partial writes violate atomicity, consistency, and durability.
* **Simultaneous Read/Write Interference:** A reader might process partial or intermediate files while a write operation is simultaneously occurring, leading to incorrect output.
* **Schema Mismatches:** Evolving data can lead to inconsistent schemas, which corrupts the data and degrades data quality.
