# Control Flow

- Rust provides several powerful constructs for control flow, primarily **`if`** **expressions, loops (`loop`, `while`, `for`), and the `match` expression**. 
- These mechanisms allow programs to make decisions and execute code repeatedly based on conditions.

## Conditional Execution

### `if` / `else if` / `else` expressions

- These are used to branch code based on a Boolean condition. 
- Unlike some other languages, the condition *must* evaluate explicitly to a `bool` type; Rust does not automatically convert non-Boolean types to a boolean. 
- `if` is an expression, meaning it can return a value that can be assigned to a variable, provided all potential return branches yield the same type.

```rust
let number = 3;

if number < 5 {
    println!("condition was true");
} else {
    println!("condition was false");
}

// Using if as an expression
let condition = true;
let num = if condition { 5 } else { 6 }; // Both arms return an integer
```

### `match` expression

- This is a powerful pattern-matching construct that compares a value against a series of patterns and executes the code for the first matching pattern. 

- `match` is exhaustive, meaning the compiler ensures all possible cases are handled, making it very safe for scenarios like working with enums and error handling (`Option` and `Result` types).

```rust
let num = 5;
match num {
    1 => println!("One"),
    2 | 3 | 4 => println!("Between 2 and 4"), // Match multiple values
    _ => println!("Something else"), // Catch-all pattern
}
```

### `if let` and `let...else`

- These are more concise syntaxes for handling specific patterns when you don't need the exhaustive checking of a full `match` expression.

## Loops

- Rust provides three kinds of loops for repeated execution.

### `loop`

- Creates an infinite loop that runs until explicitly told to stop, typically using the `break` keyword. 
- The `break` keyword can also return a value from the loop, which can be assigned to a variable.

```rust
fn main() {
    let mut counter = 0;

    while counter < 5 {
        println!("Counter: {}", counter);
        counter += 1;
    }
}
```

### `while`

- This loop runs a block of code as long as a specified Boolean condition remains `true`.

```rust
fn main() {
    let mut counter = 0;

    while counter < 5 {
        println!("Counter: {}", counter);
        counter += 1;
    }
}
```

### `for`

- This is the most commonly used and idiomatic loop in Rust. 
- It safely iterates over the items of a collection (arrays, vectors, ranges, etc.) using iterators, eliminating common errors like going out of bounds.

```rust
// Loop through a range
for number in 1..=3 {
    println!("{number}!");
}

// Loop through a collection
let a = [10, 20, 30, 40, 50];
for element in a.iter() {
    println!("the value is: {element}");
}
```

### Loop Control

The `break` and `continue` keywords provide finer control within loops.

- **`break`**: Exits the current loop immediately.

```rust
fn main() {
    let mut counter = 0;

    loop {
        counter += 1;
        println!("Counter: {}", counter);

        if counter == 3 {
            break; // exits loop when counter reaches 3
        }
    }
}
```

- **`continue`**: Skips the rest of the current loop iteration and starts the next one.

```rust
fn main() {
    for n in 1..6 {
        if n % 2 == 0 {
            continue; // skip even numbers
        }
        println!("Odd number: {}", n);
    }
}
```

- **Loop Labels**: For nested loops, you can use loop labels (e.g., `'outer:`) with `break` or `continue` to specify which loop you want to affect.

```rust
fn main() {
    'outer: for i in 1..4 {
        'inner: for j in 1..4 {
            if i == 2 && j == 2 {
                break 'outer; // exits the outer loop directly
            }
            println!("i: {}, j: {}", i, j);
        }
    }
}
```

