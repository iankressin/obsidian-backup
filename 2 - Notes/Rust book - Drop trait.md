---
date: 2024-06-05 11:00
tags:
  - final
source: https://doc.rust-lang.org/stable/book/ch15-03-drop.html
---
# Rust book - Drop trait

-  Drop trait allows to run arbitrary code when value goes out of scope
```rust
struct MyStruct {
	data: String
}

impl Drop for MyStruct {
	fn drop(&mut self) {
		println!("dropping MyStruct");
	}
}

fn main() {
	println!("before creating ms");
	let ms = MyStruct { data: String::from("hello world") };
	println!("after creating ms");
}
```

- The sequence of logs displayed is the following:
```
> before createing ms
> after createing ms
> dropping MyStruct 
```

- It is possible to drop values early using `drop` function from `std::mem::drop`, this can be useful when implementing locks, since the lock needs to be drop during the execution of the program, so another part of the program can get a hold of the lock.
  
```rust
struct MyStruct {
	data: String
}

impl Drop for MyStruct {
	fn drop(&mut self) {
		println!("dropping MyStruct");
	}
}

fn main() {
	println!("before creating ms");
	let ms = MyStruct { data: String::from("hello world") };
	println!("after creating ms");
}
```

- In this case the sequence of logs displayed is different, because `ms` goes out of scope early
```
> before createing ms
> dropping MyStruct 
> after createing ms
```