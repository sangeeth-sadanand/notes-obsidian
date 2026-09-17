---
up:
  - "[[003_DetailedNotes/DSA/Data Structures/Data Structures|Data Structures]]"
tags:
index: 2
type: topic
---
- A **stack** is a linear data structure that operates as an ordered list where elements are inserted or deleted from only one end, known as the **top of stack**. 
- The opposite, inactive end is referred to as the **bottom of stack**.
- It operates strictly on the **Last In First Out (LIFO)** principle, meaning that the element inserted last is always the first one to be removed. 
- Common real-world analogies include a pile of bread slices or passengers boarding and alighting from an elevator.

### Abstract Data Type (ADT)

The Abstract Data Type for a stack consists of a finite set of elements of the same data type and supports the following fundamental operations:

- **`CREATE(STACK)`**: Initializes an empty stack and sets up the top pointer.
- **`CHK_STACK_EMPTY(STACK)`**: A boolean check to determine if the stack contains no elements.
- **`CHK_STACK_FULL(STACK)`**: A boolean check to determine if the stack has reached its maximum allocated size (primarily relevant for sequential implementations).
- **`PUSH(STACK, ITEM)`**: Adds a new element (`ITEM`) to the top of the stack.
- **`POP(STACK, ITEM)`**: Removes the element currently occupying the top of the stack and returns it in `ITEM`.

### Implementation Approaches

#### 1. Sequential (Array-Based) Implementation

In a sequential representation, a 1D array `STACK[1:n]` of size `n` is used along with an integer variable `top` (initialized to `0`) that tracks the current top index.

- **Push Algorithm**:
    
    ```
    procedure PUSH(STACK, n, top, item)
        if (top = n) then STACK_FULL;
        else {
            top = top + 1;
            STACK[top] = item;
        }
    end PUSH
    ```
    
    _Before inserting, the program must test for stack overflow (`top = n`)_.
    
- **Pop Algorithm**:
    
    ```
    procedure POP(STACK, top, item)
        if (top = 0) then STACK_EMPTY;
        else {
            item = STACK[top];
            top = top - 1;
        }
    end POP
    ```
    
    _Before deleting, the program must test for stack underflow (`top = 0`). Note that popping merely decrements the `top` index as a mark of deletion without requiring physical erasure of the array memory_.
    
- **Time Complexity**: \(O(1)\) for both push and pop operations.
    
- **Drawbacks**: Fixed, finite capacity (\(n\)) and the mandatory condition check for `STACK_FULL` before every push operation.
    

#### 2. Linked Representation (Linked Stack)

A linked stack resolves fixed-size limitations by using a **singly linked list**, where the start pointer performs the role of the `Top` pointer pointing to the first node of the list.

- **Push Algorithm**:
    
    ```
    procedure PUSH_LINKSTACK(TOP, ITEM)
        call GETNODE(X);
        DATA(X) = ITEM;
        LINK(X) = TOP;
        TOP = X;
    end PUSH_LINKSTACK
    ```
    
    _Dynamic memory allocation (`GETNODE(X)`) creates a new node that becomes the new head of the list. No `STACK_FULL` check is needed because the capacity is non-finite (bounded only by available system memory)_.
    
- **Pop Algorithm**:
    
    ```
    procedure POP_LINKSTACK(TOP, ITEM)
        if (TOP = 0) then call LINKSTACK_EMPTY;
        else {
            TEMP = TOP;
            ITEM = DATA(TOP);
            TOP = LINK(TOP);
        }
        call RETURN(TEMP);
    end POP_LINKSTACK
    ```
    
    _The top node is detached, its data returned, and its memory recycled (`RETURN(TEMP)`) back to the free storage pool_.
    
- **Time Complexity**: \(O(1)\) for both push and pop operations.
    
- **Trade-offs**: Provides conceptual simplicity and flexible capacity, but requires extra memory to store the link/pointer field for every node.
    
### Dynamic Memory Management Application

Linked stacks play a foundational role in low-level memory allocation:

- The system's free storage pool (**`AVAIL_SPACE`**) is maintained as a **linked stack** with a top pointer `AV`.
- When an application requests memory via **`GETNODE()`**, it executes a **pop operation** on `AVAIL_SPACE` to obtain an available node.
- When memory is freed via **`RETURN()`**, it executes a **push operation** on `AVAIL_SPACE` to recycle the node back to the free pool.

### Key Applications of Stacks

1. **Recursive Programming & Call Stacks**: Tracks nested function invocations, parameter values, local variables, and return addresses across call levels (such as during recursive factorial or modulo evaluations).
2. **Expression Handling**: Used in compilers to evaluate postfix expressions and convert infix arithmetic expressions to postfix or prefix forms.
3. **Symbol Balancing**: Validates matching pairs of opening and closing symbols (like `()`, `[]`, `{}`) during syntax checking in compiler design.
4. **Tree & Graph Traversals**: Serves as the underlying mechanism for Depth-First Search (DFS) in graphs and non-recursive tree traversals (preorder, inorder, postorder).

### Special Variants

- **Multiple Stacks in a Single Array**: Two or more stacks can share a single array. For example, two stacks can have their bottoms at opposite extreme ends of an array and grow toward the middle to maximize memory utilization.
- **Double-Linked Stack Reversal**: Stacks implemented via doubly linked lists can be efficiently reversed by swapping the `LLINK` and `RLINK` pointers of each node.

