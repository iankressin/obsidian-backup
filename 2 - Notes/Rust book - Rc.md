---
date: 2024-06-05 11:53
tags:
  - final
source: https://doc.rust-lang.org/stable/book/ch15-04-rc.html
---
# Rust book - Rc

- A reference counting pointer allows multiple owners of a single heap-allocated data.
	1 value / Multiple readers

- The reference counting happens at compile time, and the compiler determines which variable does the last use of a reference, and makes that variable the owner of the heap data. Then the regular rules of ownership are applied.

- Implementation of Cons list using Rc
```rust
enum List {
	Cons(i32, Rc<List>),
	Nil
}

fn main() {
	let shared_list = Rc::new(List::Cons(10, Rc::new(Nil)));
	let list_a = Cons(2, Rc::clone(&shared_list));
	let list_b Cons(5, Rc::clone(&shared_list));
}
```

- The reference counter for a given Rc instance is updated whenever `Rc::clone` is called with a reference of that instance. Alternatively, `shared_list.clone()` could be use to increment the counter, but the current convention is to use `Rc::clone`.

-  `Rc::clone` doesn't create a deep copy of the reference, it only updates the reference counter and returns a clone of a pointer to the data on the heap.\

- Rc implements Drop trait for decreasing the ref count.

### Questions
*How a Rc is different than a regular reference?*\
A regular reference can only have a single owner. Rcs, on the other hand, can have multiple ownersj