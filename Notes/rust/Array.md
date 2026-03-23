# Array

- **Fixed Size:** The size is immutable. You cannot add or remove elements after declaration.
- **Contiguous Memory:** Elements are stored sequentially in memory (stack).
- **Homogeneous Types:** All elements must be of the same type (`T`).
- **Type Signature:** The type includes both the element type and length, written as `[T; N]` (e.g., `[i32; 5]`).
- **Compile-time Size:** The size \[N\] must be known at compile time.
- **Safety & Bounds Checking:** Accessing an index out of bounds causes a compile-time error or panic at runtime.
- **Initialization:** Can be initialized by listing elements `[1, 2, 3]` or repeating a value `[val; size]`.
- **Iterating:** Can be iterated over using `for x in array.iter()`

```rust
// 1. Explicit type and length [Type; Size]
let numbers: [i32; 5] = [1, 2, 3, 4, 5];

// 2. Implicit inference
let fruits = ["Apple", "Banana", "Orange"];

// 3. Initialize with default value: [Value; Size]
let zeroes = [0; 10]; // An array of ten 0s

// 4. Accessing elements (0-indexed)
let first = numbers[0];
println!("First number: {}", first);

// 5. Mutating elements
let mut mutable_array = [1, 2, 3];
mutable_array[1] = 10;
```
