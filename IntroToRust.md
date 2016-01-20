
![Rust Logo](files/rust-logo-64x64-blk.png)

# Rust

[http://rust-lang.org/](http://rust-lang.org)

## What this presentation is

  * Introduction to Rust
  * Discussion of design, features, tradeoffs

## What this presentation is not

  * Golang vs. Rust

** We can talk about differences, and what you like vs what you don't
like, but the two languages each bring their own strengths and aren't
really competitors **

## What is Rust?

Systems programming language developed initially by Mozilla

Now a community project

Pursuing the trifecta:

1. Safe
2. Concurrent
3. Fast

Zero-cost abstractions

Guaranteed memory safety

No data races

Close to the metal

Seamless C interop

## What does Rust look like?

```rust
// Hello, World

fn main() {
    println!("Hello, World!");
}
```

```rust
// more Hello, World

fn say_hello(s: &str) {
    println!("Hello, {}", s);
}

fn main() {
    let greeting = "World";
    say_hello(greeting); // Hello, World

    let greeting: String = "World".to_string();
    say_hello(&greeting); // Hello, World

    let greeting: String = greeting
                               .chars()
                               .rev()
                               .collect();
    say_hello(&greeting); // Hello, dlroW
}
```

```rust
// Type Inference

fn main() {
    let greeting = "World";
    println!("Hello, {}", greeting);
}
```

```rust
// Types must be specified in function parameters, and in certain
// other places where the compiler doesn't have enough information
// to determine a type.

fn add_one(x: i32) -> i32 {
    x + 1
}

add_one(1); // 2
add_one(1_i32); // 2
add_one(1_u64); // COMPILE ERROR
add_one(1_u64 as i32); // 2
```

```rust
// Immutable data

// By default, all bindings are immutable

fn main() {
    let greeting = "World";
    greeting = "Foo"; // COMPILE ERROR
}
```

```rust
// Variables must explicitly be marked as mutable

fn main() {
    let mut greeting = "World";
    greeting = "Foo" // Happy compiler
}
```

```rust
// However, re-binding is possible:

fn main() {
    let mut foo = Bar::new();
    foo.some_property = "some value";

    let foo = foo.finalize();  // previous mutable binding replaced with immutable binding
}
```

## Types

### Primitive types

<br>

```rust
// Numeric types
let x: u8 = 255;
let x: u32 = 1;
let x: i64 = -2;
let x: usize = 45;
let x: f32 = 1.0;

// Strings
let x: &'static str = "Hello, Wold"; // str

// Char
let x: char = 'a';

// Boolean
let x: bool = true;

// Tuples
let x: (u32, u32) = (1, 2);

// Array
let x: [u32; 4] = [1, 2, 3, 4];

// Slice
let x: &[u32] = &x[0..2];
```

```rust
// Aside about strings

// 1
let x: String = String::new("foo"); // or "foo".to_string()

// 2
let y: &str = &x;

// 3
let x: &'static str = "foo";
```

### Structs

<br>

```rust
struct Point {
    x: f32,
    y: f32,
}
```

** Maybe mention #[repr(C)]? **

#### Methods on structs

<br>

```rust
struct Point {
    x: f32,
    y: f32,
}

impl Point {
    fn new(x: f32, y: f32) -> Point {
        Point {
            x: x,
            y: y,
        }
    }
}

fn main() {
    let p = Point::new(1.0, 1.0);
}
```

** Keeping code & data separate FTW! **

#### Methods on structs

<br>

```rust
pub struct Point {
    x: f32,
    y: f32,
}

impl Point {
    pub fn new(x: f32, y: f32) -> Point {
        Point { x: x, y: y }
    }
    
    pub fn distance(&self, other: &Point) -> f32 {
        let x_dist = (other.x - self.x).powi(2);
        let y_dist = (other.y - self.y).powi(2);
        (x_dist + y_dist).sqrt()
    }
}

fn main() {
    let p = Point::new(1.0, 1.0);
    let q = Point::new(2.0, 2.0);
    println!("distance: {}", p.distance(&q));
}
```

### Enums

<br>

```rust
enum Color {
    Blue,
    Red,
    Green,
}
```

```rust
impl Color {
    fn what_color(&self) {
        let s = match *self {
            Color::Blue => "Blue",
            Color::Red => "Red",
            Color::Green => "Green",
        };
        println!("color: {}", s);
    }
}

fn main() {
    let x = Color::Red;
    x.what_color(); // "color: Red"
}
```

```rust
enum Either {
    Left(u32),
    Right(u32),
}
```

#### Aside on matching & patterns

<br>

```rust
// 1
match Either::Left(2) {
    Either::Right(x) => println!("got Right({})", x),
}
// Compile error, exhaustiveness checking

// 2
match Either::Left(2) {
    Either::Left(x) => println!("got Left({})", x),
    Either::Right(y) => println!("got Right({})", y),
}
```

#### Aside on matching & patterns

<br>

```rust
match (1, 2) {
    (x, 1) => {
        println!("got {}, 1", x);
    },
    (1, y) => {
        println!("got 1, {}", y);
    },
    _ => {
        println!("default case");
    },
}
```

#### Aside on matching & patterns

<br>

```rust
struct Point { x: f32, y: f32 }

enum Either { Left(u32), Right(u32) }

fn main() {
    let p = Point { x: 1.0, y: 2.0 };
    let y = match p {
        Point { x, y } => y,
    };
    println!("y: {}", y);
    
    let e = Either::Left(2);
    let l = match e {
        Either::Left(l) => l,
        Either::Right(r) => r,
    };
    println!("l: {}", l);
}
```

### References & Borrowing

There are two kinds of reference:

  * Shared reference: `&`
  * Mutable reference: `&mut`

Which obey the following rules:

  * A reference cannot outlive its referent
  * A mutable reference cannot be aliased
  
That's it. That's the entire model.

[source: https://twitter.com/QEDunham/status/689596292029251584]

```rust
struct Person;

fn main() {
    // Only one owner allowed
    let x = Person;
    let y = &x; // y "borrows" x
    let z = *y; // Error: trying to move borrowed value
}
```

```rust
struct Person;

fn main() {
    let x = Person;
    let y = &mut x;
    let z = &mut x; // Error: x is already mutably borrowed
}

fn main2() {
    let x = Person;
    {
        let y = &mut x;
        // do something with y
    }
    let z = &mut x; // Ok
}
```

```rust
fn borrow_something_immutably(x: &Either);

fn borrow_something_mutably(x: &mut Either);

fn take_ownership(x: Either);
```

### Traits

```rust
trait Display {
    fn fmt(&self, &mut Formatter);
}
```

```rust
struct Person { name: String, age: u32 }

impl Display for Person {
    fn fmt(&self, f: &mut Formatter) {
        // work
    }
}
```

```rust
trait Read {
    // required to implement trait
    fn read(&mut self, buf: &mut [u8]) -> Result<usize, io::Error>;
    
    // rest are optional...
    
    fn read_to_end(&mut self, buf: &mut Vec<u8>) -> Result<usize, io::Error> {
        // implementation using .read()
    }

    fn read_to_string(&mut self, buf: &mut String) -> Result<usize, io::Error> {
        // implementation using .read()
    }
    
    // ...more methods
}
```

### Generics

```rust
fn do_something<T>(x: &T) {
    // do something with x
}

do_something(&3);
do_something("hello, world");  // type is &'static str, so it can be a &T
```

```rust
fn add_one<T: Add>(x: T) -> usize {
    x + 1
}

add_one(1_u32) // 2
add_one(1 as usize) // 2
```

### Error Handling

```rust
enum Result<T, E> {
    Ok(T),
    Err(E)
}

fn might_fail(x: bool) -> Result<u32, String> {
    if x {
        Ok(1)
    } else {
        Err(String::new("Some error"))
    }
}

// Usually you would use a dedicated type for the Error condition, not just a String

```

```rust
fn main() {
    match might_fail(true) {
        Ok(x) => println!("got result {}", x),
        Err(e) => println!("error message was {}", e),
    }
}
```

```rust
let x: u32 = try!(might_fail(true));
```

```rust
let x: u32 = {
    match might_fail(true) {
        Ok(x) => x,
        Err(e) => return Err(e),
    }
};
```

```rust
fn always_fails() {
    panic!("This is a panic message");
}
```

## Concurrency

```rust
use std::thread;
use std::sync::mpsc::channel;

fn main() {
    let data = 0;
    let N = 10;

    let (tx, rx) = channel();

    for _ in 0..10 {
        let tx = tx.clone();
        thread::spawn(move || {
            data += 1; // COMPILE ERROR
            if data == N {
              let send_result = tx.send(());
            }
        });

    }

    match rx.recv() {
        Ok(()) => println!("Successfully received ()"),
        Ok(_) => println!("Error!"),
        Err(_) => println!("Error!"),
    }
}
```

```rust
use std::thread;
use std::sync::{Arc, Mutex};
use std::sync::mpsc::channel;

fn main() {
  let data = Arc::new(Mutex::new(0));
  let N = 10;

  let (tx, rx) = channel();

  for _ in 0..10 {
      let (data, tx) = (data.clone(), tx.clone());
      thread::spawn(move || {
          let mut data = data.lock().unwrap() // actually handle the error
          *data += 1;
          if *data == N {
              let send_result = tx.send();
          }
      });
  }

  match rx.recv() {
      Ok(()) => println!("Successfully received ()"),
      Ok(_) => println!("Error!"),
      Err(_) => println!("Error!"),
  }
}
```

** Because of the borrowing system, all this is implemented in
libraries, not the language implementation **

## Crates & Modules

#### Crate

Unit of compilation

#### Module

Used for organization within a crate

```rust
// lib.rs
mod some_module {
    // ...
}

mod other_module; // causes rustc to look for the code in either
                  // other_module.rs or other_module/mod.rs
```

### Cargo & crates.io

```bash
$ cargo new my-new-project
```

* Package manager
* Build tool
* Documentation generator
* Test runner

```ini
[package]
name = "my-new-project"
version = "0.1.0"
authors = ["John Doe <jdoe@example.com>"]
```

```ini
[lib]
name = "mylib"
path = "src/lib.rs"
```

```ini
[[bin]]
name = "mybin"
path = "src/bin/mybin.rs"
```

```ini
[dependencies]
libc = "~0.2.4"
```

```ini
[dev-dependencies]
hamcrest = "*"
```

### Testing

```rust
// src/lib.rs

pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

```rust

#[cfg(test)]
mod tests {
    use super::add_one;
    
    #[test]
    fn test_add_one() {
        assert_eq!(
            add_one(1),
            2
        );
    }

}
```

```bash
$ cargo test
```

```rust
// tests/integration.rs
extern crate mylib;
use mylib::add_one;

#[test]
fn integration_test_one() {
    assert_eq!(
        add_one(2),
        3
    );
}

```

### Documentation

```rust
// src/lib.rs
//! This is crate-level documentation. Here is where you would give a general overview of your
//! crate, along with any background information that could be useful (RFCs, etc)

/// This is specifically for the item that comes after it. 
/// Here is where you would put information about `SomeDataStructure`
struct SomeDataStructure {
    // ...
}

impl SomeDataStructure {

    /// You can include examples with your code, that will show up in the 
    /// documentation. The best part of this is that the examples are compiled
    /// and can be run with your tests. So, if the example code in your documentation
    /// goes out-of-sync with your code, your tests will fail!
    ///
    /// # Example
    ///
    /// ```rust
    /// let ds = SomeDataStructure { .. };
    /// assert_eq!(ds.some_method(), "some value");
    /// ``````
```rust
    fn some_method(&self) {
        // ...
    }
    
}
```

### Community

* Github: https://github.com/rust-lang
* Web: https://www.rust-lang.org/
* IRC: irc.mozilla.org
  * #rust
  * #rust-beginners
* Discourse forum: https://users.rust-lang.org
* Reddit: https://reddit.com/r/rust

### Stuff I didn't cover?

* Macros
* Conditional compliation
* cargo build scripts
* Probably a lot more
