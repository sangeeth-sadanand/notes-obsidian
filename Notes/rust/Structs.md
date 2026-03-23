# Structs

## Defining Structs in Rust

### Named-Field Structs

- Named-field structs are the most common form of struct in Rust.
-  Each field has a name and a type, providing clarity and self-documentation.

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

- Field names use `snake_case` by convention, while struct names use `PascalCase`.

### Tuple Structs

- Tuple structs are a hybrid between tuples and structs. 
- They have named types but their fields are accessed by position rather than by name.

```rust
struct Color(u8, u8, u8);
struct Point(f64, f64);
```

- Instances are created like tuples:

```rust
let black = Color(0, 0, 0);
let origin = Point(0.0, 0.0);
```

### Unit-Like Structs

- Unit-like structs have no fields and are defined with a semicolon:

```rust
struct Marker;
```

- They are zero-sized types and are often used as markers, for implementing traits without storing data, or as type-level flags in generic programming.

##  Instantiating and Initializing Structs

### Creating Instances

- To create an instance of a named-field struct, specify values for each field:

```rust
let user1 = User {
    active: true,
    username: String::from("alice"),
    email: String::from("alice@example.com"),
    sign_in_count: 1,
};
```

- The order of fields in the literal does not need to match the definition. 
- All fields must be initialized unless using the struct update syntax or default values.

### Field Init Shorthand

- When local variables have the same names as struct fields, you can use field init shorthand:

```rust
let username = String::from("bob");
let email = String::from("bob@example.com");
let user2 = User {
    active: true,
    username,
    email,
    sign_in_count: 1,
};
```

- This reduces redundancy and improves readability.

###  Tuple and Unit Struct Instantiation

- Tuple structs are instantiated like tuples:

```rust
let color = Color(255, 0, 0);
let point = Point(3.0, 4.0);
```

- Unit-like structs are instantiated with their name:

```rust
let marker = Marker;
```

## Accessing and Mutating Struct Fields

### Field Access

- Fields are accessed using dot notation:

```rust
println!("Username: {}", user1.username);
```

- This works for both owned instances and references. 

### Mutability

- Struct instances are immutable by default. 
- To mutate fields, the entire instance must be declared mutable:

```rust
let mut user = User {
    active: true,
    username: String::from("carol"),
    email: String::from("carol@example.com"),
    sign_in_count: 1,
};
user.email = String::from("new_email@example.com");
```

- Rust does not allow marking individual fields as mutable within an immutable struct; mutability is a property of the binding, not the fields.

## Ownership Implications

- Fields that implement the `Copy` trait (e.g., integers, bool) are copied.
- Fields that do not implement `Copy` (e.g., `String`) are moved, making them inaccessible in the original instance after the update.

```rust
let user4 = User {
    username: String::from("eve"),
    ..user3
};
// user3.username is now invalid (moved), but user3.active is still accessible.
```

## Ownership and Borrowing with Struct Fields

### Ownership of Fields

- By default, structs own their fields.
-  When a struct goes out of scope, all its owned fields are dropped, freeing resources such as heap-allocated memory.

```rust
struct DataContainer {
    id: u32,
    data: String,
}

fn main() {
    let container = DataContainer {
        id: 1,
        data: String::from("Owned data"),
    };
    // container.data is dropped when container goes out of scope
}
```

### Moving Structs and Fields

- Assigning or passing a struct by value moves ownership. 
- If the struct or any of its fields do not implement `Copy`, the original variable becomes invalid after the move.

```rust
let user5 = user4; // user4 is now invalid if any field is non-Copy
```

### Borrowing Structs

- You can borrow a struct immutably or mutably:

```rust
fn print_user(user: &User) {
    println!("{:?}", user);
}

fn update_user(user: &mut User) {
    user.sign_in_count += 1;
}
```

- Immutable references (`&User`) allow reading fields.
- Mutable references (`&mut User`) allow modifying fields.
- Only one mutable reference or any number of immutable references can exist at a time (enforced at compile time).

### Borrowing Individual Fields

Rust's borrow checker understands that you can borrow disjoint fields of a struct simultaneously:

```rust
struct Foo { a: i32, b: i32 }
let mut x = Foo { a: 0, b: 0 };
let a_ref = &mut x.a;
let b_ref = &mut x.b;
```

This is safe because `a` and `b` are independent.

### Structs with Borrowed Fields (References)

- Structs can contain references to data owned elsewhere, but must specify lifetimes to ensure the referenced data outlives the struct:

```rust
struct Book<'a> {
    title: &'a str,
    author: &'a str,
}
```

- Using borrowed fields can save memory and avoid unnecessary cloning, but introduces lifetime complexity and restricts how the struct can be used.

------

## Implementing Methods and Associated Functions with `impl`

