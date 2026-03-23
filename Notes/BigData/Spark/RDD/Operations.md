# Operations
## Read (Creating RDDs)

- `sc.parallelize([1,2,3])` → create RDD from local collection.
- `sc.textFile("path")` → read text file into RDD.
- `sc.wholeTextFiles("path")` → read directory of files (filename, content).

```python
from pyspark import SparkContext
sc = SparkContext("local", "RDD_Example")

# Create RDD from collection
rdd1 = sc.parallelize([1, 2, 3, 4, 5])

# Read RDD from file
rdd2 = sc.textFile("data.txt")

```

## Transformations (Lazy)

Transformations build new RDDs without executing immediately.

### Narrow Transformations

- Data from one partition → mapped to one partition (no shuffle).
- Examples:
    - `map(func)`
    - `filter(func)`
    - `flatMap(func)`
    - `union(rdd)`
    - `sample(withReplacement, fraction)`
```python
# map → element-wise transformation
mapped = rdd1.map(lambda x: x * 2)

# filter → keep elements satisfying condition
filtered = rdd1.filter(lambda x: x % 2 == 0)

# flatMap → multiple outputs per input
flatmapped = rdd1.flatMap(lambda x: (x, x*x))

# union → combine two RDDs
unioned = rdd1.union(sc.parallelize([6, 7]))

# Sample without replacement (approx 40% of elements)
sample1 = rdd.sample(False, 0.4)
print("Sample without replacement:", sample1.collect())
```
### Wide Transformations

- Data from multiple partitions → requires shuffle across nodes.
- Examples:
    - `groupByKey()`
    - `reduceByKey(func)`
    - `join(rdd)`
    - `distinct()`
    - `cogroup(rdd)`

```python
pairs = sc.parallelize([("a", 1), ("b", 2), ("a", 3)])

# groupByKey → groups values by key
grouped = pairs.groupByKey()

# reduceByKey → aggregation by key
reduced = pairs.reduceByKey(lambda x, y: x + y)

# join → join two RDDs by key
rddA = sc.parallelize([("a", 1), ("b", 2)])
rddB = sc.parallelize([("a", 3), ("b", 4)])
joined = rddA.join(rddB)

# distinct → remove duplicates
distincted = rdd1.distinct()

# Groups the values of two (or more) RDDs sharing the same key.
cogrouped = rdd1.cogroup(rdd2)

# Collect results
for key, (vals1, vals2) in cogrouped.collect():
    print(key, list(vals1), list(vals2))
```


## Actions (Eager)

Actions trigger execution and return results or write output.

- `collect()` → return all elements to driver.
- `count()` → number of elements.
- `first()` → first element.
- `take(n)` → first _n_ elements.
- `reduce(func)` → aggregate elements.
- `foreach(func)` → apply function to each element.
```python
print("Collect:", mapped.collect())       # → [2,4,6,8,10]
print("Count:", rdd1.count())             # → 5
print("First:", rdd1.first())             # → 1
print("Take:", rdd1.take(3))              # → [1,2,3]
print("Reduce:", rdd1.reduce(lambda x,y: x+y))  # → 15

# foreach → apply function (side effects only)
rdd1.foreach(lambda x: print(x))

```

##  Write (Persisting Results)

- `saveAsTextFile("path")` → store as text.
- `saveAsSequenceFile("path")` → Hadoop sequence format.
- `saveAsObjectFile("path")` → serialized objects.

```python
# Save as text file
mapped.saveAsTextFile("output/mapped")

# Save as Sequence file (Hadoop format)
pairs.saveAsSequenceFile("output/seq")

# Save as object file
rdd1.saveAsObjectFile("output/obj")

```

