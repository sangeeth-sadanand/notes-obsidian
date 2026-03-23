# Vector

- **Contiguous Memory:** Elements are stored next to each other on the heap, allowing for fast index-based access.
- **Dynamic Size:** Unlike fixed-size arrays, vectors can grow (using `push()`) or shrink (using `pop()`) at runtime.
- **Type Consistency:** All elements in a vector must be of the same data type (`Vec<T>`).
- **Heap Allocation:** Data is stored on the heap, allowing the size to change, while the vector handle (pointer, length, capacity) lives on the stack.
- **Capacity and Reallocation:** When a vector outgrows its reserved memory, it automatically reallocates a larger block on the heap, moves the elements, and updates the pointer.
- **Mutable/Immutable:** Created with `let` they are immutable; `let mut` is required to change their size

```rust
// 1. Create a vector (using macro)
let mut fruits = vec!["apple", "banana"];

// 2. Add elements (dynamic growth)
fruits.push("orange");

// 3. Access elements by index
println!("First fruit: {}", fruits[0]); // apple

// 4. Iterate over the vector
for fruit in &fruits {
    println!("{}", fruit);
}

// 5. Remove an element
fruits.pop(); // removes "orange"
```