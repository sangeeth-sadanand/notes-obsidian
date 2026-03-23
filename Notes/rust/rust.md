# Rust

- **Rust** is a modern, statically compiled programming language focused on **performance, reliability, and safety**.
- It was first released in **2010** and has since become one of the fastest-growing languages in systems programming.
- Rust is often compared to **C and C++**, but it eliminates many of their common pitfalls, especially around memory management.

### Key Features

- **Memory Safety without Garbage Collection** 
- **Concurrency Made Safe** 
- **Performance** 
- **Rich Tooling** 

- Rust uses [[Notes/rust/Cargo|Cargo]] as its primary build tool and package manager to handle dependencies. The foundation of the language relies on [[Notes/rust/Common Programming Concepts|Common Programming Concepts]]. The most unique feature of the language is [[Notes/rust/Understanding ownership|Understanding Ownership]], which ensures memory safety without a garbage collector.
- To build custom data structures, Rust uses [[Notes/rust/structs|Structs]] to group related data and [[Notes/rust/Enums|Enums]] to define types that can be one of several variants. 
- In Rust, **[[Notes/rust/Pattern matching|Pattern matching]]** is a powerful control flow mechanism that allows you to compare a value against a series of patterns and execute code based on which pattern matches.
- As projects grow, you use [[Notes/rust/Packages, Crates, and Modules| Packages, Crates, and Modules]] to manage the Module Tree. 
- Reliability is bolstered by a robust approach to [[Notes/rust/Error handling|Error Handling]], distinguishing between Unrecoverable Errors with panic! and Recoverable Errors with Result.
- [[Iterators]] and [[closure]] can be used to iterate through data structures with ease. 
- To write flexible code, you will use [[Generic Data Types]], [[Traits]] to define shared behaviour, and [[Lifetimes]] to ensure references remain valid.
- **[[Smart Pointers]]** are data structures that act like pointers but have additional metadata and capabilities.  
- **[[Asynchronous Programming]]** allows you to run multiple tasks concurrently on a single OS thread and **[[Concurrency]]** is the ability for different parts of a program to execute independently
- Rust is not a traditional **[[Object-Oriented Programming]]** (OOP) language like Java or C++, it provides powerful tools to achieve the same goals: Encapsulation, Abstraction, Inheritance (via composition), and Polymorphism.
- **[[Macros]]** are a way of writing code that writes other code, a concept known as meta-programming. While functions operate on values, macros operate on the source code itself, expanding into more complex structures during compilation.
