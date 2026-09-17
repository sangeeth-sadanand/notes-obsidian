---
up:
  - "[[003_DetailedNotes/DSA/Data Structures/Data Structures|Data Structures]]"
tags:
index: 4
type: topic
---
- **Linked lists** are fundamental linear data structures composed of a collection of **nodes**, where each node contains **DATA** field(s) to store information and **LINK** (pointer) field(s) to store memory addresses of adjacent or associated nodes.
## 1. Merits of Linked Lists over Sequential Data Structures (Arrays)

Sequential data structures such as arrays suffer from key limitations:

- **Inefficient Insertions/Deletions**: Inserting or deleting elements in an array requires extensive shifting or recopying of neighboring elements into temporary arrays, which is computationally expensive.
- **Memory Fragmentation**: Arrays require contiguous memory blocks, leaving fragmented free memory spaces unused if a sufficiently large contiguous block is unavailable.

Linked lists address these drawbacks:

- **No Data Movement**: Insertion and deletion merely require updating pointers rather than moving actual data elements.
- **Flexible Memory Utilization**: Nodes do not need to be physically contiguous in memory; they can be distributed across available storage and connected logically via pointer addresses.

## 2. Basic Node Architecture and Dynamic Memory Operations

A node structure consists of:

- **DATA Field(s)**: Holds the value or information content.
- **LINK / Pointer Field(s)**: Holds the address of neighboring nodes.
- **START / HEAD Pointer**: Keeps track of the address of the initial node in the list. A null link (`NIL`, `0`, or ground symbol) indicates the end of a list.

### Memory Management Functions

Dynamic memory management relies on two key procedures:

- **`GETNODE(X)`**: Obtains/allocates an available node address \(X\) from the free storage pool (`AVAIL_SPACE`).
- **`RETURN(X)`**: Releases node \(X\) back to `AVAIL_SPACE` when it is disposed.
- **Available Space Pool**: `AVAIL_SPACE` itself is maintained as a **linked stack**, where `GETNODE()` acts as a `POP` operation and `RETURN()` acts as a `PUSH` operation.

## 3. Categories of Linked Lists

![[005_assets/DSA/LinkedListTypes.png]]

### A. Singly Linked Lists

- **Structure**: Each node contains one or more data fields but only a **single link field** (`LINK`) pointing to its immediate successor.
- **Operations**:
    - **Insertion**: To insert a node \(X\) to the right of node \(NODE\), the links are updated as `LINK(X) = LINK(NODE)` and `LINK(NODE) = X`.
    - **Deletion**: Updates the predecessor's link field to point directly to the successor of the node being removed.
- **Limitation**: Unidirectional movement makes accessing a node's predecessor difficult.

### B. Circularly Linked Lists

- **Structure**: Replaces the null pointer of the last node with the address of the first node, forming a closed loop.
- **Advantages**: Allows traversal to any node starting from any given node and enables easier determination of predecessors for deletion.
- **Head Node Solution**: To prevent infinite loops during traversal, a special **head node** (`HEAD`) is introduced. An empty circular list is represented when `LINK(HEAD) = HEAD`.

### C. Doubly Linked Lists

- **Structure**: Each node contains **two link fields**: `LLINK` (pointing to the left predecessor) and `RLINK` (pointing to the right successor).
- **Advantages**:
    - Supports bidirectional (forward and backward) traversal.
    - Deletion of node \(X\) requires only knowing node \(X\) itself without needing a separate search for its predecessor (`RLINK(LLINK(X)) = RLINK(X)` and `LLINK(RLINK(X)) = LLINK(X)`).
- **Disadvantage**: Requires extra memory overhead for maintaining two pointer fields per node.

### D. Multiply Linked Lists

- **Structure**: Nodes possess multiple data fields and **multiple link fields**, forming a network of interconnected sublists.
- **Applications**: Used when nodes participate in multiple logical associations simultaneously (e.g., student records linked by department, sports membership, and day-student status), or for representing **sparse matrices** using row and column lists.

### E. Unrolled Linked Lists

- **Structure**: A hybrid variant where each node houses a **static array of elements**, a field recording the current element count (`NUMBER_OF_ELEMENTS`), and a `LINK` pointer.
- **Benefits**: Combines the low memory overhead and cache performance of arrays with the fast insertion/deletion of linked lists. Nodes are kept above a minimum storage utilization threshold (e.g., 50% full) through node splitting and merging operations.

### F. Self-Organizing Lists

- **Purpose**: Reorganizes list elements to move frequently accessed nodes closer to the front, optimizing retrieval times.
- **Methods**:
    1. **Count Method**: Tracks retrieval frequency in an extra `COUNT` field and sorts nodes in descending order.
    2. **Move to Front Method**: Pushes any retrieved node directly to the head of the list.
    3. **Transpose Method**: Swaps a retrieved node with its immediate predecessor.



## 4. Specialized Linear Structures and Applications

- **Linked Stacks and Queues**:
    - A **linked stack** uses a singly linked list where the `START` pointer acts as `Top` (Last-In-First-Out).
    - A **linked queue** uses `Front` and `Rear` pointers (First-In-First-Out).
    - **Benefit**: Provides dynamic, non-finite capacity, eliminating the need for `STACK_FULL` or `QUEUE_FULL` checks required by array implementations.
- **Key Applications**:
    - **Polynomial Addition**: Polynomial terms are represented as singly linked lists or linear queues ordered by exponent powers.
    - **Sparse Matrix Representation**: Uses multiply linked lists with `RIGHT` and `DOWN` pointers to store non-zero elements efficiently.
    - **Symbol Balancing**: Compilers use linked stacks to verify balanced parentheses and delimiters.
