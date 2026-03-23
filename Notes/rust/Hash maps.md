# Hash maps

- The hash map stores as key value pair. 
- **Key-Value Storage:** Data is accessed not by an integer index, but by a unique key.
- **Homogeneous Types:** All keys must share the same type, and all values must share the same type.
- **Ownership:** The `HashMap` takes ownership of the keys and values inserted into it for types that don't implement the `Copy` trait (like `String`).
- **Required Traits for Keys:** Any type used as a key must implement the `Eq` and `Hash` traits. The `PartialEq`, `Eq`, and `Hash` traits can often be derived automatically for custom structs using `#[derive(PartialEq, Eq, Hash)]`.
- **Unordered:** The order in which elements are inserted is not guaranteed or preserved when iterating over the map.
- **Efficient Hashing (Security by Default):** By default, Rust uses the cryptographically secure SipHash function, which is resistant to Denial of Service (DoS) attacks, trading some performance for better security.
- **Dynamic Resizing:** The map can grow and shrink in size as needed to accommodate varying data loads.
- **`Entry` API:** A powerful API (`entry().or_insert()`) allows for concise conditional logic, such as inserting a value only if the key is not already present, or updating a value based on its old value.

```rust
use std::collections::HashMap;

fn main() {
    // Create a new HashMap
    let mut scores = HashMap::new();

    // Insert key-value pairs
    scores.insert(String::from("Alice"), 50);
    scores.insert(String::from("Bob"), 30);

    // Access values
    let alice_score = scores.get("Alice");
    match alice_score {
        Some(score) => println!("Alice's score: {}", score),
        None => println!("Alice not found"),
    }

    // Iterate over key-value pairs
    for (name, score) in &scores {
        println!("{}: {}", name, score);
    }

    // Update a value
    scores.insert(String::from("Bob"), 40);

    // Check if a key exists
    if scores.contains_key("Charlie") {
        println!("Charlie is in the map");
    } else {
        println!("Charlie is not in the map");
    }
}

```