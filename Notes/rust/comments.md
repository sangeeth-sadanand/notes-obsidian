# Comments

- Rust has two primary styles of comments: **non-doc comments**, which are ignored by the compiler and used for general notes, and **doc comments**, which are used to generate HTML documentation.

##  Non-Doc Comments

- These are for explaining code to other developers and for temporarily disabling code during debugging. 

- **Line Comments**: Start with two forward slashes (`//`) and continue to the end of the line. This is the idiomatic style for most comments in Rust.

    ```rust
    // This is a single-line comment.
    
    fn main() {
        let x = 5; // A comment can also be placed at the end of a line of code.
    }
    ```

- **Block Comments**: Enclosed in `/* ... */` delimiters and can span multiple lines. They are useful for commenting out large blocks of code and support nesting (unlike in C/C++).

    ```rust
    /*
    This is a block comment.
    It can span multiple lines.
    You can even nest *//* another block comment *//* inside.
    */
    fn main() {
        /* println!("This code is commented out and will not run."); */
    }
    ```

## Documentation Comments

- These comments are parsed by the `rustdoc` tool to generate public HTML documentation for a library crate and support Markdown formatting. They are generally preferred over block comments for documentation purposes.

- **Outer Doc Comments**: Start with three forward slashes (`///`) and apply to the item immediately following the comment (e.g., a function, struct, or module).

    

    ```rust
    /// Adds two numbers together.
    ///
    /// # Arguments
    /// * `a` - The first number.
    /// * `b` - The second number.
    ///
    /// # Examples
    /// ```rust
    /// let sum = my_crate::add(10, 20);
    /// assert_eq!(sum, 30);
    /// ```
    pub fn add(a: i32, b: i32) -> i32 {
        a + b
    }
    ```

- **Inner Doc Comments**: Start with `//!` and apply to the *enclosing* item (e.g., typically used at the top of a file to document the entire module or crate).

    - Inner doc comments (using `//!` or `/*!` in Rust) are used to document the *container* holding them (such as a module, crate, or file) rather than the item following them.

    

    ```rust
    //! # My Crate
    //!
    //! This crate provides a set of useful functions for various tasks.
    
    // Code for the module or crate follows...
    ```

  