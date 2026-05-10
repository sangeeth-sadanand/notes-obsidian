---
up:
  - "[[003_skills/data-engineering/06 Spark/0603 SQL/060304 Optimization/060304 Optimization|060304 Optimization]]"
down:
prev:
topic: false
question: What is data skew? How does salting help in optimizing data skewing?
---
# What is data skew? How does salting help in optimizing data skewing?


> [!Summary] Summary
> - Data skew happens when the data is not distributed evenly across these partitions 
> - Salting is a programmatic method to break massive, skewed into smaller, evenly distributed chunk 
> - Salting is appending a random number to the end of the skewed key.

## **What is Data Skew?**
In a distributed system like Apache Spark, data is divided into "partitions" and processed in parallel by multiple tasks. **Data skew** happens when the data is not distributed evenly across these partitions.
Imagine you are joining a massive `sales` table with a `countries` table on `country_id`. If 90% of your customers are from the US, and the remaining 10% are spread across 100 other countries, the data is highly skewed.
When Spark performs a shuffle (like during a `GROUP BY` or `JOIN`), it routes all rows with the exact same key to the exact same partition on the exact same executor.
- **The Result:** One task (processing the 'US' key) receives 90% of the data. This single task might take 5 hours to run or crash with an OutOfMemory (OOM) error, while all other tasks finish in 10 seconds. This is known as a **straggler task**, and your entire job is bottle necked by it.

## **How Salting Helps Optimize Data Skew**
**Salting** is a clever programmatic trick to break up a massive, skewed key into smaller, evenly distributed chunks. You "salt" the data by appending a random number (the salt) to the end of the skewed key.
Here is the step-by-step mechanism of how salting works in a Join:
### **Step 1: Salt the Skewed Table (Table A)**
Instead of leaving the massive key as `'US'`, you append a random integer between 0 and a chosen number $N$ (e.g., $N=4$).
- `'US'` becomes `'US_0'`, `'US_1'`, `'US_2'`, `'US_3'`, or `'US_4'`.
- Now, when Spark shuffles the data, it sees 5 distinct keys instead of 1. The massive block of data is evenly distributed across 5 different partitions and tasks.

### **Step 2: Explode the Dimension Table (Table B)**
There is a catch. If Table A now has keys like `'US_0'` and `'US_1'`, but Table B only has the original `'US'` key, the join will fail. There are no matches!
To fix this, you must **replicate (explode)** the skewed row in Table B. You create a copy of the `'US'` row for every possible salt value (0 to 4).
- Table B now has rows for `'US_0'`, `'US_1'`, `'US_2'`, `'US_3'`, and `'US_4'`.
    
### **Step 3: Execute the Join**
Now you join Table A and Table B on the new salted keys.
- Because the keys match, the join is perfectly accurate.
- Because the massive `'US'` key was split into 5 distinct keys, 5 different executors can process the 'US' data in parallel. The bottleneck is destroyed.

> [!Tip] 
> While salting solves the OOM and bottleneck issues, it does require replicating data. 
> Therefore, you should only salt the specific keys that are heavily skewed.

