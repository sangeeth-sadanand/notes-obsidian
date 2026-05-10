---
up:
  - "[[003_skills/data-engineering/06 Spark/0602 Data abstraction/0602 Data abstraction|0602 Data abstraction]]"
down:
prev:
topic: false
question: What transformation and actions can be applied on RDD?
---
# What transformation and actions can be applied on RDD?

> [!Summary] Summary
> 
> | Transformation   | Description                                            |
> | ---------------- | ------------------------------------------------------ |
> | `map()`          | Applies a function to each element                     |
> | `flatMap()`      | Similar to map but returns multiple elements per input |
> | `filter()`       | Filters elements based on a condition                  |
> | `distinct()`     | Removes duplicate elements                             |
> | `union()`        | Combines two RDDs                                      |
> | `intersection()` | Returns common elements                                |
> | `subtract()`     | Removes elements present in another RDD                |
> | `groupByKey()`   | Groups values with the same key                        |
> | `reduceByKey()`  | Aggregates values using a function                     |
> | `sortByKey()`    | Sorts RDD by key                                       |
> | `sample()`       | Takes a random sample                                  |
> | `coalesce()`     | Reduces number of partitions                           |
> | `repartition()`  | Increases or decreases partitions                      |
> 
> 
> | Action             | Description                                |
> | ------------------ | ------------------------------------------ |
> | `collect()`        | Returns all elements to the driver         |
> | `count()`          | Returns number of elements                 |
> | `first()`          | Returns first element                      |
> | `take(n)`          | Returns first `n` elements                 |
> | `reduce()`         | Aggregates elements using a function       |
> | `foreach()`        | Performs an operation on each element      |
> | `saveAsTextFile()` | Saves RDD to storage                       |
> | `countByKey()`     | Counts elements per key                    |
> | `takeOrdered(n)`   | Returns first `n` elements in sorted order |
> 
> 


In **Apache Spark**, operations on **RDDs (Resilient Distributed Datasets)** are broadly divided into **Transformations** and **Actions**.

## 1. Transformations on RDD

- **Transformations are lazy operations**.  
- They do **not execute immediately**; instead, they build a **logical execution plan (DAG)** that Spark runs only when an action is called.

### **Key characteristics**

- Return a new RDD
- Lazy (execution deferred)
- Used for data processing

### Common RDD Transformations

Assume SparkContext is already available as `sc`.

#### `map()` – Applies a function to each element

```python
rdd = sc.parallelize([1, 2, 3, 4])
result = rdd.map(lambda x: x * 2)

print(result.collect())
# Output: [2, 4, 6, 8]
```

#### `flatMap()` – Returns multiple elements per input

```python
rdd = sc.parallelize(["Hello World", "Apache Spark"])
result = rdd.flatMap(lambda x: x.split(" "))

print(result.collect())
# Output: ['Hello', 'World', 'Apache', 'Spark']
```

#### `filter()` – Filters elements based on a condition

```python
rdd = sc.parallelize([10, 15, 20, 25])
result = rdd.filter(lambda x: x > 15)

print(result.collect())
# Output: [20, 25]
```

#### `distinct()` – Removes duplicate elements

```python
rdd = sc.parallelize([1, 2, 2, 3, 3, 3])
result = rdd.distinct()

print(result.collect())
# Output: [1, 2, 3]
```

#### `union()` – Combines two RDDs

```python
rdd1 = sc.parallelize([1, 2, 3])
rdd2 = sc.parallelize([4, 5])

result = rdd1.union(rdd2)
print(result.collect())
# Output: [1, 2, 3, 4, 5]
```

#### `intersection()` – Returns common elements

```python
rdd1 = sc.parallelize([1, 2, 3, 4])
rdd2 = sc.parallelize([3, 4, 5])

result = rdd1.intersection(rdd2)
print(result.collect())
# Output: [3, 4]
```

#### `subtract()` – Removes elements present in another RDD

```python
rdd1 = sc.parallelize([1, 2, 3, 4])
rdd2 = sc.parallelize([2, 4])

result = rdd1.subtract(rdd2)
print(result.collect())
# Output: [1, 3]
```

#### `groupByKey()` – Groups values with the same key

```python
rdd = sc.parallelize([("A", 1), ("B", 2), ("A", 3)])
result = rdd.groupByKey()

print([(k, list(v)) for k, v in result.collect()])
# Output: [('A', [1, 3]), ('B', [2])]
```

