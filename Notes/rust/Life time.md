# Life time

- A *lifetime* is the scope for which a reference is valid; it prevents dangling references by ensuring borrows end before the owned value is dropped. 
- **Most lifetimes are implicit.**
- You only annotate lifetimes when the compiler cannot determine how multiple references relate.

## Core rules

- **Every reference has a lifetime.**
- **A reference must not outlive the value it points to.**
- **When returning references, the returned reference’s lifetime must be tied to an input parameter or be** `'static`**.**
- **You can have many** `&T` **or one** `&mut T` **at a time; lifetimes and borrowing rules work together to prevent data races.**

## Lifetime elision 

Rust applies three elision rules so you rarely write lifetimes:

1. Each parameter that is a reference gets its own lifetime.
2. If there is exactly one input lifetime, that lifetime is assigned to the output.
3. If `&self` or `&mut self` is an input, the output lifetime is that of `self`. When these rules apply, you can omit explicit annotations