2024-04-26 09:36

Status: #final

Tags: [[6 - Tags/Rust|Rust]]

# Borrowing
Binding a heap allocated variable to another one is not the only behavior that causes a ownership to change hands in Rust. When a variable is passed the Rust compiler also consider that a move, as the example below shows:
```rust
fn get_length(s: String) -> usize {
	s.len()
}

fn main() {
	let s1 = String::from("hello");

	let length = get_length(s1);

	println!("{s1}"); // Error: s1 was moved
}
```

The `String` s1 is being passed as argument to the function `get_length`, which takes the ownership of that heap-allocated data, and when that function finishes the execution, by the rules of ownership, the reference to `s1` is dropped.

A possible solution to this problems is to return the `String` from the `get_length` function, which would create a new ownership when function returns.

```rust
fn get_length(s: String) -> (usize, String) {
	(s.len(), s)
}

fn main() {
	let s1 = String::from("hello");

	(let length, let s1) = get_length(s1);

	println!("{s1}"); // s1 exists here
}
```

Although the code above works as we expect, it's a bit tedious to return all the moved values to functions. Also, in the example above, `get_length` only takes in a single heap-allocated variable, but imagine if that same function depends one, three, four or five variables. That means the function would need to return all those values, and the caller would need to re-declare those variables after the call to `get_length` once again.
### The solution: references
Instead of moving ownership around, Rust allows to pass references to those variables, which doesn't change the original owner, instead offers a pointer to the data stored in that heap address where the variable is stored. But unlike a pointer, a reference a guarantee to point to a valid value of a particular type for the life of that reference.

> [!NOTE]
> The action of creating a reference is called *borrowing*

If `get_length` is updated to receive a reference as parameter, that's how it would look like:

```rust
fn get_length(s: &String) -> usize {
	s.len()
}

fn main() {
	let s1 = String::from("hello");

	let length = get_length(&s1);

	println!("{s1}"); // s1 exists here
}
```

The diagram below represents a mental model of how `&s` is represented in memory:

![[trpl04-05.svg]]

As variables, references are unmutable by default, which mean that if a function tries to modify the original value of a variable that is behind a reference, the compiler throws an error, like represented below:

```rust
fn change(s: &String) -> usize {
	s.push_str("world"); // ERROR: Cannot borrow s as mutable
}

fn main() {
	let s1 = String::from("hello");

	change(&s1);
}
```

### Mutable references
Mutable reference allows a variable to mutate the original value store on the heap. And to create a mutable reference is straight forward.

The `change` function would look like this to prevent the error seen above:

```rust
fn change(s: &mut String) -> usize {
	s.push_str("world"); // ERROR: Cannot borrow s as mutable
}

fn main() {
	let mut s1 = String::from("hello");

	change(&mut s1);
}
```

Notice that the owner of the reference needs to be marked as `mut`, so as the reference being passed and received as parameter on `change`.

Despite being easy to create a mutable reference, Rust only accepts a single mutable reference per scope at a time: 

```rust
let mut s1 = String::from("hello");

let s2 = &mut s1; // Valid, first mutable reference
let s3 = &mut s1; // Invalid, mutable reference already taken in this scope
```

This rule avoids data races, which occur when two pointers have access to the same data, at least one of them is trying to write to that memory location and there is no mechanism to synchronize the data between the pointers.

Rust allows to mix mutable and unmutable references within the same scope, as long as the last usage of a unmutable reference occurs before the declaration of a mutable reference. This rule avoid the uncertainty about the reference value that would affect the code, if a mutable reference was able to change the data behind an unmutable reference.

```rust {
	let mut s1 = String::from("hello");
	
	let s2 = &s1; // Valid, multiple unmut references allowed
	let s3 = &s1; // Valid, multiple unmut references allowed
	let s4 = &mut s1; // Invalid, last usage of s2 and s3 occurs after the declaration of s4 
	
	println!("{s2} {s3} {s4}");
}

	let mut s1 = String::from("hello");
	
	let s2 = &s1; // Valid, multiple unmut references allowed
	let s3 = &s1; // Valid, multiple unmut references allowed
	
	println!("{s2} {s3}");

	let s4 = &mut s1; // Valid, last usage of s2 and s3 occurs before the declaration of s4 
}
```

## References