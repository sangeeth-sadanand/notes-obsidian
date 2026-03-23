# Ownership

- **Ownership** is Rust’s most unique feature, enabling the language to make **memory safety guarantees without the need for a garbage collector**. 

- It consists of a set of rules that the compiler checks at compile time; if any rule is violated, the program will not compile

## Rules

> [!IMPORTANT]
>
> 1. **Each value in Rust has an owner**.
> 2. **There can only be one owner at a time**.
> 3. **When the owner goes out of scope, the value will be dropped**

```rust
let s1 = String::from("hello");
let s2 = s1; // ownership moves to s2
// s1 is no longer valid here
```

## Ownership and functions

- Passing a value to a function has the same mechanics as assigning a value to a variable. 
- It will either **move or copy** the value into the function's parameters. 
- Similarly, returning a value from a function **transfers ownership** back to the caller.

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope
    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here
    let x = 5;                      // x comes into scope
    makes_copy(x);                  // Because i32 implements the Copy trait,
                                    // x does NOT move into the function,
                                    // so it's okay to use x afterward.
} // Here, x goes out of scope, then s. However, because s's value was moved,
  // nothing special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{some_string}"); 
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{some_integer}");
} // Here, some_integer goes out of scope. Nothing special happens.
```

- Returning values can also transfer ownership.

```rust
fn main() {
    let s1 = gives_ownership();        // gives_ownership moves its return
                                       // value into s1
    let s2 = String::from("hello");    // s2 comes into scope
    let s3 = takes_and_gives_back(s2); // s2 is moved into
                                       // takes_and_gives_back, which also
                                       // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {       // gives_ownership will move its
                                       // return value into the function
                                       // that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                        // some_string is returned and
                                       // moves out to the calling
                                       // function
}

// This function takes a String and returns a String.
fn takes_and_gives_back(a_string: String) -> String {
    // a_string comes into
    // scope

    a_string  // a_string is returned and moves out to the calling function
}
```
