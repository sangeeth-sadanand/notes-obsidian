---
up:
  - "[[003_skills/data-engineering/01_Introduction|01_Introduction]]"
down:
prev:
topic: false
question: What is the difference between the database, data warehouse and data lake?
---
# What is the difference between the database, data warehouse and data lake?


> [!Summary] Summary
> 
> | **Feature**      | **Database**               | **Data Warehouse**             | **Data Lake**                      |
> | ---------------- | -------------------------- | ------------------------------ | ---------------------------------- |
> | **Data Type**    | Structured (Relational)    | Structured (Processed)         | All types (Raw)                    |
> | **Primary Goal** | Efficiency of transactions | Efficiency of analysis         | Flexibility and scale              |
> | **Schema**       | Write-time (Rigid)         | Write-time (Highly curated)    | Read-time (Fluid)                  |
> | **Cost**         | Expensive for large scale  | Premium (but high performance) | Low-cost (Storage is cheap)        |
> | **Agility**      | Low (hard to change)       | Medium                         | High (store now, figure out later) |


## 1. Database
A database is designed for **transactional** data. It’s built to handle quick updates, deletions, and insertions (like a banking transaction or an e-commerce order). It usually stores current, "clean" data in a highly structured format.
- **Structure:** Highly structured (Rows and Columns).
- **Purpose:** Operational tasks (Day-to-day business functions).
- **Schema on write**
- **Users:** Application developers and end-users.
- **Example**: Oracle, MySQL

## 2. Data Warehouse
A data warehouse is a central repository for **analytical** data. It pulls data from multiple databases, cleans it, and organizes it specifically for reporting. It stores historical data so you can compare "this year vs. last year."
- **Structure:** Structured (Schema-on-write; you must define the structure before loading the data).
- **Purpose:** Business Intelligence (BI) and reporting.
- Storage cost is high but less than database.
- **Users:** Business analysts and data scientists.
- it follows Extract, Transform & Load process.
- **Example**: Teradata
    

## 3. Data Lake
A data lake is a vast pool of **raw data**. It stores everything in its native format—structured, semi-structured (JSON/XML), and unstructured (images, PDFs, sensor logs). You don't worry about the structure until you are actually ready to use the data.
- **Structure:** Unstructured/Raw (Schema-on-read; you define the structure when you pull the data out).
- **Purpose:** Deep data exploration and Machine Learning. To get insights from huge amount of data.
-  It follows ELT process 
-  Cost effective
- **Users:** Data scientists and data engineers.


## Which one do you use?

In a modern Big Data architecture, companies usually use **all three**:
1. **Databases** run the apps (e.g., storing a user's password).
2. Data is dumped into a **Data Lake** for long-term storage and ML training.
3. The most important parts of that data are cleaned and moved into a **Data Warehouse** for the CEO’s monthly performance dashboard.
