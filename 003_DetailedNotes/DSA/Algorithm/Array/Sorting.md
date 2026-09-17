---
up:
  - "[[003_DetailedNotes/DSA/Algorithm/Array/Array|Array]]"
tags:
index: 2
type: topic
---

## 1. Bubble Sort

- Repeatedly compares adjacent elements and swaps them if they are in the wrong order.
- Simple but inefficient for large datasets.
- Time Complexity:
    - Best: O(n)
    - Average: O(n²)
    - Worst: O(n²)

## 2. Selection Sort

- Finds the smallest element and places it in the correct position.
- Performs fewer swaps than Bubble Sort.
- Time Complexity:
    - Best/Average/Worst: O(n²)

## 3. Insertion Sort

- Builds the sorted array one element at a time.
- Efficient for small or nearly sorted data.
- Time Complexity:
    - Best: O(n)
    - Average: O(n²)
    - Worst: O(n²)

## 4. Merge Sort

- Uses Divide and Conquer strategy.
- Splits the array into halves, sorts them, and merges them.
- Stable sorting algorithm.
- Time Complexity:
    - Best/Average/Worst: O(n log n)

## 5. Quick Sort

- Selects a pivot and partitions elements around it.
- Often the fastest in practice.
- Time Complexity:
    - Best/Average: O(n log n)
    - Worst: O(n²)

## 6. Heap Sort

- Uses a Heap data structure.
- Consistent performance.
- Time Complexity:
    - Best/Average/Worst: O(n log n)

## 7. Shell Sort

- Improved version of Insertion Sort.
- Sorts elements at specific gaps.
- Time Complexity:
    - Depends on gap sequence (typically between O(n log n) and O(n²))

## 8. Counting Sort

- Non-comparison sorting algorithm.
- Works for integers within a limited range.
- Time Complexity:
    - O(n + k)
    - k = range of input values

## 9. Radix Sort

- Sorts numbers digit by digit.
- Suitable for integers and strings.
- Time Complexity:
    - O(d × (n + k))
    - d = number of digits

## 10. Bucket Sort

- Distributes elements into buckets and sorts each bucket.
- Effective for uniformly distributed data.
- Time Complexity:
    - Best/Average: O(n + k)
    - Worst: O(n²)


|Algorithm|Best|Average|Worst|Stable|
|---|---|---|---|---|
|Bubble Sort|O(n)|O(n²)|O(n²)|Yes|
|Selection Sort|O(n²)|O(n²)|O(n²)|No|
|Insertion Sort|O(n)|O(n²)|O(n²)|Yes|
|Merge Sort|O(n log n)|O(n log n)|O(n log n)|Yes|
|Quick Sort|O(n log n)|O(n log n)|O(n²)|No|
|Heap Sort|O(n log n)|O(n log n)|O(n log n)|No|
|Counting Sort|O(n+k)|O(n+k)|O(n+k)|Yes|
|Radix Sort|O(d(n+k))|O(d(n+k))|O(d(n+k))|Yes|
|Bucket Sort|O(n+k)|O(n+k)|O(n²)|Depends|

