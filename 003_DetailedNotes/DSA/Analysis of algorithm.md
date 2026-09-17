---
up:
  - "[[001_SummaryNote/DSA|DSA]]"
tags:
index: 1
type: short_topic
---
# Analysis of algorithm
## 1. Introduction to Algorithms and Performance Evaluation

- An **algorithm** is defined as a finite sequence of instructions, each having a clear meaning, that can be performed with a finite amount of effort in a finite length of time. 
- Good program design requires that algorithm design go hand in hand with appropriate data structures to achieve efficient problem solving.

### Core Properties of Algorithms

To be classified as an algorithm, a procedure must satisfy five fundamental properties:

- **Finiteness:** The algorithm must terminate after a finite number of steps.
- **Definiteness:** Every step must be precisely defined and unambiguous.
- **Generality:** The algorithm must solve all problem instances belonging to a particular class.
- **Effectiveness:** All operations must be basic enough to be performed manually with paper and pencil.
- **Input/Output:** It accepts specific inputs and produces definite outputs.

## 2. Fundamentals of Algorithm Analysis

Analyzing an algorithm involves evaluating its resource consumption on two primary scales:

- **Time Complexity:** A function measuring the running time required by an algorithm to execute to completion.
- **Space Complexity:** A function measuring the total memory space required by the algorithm during execution.

### Apriori Analysis vs. Posteriori Testing

The efficiency of algorithms can be evaluated using two distinct paradigms:

|Characteristic|Apriori (Theoretical) Analysis|Posteriori (Empirical) Testing|
|:--|:--|:--|
|**Approach**|Mathematical determination of required resources prior to execution.|Involves coding the complete algorithm and timing its execution on a computer.|
|**Machine Dependence**|**Machine-independent**; uses statement execution frequency counts.|**Machine-dependent**; influenced by hardware architecture, CPU clock, and compiler.|
|**Scope of Input**|Can evaluate performance across arbitrarily large input sizes $n$.|Restricted to practical, moderate-sized test instances.|

### Frequency Count Method

In **apriori analysis**, the executing time is estimated by calculating the **frequency count** ($f_i$), which is the number of times each statement $i$ in an algorithm is executed. The total frequency count $T(n) = \sum f_i$ is expressed as a function of the input size $n$.

- **Single Loop Statement:** A loop header `for i = 1 to n` executes $(n + 1)$ times, while the body inside the loop executes $n$ times.
- **Nested Loops:** In a double nested loop `for j = 1 to n` and `for k = 1 to n`, inner statements execute $n^2$ times, yielding a total count of $3n^2 + 3n + 1$, which simplifies to $O(n^2)$.

## 3. Asymptotic Notations

**Asymptotic notations** are mathematical tools used in apriori analysis to approximate the growth rate of an algorithm's time or space complexity as the input size $n$ approaches infinity.

### Primary Asymptotic Notations

The three primary notations are:

- **Big O ($O$)**: Represents the **upper bound**. It describes the worst-case scenario or the maximum growth rate of an algorithm, ensuring the function grows no faster than a specified rate.
    
- **Big Omega ($\Omega$)**: Represents the **lower bound**. It describes the best-case scenario or the minimum growth rate, showing the bare minimum time or space an algorithm requires.
    
- **Big Theta ($\Theta$)**: Represents a **tight bound**. It applies when the upper and lower bounds are the same, meaning the algorithm's growth rate is precisely sandwiched between two constant multiples of the same function.
    
Two secondary notations are also sometimes used: **Little o ($o$)** for a strict upper bound (growing strictly slower) and **Little omega ($\omega$)** for a strict lower bound (growing strictly faster).

### Rules of Asymptotic Calculations

Here is a quick-reference summary of the **Rules of Asymptotic Calculations** in tabular form:

| Rule Name                | Mathematical Representation                   | Plain English Explanation                                                           | Example                                  |
| ------------------------ | --------------------------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------- |
| **Coefficient Rule**     | $O(c \cdot f(n)) = O(f(n))$                   | Constant factors do not affect growth rate and can be dropped.                      | $O(5n^2) \rightarrow O(n^2)$             |
| **Sum Rule**             | $O(f(n) + g(n)) = O(\max(f(n), g(n)))$        | Sequential operations are dominated by the fastest-growing term.                    | $O(n + n^2) \rightarrow O(n^2)$          |
| **Product Rule**         | $O(f(n) \cdot g(n))$                          | Nested structures (like loops inside loops) multiply their complexities.            | $O(n) \times O(n) \rightarrow O(n^2)$    |
| **Transitivity**         | If $f = O(g)$ and $g = O(h)$, then $f = O(h)$ | Asymptotic relationships chain together across functions.                           | $O(n) \subseteq O(n^2) \subseteq O(n^3)$ |
| **Polynomial Dominance** | Lower-order terms are ignored                 | Higher-order terms, exponentials, and factorials completely overshadow lower terms. | $O(3n^3 + 5n + 10) \rightarrow O(n^3)$   |

