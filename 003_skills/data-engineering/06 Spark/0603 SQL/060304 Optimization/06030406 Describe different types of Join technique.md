---
up:
  - "[[003_skills/data-engineering/06 Spark/0603 SQL/060304 Optimization/060304 Optimization|060304 Optimization]]"
down:
prev:
topic: false
question: Describe different types of Join technique
---
# Describe different types of Join technique


> [!Summary] Summary
> |**Join Strategy**|**Ideal Scenario**|**Join Type**|**Shuffle?**|**Sort?**|**Pros**|**Cons / Risks**|
> |---|---|---|---|---|---|---|
> |**Broadcast Hash Join (BHJ)**|One table is very small (fits in memory) and the other is large.|Equi-joins|❌ No|❌ No|Fastest strategy; avoids expensive network shuffles.|Can cause driver or executor OutOfMemory (OOM) errors if the "small" table exceeds memory limits.|
> |**Sort Merge Join (SMJ)**|Both tables are very large.|Equi-joins|✅ Yes|✅ Yes|Highly scalable; extremely memory-efficient for massive datasets due to sorting.|Slower than BHJ; heavy network and disk I/O due to shuffling and sorting phases.|
> |**Shuffle Hash Join (SHJ)**|Both tables are large, but one is ~3x smaller than the other (and its partitions fit in memory).|Equi-joins|✅ Yes|❌ No|Faster than SMJ because it skips the sorting phase.|Highly vulnerable to OOM errors if there is data skew (one partition's hash table gets too large).|
> |**Broadcast Nested Loop Join (BNLJ)**|One table is small enough to broadcast, but the condition is complex.|Non-equi-joins|❌ No|❌ No|Capable of handling complex, non-equality join conditions (like `>`, `<`).|Extremely CPU intensive $O(M \times N)$; every row is compared against every other row.|
> |**Cartesian Product Join (CPJ)**|Both tables are large, and you need a cross join or non-equi-join.|Cross / Non-equi|✅ Yes|❌ No|Evaluates every possible combination of rows across the network.|Worst performance; produces an immense amount of data and network traffic. Should generally be avoided.|
> 
> 

Choosing the wrong execution strategy can lead to massive network bottlenecks (shuffles) or OutOfMemory (OOM) errors. Spark primarily uses five physical join techniques, which dynamically depend on the size of your data and the join condition.


## **1. Broadcast Hash Join (BHJ)**
This is the fastest and most highly sought-after join strategy in Spark because it completely avoids the expensive "shuffle" phase.
- **How it works:** Spark takes the smaller of the two tables and broadcasts (copies) it entirely to the memory of every single executor in the cluster. Each executor then hashes the small table and locally joins it with the partitions of the larger table it already holds.
- **When it is used:** Used for **equi-joins** (e.g., `A.id = B.id`) when one of the tables is small enough to fit easily into memory.
- **Tuning Parameter:** Triggered automatically if one table's size is below `spark.sql.autoBroadcastJoinThreshold` (default is 10 MB).

## **2. Sort Merge Join (SMJ)**
This is Spark's default, heavy-duty join strategy. It is designed to handle massive datasets where neither table can fit entirely into the memory of a single node.
- **How it works:** It involves three distinct phases:
    1. **Shuffle:** Both tables are shuffled across the network so that rows with the same join keys end up on the same executor.
    2. **Sort:** Each executor sorts its local partitions based on the join key.
    3. **Merge:** The sorted datasets are iterated over (merged) to find matching rows.
- **When it is used:** Used for **equi-joins** between two large tables. Because the data is sorted, the merge phase is highly memory-efficient, avoiding OOM errors even with billions of rows.
- **Tuning Parameter:** `spark.sql.join.preferSortMergeJoin` (default is `true`).
    

## **3. Shuffle Hash Join (SHJ)**
This is an alternative to the Sort Merge Join. It requires a shuffle but skips the sorting phase.
- **How it works:** Similar to SMJ, both tables are shuffled over the network by their join keys. However, instead of sorting, Spark builds an in-memory hash table for the smaller of the two shuffled partitions on each executor. It then streams the larger partition and probes the hash table for matches.
- **When it is used:** Best for **equi-joins** where two tables are large, but one is at least 3x smaller than the other, and the average partition size of the smaller table can comfortably fit into memory.
- **Note:** Spark usually prefers SMJ over SHJ because SHJ is vulnerable to OOM errors if data is skewed (i.e., one hash table partition gets too large).

## **4. Broadcast Nested Loop Join (BNLJ)**
This is a fallback strategy for non-equi joins when one table is small enough to broadcast.
- **How it works:** The smaller dataset is broadcast to all executors. Spark then performs a nested loop—meaning every row in the large table's partition is compared against _every single row_ in the broadcasted table.
- **When it is used:** Used for **non-equi joins** (e.g., `A.date > B.start_date AND A.date < B.end_date`) where one table is smaller than the broadcast threshold.

## **5. Cartesian Product Join (CPJ)**
This is the absolute slowest and most resource-intensive join strategy. It should generally be avoided.
- **How it works:** Every single partition of Table A is sent over the network to be joined with every single partition of Table B. If Table A has 1 million rows and Table B has 1 million rows, Spark evaluates 1 trillion combinations.
- **When it is used:** Used as a last resort for **non-equi joins** or cross joins when neither table is small enough to be broadcasted.

