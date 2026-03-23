# Variable scope

- A variable is valid from the point it is declared until the end of the current scope
- Scopes are typically denoted by curly brackets `{}`
- When a variable goes out of scope, it is no longer valid. At this natural point, Rust automatically calls a special function called **drop** to return the variable's memory (specifically heap memory) to the allocator.

#####  Scopes of different types

- **Constants** - Constants can be declared in any scope, including the global scope. They remain valid for the **entire duration** of the program's execution within the scope where they were declared.
- **Reference** - A reference's scope starts from its introduction and continues through the **last time that reference is used**
- **Function Parameters:** Parameters in a function signature have a scope that is valid for the duration of the function body.
- **Modules and use** - The module system allows you to manage which names are in scope. The **use** **keyword** creates a shortcut for long paths, but this shortcut only applies to the specific scope in which the `use` statement occurs