---
up:
  - "[[003_DetailedNotes/DSA/Data Structures/Data Structures|Data Structures]]"
tags:
index: 1
type: topic
---
## 1. Definition and Abstract Data Type (ADT) View

- An **array** is an Abstract Data Type (ADT) whose objects form a sequence of elements of the same data type.
- As an ADT, an array natively supports two primary operations:
    1. **STORE** (`STORE(a, i, e)` or `a[i] := e`): Writes value `e` into index `i` of array `a`.
    2. **RETRIEVE** (`RETRIEVE(a, i)` or `a[i]`): Reads the element at index `i`.
- Each element resides in a contiguous unit of memory known as a **cell**.

## 2. Array Dimensions and Size Calculation

Arrays can be structured across various dimensions:

- **One-Dimensional (1D) Arrays**: Likened to mathematical vectors. For an array defined as $A[l : u]$ with lower bound $l$ and upper bound $u$, the total number of elements is given by: $\text{Size} = u - l + 1$
- **Two-Dimensional (2D) Arrays**: Likened to matrices with rows and columns. For $A[l_1 : u_1, l_2 : u_2]$, the size is: $\text{Size} = (u_1 - l_1 + 1) \cdot (u_2 - l_2 + 1)$
- **Three-Dimensional (3D) & Multidimensional ($N$-D) Arrays**: Higher-dimensional arrays can be conceptualized as nested collections (e.g., a 2D array viewed as $u_1$ one-dimensional arrays of size $u_2$; a 3D array viewed as $u_1$ two-dimensional slices). For an $N$-dimensional array $A[l_1 : u_1, l_2 : u_2, \dots, l_N : u_N]$, the total element count is: $\text{Size} = \prod_{j=1}^{N} (u_j - l_j + 1)$

### 3. Memory Representation and Address Calculations

Because physical memory is linear (1D), multidimensional arrays must be mapped to sequential memory addresses.

#### Memory Ordering Methods

1. **Row-Major Order**: Stores elements row by row in contiguous memory locations.
2. **Column-Major Order**: Stores elements column by column sequentially.

#### Address Formulas (Row-Major Order)

Let $\alpha$ be the **base address** (the starting location of the array in memory) and $w$ be the size of each memory location/element in bytes:

- **1D Array $A[l_1 : u_1]$**: $\text{Address}(A[i]) = \alpha + (i - l_1) \cdot w$
- **2D Array $A[l_1 : u_1, l_2 : u_2]$**: $\text{Address}(A[i, j]) = \alpha + \left[(i - l_1)(u_2 - l_2 + 1) + (j - l_2)\right] \cdot w$
- **3D Array $A[l_1 : u_1, l_2 : u_2, l_3 : u_3]$**: $\text{Address}(A[i, j, k]) = \alpha + \left[(i - l_1)(u_2 - l_2 + 1)(u_3 - l_3 + 1) + (j - l_2)(u_3 - l_3 + 1) + (k - l_3)\right] \cdot w$
- **$N$-Dimensional Array $A[1 : u_1, 1 : u_2, \dots, 1 : u_N]$**: $\text{Address}(A[i_1, i_2, \dots, i_N]) = \alpha + \sum_{j=1}^{N} (i_j - 1) a_j \quad \text{where } a_j = \prod_{k=j+1}^{N} u_k \quad (a_N = 1)$

![[005_assets/DSA/ArrayAddressCalculation.png]]
### 4. Primary Applications of Arrays

1. **Sparse Matrices**: For matrices dominated by zero entries, standard array storage wastes memory. Sparse matrices use a compact **3-tuple representation** $(i, j, \text{value})$ stored in an array $B[0:t, 1:3]$, where row 0 holds matrix dimensions and non-zero count $t$, while rows $1 \dots t$ record non-zero elements.
2. **Ordered Lists, Strings & Bit Arrays**:
    - **Ordered lists** map sequentially to 1D arrays.
    - **Strings** map to character arrays terminated by a null character (`\0`), and arrays of strings map to 2D character arrays.
    - **Bit arrays** optimize storage for boolean flag sets.
3. **Linear Stacks and Queues**: Stacks and queues frequently use array-based implementations with index variables (`Top`, `Front`, `Rear`) and boundary checks (`STACK_FULL`, `QUEUE_FULL`).
4. **Nonlinear & Advanced Structures**:
    - **Binary Trees**: Full or complete binary trees map efficiently to arrays where a node at index $i$ has children at indices $2i$ and $2i+1$.
    - **Segment Trees**: Represented via linear arrays to execute range queries (such as range minimum or range product) and updates in $O(\log_2 n)$ time.
    - **Graph Adjacency Matrices**: $n \times n$ binary matrices represent graph edges sequentially.

### 5. Demerits and Limitations

- **Inefficient Insertions & Deletions**: Inserting or deleting an element in a sequential array requires shifting surrounding elements or copying to temporary arrays, which is computationally expensive ($O(n)$).
- **Static Memory Allocation**: Array size must be booked in advance at declaration/compilation, risking overflow or unused memory capacity.
- **Potential Memory Wastage**: Incomplete binary trees or non-sparse representation of large matrices result in empty or zero-filled cells.

## Time complexities

| Operation                | Complexity |
| ------------------------ | ---------- |
| Access by index (arr[i]) | O(1)       |
| Update by index          | O(1)       |
| Search (unsorted array)  | O(n)       |
| Search (sorted array)    | O(log n)*  |
| Insert at end            | O(1)**     |
| Delete at end            | O(1)       |
| Insert at beginning      | O(n)       |
| Delete at beginning      | O(n)       |
| Insert in middle         | O(n)       |
| Delete from middle       | O(n)       |
| Traverse array           | O(n)       |
| Find min/max             | O(n)       |
| Reverse array            | O(n)       |
| Copy array               | O(n)       |
| Sort array               | O(n log n) |
