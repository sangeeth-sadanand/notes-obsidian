# Error handling
## Unrecoverable Errors with panic!

- Sometimes bad things happen in your code, and there’s nothing you can do about it. 
- In these cases, Rust has the `panic!` macro.
- There are two ways to cause a panic in practice: by taking an action that causes our code to panic (such as accessing an array past the end) or by explicitly calling the `panic!` macro.
- In both cases, we cause a panic in our program. 
- By default, these panics will print a failure message, unwind, clean up the stack, and quit. 
- Via an environment variable, you can also have Rust display the call stack when a panic occurs to make it easier to track down the source of the panic.
-
```rust
fn main() {
    panic!("crash and burn");
}

/* 
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.25s
     Running `target/debug/panic`

thread 'main' panicked at src/main.rs:2:5:
crash and burn
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
%/

```

### How Panic Recovery Works in Rust

#### Default Behaviour

- A `panic!` **unwinds the stack** (cleaning up resources) and terminates the thread.
- By default, if the main thread panics, the whole program exits.
#### Recovering in Threads

- Panics in spawned threads can be caught when joining:
```rust
use std::thread;

fn main() {
	let handle = thread::spawn(|| {
		panic!("Something went wrong!");
	});

	match handle.join() {
		Ok(_) => println!("Thread finished successfully."),
		Err(e) => println!("Thread panicked: {:?}", e),
	}
}
```
- Here, the main thread continues even though the spawned thread panicked.

#### Using `catch_unwind`
- Rust provides `std::panic::catch_unwind` to capture panics:
```rust
use std::panic;

fn main() {
	let result = panic::catch_unwind(|| {
		println!("About to panic!");
		panic!("Boom!");
	});

	match result {
		Ok(_) => println!("No panic occurred."),
		Err(_) => println!("Recovered from panic."),
	}
}
```
    
- This allows execution to continue after a panic.
  
## Recoverable Errors with Result
- Most errors aren’t serious enough to require the program to stop entirely. 
- Sometimes when a function fails, it’s for a reason that you can easily interpret and respond to. 
- We can use the `Result` type and the functions defined on it in many different situations where the success value and error value we want to return may differ.
  
```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");
}
```
  - The return type of `File::open` is a `Result<T, E>`. 
  - The generic parameter `T` has been filled in by the implementation of `File::open` with the type of the success value, `std::fs::File`, which is a file handle. 
  - The type of `E` used in the error value is `std::io::Error`. 
  - This return type means the call to `File::open` might succeed and return a file handle that we can read from or write to.
  
```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {error:?}"),
    };
}
```
  
  
### Propagating Errors
- When a function’s implementation calls something that might fail, instead of handling the error within the function itself, you can return the error to the calling code so that it can decide what to do. 
- This is known as _propagating_ the error and gives more control to the calling code, where there might be more information or logic that dictates how the error should be handled than what you have available in the context of your code.

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}

```
  
  
#### The ? Operator Shortcut
  
```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```
  
- The `?` placed after a `Result` value is defined to work in almost the same way as the `match` expressions that we defined to handle the `Result`.
- If the value of the `Result` is an `Ok`, the value inside the `Ok` will get returned from this expression, and the program will continue. If the value is an `Err`, the `Err` will be returned from the whole function as if we had used the `return` keyword so that the error value gets propagated to the calling code.
- we can also return Result enum in main as well

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```


  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  