### Hierarchy of Asymptotic Growth Rates

The **Hierarchy of Asymptotic Growth Rates** ranks common complexity classes from the slowest-growing (most efficient) to the fastest-growing (least efficient). Understanding this hierarchy helps you predict how an algorithm will perform as the input size ($n$) scales up.

#### Growth Rate Hierarchy Table

| Complexity Class | Name         | Description                                                                              | Practical Example                                       |
| ---------------- | ------------ | ---------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| **O(1)**         | Constant     | Execution time remains constant regardless of input size.                                | Accessing an array element by index.                    |
| **O(log n)**     | Logarithmic  | Growth scales logarithmically; highly efficient for large inputs.                        | Binary search on a sorted array.                        |
| **O(n)**         | Linear       | Execution time grows directly in proportion to the input size.                           | Finding an item in an unsorted list (linear search).    |
| **O(n log n)**   | Linearithmic | Slightly worse than linear, but standard for efficient sorting.                          | Merge sort, Quick sort (average case).                  |
| **O(n^2)**       | Quadratic    | Growth scales with the square of the input; inefficient for large data.                  | Nested loops (bubble sort, selection sort).             |
| **O(n^3)**       | Cubic        | Scales with the cube of the input; practical only for very small datasets.               | Standard matrix multiplication, 3-sum problem variants. |
| **O(2^n)**       | Exponential  | Doubling the input size adds a constant multiplicative factor to time; explodes rapidly. | Recursive Fibonacci calculation without memoization.    |
| **O(n!)**        | Factorial    | Extremely fast growth rate; unusable for anything beyond tiny inputs.                    | Traveling Salesperson Problem using brute-force search. |
![[005_assets/DSA/TimeComplexityGraph.png]]
#### Mathematical Scale Progression

Expressed mathematically as a chain of inequalities for large values of $n$:
  
$$\mathcal{O}(1) \lt \mathcal{O}(\log n) \lt \mathcal{O}(n) \lt \mathcal{O}(n \log n) \lt \mathcal{O}(n^2) \lt \mathcal{O}(n^3) \lt \mathcal{O}(2^n) \lt \mathcal{O}(n!)$$

## 4. Input Instance Cases (Best, Worst, and Average Cases)

The running time of an algorithm depends not only on the size of the input $n$ but also on the structural nature of the input data.

- **Worst-Case Complexity:** The maximum execution time across all input instances of size $n$. Crucial in safety-critical systems (e.g., nuclear power plant controllers) where maximum response time guarantees are mandatory.
- **Best-Case Complexity:** The minimum execution time across all input instances of size $n$.
- **Average-Case Complexity:** The expected running time over typical or randomly distributed input instances. Mathematically intricate, requiring probability distribution modeling over input instances.

## 5. Amortized Analysis

- **Amortized analysis** is a technique used to determine the **average cost per operation** over a sequence of multiple operations.
- Unlike standard worst-case analysis—which looks at the absolute maximum cost of a single operation in isolation—amortized analysis looks at the big picture. 
- It proves that even if a rare operation is extremely expensive, it happens so infrequently that the _average_ cost across a whole sequence remains very low.

### Why Use Amortized Analysis?

- Standard worst-case analysis can be overly pessimistic. 
- For example, if a data structure usually takes $O(1)$ time per operation, but occasionally triggers an expensive cleanup or resizing operation that takes $O(n)$ time, a naive worst-case analysis would label _every_ operation as $O(n)$. 
- Amortized analysis proves that the high cost is "paid for" by preceding cheap operations.

### The Three Main Techniques

1. **Aggregate Analysis:**
    - **How it works:** You compute the total worst-case cost of a sequence of $n$ operations and divide it by $n$ to find the average cost per operation.
    - **Use case:** Great when every operation in the sequence has a similar nature, and you can sum up the total bounds directly.
        
2. **The Accounting Method:**
    - **How it works:** You assign an **amortized cost** to each operation. Operations that take less time than their amortized cost build up a "credit balance." When an expensive operation occurs, it uses up the accumulated credits to pay for its execution.
        
    - **Rule:** The total accumulated credit must never become negative.
        
3. **The Potential Method:**
    - **How it works:** Similar to the accounting method, but instead of "credits," you use a **potential function** ($\Phi$) that maps the data structure's state to a numerical value (representing stored energy). The amortized cost is the actual cost plus the change in potential.
        
### Classic Example: Dynamic Array Resizing 

- **The Operation:** Appending an element to an array.
    
- **The Cost:** Most of the time, appending takes **$O(1)$** time because there is empty space at the end of the array. However, when the array is full, appending triggers a resize: the array allocates a new block of memory twice the size ($2n$) and copies all $n$ old elements over, which takes **$O(n)$** time.
    
- **Amortized Analysis Result:** Even though a resize costs $O(n)$, it happens so rarely (only when the size doubles) that if you spread that cost across all the cheap $O(1)$ inserts leading up to it, the **amortized cost per append is $O(1)$**.
    
