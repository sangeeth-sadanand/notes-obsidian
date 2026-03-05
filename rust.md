# Rust

## Introduction

- **Rust** is a modern, statically compiled programming language focused on **performance, reliability, and safety**.
- It was first released in **2010** and has since become one of the fastest-growing languages in systems programming.
- Rust is often compared to **C and C++**, but it eliminates many of their common pitfalls, especially around memory management.

### Key Features

- **Memory Safety without Garbage Collection** 
- **Concurrency Made Safe** 
- **Performance** 
- **Rich Tooling** 

## Cargo

- Cargo is Rust’s official **build system and package manager**
- Core functions and features
   - Dependency Management
   - Reproducible Builds
   - Project Organization
   - Release Profiles

### Create new project

- Creates a new directory and project with the specified name

```bash
cargo new <project_name>
```

- This file is in the TOML format and serves as the project's configuration file
- It also creates a src folder with main.rs file.

### Build a project

- Compiles the current project and places the executable in the target/debug directory

```bash
cargo build [--release]
```

- `--release` is need for building release version. 

###  Run the project

- Compiles the code and then runs the resulting executable in one step

```bash
cargo run
```

### Check the project

- Quickly checks the code to ensure it compiles without producing an executable, which is often much faster than a full build

```bash
cargo check
```

### Run the test cases

- Compiles and runs the project's test suite

```bash
cargo test
```

### Update the cargo package

- Ignores the `Cargo.lock` file and figures out the latest versions of dependencies that fit the specifications in `Cargo.toml`

```bash
cargo update
```

### Show documentation

- Builds documentation provided by all of your dependencies locally and opens it in a browser

```bash
cargo doc --open
```

### Publish crate

Uploads your crate to crates.io so others in the community can use it

```bash
cargo publish
```

> [!TIP]
>
> You can find the new crate on [crates.io](https://crates.io/) and find the document on [docs](https://docs.rs/)

### Adding a new crate

- Add the dependency crate to `cargo.toml` file under `[dependency]`.

```toml
[dependencies]
rand = "0.8.5"
```

Or can use

```bash
cargo add <crate_name>
```

- This will add the dependency automatically.

Then build the cargo.

## Common programming concepts

### Variables and Mutability

- In Rust, **variables are immutable by default**
- `let` keyword is used to bind a value to a variable. 

```rust
let x = 5;
```

- To make the variable mutable we need to add the `mut` keyword

```rust
let mut x = 5;
x = 10;
```

#### Constants

- Constants are **always immutable**
- `const` keyword is used to define constants

```rust
const PI = 3.1416
```

- Type must always be annotated.
- Constants value should me known at compile time. It cannot be assigned at run time.
- It can be declared at global scope as well.
- Use uppercase naming convention 

#### Shadowing

- Shadowing allows to declare a new variable with the same name as a previous variable. 

- This shadowed variable can be used until either it is shadowed again or its scope ends.

- `let` keyword is used to shadow a variable.

- In shadowing the datatype and value can be re-assigned.

- If you shadow a variable within an inner scope (created by nested curly brackets), the shadowing remains in effect until that inner scope ends. Once the inner scope is over, the inner variable is dropped, and the outer variable becomes visible to the compiler again.

```rust
let x = 10;
{
    let mut x = x + 10;
    x = x * 20;
    println!("{}", x) // 400
}
println!("{}", x) // 10
```

​    

#### Scope

- A variable is valid from the point it is declared until the end of the current scope
- Scopes are typically denoted by curly brackets `{}`
- When a variable goes out of scope, it is no longer valid. At this natural point, Rust automatically calls a special function called **drop** to return the variable's memory (specifically heap memory) to the allocator.

#####  Scopes of different types

- **Constants** - Constants can be declared in any scope, including the global scope. They remain valid for the **entire duration** of the program's execution within the scope where they were declared.
- **Reference** - A reference's scope starts from its introduction and continues through the **last time that reference is used**
- **Function Parameters:** Parameters in a function signature have a scope that is valid for the duration of the function body.
- **Modules and use** - The module system allows you to manage which names are in scope. The **use** **keyword** creates a shortcut for long paths, but this shortcut only applies to the specific scope in which the `use` statement occurs

### Data types

- Rust is a **statically typed language**, meaning it must know the types of all variables at compile time, though it can often infer them based on the value and usage. 
- Data types are primarily categorized into **scalar** and **compound** types.

#### Scalar

- A scalar type represents a **single value**. Rust has four primary scalar types

##### integer

- Numbers without fractional components. 
- They can be **signed** (starting with `i`) or **unsigned** (starting with `u`) 
- range in size from 8-bit to 128-bit (8, 16, 32, 64, 128)
- along with architecture-dependent `isize` and `usize` types. 
- Rust defaults to **i32** for integer types

##### float

- Numbers with decimal points. 
- Rust provides **f32** and **f64** (the default), both of which are signed

##### char

- The **char** type represents a **Unicode scalar value** and is four bytes in size. 
- It can represent emojis, accented letters, and non-Latin characters.
- `'` (single quote) is used to denote a char

##### Boolean

- Represented by the **bool** type, these have two possible values: 

- **true** and **false**. 

#### Compound types

- Compound types **group multiple values** into one type

##### tuple

- **Heterogeneous Types**: Tuples can store values of different data types.
- **Fixed Length**: Once a tuple is declared, its size cannot grow or shrink.
- **Indexing**: Elements are accessed by position using dot notation and an index number (e.g., `person.0`), which starts at 0.
- **Immutability by Default**: Tuples are immutable by default, but you can make them mutable using the `mut` keyword.
- **Type Signature**: A tuple's type is defined by the types of its members in order 
- **Returning Multiple Values**: Tuples are commonly used to enable functions to return multiple values.
- A tuple with no values is called **unit** and written as `()`

```rust
// A tuple containing a string slice, an integer, and a boolean
let person = ("John", 30, true);

// Accessing elements using tuple indexing (starts from 0)
println!("Name: {}", person.0);
println!("Age: {}", person.1);
println!("Is active: {}", person.2);

// Destructuring a tuple into individual variables
let (name, age, is_active) = person;
println!("Destructured Name: {}", name);
println!("Destructured Age: {}", age);
println!("Destructured Is active: {}", is_active);
```

##### array

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



##### Vector

- **Contiguous Memory:** Elements are stored next to each other on the heap, allowing for fast ![img](data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==) index-based access.
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



##### String

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



##### Hash maps

### Functions

### Control Flow

### Comments

## Understanding Ownership

### Ownership

### Reference and Borrowing

### Life cycle

## Structs

## Enums

## Package, modules and crates

## Error Handling

## Iterators and closures

## Generics and Traits

## File IO

## Smart pointers

## Concurrency

## async and await

## OOPS

## Macro





