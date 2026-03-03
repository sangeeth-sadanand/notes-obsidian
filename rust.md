# Rust

## Introduction

- **Rust** is a modern, statically compiled programming language focused on **performance, reliability, and safety**.
- It was first released in **2010** and has since become one of the fastest-growing languages in systems programming.
- Rust is often compared to **C and C++**, but it eliminates many of their common pitfalls, especially around memory management.

### Key Features

- **Memory Safety without Garbage Collection** 
- **Concurrency Made Safe** 
- **Performance** 
- **Rich Tooling** 



## Cargo

- Cargo is Rust’s official **build system and package manager**
- Core functions and features
   - Dependency Management
   - Reproducible Builds
   - Project Organization
   - Release Profiles

### Create new project

- Creates a new directory and project with the specified name

```bash
cargo new <project_name>
```

- This file is in the TOML format and serves as the project's configuration file
- It also creates a src folder with main.rs file.

### Build a project

- Compiles the current project and places the executable in the target/debug directory

```bash
cargo build [--release]
```

- `--release` is need for building release version. 

###  Run the project

- Compiles the code and then runs the resulting executable in one step

```bash
cargo run
```

### Check the project

- Quickly checks the code to ensure it compiles without producing an executable, which is often much faster than a full build

```bash
cargo check
```

### Run the test cases

- Compiles and runs the project's test suite

```bash
cargo test
```

### Update the cargo package

- Ignores the `Cargo.lock` file and figures out the latest versions of dependencies that fit the specifications in `Cargo.toml`

```bash
cargo update
```

### Show documentation

- Builds documentation provided by all of your dependencies locally and opens it in a browser

```bash
cargo doc --open
```

### Publish crate

Uploads your crate to crates.io so others in the community can use it

```bash
cargo publish
```

> [!TIP]
>
> You can find the new crate on [crate.io](https://crates.io/) and find the document on [docs](https://docs.rs/)

### Adding a new crate

- Add the dependency crate to `cargo.toml` file under `[dependency]`.

```toml
[dependencies]
rand = "0.8.5"
```

Or can use

```bash
cargo add <crate_name>
```

- This will add the dependency automatically.

Then build the cargo.

## Common programming concepts

### Variables and Mutability

#### Constants 

#### Shadowing

#### Scope

### Data types

#### Scalar

##### integer and float

##### char

##### boolean

#### Structures

##### String

##### tuple

##### array

##### Vector

##### Hash maps

### Functions

### Control Flow

### Comments

## Understanding Ownership

### Ownership

### Reference and Borrowing

### Life cycle

## Structs

## Enums

## Package, modules and crates

## Error Handling

## Iterators and closures

## Generics and Traits

## File IO

## Smart pointers

## Concurrency

## async and await

## OOPS

## Macro