> [!Warning] 
> Not recommended for large datasets due to shuffle overhead.

#### `reduceByKey()` – Aggregates values using a function

```python
rdd = sc.parallelize([("A", 1), ("B", 2), ("A", 3)])
result = rdd.reduceByKey(lambda a, b: a + b)

print(result.collect())
# Output: [('A', 4), ('B', 2)]
```

> [!tip]
> Preferred over `groupByKey()` for performance.

#### `sortByKey()` – Sorts RDD by key

```python
rdd = sc.parallelize([(3, "C"), (1, "A"), (2, "B")])
result = rdd.sortByKey()

print(result.collect())
# Output: [(1, 'A'), (2, 'B'), (3, 'C')]
```

#### `sample()` – Takes a random sample

```python
rdd = sc.parallelize(range(1, 11))
result = rdd.sample(withReplacement=False, fraction=0.4)

print(result.collect())
# Output: Random ~40% of elements
```

#### `coalesce()` – Reduces number of partitions

```python
rdd = sc.parallelize(range(1, 11), 5)
coalesced = rdd.coalesce(2)

print(rdd.getNumPartitions())       # 5
print(coalesced.getNumPartitions()) # 2
```

Minimizes shuffle when reducing partitions.

#### `repartition()` – Increases or decreases partitions

```python
rdd = sc.parallelize(range(1, 11), 2)
repartitioned = rdd.repartition(4)

print(rdd.getNumPartitions())          # 2
print(repartitioned.getNumPartitions()) # 4
```
 Triggers a full shuffle.

## 2. Actions on RDD

- **Actions trigger execution** of all the transformations defined so far.  
- They return a **result** to the driver or write data to storage.

### **Key characteristics**

- Trigger Spark job execution
- Return values or persist results
- End of RDD processing pipeline

### Common RDD Actions

#### `collect()` – Returns all elements to the driver

```python
rdd = sc.parallelize([1, 2, 3, 4])
result = rdd.collect()

print(result)
# Output: [1, 2, 3, 4]
```

> [!warning] 
> Avoid using `collect()` on **large datasets** (driver memory risk).

#### `count()` – Returns number of elements

```python
rdd = sc.parallelize([10, 20, 30, 40])
print(rdd.count())
# Output: 4
```

#### `first()` – Returns first element

```python
rdd = sc.parallelize([100, 200, 300])
print(rdd.first())
# Output: 100
```

> [!tip]
> Efficient for previewing data.

#### `take(n)` – Returns first `n` elements

```python
rdd = sc.parallelize(range(1, 10))
print(rdd.take(3))
# Output: [1, 2, 3]
```

> [!tip]
> Safer alternative to `collect()`.

#### `reduce()` – Aggregates elements using a function

```python
rdd = sc.parallelize([1, 2, 3, 4])
result = rdd.reduce(lambda a, b: a + b)

print(result)
# Output: 10
```

> [!tip]
> Used when a **single aggregated value** is needed.

#### `foreach()` – Performs an operation on each element

```python
rdd = sc.parallelize([1, 2, 3])
rdd.foreach(lambda x: print(x))
```

> [!tip]
> Executes on **executors**, not the driver  
> Not suitable for collecting results

#### `saveAsTextFile()` – Saves RDD to storage

```python
rdd = sc.parallelize(["Spark", "Hadoop", "Hive"])
rdd.saveAsTextFile("/tmp/tech_output")
```

> [!tip]
> Output is saved as **multiple part files**, one per partition.

#### `countByKey()` – Counts elements per key

```python
rdd = sc.parallelize([("A", 1), ("B", 1), ("A", 1)])
result = rdd.countByKey()

print(result)
# Output: {'A': 2, 'B': 1}
```

> [!tip]
> Result is a **Python dictionary**, returned to driver.

#### `takeOrdered(n)` – Returns first `n` elements in sorted order

```python
rdd = sc.parallelize([5, 1, 3, 2, 4])
result = rdd.takeOrdered(3)

print(result)
# Output: [1, 2, 3]
```

For descending order:

```python
rdd.takeOrdered(3, key=lambda x: -x)
# Output: [5, 4, 3]
```

## 3. Transformation vs Action (Quick Comparison)

| Aspect    | Transformation  | Action          |
| --------- | --------------- | --------------- |
| Execution | Lazy            | Immediate       |
| Returns   | New RDD         | Result / Output |
| Spark Job | Not triggered   | Triggered       |
| Purpose   | Data processing | Get results     |
