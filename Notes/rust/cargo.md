# Cargo
- Cargo is Rust’s official **build system and package manager**
- Core functions and features
   - Dependency Management
   - Reproducible Builds
   - Project Organization
   - Release Profiles

## Create new project

- Creates a new directory and project with the specified name

```bash
cargo new <project_name>
```

- This file is in the TOML format and serves as the project's configuration file
- It also creates a src folder with main.rs file.

## Build a project

- Compiles the current project and places the executable in the target/debug directory

```bash
cargo build [--release]
```

- `--release` is need for building release version. 

##  Run the project

- Compiles the code and then runs the resulting executable in one step

```bash
cargo run
```

## Check the project

- Quickly checks the code to ensure it compiles without producing an executable, which is often much faster than a full build

```bash
cargo check
```

## Run the test cases

- Compiles and runs the project's test suite

```bash
cargo test
```

## Update the cargo package

- Ignores the `Cargo.lock` file and figures out the latest versions of dependencies that fit the specifications in `Cargo.toml`

```bash
cargo update
```

## Show documentation

- Builds documentation provided by all of your dependencies locally and opens it in a browser

```bash
cargo doc --open
```

## Publish crate

Uploads your crate to crates.io so others in the community can use it

```bash
cargo publish
```

> [!TIP]
>
> You can find the new crate on [crates.io](https://crates.io/) and find the document on [docs](https://docs.rs/)

## Adding a new crate

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