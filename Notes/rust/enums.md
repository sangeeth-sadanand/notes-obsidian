# Enums

## Defining Enums in Rust

### Basic Enum Definitions (Unit Variants)

- The simplest form of an enum in Rust is a set of unit variants—named constants that do not carry any data. 
- This is similar to C-style enums, but with Rust’s strong type safety.

```rust
enum Direction {
    North,
    South,
    East,
    West,
}
```

- Here, `Direction` is a type that can be one of four possible values. 
- Each variant is namespaced under the enum name, so you refer to them as `Direction::North`, `Direction::South`, etc.

- Unit variants are ideal for representing a fixed set of options or states where no additional data is needed. 

### Enums with Associated Data

- Rust’s enums can have variants that carry data, making them much more powerful than C-style enums. 
- There are two primary forms:

#### Tuple Variants

- Tuple variants hold unnamed fields, similar to tuple structs.

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}
```

- Here, `IpAddr::V4` holds four `u8` values (an IPv4 address), while `IpAddr::V6` holds a `String` (an IPv6 address). 
- Each variant can have different types and numbers of fields.

#### Struct Variants (Named Fields)

- Struct variants hold named fields, similar to regular structs.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

- This flexibility allows you to model complex data structures in a concise and type-safe way.

## Constructors and Variant Names as Functions

- Each variant acts as a constructor function. For example:

```rust
let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
let msg = Message::Move { x: 10, y: 20 };
```

- This means you can use the variant name as a function to create instances, which is especially useful for variants with associated data.

## Pattern Matching with Enums

- Pattern matching is one of Rust’s most powerful features, and enums are designed to work seamlessly with it.

### The `match` Expression

- The `match` expression allows you to branch on the value of an enum, handling each variant explicitly.
- Rust enforces *exhaustive matching*: every possible variant must be covered, or the code will not compile.

```rust
fn describe_direction(direction: Direction) -> &'static str {
    match direction {
        Direction::North => "Heading North",
        Direction::South => "Heading South",
        Direction::East  => "Heading East",
        Direction::West  => "Heading West",
    }
}
```

- If you add a new variant to `Direction`, the compiler will force you to update all `match` expressions that handle it, preventing bugs from unhandled cases.

### Matching Variants with Data

- When matching variants that carry data, you can de-structure them to access their fields:

```rust
match msg {
    Message::Quit => println!("Quit"),
    Message::Move { x, y } => println!("Move to ({}, {})", x, y),
    Message::Write(text) => println!("Write: {}", text),
    Message::ChangeColor(r, g, b) => println!("Change color to RGB({}, {}, {})", r, g, b),
}
```

- This pattern matching can bind fields to variables, enabling concise and expressive code.

### Pattern Guards

- You can add extra conditions to match arms using pattern guards:

```rust
enum Message {
    Text(String),
    Number(i32),
}

fn main() {
    let msg = Message::Number(42);

    match msg {
        Message::Number(n) if n % 2 == 0 => {
            println!("Even number: {}", n);
        }
        Message::Number(n) => {
            println!("Odd number: {}", n);
        }
        Message::Text(t) if t.len() > 5 => {
            println!("Long text: {}", t);
        }
        Message::Text(t) => {
            println!("Short text: {}", t);
        }
    }
}
```

- Pattern guards are useful for refining matches beyond structural patterns.

## De-structuring and Binding Fields

- Rust allows de-structuring of tuple and struct variants directly in the match arm:

```rust
enum Color {
    Red,
    RGB(u8, u8, u8),
    HSV(u8, u8, u8),
}

let color = Color::RGB(122, 17, 40);

