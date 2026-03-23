# Functions

- Functions in Rust are reusable blocks of code defined using the `fn` keyword. 
- They are central to organising code and accept typed parameters, optionally returning a single value. 
- The primary function in every Rust program is `main()`. 

## Defining a Function

- Functions are declared with `fn` followed by the function name, parentheses for parameters, an optional return type annotation `->` Type, and curly braces for the body. 

```rust
fn function_name(parameter_name: Type) -> ReturnType {
    // function body
}
```
-  **`fn` keyword**: Declares a new function. 
-  **snake case naming**: Rust convention for function names. 
- **Parameters**: Each parameter must have a type annotation declared after a colon. 
- **Return Type**: An arrow `->` indicates the type of the value the function will return. If no value is returned, the annotation and the  are omitted, and the function implicitly returns the unit type.

## Calling a Function 

- Functions are called by using their name followed by parentheses `()` and passing arguments that match the parameter types. 

```rust
fn main() {
    another_function(5); // Call with an argument
}

fn another_function(x: i32) {
    println!("The value of x is: {}", x);
}
```

## Returning Values

Functions return a value in two ways:

- **Implicit Return (Expression)**: The last line in the function body without a semicolon is treated as an expression whose value is returned. This is the idiomatic Rust style. 
- **Explicit Return (Statement)**: The  keyword can be used to return a value explicitly, often for early returns.

## Key Concepts 

- **Statements vs. Expressions**: Rust functions are primarily expression-based. Statements perform actions and do not return values (e.g., `let y = 6`;). Expressions evaluate to a resultant value (e.g., `5+1` or a code block `{ x + 1 }` ). 
- **Ownership and Borrowing**: When passing values to functions, Rust's ownership system determines how the data is handled. By default, ownership might be moved, but references (`&`) can be used to "borrow" data without taking ownership, which is a key part of Rust's memory safety. 
- **Methods**: Functions can be defined within `impl` blocks for structs or enums. If the first parameter is `&self`, `&mut self` , `self` or , the function is a method callable using the `.` syntax (e.g., `instance.method()`). 
- **Closures**: Rust supports anonymous functions called closures, which can capture values from their surrounding scope. Closures are a powerful feature used extensively with iterators.

