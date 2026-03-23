# Pattern matching
- Pattern matching is one of Rust’s most powerful features. 
- It allows you to compare a value against a series of patterns and execute specific code based on which pattern matches. 

## The `match` Expression

- The most common way to use pattern matching is the `match` control flow operator. 
- It consists of a value to evaluate, followed by one or more "arms". 
- Each arm has a pattern and some code to run if the pattern matches.

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

> [!IMPORTANT]
> 
>  **Key Rule: Exhaustiveness** 
>  - Rust requires `match` expressions to be _exhaustive_. 
>  - This means you must explicitly handle every possible case, or the code will not compile. 
>  - This prevents subtle bugs where an edge case is forgotten.

## Extracting Values

Pattern matching really shines when you use it to extract values wrapped inside Enums.

```rust
#[derive(Debug)]
enum UsState {
    Alaska,
    Hawaii,
    // ...
}

enum Coin {
    Penny,
    Quarter(UsState), // A quarter holding a state value
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

If `coin` is `Coin::Quarter(UsState::Alaska)`, the `state` variable binds to the `UsState::Alaska` value, allowing you to use it in that arm.

## Matching with `Option<T>`

- Rust doesn't have `null`. 
- Instead, it uses the `Option` enum to represent the presence or absence of a value. 
- Pattern matching is the standard way to handle `Option`.

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

## 4. Catch-all Patterns and the `_` Placeholder

- If you only care about a few specific values and want a default action for everything else, you can use the `_` placeholder. 
- `_` is a special pattern that matches any value and does not bind to it.

```rust
let dice_roll = 9;
match dice_roll {
    3 => println!("You rolled a 3!"),
    7 => println!("You rolled a 7!"),
    _ => println!("You rolled something else."), // Handles everything else
}
```

## 5. Concise Control Flow with `if let`

- Sometimes a `match` expression is too verbose if you only care about _one_ specific case and want to ignore the rest. 
- For this, Rust provides `if let`.

```rust
let config_max = Some(3u8);
match config_max {
    Some(max) => println!("The maximum is configured to be {}", max),
    _ => (),
}
```

You can do this:


```rust
let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximum is configured to be {}", max);
}
```

## Other Places Patterns Appear

While `match` and `if let` are the most common, pattern matching is baked into many other parts of Rust:

- **`while let` loops:** Runs a loop as long as a pattern continues to match.
- **`for` loops:** The `x` in `for x in y` is actually a pattern.
- **`let` statements:** `let (x, y, z) = (1, 2, 3);` destructures a tuple using a pattern.
- **Function parameters:** You can destructure arguments right in the function signature: `fn print_coordinates(&(x, y): &(i32, i32))`.