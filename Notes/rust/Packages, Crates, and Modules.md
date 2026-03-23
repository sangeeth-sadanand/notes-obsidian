# Packages, Crates, and Modules

- A package can contain multiple binary crates and optionally one library crate. As a package grows, you can extract parts into separate crates that become external dependencies
- These features, sometimes collectively referred to as the _module system_, include:

	- **Packages**: A Cargo feature that lets you build, test, and share crates
	- **Crates**: A tree of modules that produces a library or executable
	- **Modules and use**: Let you control the organization, scope, and privacy of paths
	- **Paths**: A way of naming an item, such as a struct, function, or module

## The Hierarchy: From 10,000 Feet

- The relationship looks like this:

```mermaid
flowchart LR
	A[Workspace] --> B[Package]
	B --> C[Crates]
	C --> D[Modules]
```

## Packages and Crates

### Crate
- A _crate_ is the smallest amount of code that the Rust compiler considers at a time.
- Crates can contain modules, and the modules may be defined in other files that get compiled with the crate
- A crate can come in one of two forms: a binary crate or a library crate. 
- The _crate root_ is a source file that the Rust compiler starts from and makes up the root module of your crate
#### Binary crates 

- binary crates are programs you can compile to an executable that you can run, such as a command line program or a server.
- Each must have a function called `main` that defines what happens when the executable runs.

#### Library crates

- Library crates don’t have a `main` function, and they don’t compile to an executable. 
- Instead, they define functionality intended to be shared with multiple projects.
- Example the `rand` crate provides functionality that generates random numbers.

###  Package
- A _package_ is a bundle of one or more crates that provides a set of functionality. 
- A package contains a _Cargo.toml_ file that describes how to build those crates.
- Cargo is actually a package that contains the binary crate for the command line tool you’ve been using to build your code. 
- The Cargo package also contains a library crate that the binary crate depends on. 
- Other projects can depend on the Cargo library crate to use the same logic the Cargo command line tool uses.
- A package can contain as many binary crates as you like, but at most only one library crate. A package must contain at least one crate, whether that’s a library or binary crate.

#### Create a package
- After we run `cargo new my-project`, we use `ls` to see what Cargo creates. 
- In the _my-project_ directory, there’s a _Cargo.toml_ file, giving us a package. 
- There’s also a _src_ directory that contains _main.rs_. 
- Open _Cargo.toml_ in your text editor and note that there’s no mention of _src/main.rs_. 
- Cargo follows a convention that _src/main.rs_ is the crate root of a binary crate with the same name as the package. 
- Likewise, Cargo knows that if the package directory contains _src/lib.rs_, the package contains a library crate with the same name as the package, and _src/lib.rs_ is its crate root. 
- Cargo passes the crate root files to `rustc` to build the library or binary.
- Here, we have a package that only contains _src/main.rs_, meaning it only contains a binary crate named `my-project`. 
- If a package contains _src/main.rs_ and _src/lib.rs_, it has two crates: a binary and a library, both with the same name as the package. 
- A package can have multiple binary crates by placing files in the _src/bin_ directory: Each file will be a separate binary crate.


## Modules

- Rust modules let you organize code into namespaces, control visibility with `pub`, and map modules to files or folders using `mod` and paths like `crate::foo::bar`. 
- Use `use` to bring paths into scope and `pub(crate)`/`pub(super)` for finer-grained access.

### What a module is
- **Definition:** A module (`mod`) groups related items (functions, structs, enums, constants, other modules). Modules form a tree of namespaces.  
- **Default privacy:** Items are **private by default**; add `pub` to expose them outside the module. 
### Core keywords and meanings
- **`mod name`** — declares a module (inline or file).
- **`pub`** — makes an item public; variants: **`pub(crate)`**, **`pub(super)`**, **`pub(in path)`** for restricted visibility. 
- **`use path::to::Item`** — brings a path into scope (like an import/alias).  
- **Paths:** **absolute** (`crate::a::b`) or **relative** (`self::`, `super::`). 

### File layout rules (common patterns)
#### Inline
- Inline modules are defined with** `mod name { ... }` inside the same file
- They create a nested namespace, keep items private by default, and use** `pub` to expose items.

```rust
fn main() {
    // call a public function from the inline module
    crate::math::add_and_print(2, 3);
}

mod math {
    // private helper function (not visible outside `math`)
    fn add(a: i32, b: i32) -> i32 {
        a + b
    }

    // public function exposed to parent module / crate
    pub fn add_and_print(a: i32, b: i32) {
        let sum = add(a, b); // can call private `add` inside same module
        println!("{} + {} = {}", a, b, sum);
    }

    // nested inline module
    pub mod utils {
        pub fn double(x: i32) -> i32 {
            x * 2
        }
    }
}
```

#### File modules 
- A _file module_ is created when you write `mod foo;` in a parent file (e.g., `main.rs` or `lib.rs`) and put the module body in a separate file `src/foo.rs` or in a directory `src/foo/mod.rs`.
- Rust maps module names to files on disk using these conventions.

```
project/
├─ src/
│  ├─ main.rs
│  └─ math.rs
```

**`src/main.rs`**

```rust
mod math; // loads src/math.rs

fn main() {
    // call public function from file module
    math::add_and_print(4, 5);
}
```

**`src/math.rs`**

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

pub fn add_and_print(a: i32, b: i32) {
    let sum = add(a, b);
    println!("{} + {} = {}", a, b, sum);
}
```

**Key points**

- **`mod math;`** in `main.rs` tells the compiler to load `src/math.rs`.
- **Items are private by default**; `pub fn add_and_print` is required to call it from `main`.
- Private `add` is accessible inside `math.rs` but not from `main`.

#### Directory module variant (`mod.rs` or `foo/mod.rs`)

- When you need submodules, use a directory:

```
src/
├─ lib.rs
└─ math/
   ├─ mod.rs    // or use src/math.rs + src/math/*.rs alternative
   └─ utils.rs
```

**`src/lib.rs`**

```rust
mod math; // loads src/math/mod.rs
pub use math::add_and_print;
```

**`src/math/mod.rs`**

```rust
mod utils; // loads src/math/utils.rs

fn add(a: i32, b: i32) -> i32 { a + b }

pub fn add_and_print(a: i32, b: i32) {
    let sum = add(a, b);
    println!("{} + {} = {}", a, b, sum);
}
```

**`src/math/utils.rs`**

```rust
pub fn double(x: i32) -> i32 { x * 2 }
```


> [!tip]
> - Below is a **modern** directory-module layout that avoids `mod.rs` by using `src/math.rs` as the parent module and `src/math/*.rs` for submodules.
> ```rust
> my_lib/
> ├─ Cargo.toml
> └─ src/
> 	├─ lib.rs
> 	├─ math.rs
> 	└─ math/
> 	    ├─ utils.rs
> 	    └─ ops.rs