match color {
    Color::Red => println!("Red!"),
    Color::RGB(r, g, b) => println!("RGB({}, {}, {})", r, g, b),
    Color::HSV(h, s, v) => println!("HSV({}, {}, {})", h, s, v),
}
```

- This makes it easy to extract and use the data stored in each variant.

### `if let`, `while let`, and `let else` Ergonomics

- For cases where you only care about one variant, `if let` provides a concise alternative to `match`:

```rust
if let Some(value) = option {
    println!("Got a value: {}", value);
}
```

- This is especially useful for enums like `Option` and `Result`, where you often only care about the `Some` or `Ok` case.

- Similarly, `while let` can be used to loop while a pattern matches:

```rust
while let Some(item) = stack.pop() {
    println!("Popped: {}", item);
}
```

- The `let else` syntax (stable since Rust 1.65) allows early returns on failed pattern matches:

```rust
let Some(value) = maybe_value else {
    return;
};
```

### The `matches!` Macro

- The `matches!` macro provides a concise way to check if a value matches a pattern:

```rust
let bar = Some(4);
assert!(matches!(bar, Some(x) if x > 2));
```

- This is useful for quick checks, especially in assertions or filtering logic.


## Implementing Methods and Associated Functions on Enums

- Enums in Rust can have methods and associated functions, just like structs. 
- This is done using the `impl` block.

### Methods

- Methods take `self`, `&self`, or `&mut self` as their first parameter and are called using the dot syntax.

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

impl TrafficLight {
    fn duration(&self) -> u8 {
        match self {
            TrafficLight::Red => 60,
            TrafficLight::Yellow => 5,
            TrafficLight::Green => 30,
        }
    }
}

let light = TrafficLight::Red;
println!("Red light duration: {} seconds", light.duration());
```

- This pattern allows you to encapsulate behavior related to the enum, keeping your code organized and idiomatic.

### Associated Functions

- Associated functions do not take `self` and are called using the double colon syntax. 
- They are often used as constructors or utility functions.

```rust
impl TrafficLight {
    fn new_red() -> TrafficLight {
        TrafficLight::Red
    }
}
let red_light = TrafficLight::new_red();
```

- You can define as many associated functions as needed, providing clear and intention-revealing APIs.

### Enum Variant Constructors

- While each variant acts as a constructor, you can define additional associated functions for more complex construction logic, validation, or to abstract away internal details:

```rust
impl Message {
    fn new_move(x: i32, y: i32) -> Self {
        Message::Move { x, y }
    }
}
let m = Message::new_move(10, 20);
```

- This approach is especially useful for large enums or when you want to hide internal representation details from users of your API.

## Ownership, Borrowing, and Pattern Matching with Enums

- Rust’s ownership and borrowing rules apply to enums just as they do to other types. 
- Understanding how these interact with pattern matching is crucial for writing safe and efficient code.

### Moving and Borrowing Enum Values

- When you match on an enum, you can either move or borrow its contents. 
- If you match on a value directly, ownership of any fields is moved:

```rust
let msg = Message::Write(String::from("hello"));
match msg {
    Message::Write(text) => println!("Text: {}", text), // text is moved here
    _ => (),
}
```

- If you need to use the value after matching, borrow it:

```rust
let msg = Message::Write(String::from("hello"));
match &msg {
    Message::Write(text) => println!("Text: {}", text), // text is a &String
    _ => (),
}
println!("{:?}", msg); // msg is still usable
```

### The `ref` and `ref mut` Patterns

- You can use `ref` and `ref mut` in patterns to borrow fields immutably or mutably:

```rust
match &mut msg {
    Message::Write(ref mut text) => text.push_str(" world!"),
    _ => (),
}
```

- This is especially useful when you want to modify data inside an enum variant without moving it out.
## Memory Layout

The size of an enum is determined by the size of its largest variant plus the size of the discriminant. For example:

```rust
enum ExampleEnum {
    VariantA(u8),
    VariantB(i64),
    VariantC([u8; 128]),
}
```

All instances of `ExampleEnum` have the same size, large enough to hold the biggest variant plus the discriminant.
### Optimising with `Box`

If one variant is much larger than others, you can store its data on the heap using `Box`, reducing the enum’s stack size:

```rust
enum OptimizedEnum {
    Small(u8),
    Large(Box<[u8; 1024]>),
}
```