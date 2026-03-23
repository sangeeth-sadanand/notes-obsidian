# Variables and Mutability
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

## Constants

- Constants are **always immutable**
- `const` keyword is used to define constants

```rust
const PI = 3.1416
```

- Type must always be annotated.
- Constants value should me known at compile time. It cannot be assigned at run time.
- It can be declared at global scope as well.
- Use uppercase naming convention 

## Shadowing

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

