# Data types and memory

- The choice between stack and heap depends primarily on ==whether the data size is known at **compile time**==.
## Types on the Stack

The stack is used for data with a **fixed, known size**. These types are generally "Copy" types, meaning they are duplicated rather than moved. 

- **Scalar Primitives**: All fixed-size integers (`i32`, `u64`, etc.), floating-point numbers (`f32`, `f64`), booleans (`bool`), and characters (`char`).
- **Compound Primitives**: Tuples and arrays (`[T; N]`) that contain only other stack-allocated types.
- **Pointers & Metadata**: The "bookkeeping" for heap types (like a `String`'s pointer, length, and capacity) actually lives on the stack.
- **Function Local Variables**: Any variable declared with `let` inside a function scope is allocated on that function's stack frame by default. 

## Types on the Heap

The heap is used for data that is **dynamically sized** or needs to live beyond the scope of a single function. 
- **Grow-able Collections**: `String` and `Vec<T>` store their actual contents on the heap because they can change size during runtime.
- **Smart Pointers**: `Box<T>` explicitly puts a value on the heap that would otherwise be on the stack.
- **Shared Data**: Types like `Rc<T>` or `Arc<T>` use the heap to allow multiple owners to access the same data.
- **Map-based Collections**: `HashMap` and `HashSet` and `HashSet` are heap-allocated to manage their complex internal structures.