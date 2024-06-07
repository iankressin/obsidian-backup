2024-05-28 13:13

Link: https://doc.rust-lang.org/book/ch15-02-deref.html

# Rust book - Deref trait

- We can use the dereference operator (\*\) on Boxes just like a regular reference because Box implements the deref trait
```rust
fn main() {
	let x = 5;
	let y = &x;

	assert_eq!(5, *y);
}

// same as above
fn main() {
	let x = 5;
	let y = Box::new(x);

	assert_eq!(5, *y)
}
```

- In order to be able to use the dereference in a custom type, we need to implement the Deref trait
```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
	fn new(t: T) -> Self {
		MyBox(t);
	}
}

impl<T> Deref for MyBox<T> {
	type Target = T;

	fn deref(&self) -> &Self::Target {
		&self.0 // reference to inner data
	}
}
```

- When rust dereference a struct that implements the Deref trait, what's actually doing is: `*(y.deref())`

- The reason why `fn deref(&self) -> &Self::Target` returns a reference to a value is because the compiler still use the dereference operator on the return value, because of the ownership system. If `fn deref` was to return the actual value, `MyBox` would be moved out of self.

- Rust provides Deref coercion, which allows to use types that implement the Deref trait to be used as params in functions and methods that receives references, because the `deref` function of `Deref` trait returns a reference. 
```rust
fn hello(name: &str) {
	println!("Hello {name}");
}

fn main() {
	let my_box = Mybox::new(String::from("Ian"));
	hello(&my_box);
}
```

- Rust compiler runs `Deref::deref` as many times needed until it founds a type that matches the function signature. This happens at compile time, so there's no penalty at runtime 