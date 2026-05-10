---
up:
  - "[[003_skills/data-engineering/04 processing layer/04 processing layer|04 processing layer]]"
down:
prev:
topic: false
question: How does map phase differ from reduce phase?
---
# How does map phase differ from reduce phase


> [!Summary] Summary
> 
> - **Map phase**
> 	- Take raw files, break into small pieces.
> 	- Then applies map function which gives key and value pair
> 	- It runs in parallel in multiple node
> 	- Performs filtering, transformation or extraction
> 	- Does not aggregate
> - **Reduce phase**
> 	- It receives a grouped data from shuffle from mapped data
> 	- Applies a reduce function to each key
> 	- Combine value associated with same key
> 	- Produce the final output
> 	- It focus on aggregate


The **Map phase** and **Reduce phase** are the two core stages of the **MapReduce** programming model.

## 1. Map Phase

### Purpose

The **Map phase** processes input data and converts it into **intermediate key–value pairs**.

### What it does

*   Takes raw input data (files, records, lines of text, etc.)
*   Breaks it into smaller pieces
*   Applies a **map function** to each piece independently
*   Produces intermediate results as `(key, value)` pairs

### Characteristics

*   Runs **in parallel** on multiple nodes
*   Performs **filtering, transformation, or extraction**
*   Does **not aggregate** data globally

### Example (Word Count)

Input:

    "big data big analytics"

Map output:

    (big, 1)
    (data, 1)
    (big, 1)
    (analytics, 1)

***

## 2. Reduce Phase

### Purpose

The **Reduce phase** aggregates and processes the intermediate key–value pairs produced by the Map phase.

### What it does

*   Receives grouped data from the Map phase
*   Applies a **reduce function** to each key
*   Combines values associated with the same key
*   Produces the **final output**

### Characteristics

*   Runs after the Map phase completes
*   Works on **grouped and sorted** data
*   Focuses on **aggregation, summarization, or computation**

### Example (Word Count)

Input to Reduce:

    (big, [1, 1])
    (data, [1])
    (analytics, [1])

Reduce output:

    (big, 2)
    (data, 1)
    (analytics, 1)

***

## 3. Key Differences at a Glance

| Aspect             | Map Phase                    | Reduce Phase                   |
| ------------------ | ---------------------------- | ------------------------------ |
| Main role          | Data transformation          | Data aggregation               |
| Input              | Raw input data               | Intermediate key–value pairs   |
| Output             | Intermediate key–value pairs | Final result                   |
| Execution          | Runs first                   | Runs after map phase           |
| Parallelism        | Highly parallel              | Parallel per key               |
| Data handling      | Independent processing       | Grouped by key                 |
| Typical operations | Filtering, parsing, mapping  | Summation, counting, averaging |
|                    |                              |                                |

***

## 4. How They Work Together

1.  **Map phase** processes raw data and emits key–value pairs.
2.  **Shuffle & Sort** (automatic framework step) groups values by key.
3.  **Reduce phase** processes each group to generate final results.

***
