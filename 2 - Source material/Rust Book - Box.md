2024-05-16 05:17

Link: https://doc.rust-lang.org/book/ch15-01-box.html

# Rust Book - Box
- The difference between references and smart-pointers is while references borrow data, smart-pointers own the data they point to.

- Box allows to store data on the heap rather than the stack, and keep a pointer to the heap data on the stack.

- Boxes are used in the following situations:
	1. when a type is dynamic and its size is unknown at compile and you want to use a value of that type in a context that requires the exact size
	  
	2. when you poses a large chunk of data which you want to transfer the ownership, but want to guarantee that the data won't be copied.
		- when ownership is transferred the data behind the reference is copied around the stack, and this can become a time costly task when the data is too large. then we can store the whole data on the heap, and the only value being passed around is a pointer on the stack. 
		  
	3. when you want to own a type, that implements a particular trait, but you only care about that type implementing that particular trait

- Deallocation work for boxes in the dame way to work for any other value. When it goes out of scope, it gets deallocated

- The code below won't compile since rust cannot predict the size of type `List` to allocate the initial memory for the program to run. This happens because a `List` contains a `List` and this can go on forever. So in order to break the recursion, we need to insert what rust calls *indirection*, which is a pointer to a data on the heap.
```rust
enum List {
	Cons(i32, List),
	Nil,
}
```
![[trpl15-01.svg]]

- With the indirection in the form of a Box pointer, rust can now predict the amount of data needed to store a List in memory, since `Cons` now holds two fixed length values, `i32` and `Box`, which as a pointer, has a fixed length `usize`.
``` rust
enum List {
	Cons(i32, Box<List>),
	Nil,
}
```
![[trpl15-02.svg]]
