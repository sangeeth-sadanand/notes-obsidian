# 1. Array

An array stores elements in contiguous memory locations.

Example:

[10, 20, 30, 40]

### Basic Operations

|Operation|Description|Time Complexity|
|---|---|---|
|Access|Get element by index|O(1)|
|Update|Change value at index|O(1)|
|Search|Find an element|O(n)|
|Insert (end)|Add element at end|O(1)*|
|Insert (middle)|Shift elements and insert|O(n)|
|Delete|Remove element and shift|O(n)|

Example:

arr = [10, 20, 30]  

  

arr.append(40) # Insert  

arr[1] = 25 # Update  

x = arr[0] # Access  

arr.remove(25) # Delete

---

# 2. Linked List

A collection of nodes where each node contains:

- Data
- Pointer/reference to next node

Example:

10 → 20 → 30 → NULL

### Basic Operations

|Operation|Time Complexity|
|---|---|
|Access|O(n)|
|Search|O(n)|
|Insert at Beginning|O(1)|
|Insert at End|O(n)|
|Delete at Beginning|O(1)|
|Delete by Value|O(n)|

Advantages:

- Dynamic size
- Efficient insertion/deletion

---

# 3. Stack (LIFO)

Last In, First Out

Example:

Top  

30  

20  

10

### Basic Operations

|Operation|Description|Complexity|
|---|---|---|
|Push|Insert element|O(1)|
|Pop|Remove top element|O(1)|
|Peek/Top|View top|O(1)|
|isEmpty|Check empty|O(1)|

Example:

stack = []  

  

stack.append(10) # Push  

stack.append(20)  

stack.pop() # Pop

Applications:

- Function calls
- Undo operations
- Expression evaluation

---

# 4. Queue (FIFO)

First In, First Out

Example:

Front → 10 → 20 → 30 → Rear

### Basic Operations

|Operation|Complexity|
|---|---|
|Enqueue|O(1)|
|Dequeue|O(1)|
|Front/Peek|O(1)|
|isEmpty|O(1)|

Example:

from collections import deque  

  

q = deque()  

  

q.append(10) # Enqueue  

q.append(20)  

q.popleft() # Dequeue

Applications:

- Scheduling
- Print queues
- BFS traversal

---

# 5. Hash Table / Hash Map

Stores data as:

Key → Value

Example:

{  

"id": 101,  

"name": "John"  

}

### Basic Operations

|Operation|Average Complexity|
|---|---|
|Insert|O(1)|
|Search|O(1)|
|Delete|O(1)|
|Update|O(1)|

Example:

student = {}  

  

student["id"] = 101  

student["name"] = "John"  

  

print(student["name"])

Applications:

- Database indexing
- Caching
- Symbol tables

---

# 6. Tree

Hierarchical structure.

Example:

10  

/ </span>  

5 20

### Basic Operations

|Operation|Complexity (BST Average)|
|---|---|
|Search|O(log n)|
|Insert|O(log n)|
|Delete|O(log n)|
|Traversal|O(n)|

Common Traversals:

- Inorder
- Preorder
- Postorder
- Level Order

Applications:

- File systems
- XML/HTML representation
- Database indexing

---

# 7. Binary Search Tree (BST)

Special tree:

Left < Root < Right

Example:

15  

/ </span>  

10 20

### Operations

|Operation|Average|
|---|---|
|Search|O(log n)|
|Insert|O(log n)|
|Delete|O(log n)|

Worst Case:

10  

</span>  

20  

</span>  

30

Complexity becomes:

O(n)

---

# 8. Heap

Complete binary tree used for priority management.

### Max Heap

50  

/ </span>  

30 40

### Operations

|Operation|Complexity|
|---|---|
|Insert|O(log n)|
|Delete Max/Min|O(log n)|
|Peek|O(1)|

Applications:

- Priority Queue
- Scheduling
- Heap Sort

---

# 9. Graph

Collection of:

- Vertices (Nodes)
- Edges (Connections)

Example:

A --- B  

| |  

C --- D

### Basic Operations

|Operation|Complexity|
|---|---|
|Add Vertex|O(1)|
|Add Edge|O(1)|
|DFS|O(V + E)|
|BFS|O(V + E)|
|Search|O(V + E)|

Applications:

- Social Networks
- Maps
- Recommendation Systems

---

# 10. Set

Stores unique elements only.

Example:

{1, 2, 3, 4}

### Basic Operations

|Operation|Complexity|
|---|---|
|Add|O(1)|
|Remove|O(1)|
|Search|O(1)|
|Union|O(n)|
|Intersection|O(n)|

Example:

s = {1, 2, 3}  

  

s.add(4)  

s.remove(2)

---

# Quick Revision Table

|Data Structure|Access|Search|Insert|Delete|
|---|---|---|---|---|
|Array|O(1)|O(n)|O(n)|O(n)|
|Linked List|O(n)|O(n)|O(1)*|O(1)*|
|Stack|O(n)|O(n)|O(1)|O(1)|
|Queue|O(n)|O(n)|O(1)|O(1)|
|Hash Map|N/A|O(1)|O(1)|O(1)|
|BST|O(log n)|O(log n)|O(log n)|O(log n)|
|Heap|O(n)|O(n)|O(log n)|O(log n)|
|Graph|O(V)|O(V+E)|O(1)|O(1)|

### Core Operations to Remember

Every data structure mainly supports:

1. **Traversal** – Visit elements
2. **Insertion** – Add element
3. **Deletion** – Remove element
4. **Searching** – Find element
5. **Sorting** – Arrange elements
6. **Access/Update** – Read or modify data

For DSA interview preparation, focus first on **Arrays → Linked Lists → Stacks → Queues → Hash Maps → Trees → Heaps → Graphs**, as they form the foundation for most coding problems.