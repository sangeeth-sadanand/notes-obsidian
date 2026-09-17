---
up:
  - "[[003_DetailedNotes/DSA/Algorithm/Array/Array|Array]]"
tags:
index: 1
type: topic
---
## 1. Linear Search

- **Requirement:** Works on any array (sorted or unsorted).
- **How it works:** Checks every element sequentially from start to finish.
- **Time Complexity:** O(n)

## 2. Binary Search

- **Requirement:** The array **must be sorted**.
- **How it works:** Repeatedly divides the search interval in half.
- **Time Complexity:** O(log n)

![[005_assets/DSA/Algorithm/binary-and-linear-search-animations.gif]]
## 3. Jump Search

- **Requirement:** The array **must be sorted**.
- **How it works:** Jumps ahead by fixed steps (blocks) rather than checking every element, then performs a linear search within the block.
- **Time Complexity:** O(sqrt(n))


![[005_assets/DSA/Algorithm/Jumpsearch.png]]
## 4. Interpolation Search

- **Requirement:** The array **must be sorted** and have uniformly distributed data.
- **How it works:** Estimates the position of the target value based on the values at the endpoints of the search range (like looking up a name in a phone book).
- **Time Complexity:** O(log log n) on average

$$
pos = low + \left\lfloor \frac{(target - arr[low]) \times (high - low)}{arr[high] - arr[low]} \right\rfloor
$$
## 5. Exponential Search

- **Requirement:** The array **must be sorted**.
- **How it works:** Finds the range where the element exists by exponentially increasing steps, then applies binary search within that range. Ideal for unbounded or infinite arrays.
- **Time Complexity:** O(log n)

## 6. Ternary Search

- **Requirement:** The array **must be sorted**.
- **How it works:** Divides the array into three parts instead of two to narrow down the location of the target.
- **Time Complexity:** O(log3 n)
    