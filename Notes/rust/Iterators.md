In Rust, an **iterator** is a pattern that allows you to perform a task on a sequence of items one by one. It handles the logic of stepping through each item in a collection and determining when the sequence has finished.

What makes Rust iterators special is that they are **lazy**—they don't do any work until you explicitly call methods that consume them. They are also incredibly fast; because they are compiled down to highly optimized state machines, they often perform just as well as (or faster than) manually written loops.

Here is a breakdown of how iterators work in Rust:

### 1. The `Iterator` Trait

At the heart of Rust's iteration is the `Iterator` trait from the standard library. It looks something like this:

Rust

```
pub trait Iterator {
    type Item; // The type of elements being iterated over

    // The only required method to implement
    fn next(&mut self) -> Option<Self::Item>;

    // ... plus dozens of default methods
}
```

The `next` method returns `Some(Item)` when there is a value to yield, and `None` when the iteration is over.

### 2. Creating an Iterator

Collections in Rust (like `Vec`, `HashMap`, etc.) us ually provide three main ways to create an iterator, depending on what kind of access you need:

- **`iter()`**: Borrows each element (creates an iterator of `&T`).
    
- **`iter_mut()`**: Mutably borrows each element (creates an iterator of `&mut T`), allowing you to modify the elements in place.
    
- **`into_iter()`**: Takes ownership of the collection (creates an iterator of `T`). The original collection can no longer be used after this.
    

Rust

```
let v = vec![1, 2, 3];

let mut iter = v.iter(); // Returns immutable references

assert_eq!(iter.next(), Some(&1));
assert_eq!(iter.next(), Some(&2));
assert_eq!(iter.next(), Some(&3));
assert_eq!(iter.next(), None);
```

### 3. Consuming Adaptors

Because iterators are lazy, just creating one does nothing. You have to call a **consuming adaptor** to actually make the iterator do work. Consuming adaptors call `next` internally and use up the iterator.

- **The `for` loop**: The most common way to consume an iterator. (Under the hood, a `for` loop calls `into_iter()` on the collection).
    
- **`collect()`**: Gathers the iterated values back into a new collection (like a `Vec` or `HashMap`).
    
- **`sum()`, `count()`, `max()`**: Math and aggregation functions.
    

Rust

```
let v1 = vec![1, 2, 3];

// 'sum()' consumes the iterator
let total: i32 = v1.iter().sum(); 
println!("Total: {}", total); // Prints: Total: 6
```

### 4. Iterator Adaptors

Iterator adaptors are methods defined on the `Iterator` trait that **change one iterator into a different kind of iterator**. Because they return a new iterator, they are also lazy, meaning you can chain multiple adaptors together without a performance penalty.

Common adaptors include:

- **`map()`**: Applies a closure to every item.
    
- **`filter()`**: Keeps only the items that match a certain condition.
    
- **`take()`**: Stops after yielding a specific number of items.
    

### Putting it all together

Here is an example that demonstrates chaining iterator adaptors and then consuming the result:

Rust

```
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6];

    // Let's take the numbers, filter for even ones, square them, 
    // and collect them into a new vector.
    let squared_evens: Vec<i32> = numbers
        .into_iter()          // 1. Create an owned iterator
        .filter(|x| x % 2 == 0) // 2. Adaptor: Keep only even numbers
        .map(|x| x * x)         // 3. Adaptor: Square them
        .collect();             // 4. Consuming adaptor: Execute and gather into a Vec

    println!("{:?}", squared_evens); // Prints: [4, 16, 36]
}
```

In this example, no actual work (filtering or mapping) happens on lines 2 and 3. The entire pipeline is executed simultaneously only when `collect()` is called on line 4.