# Stack and heap

- In Rust, values can be stored either on the **stack** (fast, fixed-size memory) or the **heap** (dynamic, flexible memory). 
- The stack is used for simple, known-size data like integers, while the heap is used for complex or variable-sized data like `String` or `Vec`. 

## Stack

- **Definition:** A region of memory that stores values in a *Last In, First Out (LIFO)* manner.

- **Characteristics:**

    - Stores **fixed-size data** known at compile time.
    - Very fast allocation and deallocation.
    - Data is automatically removed when it goes out of scope.

- **Examples:**

    ```rust
    fn main() {
        let x = 42; // stored on the stack
        let y = true; // also on the stack
    }
    ```

## Heap

- **Definition:** A region of memory for dynamically allocated data.

- **Characteristics:**

    - Stores **variable-size data** or data whose size is unknown at compile time.
    - Requires a pointer on the stack to reference the heap data.
    - Slightly slower than stack due to allocation overhead.

- **Examples:**

    ```rust
    fn main() {
        let s = String::from("Hello, Rust!"); // data stored on the heap
        let v = vec![1, 2, 3]; // vector contents on the heap
    }
    ```
