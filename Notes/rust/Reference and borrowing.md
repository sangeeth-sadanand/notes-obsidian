# Reference and borrowing

- A **reference** in Rust is a type of pointer that contains an address in memory referring to data owned by another variable. 
- Indicated by the **ampersand symbol** `&`, references allow multiple parts of your code to access the same piece of data without needing to copy that data into memory multiple times

## Core Concepts

- **Borrowing:** 
    - The action of creating a reference is called borrowing. 
    - Just like in real life, when you borrow something, you do not own it, and you must give it back when you are finished.
    
```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{s1}' is {len}.");
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

- **Immutability by Default:** 
    - Like variables, references are **immutable by default**. 
    - This means you cannot modify the data you are borrowing unless you explicitly create a mutable reference.
```rust
fn main() {
	let mut s = String::from("Hello");
	// Create a mutable reference
	let r = &mut s;
	// Use the mutable reference to change the value
	r.push_str(", world!");
	println!("{}", r); // prints: Hello, world!
}
```
- **De-referencing:** 
    - The opposite of referencing is de-referencing, which uses the **de-reference operator** `*` to follow a pointer and access the actual value stored at the address.

## The Rules of References

To ensure memory safety, the Rust compiler enforces two strict rules for references at any given time:

> [!IMPORTANT]
>
> 1. You can have **either** one mutable reference **or** any number of immutable references to a single piece of data.
>
> 2. References must always be **valid**.
