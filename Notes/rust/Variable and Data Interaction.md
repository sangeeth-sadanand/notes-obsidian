# Variable and Data Interaction

- The interaction between variables depends on whether the data is stored on the stack or the heap.

## **Move:**

- For heap-allocated data (like the `String` type), assigning one variable to another copies the pointer, length, and capacity on the stack but **does not copy the heap data**. 
- To prevent "double free" errors where two variables try to free the same memory, Rust **invalidates the original variable**. 
- This process is called a **move**.
```rust
let a = vec![1,2,3]
let b = a // Here a move occurs
```
## **Clone:** 

- If you explicitly want a **deep copy** of heap data, you must use the `clone` method, which can be an expensive operation.

## **Copy Trait:** 

- Simple types stored entirely on the stack (integers, booleans, characters) implement the `Copy` trait. 
- For these types, variables remain valid after being assigned to another variable because the values are trivially duplicated