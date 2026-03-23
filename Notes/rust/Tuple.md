# Tuple

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
​  
// Accessing elements using tuple indexing (starts from 0)  
println!("Name: {}", person.0);  
println!("Age: {}", person.1);  
println!("Is active: {}", person.2);  
​  
// Destructuring a tuple into individual variables  
let (name, age, is_active) = person;  
println!("Destructured Name: {}", name);  
println!("Destructured Age: {}", age);  
println!("Destructured Is active: {}", is_active);
```

