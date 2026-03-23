# String

- The primary string types are `str` and `String`, which differ in how they handle memory and ownership. 
- `&String` is a reference to an owned `String`.
- String start and ends with `“`

###### `str` (String Slice - Dynamically Sized Type)

- **Nature:** `str` is an *unsized* or dynamically sized type (DST), meaning its size is not known at compile time. It represents the actual sequence of UTF-8 bytes. You cannot create a variable of type `str` on its own; it must always be behind a pointer, such as `&str` or `Box<str>`.

- **Ownership:** It does not own the data it points to; it is just a view or slice into memory owned by something else.

- **Mutability:** The underlying data is immutable through a `&str` reference. A `&mut str` allows in-place modifications, but the length cannot change.

- **Usage:** Most commonly used as `&str`, a "fat pointer" (pointer + length). String literals like `"hello"` are of type `&'static str` (a string slice with a static lifetime, embedded in the program's binary).

- **When to use:** Primarily used for read-only views of string data, especially for function arguments, as it is efficient and flexible. 

```rust
fn main() {
    let literal: &str = "Hello, world!"; // Stored in static memory
    println!("{}", literal);
}
```

​    

###### `String` (Owned, Mutable, Heap-Allocated)

- **Nature:** `String` is a growable, owned data type provided by the standard library, similar to `Vec<u8>` but guaranteed to be valid UTF-8. It is a struct containing a pointer, a length, and a capacity.
- **Ownership:** It owns its data, which is stored on the heap. When the `String` goes out of scope, the memory is automatically freed.
- **Mutability:** It is mutable and can be grown or shrunk (e.g., using `push_str()`).
- **Usage:** Used when you need to modify the string content, return an owned string from a function, or store owned string data in a struct.
- **When to use:** Use `String` when you need ownership or the ability to modify the text dynamically.

```rust
fn main() {
    let mut owned_string: String = String::from("Hello"); // Stored on the heap
    owned_string.push_str(", world!"); // Can be modified
    println!("{}", owned_string);
}

```

###### `&String` (Reference to a String)

- **Nature:** This is an immutable reference to a `String` object. It points to the `String` struct on the stack, which in turn manages the heap data.
- **Usage:** It is generally considered an anti-pattern in most scenarios.
- **`Deref` Coercion:** `&String` can be automatically *deref-coerced* by the Rust compiler into a `&str` when passed as a function argument, because `String` implements the `Deref<Target=str>` trait. This makes accepting `&str` in function signatures more flexible, as it can accept both `&str` and `&String` seamlessly.
- **When to use:** There is rarely a need to use `&String` explicitly; functions should typically accept `&str` instead to be more generic and accept both owned `String` references and string literals. 

```rust
fn main() {
    let owned_string = String::from("Hello, Rust!");

    // Get a &str from a String (cheap reference)
    let string_slice: &str = owned_string.as_str(); 
    // Can also use a simple borrow
    let string_slice_deref: &str = &owned_string; 

    // Convert &str to String (allocates new memory)
    let new_owned_string = string_slice.to_string(); 
}
```