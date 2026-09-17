---
up:
  - "[[003_DetailedNotes/DSA/Data Structures/Data Structures|Data Structures]]"
tags:
index: 3
type: topic
---
- A **queue** is a fundamental linear data structure in computer science that operates on the **First-In First-Out (FIFO)** or **First-Come First-Served (FCFS)** principle. 
- In a queue, all insertions take place at one end called the **rear** (or tail), while all deletions occur at the opposite end called the **front** (or head).

## 1. Fundamental Operations & Abstract Data Type (ADT)

The core operations defined on a queue include:

- **ENQUEUE / INSERTQ**: Adds a new element to the rear end of the queue.
- **DEQUEUE / DELETEQ**: Removes and returns the element at the front end of the queue.
- **CHK_QUEUE_EMPTY**: A Boolean function that checks whether the queue contains any elements.
- **CHK_QUEUE_FULL**: A Boolean function that checks whether a sequential queue has reached its maximum capacity.

In an **Abstract Data Type (ADT)** specification, a queue focuses on _what_ operations are performed rather than implementation details.

## 2. Sequential (Array-Based) Linear Queues

In an array-based implementation, a queue is accommodated in a one-dimensional array `Q[1:n]` of capacity `n`, tracked using two pointer variables, `FRONT` and `REAR`, both initialized to `0`.

### Mechanics and Pointer Movement

- **Positioning**: The `FRONT` pointer physically points to one position _before_ the actual front element, while `REAR` points directly to the position of the last inserted element.
- **Insertion (Algorithm 5.1)**: Increments `REAR` by 1 (`REAR = REAR + 1`) and stores `ITEM` at `Q[REAR]`. It requires checking for `REAR = n` (**QUEUE_FULL**) beforehand.
- **Deletion (Algorithm 5.2)**: Increments `FRONT` by 1 (`FRONT = FRONT + 1`) and retrieves the element at `Q[FRONT]`. It checks for `FRONT = REAR` (**QUEUE_EMPTY**) beforehand.

Both insertion and deletion in a linear queue execute in **\(O(1)\) time complexity**.

### Drawback: False Overflow

A major limitation of a linear queue is **false overflow**. As deletions occur, `FRONT` moves forward toward `REAR`. When `REAR` reaches `n` (`REAR = n`), any further `ENQUEUE` attempt triggers a `QUEUE_FULL` condition, even if previous deletions have freed up substantial empty slots at the beginning of the array. Moving array elements forward upon every deletion to reclaim space would require \(O(n)\) data shifting, making queue maintenance inefficient.

## 3. Circular Queues

To solve the false overflow problem without costly data movement, **circular queues** conceptualize the linear array as a continuous ring using modulo arithmetic.

### Mechanics and Pointer Movement

- **Declaration**: Declared over an array `CIRC_Q[0:n-1]` with an effective physical capacity of \(n - 1\) elements. Both `FRONT` and `REAR` are initialized to `0`.
- **Circular Insertion**: `REAR` advances clockwise via `REAR = (REAR + 1) mod n`.
- **Circular Deletion**: `FRONT` advances clockwise via `FRONT = (FRONT + 1) mod n`.
- **Conditions**:
    - **QUEUE_EMPTY**: Satisfied when `FRONT = REAR`.
    - **QUEUE_FULL**: Checked when advancing `REAR` causes `(REAR + 1) mod n == FRONT` (or `FRONT == REAR` after incrementing).

Circular queue insertion and deletion both retain an **\(O(1)\) time complexity** while fully utilizing array storage.

## 4. Linked Queues (Dynamic Representation)

A **linked queue** overcomes all sequential array limitations (such as fixed size and overflow checking) by implementing the queue as a singly linked list.

### Mechanics and Node Pointers

- **Structure**: Uses two pointers: `FRONT` pointing to the first node of the list and `REAR` pointing to the last node.
- **Insertion**: Allocates a new node via `GETNODE(X)`, attaches it after `REAR` (`LINK(REAR) = X`), and updates `REAR = X`. No `QUEUE_FULL` test is needed because capacity is non-finite (bounded only by available system memory).
- **Deletion**: Removes the node at `FRONT`, advances `FRONT = LINK(FRONT)`, and returns the deleted node to the memory pool via `RETURN(TEMP)`. If `FRONT == NIL`, it triggers `QUEUE_EMPTY`.

### Comparison: Array vs. Linked Representations

- **Advantages**: Conceptual simplicity, dynamic allocation, non-finite capacity, and complete elimination of overflow/circular array management.
- **Disadvantages**: Extra memory overhead required to store explicit pointer/link fields in each node.


## 5. Queue Variants

### Priority Queues

In a **priority queue**, elements are inserted or removed based on an assigned priority factor rather than purely by arrival time. When items share the same priority, they follow standard FIFO order.

**Implementation Strategies**:

1. **Cluster / Multiple Queues**: Maintains an individual queue for each priority level. Deletion removes elements from the highest-priority non-empty queue. Insertion takes \(O(1)\) time, but deletion requires checking higher-priority queues.
2. **Sorted Queue**: Uses a single queue where elements are kept sorted by priority. Insertion requires sorting (\(O(n \log n)\) or \(O(n)\) time), while deletion from the front takes \(O(1)\) time.
3. **Two-Dimensional Array**: Uses an array `PRIO_QUE[1:m, 1:p]`, where \(m\) represents priority levels and \(p\) represents capacity per priority level.

### Deques (Double-Ended Queues)

A **deque** (pronounced _"deck"_) allows insertions and deletions at **both** ends (referred to as the `LEFT` and `RIGHT` ends). It serves as a general structure encompassing both stacks and queues.

- **Input-Restricted Deque**: Permits insertions at one end only, but deletions from both ends.
- **Output-Restricted Deque**: Permits insertions at both ends, but deletions from one end only.

## 6. Primary Applications of Queues

1. **Job Scheduling in Time-Sharing Systems**:
    - **Linear Queues**: Used by CPU schedulers in Round-Robin fashion to allocate fixed time slices across multiple active user processes.
    - **Priority Queues**: Manages job streams with differing priorities (e.g., Real-Time jobs > Online processing > Batch processing).
2. **Polynomial Representation and Manipulation**: Polynomials can be represented as **traversable queues** sorted by exponent. Performing additions simply appends combined terms to the rear of the resulting queue.
3. **Graph Traversal (Breadth-First Search / BFT)**: BFS uses a queue to track visited vertices whose adjacent neighboring nodes are waiting to be explored.
4. **Radix / Bin Sorting**: Distributes numbers into array-based linked queues (bins) based on digit values pass-by-pass without disturbing relative ordering.

