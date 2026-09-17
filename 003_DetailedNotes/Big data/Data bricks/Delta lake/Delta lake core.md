---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Delta lake/Delta lake|Delta lake]]"
tags:
  - DataBricks/deltaLake
index: 2
type: topic
---
# Delta lake core

```ad-summary
collapse: true 
title: Summary

**Delta Lake** is an open-source storage layer that sits on top of your existing data lake to bring reliability and full ACID transactions.

## Core Concept: Parquet + Transaction Logs

At its foundation, Delta Lake relies on a simple formula: **Raw Parquet files + a Transactional Log (`_delta_log`)**. This log acts as the single source of truth, using JSON files to track every single change to the data.

## How It Solves Data Lake Challenges

By forcing all operations to go through the `_delta_log`, Delta Lake ensures consistency, schema enforcement, and data versioning:
- **Reliable Writes:** Data is written to Parquet files _first_. A transaction is only considered successful when a JSON log file is subsequently added to commit those changes.
- **Consistent Reads:** Delta always checks the log first to determine the latest valid state, preventing users from seeing partial or corrupt data.
    > **Crucial Rule:** Because Delta tracks active vs. inactive files, you must _never_ read the raw Parquet files directly. Always use a Delta format reader.
- **Failure Protection:** If a job crashes mid-write, the orphaned Parquet files simply aren't logged. The system treats them as invisible, leaving the original data perfectly intact.
- **Concurrency:** If a read and write happen simultaneously, the reader will safely query the older, committed files until the new write transaction is officially logged.
### Inside the `_delta_log`
The log calculates the current state of your table by recording chronological **add** and **remove** actions.
- **Log Contents:** The log tracks commit metadata (why the change happened), table schema, and statistics (min/max values for up to 32 columns), which allows Delta to skip irrelevant files and vastly improve query speeds.
- **Computing State:** When you query the data, Delta sequentially reads the transaction logs. It applies all the `add` actions and ignores any files that have a subsequent `remove` action, ensuring it only reads the currently active Parquet files.

```
---
## Introduction to Delta Lake
**Delta Lake** is an open-source storage layer that brings reliability and ACID transactions to data lakes. It sits directly on top of your existing data lake architecture.

**Core Concept:** 
> `Delta Lake = Parquet files + Transactional logs (_delta_log)`

### How Delta Lake Solves Traditional Problems
- Delta Lake enforces ACID guarantees on top of a traditional data lake by maintaining a transactional log (`_delta_log`). 
- This log contains JSON files that track every single change, ensuring reliable file operations, schema enforcement, schema evolution, data versioning, and consistent reads/writes.

#### Handling Read and Write Operations
* **Write Operations:** All Parquet files are written to storage first. Only after the files are fully written is a JSON transactional log file added to the `_delta_log` directory committing the changes.
* **Read Operations:** When reading, Delta queries the transactional logs to determine the latest valid state. It **only** reads committed files, preventing partial or corrupt data from being exposed to readers.
* **Failure Handling:** If a job fails during an append, the Parquet files might be written, but the transactional log is never updated. Therefore, the old data remains unchanged and valid.
* **Concurrent Operations:** When two processes simultaneously read and write, the reader will only see the old files. The newly written files are completely invisible until their transaction log is committed.

> [!note]
>Because Delta tracks all file states, it is strongly advised **not** to read the raw Parquet files inside a Delta Lake directly. Always use the Delta format reader to ensure you get the accurate, consistent state of the table.

#### Delta Log Directory
The Delta log directory acts as the single source of truth for the table. It tracks changes using `add` and `remove` entries. 
The log contains:
* **Commit Info:** Details on what triggered the transaction.
* **Metadata:** The schema of the data.
* **Add Actions:** Part file names and statistical min/max values of up to 32 columns. This is heavily utilized for **Data Skipping** to improve query performance.
## How Delta Lake Computes the Latest State

- Delta Lake **reads transaction log JSON files in order** and applies **add** and **remove** actions to determine which parquet files are active.
- Each transaction log entry records **add** and **remove** actions; files removed later are ignored for the latest state.
- Example flow:
    1. Table creation → add a transaction log entry.
    2. Insert → add a parquet file and a log entry.
    3. Update → create a new parquet file with updated rows, add a **remove** for the old parquet file, and add entries for deletion vectors or new files as needed.
- When reading, Delta Lake scans logs sequentially and applies only the final active add entries.