###  The `impl` Block

- Methods and associated functions are defined in `impl` blocks:

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Associated function (constructor)
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }

    // Method (borrows self immutably)
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // Method (borrows self mutably)
    fn scale(&mut self, factor: u32) {
        self.width *= factor;
        self.height *= factor;
    }
}
```

- Associated functions do not take `self` and are called with `::` (e.g., `Rectangle::new(10, 20)`).
- Methods take `self`, `&self`, or `&mut self` and are called with `.` (e.g., `rect.area()`).

### Method Receivers: `self`, `&self`, `&mut self`

| Receiver    | Meaning           | Use Case                              |
| ----------- | ----------------- | ------------------------------------- |
| `self`      | Takes ownership   | Consumes the instance, e.g., `into_*` |
| `&self`     | Borrows immutably | Read-only methods (most common)       |
| `&mut self` | Borrows mutably   | Methods that modify the instance      |

- Use `&self` for read-only methods.
- Use `&mut self` for methods that mutate fields.
- Use `self` when the method consumes the instance (rare).

```rust
impl Rectangle {
    fn set_width(mut self, width: u32) -> Self {
        self.width = width;
        self
    }
}
```

### Multiple `impl` Blocks

- You can have multiple `impl` blocks for the same struct, which is useful for organizing code or separating trait implementations from inherent methods.

## Pattern Matching and De-structuring Structs

### De-structuring with `let`

- You can de-structure structs to bind fields to local variables:

```rust
let Point { x, y } = point;
```

- This moves or copies the fields, depending on their types. 
- If a field is not `Copy`, it is moved out of the struct, making the original field inaccessible.

### Pattern Matching in `match`, `if let`, and `while let`

- Structs can be matched in `match` expressions:

```rust
match point {
    Point { x: 0, y } => println!("On the y axis at {}", y),
    Point { x, y: 0 } => println!("On the x axis at {}", x),
    Point { x, y } => println!("At ({}, {})", x, y),
}
```

- You can also use `if let` and `while let` for concise pattern matching:

```rust
if let Point { x: 0, y } = point {
    println!("On the y axis at {}", y);
}
```

### Ignoring Fields

- Use `..` to ignore fields you don't care about:

```rust
match point {
    Point { x, .. } => println!("x is {}", x),
}
```

- This is especially useful for structs with many fields.

### Nested Destructuring

- Pattern matching can be nested for structs containing other structs or tuples:

```rust
struct Rectangle { top_left: Point, bottom_right: Point }
let Rectangle { top_left: Point { x, y }, bottom_right } = rect;
```

## Visibility, Modules, and Encapsulation

### Default Privacy

- By default, structs and their fields are private to the module in which they are defined. 
- To make a struct or field public, use the `pub` keyword:

```rust
pub struct Person {
    pub name: String,
    age: u8, // private field
}
```

- Other modules can use the struct and its public methods, but cannot access private fields directly.

### Fine-Grained Visibility

Rust supports more granular visibility controls:

- `pub(crate)`: visible within the current crate
- `pub(super)`: visible to the parent module
- `pub(in path)`: visible within a specific module path

This allows for precise encapsulation and API design.

### Encapsulation Patterns

- Expose only necessary fields as `pub`.
- Provide public constructors and accessor methods for private fields.
- Use modules to group related types and functions, controlling visibility at the module boundary.

## Structs vs. Classes in Python and C++

| Feature              | Rust Structs                      | Python Classes             | C++ Classes/Structs              |
| -------------------- | --------------------------------- | -------------------------- | -------------------------------- |
| Data + Methods       | Data in struct, methods in `impl` | Data and methods together  | Data and methods together        |
| Inheritance          | No (use traits, composition)      | Yes (single/multiple)      | Yes (single/multiple)            |
| Polymorphism         | Traits, trait objects             | Duck typing, ABCs          | Virtual functions, vtables       |
| Visibility           | Fine-grained (`pub`, private)     | Public, protected, private | Public, protected, private       |
| Memory Management    | Ownership, borrowing, lifetimes   | Garbage collected          | Manual, RAII, smart pointers     |
| Field Mutability     | By binding, not per field         | Per field                  | Per field                        |
| Default Field Access | Private                           | Public                     | Private (class), public (struct) |
| Constructors         | Associated functions              | `__init__`                 | Constructors                     |
| Dynamic Features     | No (static)                       | Yes (dynamic, metaclasses) | Limited (templates, RTTI)        |
| Method Receivers     | `self`, `&self`, `&mut self`      | `self`                     | `this` pointer                   |
| Multiple Inheritance | No (traits can be composed)       | Yes                        | Yes                              |
| Reflection           | Limited (macros, derive)          | Yes                        | Limited (RTTI)                   |
|                      |                                   |                            |                                  |


